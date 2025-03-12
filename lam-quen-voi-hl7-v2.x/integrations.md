---
description: >-
  Tin nhắn HL7 v2.x được xây dựng theo một cấu trúc phân cấp nghiêm ngặt với 5
  cấp độ. Hiểu được cấu trúc này là nền tảng để làm việc hiệu quả với HL7.
icon: messages
---

# Message in HL7 v2.x

#### 1. Message (Tin nhắn)

**Tin nhắn (Message)** là đơn vị trao đổi thông tin lớn nhất, đại diện cho một sự kiện hoàn chỉnh trong hệ thống y tế như đăng ký bệnh nhân, đặt chỉ định xét nghiệm, hay báo cáo kết quả.

Mỗi tin nhắn:

* Luôn bắt đầu bằng phân đoạn MSH (Message Header)
* Bao gồm một tập hợp các phân đoạn theo thứ tự xác định
* Có một loại tin nhắn cụ thể được định nghĩa trong MSH-9 (ví dụ: ADT^A01, ORU^R01)
* Được phân định bởi các ký tự xuống dòng (CR, ASCII 13) giữa các phân đoạn

**Ví dụ tin nhắn hoàn chỉnh:** (ADT^A01 - Nhập viện)

```
MSH|^~\&|SENDING_APP|SENDING_FACILITY|RECEIVING_APP|RECEIVING_FACILITY|20230115120000||ADT^A01|MSG00001|P|2.5.1
EVN|A01|20230115120000|||
PID|1||12345^^^MRN||Doe^John^A||19800101|M|||123 Main St^^Seattle^WA^98101
PV1|1|I|2NORTH^2104^01|1|||004777^GOOD^SIDNEY^J^^^MD
```

#### 2. Segment (Phân đoạn)

**Phân đoạn (Segment)** là một nhóm logic các trường thông tin liên quan đến một khía cạnh cụ thể của dữ liệu y tế (như thông tin bệnh nhân, thông tin thăm khám, v.v.).

Đặc điểm của phân đoạn:

* Mỗi phân đoạn bắt đầu bằng một mã 3 ký tự (ví dụ: MSH, PID, OBR)
* Phân đoạn được phân tách bằng ký tự xuống dòng ( hoặc ASCII 13)
* Thứ tự của phân đoạn có ý nghĩa quan trọng và được xác định bởi định nghĩa tin nhắn
* Một số phân đoạn là bắt buộc, một số khác là tùy chọn
* Một số phân đoạn có thể lặp lại (như OBX trong ORU)

**Cách định vị phân đoạn lặp lại:**

* Khi một phân đoạn có thể lặp lại, vị trí của chúng trong tin nhắn được đánh số bắt đầu từ 1
* Ví dụ: OBX\[1], OBX\[2], OBX\[3], ...

#### 3. Field (Trường)

**Trường (Field)** là đơn vị dữ liệu cơ bản trong một phân đoạn, đại diện cho một phần thông tin cụ thể (như tên bệnh nhân, ID, ngày sinh, v.v.).

Đặc điểm của trường:

* Các trường được phân tách bằng ký tự pipe (`|`, ASCII 124)
* Đánh số bắt đầu từ 1 (không phải từ 0)
* Vị trí của trường trong phân đoạn quyết định ý nghĩa của nó
* Trường có thể rỗng (biểu thị bằng hai ký tự pipe liên tiếp `||`)
* Một số trường có thể lặp lại (sử dụng ký tự `~`)
* Trường có thể đơn giản (primitive) hoặc phức tạp (composite)

**Cách tham chiếu đến trường:**

* Sử dụng cú pháp: SegmentID-FieldNumber
* Ví dụ: PID-3 đề cập đến trường 3 (Patient ID) trong phân đoạn PID

**Đặc biệt đối với MSH:**

* MSH-1 chỉ định ký tự phân tách trường (luôn là `|`)
* MSH-2 chỉ định các ký tự mã hóa khác (thường là `^~\&`)
* Các trường trong MSH được đánh số khác biệt (MSH-1 là ký tự phân tách đầu tiên, không phải sau nó)

#### 4. Component (Thành phần)

**Thành phần (Component)** được sử dụng khi một trường cần chứa nhiều mẩu thông tin liên quan. Ví dụ, tên đầy đủ của một người có thể bao gồm họ, tên, tên đệm, v.v.

Đặc điểm của thành phần:

* Các thành phần được phân tách bằng ký tự mũ (`^`, ASCII 94)
* Thứ tự của thành phần xác định ý nghĩa của nó
* Thành phần có thể rỗng (biểu thị bằng hai ký tự mũ liên tiếp `^^`)
* Không có giới hạn số lượng thành phần trong một trường

**Cách tham chiếu đến thành phần:**

* Sử dụng cú pháp: SegmentID-FieldNumber.ComponentNumber
* Ví dụ: PID-5.1 đề cập đến thành phần 1 (họ) của trường 5 (tên bệnh nhân) trong phân đoạn PID

**Ví dụ trường với nhiều thành phần:**

```
PID-5: Smith^John^A^^Dr.
```

Ở đây:

* PID-5.1: Smith (Họ)
* PID-5.2: John (Tên)
* PID-5.3: A (Tên đệm)
* PID-5.4: (rỗng)
* PID-5.5: Dr. (Tiền tố)

#### 5. Sub-component (Thành phần phụ)

**Thành phần phụ (Sub-component)** là cấp độ chi tiết nhất trong tin nhắn HL7, được sử dụng khi một thành phần cần chia nhỏ hơn nữa.

Đặc điểm của thành phần phụ:

* Các thành phần phụ được phân tách bằng ký tự và (`&`, ASCII 38)
* Ít được sử dụng hơn so với các cấp độ khác
* Chủ yếu dùng cho dữ liệu phức tạp như tên địa lý hỗn hợp, mã hóa, v.v.

**Cách tham chiếu đến thành phần phụ:**

* Sử dụng cú pháp: SegmentID-FieldNumber.ComponentNumber.SubcomponentNumber
* Ví dụ: PID-11.3.1 đề cập đến thành phần phụ 1 của thành phần 3 (thành phố) của trường 11 (địa chỉ) trong phân đoạn PID

**Ví dụ trường với thành phần phụ:**

```
PID-11: 123 Main St^Apt 4B^Seattle&Washington^WA^98101
```

Ở đây:

* PID-11.3.1: Seattle (Tên thành phố)
* PID-11.3.2: Washington (Tên tiểu bang đầy đủ)

### Ký tự phân tách (Delimiters) chi tiết

Ký tự phân tách là nền tảng của cú pháp HL7 v2.x, cho phép phân tích và giải mã tin nhắn. Hiểu rõ về chúng là điều cần thiết.

#### Ký tự phân tách tiêu chuẩn

| Ký tự | Mã ASCII | Tên                 | Mục đích                     | Ví dụ                         |
| ----- | -------- | ------------------- | ---------------------------- | ----------------------------- |
|       | 13       | Carriage Return     | Phân tách các phân đoạn      | MSH...\rPID...\rPV1...        |
| `\|`  | 124      | Vertical Bar (Pipe) | Phân tách các trường         | PID\|1\|\|12345\|\|Smith^John |
| `^`   | 94       | Caret               | Phân tách các thành phần     | Smith^John^A                  |
| `&`   | 38       | Ampersand           | Phân tách các thành phần phụ | Seattle\&Washington           |
| `~`   | 126      | Tilde               | Phân tách các trường lặp lại | 12345^^^MRN\~67890^^^HIS      |
| `\\`  | 92       | Backslash           | Ký tự escape                 | Smith \T\ Jones               |

#### Khai báo ký tự phân tách

Các ký tự phân tách được khai báo trong MSH-1 và MSH-2. Đây là quy ước quan trọng trong HL7:

* **MSH-1**: Luôn chứa ký tự pipe (`|`), xác định dấu phân cách trường
* **MSH-2**: Chứa 4 ký tự theo thứ tự: phân cách thành phần, phân cách lặp lại, ký tự escape, phân cách thành phần phụ (thường là `^~\&`)

**Ví dụ:**

```
MSH|^~\&|...
```

#### Thứ tự ưu tiên của ký tự phân tách

Khi phân tích tin nhắn HL7, ký tự phân tách được áp dụng theo thứ tự sau:

1. Phân tách phân đoạn ()
2. Phân tách trường (`|`)
3. Phân tách lặp lại (`~`)
4. Phân tách thành phần (`^`)
5. Phân tách thành phần phụ (`&`)

### Trường lặp lại (Repeated Fields)

HL7 v2.x cho phép một trường chứa nhiều giá trị tương tự, được gọi là trường lặp lại.

#### Cách sử dụng trường lặp lại:

* Các giá trị lặp lại được phân tách bằng ký tự tilde (`~`)
* Mỗi lần lặp lại có cùng cấu trúc dữ liệu
* Việc lặp lại áp dụng cho toàn bộ trường, không phải cho từng thành phần
* Không phải tất cả các trường đều cho phép lặp lại (phải kiểm tra định nghĩa tiêu chuẩn)

#### Khi nào sử dụng trường lặp lại:

* Danh sách định danh bệnh nhân (PID-3)
* Danh sách dị ứng (AL1)
* Danh sách chẩn đoán (DG1)
* Danh sách liên hệ (NK1)
* Nhiều giá trị cho cùng một thông số

**Ví dụ trường lặp lại - Nhiều định danh bệnh nhân:**

```
PID-3: 12345^^^MRN~67890^^^HIS~54321^^^SSN
```

Ở đây, bệnh nhân có 3 định danh:

* 12345 là Medical Record Number (MRN)
* 67890 là Hospital Information System ID (HIS)
* 54321 là Social Security Number (SSN)

#### Cách tham chiếu đến trường lặp lại:

* Sử dụng cú pháp: SegmentID-FieldNumber\[RepeatNumber]
* Ví dụ: PID-3\[1] đề cập đến lần lặp lại đầu tiên của trường 3
* Chỉ số lặp lại bắt đầu từ 1 (không phải từ 0)

**Ví dụ chi tiết - Phân tích trường lặp lại:**

```
PID-3: 12345^^^MRN~67890^^^HIS
```

* PID-3\[1] = 12345^^^MRN
* PID-3\[1].1 = 12345 (ID Number)
* PID-3\[1].4 = MRN (Assigning Authority)
* PID-3\[2] = 67890^^^HIS
* PID-3\[2].1 = 67890 (ID Number)
* PID-3\[2].4 = HIS (Assigning Authority)

### Ký tự escape

Ký tự escape cho phép đưa các ký tự đặc biệt (như các ký tự phân tách) vào dữ liệu mà không làm gián đoạn cấu trúc tin nhắn.

#### Chuỗi escape tiêu chuẩn:

| Chuỗi escape | Ý nghĩa                 | Kết quả                 |
| ------------ | ----------------------- | ----------------------- |
| `\F\`        | Field separator         | `\|`                    |
| `\S\`        | Component separator     | `^`                     |
| `\T\`        | Subcomponent separator  | `&`                     |
| `\R\`        | Repetition separator    | `~`                     |
| `\E\`        | Escape character        | `\\`                    |
| `\.br\`      | Line break within data  | Xuống dòng              |
| `\.sp\`      | Space retention         | Giữ nguyên khoảng trắng |
| `\.sk\`      | Skip to next segment    | Bỏ qua phần còn lại     |
| `\Hnnn\`     | Single byte character   | Ký tự hex nnn           |
| `\Mnnn...\`  | Multiple byte character | Chuỗi hex               |

#### Ví dụ sử dụng ký tự escape:

1.  **Văn bản chứa ký tự pipe:**

    ```
    OBX|5|TX|COMMENT||The result shows 10\F\20 ratio||||||F
    ```

    Khi hiển thị: "The result shows 10|20 ratio"
2.  **Văn bản chứa ký tự mũ:**

    ```
    OBX|5|TX|COMMENT||Temperature is 37.5\S\C||||||F
    ```

    Khi hiển thị: "Temperature is 37.5^C"
3.  **Văn bản có nhiều dòng:**

    ```
    OBX|5|TX|REPORT||Patient came with fever.\.br\Temperature was 39C.\.br\Prescribed antibiotics.||||||F
    ```

    Khi hiển thị:

    ```
    Patient came with fever.
    Temperature was 39C.
    Prescribed antibiotics.
    ```

#### Quy tắc xử lý escape:

1. Xử lý các chuỗi escape trước khi phân tích cấu trúc tin nhắn
2. Escape phải được áp dụng khi gửi dữ liệu chứa ký tự phân tách
3. Unescape phải được áp dụng khi hiển thị dữ liệu cho người dùng
4. Escape phải được bảo tồn khi chuyển tiếp tin nhắn

### Cách tham chiếu đến fields và components chi tiết

HL7 có một hệ thống tham chiếu rõ ràng để định vị chính xác bất kỳ phần tử nào trong tin nhắn.

#### Cú pháp tham chiếu đầy đủ:

```
SegmentID-FieldNumber[RepeatNumber].ComponentNumber.SubcomponentNumber
```

Trong đó:

* **SegmentID**: Mã 3 ký tự của phân đoạn (PID, OBX, MSH, v.v.)
* **FieldNumber**: Số thứ tự của trường, bắt đầu từ 1
* **RepeatNumber**: \[Tùy chọn] Chỉ số của lần lặp lại, bắt đầu từ 1
* **ComponentNumber**: \[Tùy chọn] Số thứ tự của thành phần, bắt đầu từ 1
* **SubcomponentNumber**: \[Tùy chọn] Số thứ tự của thành phần phụ, bắt đầu từ 1

#### Ví dụ tham chiếu chi tiết:

Với tin nhắn:

```
MSH|^~\&|SENDING_APP|SENDING_FACILITY|RECEIVING_APP|RECEIVING_FACILITY|20230115120000||ADT^A01|MSG00001|P|2.5.1
PID|1||12345^^^MRN~67890^^^HIS||Doe^John^A^^Dr.||19800101|M|||123 Main St^Apt 4B^Seattle&Washington^WA^98101
```

Các tham chiếu:

* `MSH-3` = SENDING\_APP (Trường 3 của MSH)
* `MSH-9.1` = ADT (Thành phần 1 của trường 9 trong MSH)
* `MSH-9.2` = A01 (Thành phần 2 của trường 9 trong MSH)
* `PID-3[1]` = 12345^^^MRN (Lần lặp lại đầu tiên của trường 3 trong PID)
* `PID-3[2]` = 67890^^^HIS (Lần lặp lại thứ hai của trường 3 trong PID)
* `PID-3[1].1` = 12345 (Thành phần 1 của lần lặp lại đầu tiên của trường 3 trong PID)
* `PID-3[1].4` = MRN (Thành phần 4 của lần lặp lại đầu tiên của trường 3 trong PID)
* `PID-5.1` = Doe (Thành phần 1 của trường 5 trong PID)
* `PID-5.2` = John (Thành phần 2 của trường 5 trong PID)
* `PID-11.3.1` = Seattle (Thành phần phụ 1 của thành phần 3 của trường 11 trong PID)
* `PID-11.3.2` = Washington (Thành phần phụ 2 của thành phần 3 của trường 11 trong PID)

#### Trường hợp đặc biệt cho MSH:

Phân đoạn MSH có cách đánh số trường đặc biệt:

* MSH-1: Ký tự phân tách trường (`|`)
* MSH-2: Các ký tự mã hóa khác (`^~\&`)

Các trường MSH được đánh số bao gồm cả ký tự phân tách đầu tiên, không giống các phân đoạn khác.

#### Tầm quan trọng của hệ thống tham chiếu:

1. **Truy xuất dữ liệu**: Cho phép truy xuất chính xác thông tin từ tin nhắn
2. **Ánh xạ dữ liệu**: Hỗ trợ ánh xạ giữa các hệ thống khác nhau
3. **Xác thực và kiểm tra**: Cho phép xác định chính xác vị trí lỗi
4. **Tạo và chỉnh sửa**: Giúp cập nhật tin nhắn một cách có cấu trúc

### Phân tích ví dụ tin nhắn HL7 hoàn chỉnh

Để hiểu rõ hơn cấu trúc tin nhắn HL7, hãy phân tích một tin nhắn ADT^A01 hoàn chỉnh:

```
MSH|^~\&|SENDING_APP|SENDING_FACILITY|RECEIVING_APP|RECEIVING_FACILITY|20230115120000||ADT^A01|MSG00001|P|2.5.1
EVN|A01|20230115120000|||
PID|1||12345^^^MRN~67890^^^HIS||Doe^John^A^^Dr.||19800101|M|||123 Main St^Apt 4B^Seattle&Washington^WA^98101||5551234^PRN^PH^^^206^5551234||EN|S||MRN12345|123-45-6789
PV1|1|I|2NORTH^2104^01|1|||004777^GOOD^SIDNEY^J^^^MD|004488^ATTEND^AARON^A^^^MD|001254^REFER^RICKY^R^^^MD|CAR|||2|A0|||||||||||||||||||||||||20230115
AL1|1|DA|ASPIRIN|MO|HIVES|20220610
```

#### Phân tích từng phân đoạn:

**MSH (Message Header):**

* `MSH-1`: `|` (Field Separator)
* `MSH-2`: `^~\&` (Encoding Characters)
* `MSH-3`: `SENDING_APP` (Sending Application)
* `MSH-4`: `SENDING_FACILITY` (Sending Facility)
* `MSH-5`: `RECEIVING_APP` (Receiving Application)
* `MSH-6`: `RECEIVING_FACILITY` (Receiving Facility)
* `MSH-7`: `20230115120000` (Date/Time of Message - 12:00:00, 15/01/2023)
* `MSH-9`: `ADT^A01` (Message Type)
  * `MSH-9.1`: `ADT` (Message Code)
  * `MSH-9.2`: `A01` (Trigger Event)
* `MSH-10`: `MSG00001` (Message Control ID)
* `MSH-11`: `P` (Processing ID - Production)
* `MSH-12`: `2.5.1` (Version ID)

**EVN (Event Type):**

* `EVN-1`: `A01` (Event Type Code - Admission)
* `EVN-2`: `20230115120000` (Recorded Date/Time - 12:00:00, 15/01/2023)

**PID (Patient Identification):**

* `PID-1`: `1` (Set ID)
* `PID-3`: (Patient Identifier List)
  * `PID-3[1]`: `12345^^^MRN` (First identifier)
    * `PID-3[1].1`: `12345` (ID Number)
    * `PID-3[1].4`: `MRN` (Assigning Authority)
  * `PID-3[2]`: `67890^^^HIS` (Second identifier)
    * `PID-3[2].1`: `67890` (ID Number)
    * `PID-3[2].4`: `HIS` (Assigning Authority)
* `PID-5`: (Patient Name)
  * `PID-5.1`: `Doe` (Family Name)
  * `PID-5.2`: `John` (Given Name)
  * `PID-5.3`: `A` (Middle Initial)
  * `PID-5.5`: `Dr.` (Prefix)
* `PID-7`: `19800101` (Date of Birth - 01/01/1980)
* `PID-8`: `M` (Administrative Sex - Male)
* `PID-11`: (Patient Address)
  * `PID-11.1`: `123 Main St` (Street Address)
  * `PID-11.2`: `Apt 4B` (Other Designation)
  * `PID-11.3`: `Seattle&Washington` (City)
    * `PID-11.3.1`: `Seattle` (City Name)
    * `PID-11.3.2`: `Washington` (State Name)
  * `PID-11.4`: `WA` (State or Province)
  * `PID-11.5`: `98101` (Zip or Postal Code)
* `PID-13`: (Phone Number - Home)
  * `PID-13.1`: `5551234` (Number)
  * `PID-13.2`: `PRN` (Telecommunication Use Code)
  * `PID-13.3`: `PH` (Telecommunication Equipment Type)
  * `PID-13.6`: `206` (Area Code)
* `PID-15`: `EN` (Primary Language - English)
* `PID-16`: `S` (Marital Status - Single)
* `PID-18`: `MRN12345` (Patient Account Number)
* `PID-19`: `123-45-6789` (SSN Number)

**PV1 (Patient Visit):**

* `PV1-1`: `1` (Set ID)
* `PV1-2`: `I` (Patient Class - Inpatient)
* `PV1-3`: (Assigned Patient Location)
  * `PV1-3.1`: `2NORTH` (Point of Care)
  * `PV1-3.2`: `2104` (Room)
  * `PV1-3.3`: `01` (Bed)
* `PV1-7`: (Attending Doctor)
  * `PV1-7.1`: `004777` (ID Number)
  * `PV1-7.2`: `GOOD` (Family Name)
  * `PV1-7.3`: `SIDNEY` (Given Name)
  * `PV1-7.4`: `J` (Middle Initial)
* `PV1-8`: (Referring Doctor)
* `PV1-10`: `CAR` (Hospital Service - Cardiology)
* `PV1-14`: `2` (Admit Source)
* `PV1-15`: `A0` (Ambulatory Status)
* `PV1-44`: `20230115` (Admit Date/Time - 15/01/2023)

**AL1 (Allergy Information):**

* `AL1-1`: `1` (Set ID)
* `AL1-2`: `DA` (Allergen Type Code - Drug Allergy)
* `AL1-3`: `ASPIRIN` (Allergen Code/Mnemonic/Description)
* `AL1-4`: `MO` (Allergy Severity Code - Moderate)
* `AL1-5`: `HIVES` (Allergy Reaction Code)
* `AL1-6`: `20220610` (Identification Date - 10/06/2022)

### Tóm tắt & Các thực hành tốt nhất

#### Tóm tắt cấu trúc tin nhắn HL7 v2.x:

1. **Message (Tin nhắn)**: Đơn vị trao đổi thông tin lớn nhất
2. **Segment (Phân đoạn)**: Nhóm logic các trường liên quan (MSH, PID, PV1...)
3. **Field (Trường)**: Đơn vị dữ liệu cơ bản, phân tách bằng `|`
4. **Component (Thành phần)**: Phân chia trường phức tạp, phân tách bằng `^`
5. **Sub-component (Thành phần phụ)**: Phân chia thành phần, phân tách bằng `&`

#### Các thực hành tốt nhất khi làm việc với tin nhắn HL7:

1. **Luôn kiểm tra phiên bản HL7** (MSH-12) trước khi xử lý tin nhắn
2. **Xử lý các ký tự escape** khi hiển thị dữ liệu hoặc lưu trữ
3. **Sử dụng hệ thống tham chiếu chính xác** khi truy xuất dữ liệu
4. **Bảo tồn các trường rỗng** (không loại bỏ)
5. **Giữ nguyên thứ tự các phân đoạn** khi chuyển tiếp tin nhắn
6. **Xác thực tin nhắn** theo profile hoặc định nghĩa
7. **Hiểu rõ ngữ cảnh của từng loại tin nhắn** (ADT, ORM, ORU...)
8. **Cẩn thận với ngày tháng** (định dạng YYYYMMDDHHMMSS)
9. **Sử dụng các công cụ phân tích HL7** để kiểm tra và gỡ lỗi tin nhắn
10. **Ghi lại tin nhắn gốc** trước khi xử lý hoặc sửa đổi
11. **Xử lý các ký tự Unicode đúng cách** để hỗ trợ nhiều ngôn ngữ
12. **Kiểm tra tính nhất quán của ID** giữa các phân đoạn liên quan

### Ví dụ phân tích tin nhắn thực tế và thách thức

#### Thử thách phân tích tin nhắn HL7 thực tế

Khi phân tích tin nhắn HL7 trong môi trường thực tế, bạn sẽ gặp nhiều thách thức. Hãy xem xét một tin nhắn ORU phức tạp và quy trình phân tích:

```
MSH|^~\&|LABSYS|MYLAB|HIS|HOSPITAL|20230512131530||ORU^R01|203945701|P|2.3|||AL|NE|USA||||
PID|1||12001423^^^MYLAB^MRN~90001234^^^HOSPITAL^MRN||SMITH^JOHN^Q^JR^DR||19670429|M|||123 MAIN ST.^APT. 3B^CHICAGO^IL^60610||312-555-1212^PRN^PH|312-555-2323^WPN^PH|||||90001234||||||||
ORC|RE|T-129375-1^LABSYS|80391273^HOSPITAL|||CM||||128473^JONES^MARY|||||||||
OBR|1|T-129375-1^LABSYS|80391273^HOSPITAL|CBC^COMPLETE BLOOD COUNT^L|||20230512094000|20230512095000|||||||||128473^JONES^MARY||||20230512131200|||F|||||||
OBX|1|NM|WBC^WHITE BLOOD CELL COUNT^L||10.5|10*3/uL|4.0-11.0|N|||F|||20230512130000|LAB1^FIRST LAB|
OBX|2|NM|RBC^RED BLOOD CELL COUNT^L||4.8|10*6/uL|4.5-5.5|N|||F|||20230512130000|LAB1^FIRST LAB|
OBX|3|NM|HGB^HEMOGLOBIN^L||14.2|g/dL|13.5-17.5|N|||F|||20230512130000|LAB1^FIRST LAB|
OBX|4|NM|HCT^HEMATOCRIT^L||42|%|41-50|N|||F|||20230512130000|LAB1^FIRST LAB|
OBX|5|NM|PLT^PLATELET COUNT^L||385|10*3/uL|150-450|N|||F|||20230512130000|LAB1^FIRST LAB|
OBX|6|FT|NOTE^NOTES AND COMMENTS^L||Patient sample slightly hemolyzed\.br\May affect some results\.br\Recommend repeat if clinically indicated.||||||F|||20230512130500|LAB1^FIRST LAB|
```

#### Thách thức phân tích:

1. **Nhiều định danh bệnh nhân**:
   * PID-3 chứa hai định danh: ID của phòng xét nghiệm và ID của bệnh viện
   * Phải xác định đúng định danh để khớp bệnh nhân
2. **Theo dõi mối quan hệ giữa phân đoạn**:
   * ORC chứa thông tin về chỉ định
   * OBR chứa thông tin về yêu cầu xét nghiệm cụ thể
   * Nhiều OBX chứa kết quả khác nhau cho cùng một xét nghiệm
3. **Xử lý các kiểu dữ liệu khác nhau**:
   * OBX-2 xác định kiểu dữ liệu: NM (Numeric) cho các kết quả số
   * FT (Formatted Text) cho ghi chú có định dạng
4. **Xử lý ký tự escape trong văn bản**:
   * Ghi chú trong OBX-6 sử dụng `\.br\` cho ngắt dòng
   * Cần xử lý đúng khi hiển thị cho người dùng

#### Quy trình phân tích:

1. **Xác minh MSH**:
   * Đây là tin nhắn ORU^R01 (kết quả quan sát)
   * Phiên bản HL7 2.3
   * Gửi từ LABSYS tại MYLAB đến HIS tại HOSPITAL
   * Thời gian gửi: 13:15:30, 12/05/2023
2. **Xác định thông tin bệnh nhân (PID)**:
   * Tên: SMITH JOHN Q JR DR
   * Ngày sinh: 29/04/1967
   * Giới tính: Nam
   * Địa chỉ: 123 MAIN ST., APT. 3B, CHICAGO, IL, 60610
   * Số điện thoại: 312-555-1212 (nhà) và 312-555-2323 (nơi làm việc)
3. **Xác minh thông tin chỉ định (ORC + OBR)**:
   * Thông tin ORC chung về chỉ định
   * ID chỉ định: T-129375-1 (hệ thống gốc), 80391273 (hệ thống nhận)
   * Xét nghiệm: CBC (Complete Blood Count)
   * Bác sĩ chỉ định: JONES MARY (ID: 128473)
   * Thời gian lấy mẫu: 09:40:00, 12/05/2023
   * Trạng thái kết quả: F (Final)
4. **Phân tích các kết quả (OBX)**:
   * 5 kết quả số (OBX-2 = NM):
     * WBC: 10.5 10\*3/uL (bình thường)
     * RBC: 4.8 10\*6/uL (bình thường)
     * HGB: 14.2 g/dL (bình thường)
     * HCT: 42 % (bình thường)
     * PLT: 385 10\*3/uL (bình thường)
   * 1 ghi chú (OBX-2 = FT):
     * Mẫu hơi bị tan máu
     * Có thể ảnh hưởng đến một số kết quả
     * Khuyến nghị lặp lại nếu có chỉ định lâm sàng
5. **Lưu ý các chi tiết quan trọng**:
   * Tất cả các kết quả đều trong phạm vi bình thường (OBX-8 = N)
   * Thời gian xét nghiệm: 13:00:00, 12/05/2023
   * Phòng xét nghiệm thực hiện: LAB1 (FIRST LAB)

#### Tầm quan trọng của phân tích chi tiết:

1. **Đảm bảo an toàn bệnh nhân**: Xác định đúng bệnh nhân và các kết quả
2. **Tích hợp hệ thống chính xác**: Ánh xạ dữ liệu chính xác giữa các hệ thống
3. **Xử lý ngoại lệ**: Phát hiện và xử lý các trường hợp đặc biệt
4. **Hiểu ngữ cảnh lâm sàng**: Kết hợp các thông tin từ nhiều phân đoạn

### Sử dụng cấu trúc HL7 để truy xuất thông tin

Khi làm việc với tin nhắn HL7, bạn thường cần truy xuất thông tin cụ thể. Dưới đây là cách tiếp cận có cấu trúc:

#### Các câu hỏi phổ biến và cách tìm thông tin:

1. **Ai gửi tin nhắn này và khi nào?**
   * Nguồn gửi: MSH-3 và MSH-4
   * Thời gian gửi: MSH-7
2. **Tin nhắn này về bệnh nhân nào?**
   * Định danh: PID-3
   * Tên: PID-5
   * Ngày sinh: PID-7
   * Giới tính: PID-8
3. **Loại sự kiện/thông tin này là gì?**
   * Loại tin nhắn: MSH-9
   * Sự kiện cụ thể: MSH-9.2
4. **Ai chỉ định xét nghiệm/thủ thuật này?**
   * Bác sĩ chỉ định: ORC-12 hoặc OBR-16
5. **Kết quả là gì và có bất thường không?**
   * Loại xét nghiệm: OBR-4
   * Kết quả: OBX-5
   * Giá trị tham chiếu: OBX-7
   * Cờ bất thường: OBX-8
6. **Thời gian của các sự kiện chính?**
   * Thời gian chỉ định: ORC-9
   * Thời gian lấy mẫu: OBR-7
   * Thời gian xét nghiệm: OBX-14
   * Thời gian kết quả: OBR-22

#### Ví dụ truy xuất thông tin:

**Tìm tất cả kết quả xét nghiệm bất thường:**

1. Quét qua tất cả phân đoạn OBX
2. Kiểm tra giá trị OBX-8 (Abnormal Flags)
3. Trích xuất kết quả (OBX-5) và tên xét nghiệm (OBX-3) khi OBX-8 = "H" hoặc "L"

**Tìm thông tin liên hệ của bệnh nhân:**

1. Xác định số điện thoại từ PID-13 và PID-14
2. Xác định địa chỉ từ PID-11
3. Kiểm tra thông tin liên hệ khẩn cấp từ NK1 (nếu có)

**Xác định lịch sử chỉ định:**

1. Lấy Placer Order Number từ ORC-2 hoặc OBR-2
2. Lấy Filler Order Number từ ORC-3 hoặc OBR-3
3. Kết hợp với thời gian chỉ định và thời gian kết quả

### Kết luận

Hiểu rõ cấu trúc tin nhắn HL7 v2.x là nền tảng cần thiết cho bất kỳ ai làm việc với hệ thống thông tin y tế. Cấu trúc phân cấp với segments, fields, components và sub-components tạo nên một hệ thống linh hoạt cho việc trao đổi dữ liệu y tế phức tạp.

Các ký tự phân tách, trường lặp lại, và ký tự escape tạo thành cú pháp HL7 đặc trưng, cho phép truyền tải thông tin có cấu trúc một cách hiệu quả. Hệ thống tham chiếu rõ ràng của HL7 giúp truy xuất thông tin chính xác từ tin nhắn.

Khi làm việc với HL7 v2.x, việc tuân thủ các thực hành tốt nhất, hiểu rõ cấu trúc tin nhắn, và sử dụng các công cụ phù hợp sẽ giúp bạn xử lý dữ liệu y tế một cách hiệu quả và chính xác.

