# VTC-ITC LMS

---

## I. MEETING NOTE
- [MeetingNote](MeetingNote.md)

---
## II. LIỆT KÊ VÀ PHÂN TÍCH YÊU CẦU NGHIỆP VỤ
Dựa vào biên bản, gom nhóm và phân tích các yêu cầu mà VTC-ITC mong muốn như sau:

### 1. Nền tảng và Vận hành
**Yêu cầu:**
- Hệ thống LMS phải được cung cấp theo hình thức white-label
- VTC-ITC sẽ cần một landing page riêng
- VTC-ITC sẽ cần 1 CMS riêng: trực tiếp quản lý, vận hành hệ thống.

> Đánh giá: EduCore đang có thể dáp ứng, tính chỉnh thêm các nhận diện thương hiệu trên các page cho VTC-ITC (logo, tên đơn vị, màu săc ...)
> - Landing Page: Tích hợp hoặc cung cấp giao diện Landing page riêng cho người dùng VTC-ITC.
> - Customer Page: Page học viện học
> - Admin CMS: Giao diện quản trị toàn quyền cho đội ngũ VTC Intecom.

### 2. Quản lý học viên
**Yêu cầu:**
- Phải có tính năng import user lên hệ thống _( format file import 2 bên sẽ trao đổi )_

> Đánh giá: EduCore hiện đang có đủ các tính năng cơ bản để quản lý học viên, nhưng để vận hành thực tế thì vẫn có các tính năng chưa được thuận tiện lắm. Cần xem xét nâng cấp
> - import học viên: 
>   - chưa có logic update học viên đã có: 1 case cần update thông tin học viên hàng loạt sẽ rất cần
> - Chưa có tính năng quên mật khẩu cho học viên: tình trạng quên mật khẩu khá nhiều
> - Nhóm học viên:
>   - chưa có tính năng thêm học viên vào nhóm:
>   - chưa có tính năng bỏ học viên khỏi nhóm
> - Chức vụ:
>   - chưa có tính năng thêm học viên vào Chức vụ:
>   - chưa có tính năng bỏ học viên khỏi Chức vụ
> - Đăng ký học viên vào chương trinh học:
>   - chưa có tính năng thêm học viên hàng loạt : 
>     - => hiện mới có add thủ công
>     - => nên có thêm user theo nhóm học viên hoặc chức vụ
>     - => bỏ user theo nhóm và chức vụ

### 3. Lộ trình đào tạo & Khung năng lực
**Yêu cầu:**
- Triển khai học tập theo khung năng lực gồm 4 nhóm: 
  + Năng lực lõi: ...
  + Năng lực chuyên môn: ...
  + Năng lực quản lý: ...
  + Năng lực bổ trợ: ...
- Lộ trình học của mỗi vị trí chia theo giai đoạn _(ví dụ: Foundation → Intermediate → Advanced)_
- Học viên phải hoàn thành khóa học và đạt điểm tối thiểu để được chuyển sang giai đoạn tiếp theo


> Đánh giá: 
> - Phần năng lực:
>   + Chưa rõ yêu cầu cụ thể , cần làm rõ thêm: ý hiểu cá nhân thì 
>     + 4 năng lực chính, mỗi cái sẽ có 3 cấp độ (Foundation, Intermediate, Advanced) 
>       + => giống luồng năng lực của mình đang chạy
>       + => có thể có thêm 1 bài kiểm tra trong lộ trình để xác định hoàn thành cấp độ năng lực
>       + => chuyển tiếp các năng lực có thể tận dụng điều kiện bắt đầu bài học 
>     + có issue là: 
>       + có khái niệm điểm trong bài học , khóa học => mình chưa có
> - Điều kiện bắt đầu bài học , khóa học :
>   + phát sinh thêm điều kiện điểm
>   + => logic về điểm bài học, khóa học: mình cũng đang chưa có 
>   + => nano material: cũng chưa có setting điểm từng quiz, điểm vần đang fix cứng từng câu
>   + => khả năng sẽ phát sinh thêm điều kiện hoàn thành liên quan đến điểm

### 4. Quản lý Nội dung học liệu
**Yêu cầu:**
- Sủ dung dữ liệu có sẵn trong kho EDU-VTC và cả các học liệu 2 bên hợp tác biên soạn

> Đánh giá: phần này theo đánh giá của em thì đang ko có vấn đề gì
> - các tính năng quản lý, editor học liệu em đánh giá là đang khá đủ dùng
> - issue: chưa có cấu hình, setting điểm cho từng loại bài học 
>   + hiện nano chưa có đang fix cững điểm
>   + scorm thì có thể áp dụng luôn điểm của học liệu

### 5. Kiểm tra, đánh giá khóa học
**Yêu cầu:**
- Mỗi khóa học phải được gắn điểm số/credit dựa trên độ khó
- Điểm này được tích lũy để làm căn cứ đánh giá lộ trình và xét thăng tiến
- Việc kiểm tra đánh giá đa dạng: kiểm tra cuối kỳ, case study, project và review từ cấp quản lý

> Đánh giá: các yêu cầu trên hiện em đáng giá là hệ thống chưa có
> - hệ thống chưa có logic về điểm bài học, khóa học và điệu kiện hoàn thành liên quan đến điểm
> - Kiểm tra đánh giá phải bổ sung thêm:
>   + kiểm tra tự luận -  đánh giá bài - chấm điểm

---