# AI IDE & AI CLI
## Kinh nghiệm ứng dụng trong phát triển Backend

> Chia sẻ kinh nghiệm sử dụng AI IDE và AI CLI trong quá trình phát triển Backend tại VTC Edu.

---

# Mục tiêu

Sau buổi chia sẻ, người nghe có thể:

- Hiểu bức tranh AI Coding hiện nay.
- Hiểu AI IDE và AI CLI.
- Biết workflow sử dụng AI trong phát triển Backend.
- Có thể áp dụng ngay vào công việc hằng ngày.

---

# Slide 1 - Các mô hình tích hợp AI vào quy trình phát triển phần mềm

## AI Coding hiện nay

| Nhóm | Mô tả | Ví dụ |
|------|------|-------|
| **Chat AI** | AI hoạt động độc lập, chủ yếu dùng để hỏi đáp, giải thích, phân tích hoặc sinh code. Người dùng thường phải copy/paste giữa AI và IDE. | ChatGPT, Claude, Gemini |
| **AI Plugin** | AI tích hợp vào IDE hiện có dưới dạng plugin, bổ sung autocomplete, chat, refactor... nhưng vẫn phụ thuộc IDE gốc. | GitHub Copilot, Continue, Cline, Roo Code |
| **AI IDE** | IDE được thiết kế xoay quanh AI, AI trở thành một phần của môi trường phát triển. | Cursor, Windsurf, Antigravity IDE |
| **AI CLI** | AI hoạt động qua Terminal, độc lập với IDE và có thể kết hợp với bất kỳ IDE nào. | Claude Code, Gemini CLI, Codex CLI, Antigravity CLI |

### Xu hướng phát triển

```text
Chat AI
      ↓
AI Plugin
      ↓
AI IDE
      ↓
AI CLI
```

### Thông điệp

AI đang ngày càng được tích hợp sâu hơn vào quy trình phát triển phần mềm, giúp:

- Giảm việc chuyển đổi ngữ cảnh (Context Switching).
- Hiểu tốt hơn context của project.
- Tăng hiệu quả trong toàn bộ Software Development Lifecycle.

---

# Slide 2 - Vì sao AI IDE & AI CLI?

## Trước đây

```text
Requirement

↓

ChatGPT

↓

Copy

↓

Paste

↓

IDE

↓

Lặp lại
```

## Hiện nay

```text
Requirement

↓

AI ngay trong môi trường phát triển

↓

Hiểu project

↓

Coding

↓

Review

↓

Testing
```

### Thông điệp

AI IDE và AI CLI đều hướng tới cùng một mục tiêu:

> Đưa AI đến gần quy trình phát triển phần mềm hơn, thay vì chỉ là một cửa sổ chat.

---

# Slide 3 - Khả năng của các AI Coding Tools hiện nay

Các AI Coding Tools hiện nay (AI Plugin, AI IDE, AI CLI) đều có thể hỗ trợ:

- Phân tích Requirement
- Giải thích source code
- Hiểu cấu trúc project
- Sinh code
- Refactor
- Review Pull Request
- Sinh Unit Test
- Sinh Documentation
- Chạy Terminal Command
- Agent Workflow
- MCP Integration
- ...

### Thông điệp

Khác biệt giữa các AI Coding Tools ngày nay không còn nằm ở khả năng của AI.

Khác biệt chủ yếu nằm ở:

- Cách tích hợp vào workflow.
- Trải nghiệm sử dụng.
- Triết lý thiết kế của công cụ.

---

# Slide 4 - AI IDE và AI CLI khác nhau ở đâu?

## AI IDE

### Triết lý

AI là một phần của IDE.

### Đặc điểm

- IDE hoàn chỉnh.
- Giao diện trực quan.
- Nhiều tính năng tích hợp.
- Trải nghiệm liền mạch trong quá trình coding.
- Thay thế IDE truyền thống.

---

## AI CLI

### Triết lý

AI hoạt động độc lập ở bất kỳ đâu có Terminal.

### Đặc điểm

- Nhỏ gọn.
- Tối giản.
- Linh hoạt.
- Không phụ thuộc IDE.
- Kết hợp với mọi IDE.
- Dễ mở nhiều phiên làm việc song song.
- Phù hợp với workflow automation.

---

### Thông điệp

AI IDE và AI CLI không thay thế nhau.

Hai công cụ bổ trợ cho nhau.

Việc lựa chọn phụ thuộc vào:

- Thói quen làm việc.
- Workflow của từng lập trình viên.
- Bài toán cần giải quyết.

---

# Slide 5 - Cách mình sử dụng tại VTC Edu

Trong công việc Backend mình sử dụng kết hợp:

- Antigravity IDE
- Antigravity CLI

## AI IDE

Thường sử dụng khi:

- Coding
- Refactor
- Explain Code
- Quick Fix

## AI CLI

Thường sử dụng khi:

- Phân tích Requirement
- Phân tích project
- Lập kế hoạch triển khai
- Review
- Automation
- Chạy nhiều task song song

### Thông điệp

Workflow này không phụ thuộc Antigravity.

Hoàn toàn có thể áp dụng tương tự với:

- Claude Code
- Gemini CLI
- Codex CLI

---

# Slide 6 - Workflow mình đang sử dụng

```text
Requirement

↓

AI Analysis

↓

AI Planning

↓

Developer Review

↓

AI Coding

↓

Developer Review

↓

Testing

↓

Merge
```

## Vai trò của AI và Developer

```text
Developer
(Cung cấp Context)

↓

AI
(Phân tích & Đề xuất)

↓

Developer
(Đánh giá & Quyết định)

↓

AI
(Triển khai)

↓

Developer
(Review & Merge)
```

### Thông điệp

AI không bắt đầu bằng Coding.

Mình luôn yêu cầu AI:

- Phân tích.
- Lập kế hoạch.
- Sau đó mới Coding.

---

# Slide 7 - Ví dụ thực tế

## Task

### Thêm API cập nhật Avatar cho User

---

## Bước 1

Đưa Requirement cho AI

Context:

- Laravel 12
- Repository Pattern
- Service Pattern
- Resource
- Module User
- MinIO Storage
- Coding Convention
- Chỉ cập nhật Avatar
- Không ảnh hưởng API khác

---

## Bước 2

Yêu cầu AI phân tích

Output mong muốn:

- Requirement Summary
- Business Flow
- Module ảnh hưởng
- Validation
- Security
- API Contract
- Database
- Storage Flow
- Risk
- Implementation Plan

---

## Bước 3

Developer Review Plan

Review:

- Business Flow
- Validation
- Permission
- Upload Flow
- Storage
- Rollback
- Exception
- URL trả về
- Edge Cases

Sau khi thống nhất mới Coding.

---

## Bước 4

AI Coding

Yêu cầu AI:

- Follow đúng Plan.
- Không thay đổi Architecture.
- Không sửa ngoài phạm vi Task.

---

## Bước 5

Developer Review

Review:

- Business Logic
- Validation
- Security
- Exception
- Performance
- Coding Convention

Sau đó mới Merge.

---

# Slide 8 - Một số kinh nghiệm khi sử dụng AI

## Context quan trọng hơn Prompt

Luôn cung cấp:

- Requirement
- Architecture
- Coding Convention
- Business Rules
- Constraint

---

## Luôn Review Plan trước khi Coding

Không yêu cầu AI viết code ngay.

---

## Kiểm soát Business Logic

AI hỗ trợ.

Developer quyết định.

---

## Không cung cấp thông tin nhạy cảm

Không gửi:

- .env
- Password
- API Key
- Secret
- Private Key
- Dữ liệu Production

---

## Không Merge nếu chưa Review

AI có thể sai.

Developer chịu trách nhiệm cuối cùng.

---

# Slide 9 - Kết luận

AI giúp tăng tốc:

- Phân tích
- Thiết kế
- Coding
- Review
- Documentation
- Testing

Nhưng:

> AI không thay thế lập trình viên.

Developer vẫn chịu trách nhiệm về:

- Business Logic
- Kiến trúc hệ thống
- Security
- Chất lượng sản phẩm

---
