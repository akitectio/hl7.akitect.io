---
icon: file-plus-minus
---

# CRUD Operations

Chào các bạn! Trong bài viết hôm nay, tôi sẽ chia sẻ chi tiết về các thao tác CRUD (Create, Read, Update, Delete) trong FHIR R5.&#x20;

### 1. CREATE: Tạo mới tài nguyên

#### 1.1. POST cơ bản

Để tạo tài nguyên mới trong FHIR, chúng ta sử dụng phương thức HTTP POST với dữ liệu dạng JSON hoặc XML.

**Ví dụ tạo mới bệnh nhân:**

```http
POST https://server.com/fhir/Patient
Content-Type: application/fhir+json

{
  "resourceType": "Patient",
  "name": [
    {
      "family": "Nguyễn",
      "given": ["Văn", "A"]
    }
  ],
  "birthDate": "1980-07-15",
  "gender": "male",
  "active": true,
  "telecom": [
    {
      "system": "phone",
      "value": "0912345678",
      "use": "mobile"
    }
  ],
  "address": [
    {
      "use": "home",
      "line": ["123 Đường Lê Lợi"],
      "city": "Hà Nội",
      "country": "VN"
    }
  ]
}
```

**Kết quả trả về khi thành công:**

```http
HTTP/1.1 201 Created
Location: https://server.com/fhir/Patient/456
ETag: W/"1"
Last-Modified: 2025-03-13T10:30:15.123+07:00

{
  "resourceType": "Patient",
  "id": "456",
  "meta": {
    "versionId": "1",
    "lastUpdated": "2025-03-13T10:30:15.123+07:00"
  },
  "name": [
    {
      "family": "Nguyễn",
      "given": ["Văn", "A"]
    }
  ],
  // ... dữ liệu còn lại
}
```

> **Lưu ý:** Server sẽ tự động:
>
> * Gán ID cho tài nguyên mới (ở đây là "456")
> * Tạo thông tin meta với phiên bản đầu tiên (versionId: "1")
> * Thêm timestamp khi tạo (lastUpdated)

#### 1.2. Conditional Create - Tạo có điều kiện

FHIR cho phép tạo tài nguyên chỉ khi chưa tồn tại, giúp tránh dữ liệu trùng lặp. Đây là tính năng đặc biệt hữu ích khi cần nhập dữ liệu hàng loạt.

**Ví dụ tạo có điều kiện:**

```http
POST https://server.com/fhir/Patient
Content-Type: application/fhir+json
If-None-Exist: identifier=http://hospital.vn/fhir/identifier/mrn|MRN12345

{
  "resourceType": "Patient",
  "identifier": [
    {
      "system": "http://hospital.vn/fhir/identifier/mrn",
      "value": "MRN12345"
    }
  ],
  "name": [
    {
      "family": "Nguyễn",
      "given": ["Văn", "A"]
    }
  ],
  "birthDate": "1980-07-15",
  "gender": "male"
}
```

Header `If-None-Exist` chứa điều kiện tìm kiếm. Trong ví dụ trên, chúng ta chỉ tạo bệnh nhân nếu chưa có bệnh nhân nào có mã MRN12345.

**Kết quả có thể xảy ra:**

* **201 Created:** Nếu không tìm thấy bệnh nhân thỏa điều kiện → tạo mới
* **200 OK:** Nếu đã tồn tại bệnh nhân thỏa điều kiện → trả về tài nguyên hiện có
* **412 Precondition Failed:** Nếu tìm thấy nhiều tài nguyên thỏa điều kiện

### 2. READ: Đọc tài nguyên

#### 2.1. Đọc tài nguyên đơn

Để đọc một tài nguyên, sử dụng phương thức HTTP GET với ID của tài nguyên.

**Ví dụ đọc thông tin bệnh nhân:**

```http
GET https://server.com/fhir/Patient/456
```

**Kết quả:**

```http
HTTP/1.1 200 OK
ETag: W/"1"
Last-Modified: 2025-03-13T10:30:15.123+07:00

{
  "resourceType": "Patient",
  "id": "456",
  "meta": {
    "versionId": "1",
    "lastUpdated": "2025-03-13T10:30:15.123+07:00"
  },
  "name": [
    {
      "family": "Nguyễn",
      "given": ["Văn", "A"]
    }
  ],
  "birthDate": "1980-07-15",
  "gender": "male",
  // ... dữ liệu còn lại
}
```

#### 2.2. Đọc phiên bản cụ thể

FHIR lưu giữ lịch sử các phiên bản của tài nguyên. Để truy xuất một phiên bản cụ thể, thêm `_history/[versionId]` vào URL.

**Ví dụ đọc phiên bản 2 của bệnh nhân:**

```http
GET https://server.com/fhir/Patient/456/_history/2
```

Lệnh này sẽ trả về phiên bản số 2 của bệnh nhân có ID 456, ngay cả khi đã có phiên bản mới hơn.

#### 2.3. Xem lịch sử thay đổi

Để xem toàn bộ lịch sử thay đổi của một tài nguyên:

```http
GET https://server.com/fhir/Patient/456/_history
```

**Kết quả:**

```http
HTTP/1.1 200 OK

{
  "resourceType": "Bundle",
  "type": "history",
  "total": 3,
  "entry": [
    {
      "fullUrl": "https://server.com/fhir/Patient/456/_history/3",
      "resource": {
        "resourceType": "Patient",
        "id": "456",
        "meta": {
          "versionId": "3",
          "lastUpdated": "2025-03-13T14:25:30.000+07:00"
        },
        // ... phiên bản mới nhất
      }
    },
    {
      "fullUrl": "https://server.com/fhir/Patient/456/_history/2",
      "resource": {
        "resourceType": "Patient",
        "id": "456",
        "meta": {
          "versionId": "2",
          "lastUpdated": "2025-03-13T12:15:45.000+07:00"
        },
        // ... phiên bản 2
      }
    },
    {
      "fullUrl": "https://server.com/fhir/Patient/456/_history/1",
      "resource": {
        "resourceType": "Patient",
        "id": "456",
        "meta": {
          "versionId": "1",
          "lastUpdated": "2025-03-13T10:30:15.123+07:00"
        },
        // ... phiên bản 1
      }
    }
  ]
}
```

Kết quả trả về là một Bundle chứa tất cả các phiên bản, sắp xếp từ mới nhất đến cũ nhất.

#### 2.4. Tìm kiếm theo tham số

Thay vì sử dụng ID, chúng ta có thể tìm kiếm tài nguyên bằng các tham số:

```http
GET https://server.com/fhir/Patient?family=Nguyễn&given=Văn
```

Tìm tất cả bệnh nhân có họ "Nguyễn" và tên đệm "Văn".

```http
GET https://server.com/fhir/Patient?identifier=http://hospital.vn/fhir/identifier/mrn|MRN12345
```

Tìm bệnh nhân có mã MRN12345.

### 3. UPDATE: Cập nhật tài nguyên

#### 3.1. Cập nhật toàn bộ với PUT

Phương thức PUT thay thế hoàn toàn tài nguyên hiện tại bằng nội dung mới.

**Ví dụ cập nhật thông tin bệnh nhân:**

```http
PUT https://server.com/fhir/Patient/456
Content-Type: application/fhir+json
If-Match: W/"1"

{
  "resourceType": "Patient",
  "id": "456",
  "name": [
    {
      "family": "Nguyễn",
      "given": ["Văn", "A"]
    }
  ],
  "birthDate": "1980-07-15",
  "gender": "male",
  "telecom": [
    {
      "system": "phone",
      "value": "0987654321",  // Số điện thoại đã thay đổi
      "use": "mobile"
    }
  ],
  "address": [
    {
      "use": "home",
      "line": ["456 Đường Nguyễn Huệ"],  // Địa chỉ đã thay đổi
      "city": "Hồ Chí Minh",  // Thành phố đã thay đổi
      "country": "VN"
    }
  ]
}
```

Header `If-Match` đảm bảo chúng ta chỉ cập nhật nếu phiên bản hiện tại khớp với điều kiện. Điều này giúp tránh xung đột khi có nhiều người cùng cập nhật.

**Kết quả khi thành công:**

```http
HTTP/1.1 200 OK
ETag: W/"2"
Last-Modified: 2025-03-13T12:15:45.000+07:00

{
  "resourceType": "Patient",
  "id": "456",
  "meta": {
    "versionId": "2",  // Phiên bản tăng lên
    "lastUpdated": "2025-03-13T12:15:45.000+07:00"
  },
  // ... dữ liệu đã cập nhật
}
```

> **Quan trọng:** Khi sử dụng PUT, bạn phải cung cấp đầy đủ tất cả dữ liệu, kể cả những phần không thay đổi. Nếu bỏ sót bất kỳ trường nào, trường đó sẽ bị xóa trong phiên bản mới.

#### 3.2. Cập nhật một phần với PATCH

FHIR R5 hỗ trợ PATCH để cập nhật từng phần của tài nguyên, không cần gửi toàn bộ nội dung.

**Ví dụ sử dụng JSON Patch:**

```http
PATCH https://server.com/fhir/Patient/456
Content-Type: application/json-patch+json
If-Match: W/"2"

[
  { "op": "replace", "path": "/telecom/0/value", "value": "0909123456" },
  { "op": "replace", "path": "/address/0/city", "value": "Đà Nẵng" },
  { "op": "add", "path": "/contact", "value": [
    {
      "relationship": [
        {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/v2-0131",
              "code": "N",
              "display": "Next-of-Kin"
            }
          ]
        }
      ],
      "name": {
        "family": "Nguyễn",
        "given": ["Thị", "B"]
      },
      "telecom": [
        {
          "system": "phone",
          "value": "0912345678"
        }
      ]
    }
  ]}
]
```

Trong ví dụ trên:

* Thay đổi số điện thoại
* Cập nhật thành phố thành "Đà Nẵng"
* Thêm thông tin liên hệ người thân

FHIR R5 cũng hỗ trợ FHIRPath Patch, cho phép cập nhật dựa trên biểu thức FHIRPath:

```http
PATCH https://server.com/fhir/Patient/456
Content-Type: application/fhir-patch+json
If-Match: W/"2"

{
  "resourceType": "Parameters",
  "parameter": [
    {
      "name": "operation",
      "part": [
        {
          "name": "type",
          "valueString": "replace"
        },
        {
          "name": "path",
          "valueString": "Patient.telecom.where(system='phone').value"
        },
        {
          "name": "value",
          "valueString": "0909123456"
        }
      ]
    }
  ]
}
```

#### 3.3. Cập nhật có điều kiện

Cập nhật có điều kiện cho phép bạn cập nhật tài nguyên dựa trên điều kiện tìm kiếm thay vì ID.

**Ví dụ cập nhật theo mã MRN:**

```http
PUT https://server.com/fhir/Patient?identifier=http://hospital.vn/fhir/identifier/mrn|MRN12345
Content-Type: application/fhir+json

{
  "resourceType": "Patient",
  "identifier": [
    {
      "system": "http://hospital.vn/fhir/identifier/mrn",
      "value": "MRN12345"
    }
  ],
  "name": [
    {
      "family": "Nguyễn",
      "given": ["Văn", "A"]
    }
  ],
  "birthDate": "1980-07-15",
  "gender": "male",
  "active": true
}
```

**Các kết quả có thể xảy ra:**

* **200 OK:** Tìm thấy và cập nhật chính xác một tài nguyên
* **201 Created:** Không tìm thấy, tạo mới tài nguyên (nếu server hỗ trợ)
* **412 Precondition Failed:** Tìm thấy nhiều tài nguyên thỏa điều kiện

### 4. DELETE: Xóa tài nguyên

#### 4.1. Xóa theo ID

```http
DELETE https://server.com/fhir/Patient/456
```

**Kết quả có thể:**

* **200 OK:** Xóa thành công, trả về thông tin về tài nguyên đã xóa
* **204 No Content:** Xóa thành công, không trả về nội dung
* **405 Method Not Allowed:** Server không cho phép xóa
* **409 Conflict:** Không thể xóa vì tài nguyên đang được tham chiếu

> **Lưu ý quan trọng:** Trong hầu hết hệ thống y tế, việc "xóa" thường không phải là xóa vĩnh viễn mà là đánh dấu tài nguyên đó không còn hoạt động. Dữ liệu y tế thường cần được lưu trữ lâu dài vì lý do pháp lý và lâm sàng.

**Ví dụ về tài nguyên đã bị xóa:**

```json
{
  "resourceType": "Patient",
  "id": "456",
  "meta": {
    "versionId": "3",
    "lastUpdated": "2025-03-13T14:25:30.000+07:00"
  },
  "name": [
    {
      "family": "Nguyễn",
      "given": ["Văn", "A"]
    }
  ],
  "birthDate": "1980-07-15",
  "gender": "male",
  "active": false  // Đánh dấu bệnh nhân không còn hoạt động
}
```

#### 4.2. Xóa có điều kiện

```http
DELETE https://server.com/fhir/Patient?identifier=http://hospital.vn/fhir/identifier/mrn|MRN12345&active=false
```

Lệnh này sẽ xóa tất cả bệnh nhân có mã MRN12345 và đã không còn hoạt động.

**Kết quả trả về thêm thông tin về số lượng tài nguyên bị ảnh hưởng:**

```http
HTTP/1.1 200 OK

{
  "resourceType": "OperationOutcome",
  "issue": [
    {
      "severity": "information",
      "code": "informational",
      "details": {
        "text": "Successfully deleted 1 resource(s)"
      }
    }
  ]
}
```

### 5. Versioning, History và Tham số \_history

#### 5.1. Cơ chế versioning trong FHIR

FHIR quản lý phiên bản tài nguyên thông qua hai thông tin chính trong thuộc tính `meta`:

* **versionId:** Số phiên bản, thường bắt đầu từ 1 và tăng dần
* **lastUpdated:** Thời điểm cập nhật phiên bản này

Mỗi khi tài nguyên được cập nhật (PUT/PATCH) hoặc thay đổi trạng thái (như đánh dấu xóa), phiên bản mới sẽ được tạo.

**Ví dụ về meta:**

```json
"meta": {
  "versionId": "3",
  "lastUpdated": "2025-03-13T14:25:30.000+07:00",
  "source": "#LmZmZjU5YzM2ZWFkZTM2Mg",
  "profile": [
    "http://hl7.org/fhir/StructureDefinition/Patient"
  ],
  "security": [
    {
      "system": "http://terminology.hl7.org/CodeSystem/v3-Confidentiality",
      "code": "N",
      "display": "normal"
    }
  ],
  "tag": [
    {
      "system": "http://hospital.vn/fhir/tags",
      "code": "inpatient",
      "display": "Inpatient"
    }
  ]
}
```

#### 5.2. Tham số \_history trong FHIR

Tham số `_history` cho phép truy vấn lịch sử thay đổi:

**1. Xem lịch sử của một tài nguyên cụ thể:**

```http
GET https://server.com/fhir/Patient/456/_history
```

**2. Xem lịch sử của tất cả tài nguyên cùng loại:**

```http
GET https://server.com/fhir/Patient/_history
```

**3. Xem lịch sử của toàn bộ server:**

```http
GET https://server.com/fhir/_history
```

**4. Lọc theo thời gian:**

```http
GET https://server.com/fhir/Patient/456/_history?_since=2025-03-10T00:00:00Z
```

Lệnh này trả về tất cả phiên bản của bệnh nhân 456 được cập nhật từ ngày 10/03/2025.

**5. Giới hạn số lượng phiên bản:**

```http
GET https://server.com/fhir/Patient/456/_history?_count=5
```

#### 5.3. Ví dụ thực tế về theo dõi thay đổi

**Giả sử kịch bản:**

1. Bệnh nhân Nguyễn Văn A được đăng ký vào hệ thống
2. Thông tin liên lạc cập nhật sau đó
3. Bệnh nhân chuyển từ Hà Nội vào Hồ Chí Minh
4. Bệnh nhân không còn điều trị (đánh dấu không hoạt động)

**Truy vấn lịch sử:**

```http
GET https://server.com/fhir/Patient/456/_history
```

**Kết quả:**

```json
{
  "resourceType": "Bundle",
  "type": "history",
  "total": 4,
  "entry": [
    {
      "fullUrl": "https://server.com/fhir/Patient/456/_history/4",
      "resource": {
        "resourceType": "Patient",
        "id": "456",
        "meta": {
          "versionId": "4",
          "lastUpdated": "2025-03-15T09:45:20.000+07:00"
        },
        "active": false,
        // ...thông tin khác
      }
    },
    {
      "fullUrl": "https://server.com/fhir/Patient/456/_history/3",
      "resource": {
        "resourceType": "Patient",
        "id": "456",
        "meta": {
          "versionId": "3",
          "lastUpdated": "2025-03-14T16:30:45.000+07:00"
        },
        "active": true,
        "address": [
          {
            "use": "home",
            "line": ["456 Đường Nguyễn Huệ"],
            "city": "Hồ Chí Minh",
            "country": "VN"
          }
        ],
        // ...thông tin khác
      }
    },
    // ...phiên bản 2 và 1
  ]
}
```

Với kết quả này, chúng ta có thể xem toàn bộ lịch sử thay đổi của bệnh nhân, điều rất quan trọng trong hồ sơ y tế.

### Một số mẹo khi làm việc với CRUD trong FHIR

#### 1. Sử dụng ETag để tránh xung đột

Luôn sử dụng header `If-Match` khi cập nhật để tránh ghi đè dữ liệu đã được người khác thay đổi:

```http
PUT https://server.com/fhir/Patient/456
If-Match: W/"2"
```

#### 2. Xử lý lỗi 409 Conflict

Khi gặp lỗi 409 (Conflict), đọc lại tài nguyên hiện tại, so sánh sự khác biệt, và thử cập nhật lại.

#### 3. Sử dụng Conditional Operations cho batch processing

Khi nhập dữ liệu hàng loạt, conditional create và update giúp tránh trùng lặp và đơn giản hóa quy trình.

#### 4. Bundle Transaction

Khi cần thực hiện nhiều thao tác CRUD cùng lúc, sử dụng Bundle kiểu transaction:

```http
POST https://server.com/fhir
Content-Type: application/fhir+json

{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "request": {
        "method": "POST",
        "url": "Patient"
      },
      "resource": {
        "resourceType": "Patient",
        "name": [
          {
            "family": "Lê",
            "given": ["Thị", "C"]
          }
        ]
      }
    },
    {
      "request": {
        "method": "PUT",
        "url": "Patient/456"
      },
      "resource": {
        "resourceType": "Patient",
        "id": "456",
        "active": false
      }
    }
  ]
}
```

Transaction đảm bảo tất cả hoặc không có thao tác nào được thực hiện (all-or-nothing).

### Kết luận

CRUD operations trong FHIR R5 cung cấp nền tảng vững chắc cho việc quản lý dữ liệu y tế. Với cơ chế versioning tích hợp sẵn, FHIR cho phép theo dõi các thay đổi quan trọng - yếu tố thiết yếu trong môi trường y tế nơi tính chính xác và khả năng kiểm toán là bắt buộc.

Qua bài viết này, tôi mong rằng các bạn có thể thấy được sự mạnh mẽ và linh hoạt của FHIR R5 trong việc thao tác với dữ liệu y tế. Các operations không chỉ đơn thuần là CRUD cơ bản mà còn bao gồm nhiều tính năng nâng cao như điều kiện, patching và giao dịch.

Ở bài viết tiếp theo, tôi sẽ đi sâu hơn vào các khía cạnh nâng cao của FHIR như Search Operations và Custom Operations. Hãy theo dõi blog của tôi để không bỏ lỡ!

