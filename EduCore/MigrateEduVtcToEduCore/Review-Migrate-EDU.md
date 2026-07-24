# Review Migrate EDU

## I. Mục tiêu & phạm vi

### 1. Mục tiêu

- Chuyển đổi toàn bộ dữ liệu học tập từ hệ thống **edu.vtc.vn (Moodle 4.2)** sang hệ thống mới **EduCore (microservice)**.
- Đảm bảo:
  - Không mất dữ liệu quan trọng.
  - Giữ được:
    - Dữ liệu khóa học  
    - Tài khoản  
    - Dữ liệu học tập (hoàn thành, bài học, điểm, chứng chỉ)

### 2. Phạm vi

#### Bao gồm

- **Tài khoản người dùng**
  - Học viên  
  - Mật khẩu  
  - Nhóm/khóa học viên  
  - Chức danh  

- **Khóa học**
  - Lĩnh vực khóa học  
  - Khóa học  
  - Bài học  
  - Phôi chứng chỉ  
  - Cấu hình hoàn thành:
    - Khóa học  
    - Bài học  
    - Chứng chỉ  

- **Dữ liệu học tập**
  - Đăng ký khóa học  
  - Bài học: điểm, thông tin hoàn thành  
  - Khóa học: điểm, thông tin hoàn thành  
  - Chứng chỉ khóa học  

#### Không bao gồm (nếu có)

- Log hệ thống cũ

---

## II. Kịch bản migrate

- `moodle.cohort` → **Đơn vị Tennal**
  - Danh sách user trong cohort → **Customer của Tennal**
  - Danh sách khóa học user được đăng ký:
    - Lấy lĩnh vực khóa học → **Chương trình học**
      - Khóa học trong lĩnh vực → **Khóa học trong chương trình**
        - Bài học → **Bài học trong chương trình**

---

## III. Plan

### 1. Tài khoản người dùng

| Edu Moodle | Edu Core |
|-------------|-----------|
| cohort | Tenants |
| user_info.parent_department | Groups |
| user.department | JobTitles |
| user | Customers |

- Tổng số account active: **> 353,418**

```sql
SELECT count(id) FROM mdl_user WHERE confirmed = '1' AND deleted = '0' AND suspended = '0';
```

#### Mapping

- `cohort` → `Tenants`
- `parent_department` → `Groups`
- `department` → `JobTitles`
- `user` → `Customers`

---

### Password

- Password hashing functions được PHP Core cung cấp:
  - Thuật toán: PASSWORD_BCRYPT, PASSWORD_ARGON2I, PASSWORD_ARGON2ID
  - moodle đang sử dụng PASSWORD_DEFAULT: từ php > 5.5 => sẽ là PASSWORD_DEFAULT = PASSWORD_BCRYPT
  - php doc: https://www.php.net/manual/en/function.password-hash.php
- Password verify functions được PHP Core cung cấp:
  - function sẽ dựa vào các ký tự đầu có format như: $2y$10$4........ để phân biệt mà verify theo thuật toán nào
  - php doc: https://www.php.net/manual/en/function.password-verify.php

---

### 1.1 Cohort → Tenants

| Moodle | EduCore |
|--------|---------|
| name | Name |
| idnumber | Code |

---

### 1.2 Parent Department → Groups

| Moodle | EduCore |
|--------|---------|
| user_info.parent_department.value | Name |

---

### 1.3 Department → JobTitles

| Moodle | EduCore |
|--------|---------|
| user.department | Name |

---

### 1.4 User → Customers

#### Thông tin tài khoản

| Moodle | EduCore |
|--------|---------|
| username | Username |
| password | Password |
| idnumber | IdNumber |
| firstname, lastname | Name |
| email | Email |
| phone1 | PhoneNumber |
| institution | (chưa có) |
| address | Address |
| city | Province |
| profile.dateofbirth | Birthday |
| profile.gender | Gender |

#### Avatar

- Lấy từ `mdl_files` với `id = user.picture`
- Upload lại qua API

---

### Quan hệ học viên

- `Customers.TenantId`
- `CustomerGroups`
- `CustomerJobTitles`

---

### SSO Profile

#### Google / Facebook (OAuth2)

| Moodle | EduCore |
|--------|---------|
| username | Name |
| email | ProviderId |
| oauth2_issuer.name | Provider |

> Moodle chỉ linked bằng email, chưa có providerId.

---

#### MyVTC

| Moodle | EduCore |
|--------|---------|
| accountid | ProviderId |
| "myvtc" | Provider |
| fullname | Name |
| linkavatar | Avatar |

---

#### Zalo

| Moodle | EduCore |
|--------|---------|
| accountid | ProviderId |
| "zalo" | Provider |
| accountfullname | Name |
| accountlinkavatar | Avatar |

---

## 2. Khóa học

### 2.2 Lĩnh vực khóa học

- `course_categories` (depth = 2, visible = 1)

| Moodle | EduCore |
|--------|---------|
| idnumber | Code |
| name | Title |
| description | Description |

Ảnh:
- `component = course`
- `filearea = category`

---

### 2.3 Khóa học

| Moodle | EduCore |
|--------|---------|
| fullname | Title |
| shortname | Shortname |
| idnumber | Code |
| summary | Description |

Ảnh:
- `filearea = overviewfiles`

---

### 2.4 Bài học

| Module | Số lượng |
|--------|---------|
| scorm | 370 |
| resource | 317 |
| quiz | 122 |
| customcert | 52 |
| url | 41 |
| forum | 39 |
| feedback | 12 |
| zoom | 3 |
| assign | 2 |
| page | 1 |
| book | 1 |
| label | 1 |

#### Quiz
- Type: Multiple choice, True/False, Matching, Short answer, Numerical, Essay, All-or-Nothing Multiple Choice, Calculated, Calculated multichoice, Calculated simple, Drag and drop into text, Drag and drop markers, Drag and drop onto image, Embedded answers (Cloze), Random short-answer matching, Select missing words
- Các loại Check trên DB đang dùng:
  - Multiple choice  
  - True/False  
  - Matching  
  - Short answer  
  - Essay  
  - Cloze  

> Thực tế dùng [hỏi anh trung]: **Multiple choice, All-or-Nothing, True/False**

---

### 2.5 Phôi chứng chỉ

- Sử dụng phôi mới

---

### 2.6 Cấu hình điều kiện hoàn thành

- Cấu hình điều kiện Khóa học  
  - Điều kiện chung:
    - Hoàn thành khi thỏa mãn tất cả các điều kiện
    - Hoàn thành khi bất kỳ 1 điều kiện nào được thỏa mãn
  - Điều kiện: Hoạt động hoàn thành
  - Điều kiện: Hoàn thành các khóa học khác
  - Điều kiện: Ngày giờ
  - Điều kiện: Thời gian ghi danh
  - Điều kiện: Rút tên
  - Điều kiện: Điểm số khóa học
  - Điều kiện: Tự hoàn thành
  - Điều kiện: Hoàn thành thủ công bởi những người khác
- Cấu hình điều kiện Bài học 
  - ko xác đinh hoàn thành
  - học viên tự đánh dấu là hoàn thành
  - phải xem hoạt động để hoàn thành
  - Mỗi loại bài học sẽ có thêm các config điều kiện riêng theo loại đó 

> Đây là 1 trong các plugin phức tạp nhất của moodle, là tổ hợp của rất nhiều plugin khác nhau => rất khó để migrate 

---

## 3. Dữ liệu học tập

- Đăng ký khóa học
- Bài học: điểm, hoàn thành
- Khóa học: điểm, hoàn thành
- Chứng chỉ khóa học

> Dữ liệu về điểm và trạng thái hoàn thành từng bài học khóa học có thể lấy và migrate được

> Dữ liệu học của từng attempt thì đang chưa có cách migrate sang hệt thống mới 
