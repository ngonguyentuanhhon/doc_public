# Thiết kế Forum Service (Bản chốt cuối)

> **Trạng thái:** Final sau toàn bộ quá trình brainstorm\
> **Phạm vi:** Forum Service cho LMS\
> **CSDL:** SQL Server\
> **Framework:** Laravel

------------------------------------------------------------------------

# 1. Tổng quan Domain

Forum Service chỉ quản lý **nội dung thảo luận**, không sở hữu:

-   Tenant
-   Program/Course/Lesson
-   Customer/Teacher
-   Enrollment
-   Authentication/SSO
-   File binary

``` text
Tenant/LMS Context
        │
        ▼
    Category
        │
        ▼
      Topic
        │
        ▼
      Post
     /    \
Reaction  Attachment
    │
    ▼
Reaction Stats
```

Toàn bộ domain chỉ còn **6 bảng**:

-   `forum_categories`
-   `forum_topics`
-   `forum_posts`
-   `forum_post_reactions`
-   `forum_post_reaction_stats`
-   `forum_attachments`

Không còn các entity như `DiscussionTopic`, `QnaTopic`, `Answer`,
`Reply`...

------------------------------------------------------------------------

# 2. Nguyên tắc thiết kế

## 2.1 Không dùng `topic_type`

Không lưu:

-   discussion
-   qna
-   announcement

Thay vào đó Topic được quyết định bởi **capability**.

``` text
allow_replies
allow_reactions
allow_attachments
allow_accept_answers
```

Ví dụ preset:

  Loại           replies   reactions   attachments   accept answers
  -------------- --------- ----------- ------------- ----------------
  Discussion     ✅        ✅          ✅            ❌
  Q&A            ✅        ✅          ✅            ✅
  Announcement   ❌        ✅          ✅            ❌

Đây chỉ là preset của UI, không phải kiểu dữ liệu trong DB.

------------------------------------------------------------------------

## 2.2 Capability khác Permission

-   Capability = Topic hỗ trợ chức năng gì.
-   Permission = Actor có quyền thực hiện hay không.

Ví dụ:

``` text
allow_accept_answers = true
```

không có nghĩa mọi người đều được đánh dấu đáp án.

------------------------------------------------------------------------

# 3. Forum Category

Category là nơi chứa Topic.

Có thể gắn vào nhiều context:

``` text
tenant
program
course
lesson
```

thông qua:

``` text
context_type
context_id
```

## Schema cuối

``` text
forum_categories
------------------------
id
tenant_id

context_type
context_id

name
description

image_id
image_path

state
topic_count

actor_type
actor_id

created_at
updated_at
deleted_at
```

## State

``` text
active
locked
hidden
```

  State    Hiển thị   Tạo Topic   Tương tác
  -------- ---------- ----------- -----------
  active   ✅         ✅          ✅
  locked   ✅         ❌          ❌
  hidden   ❌         ❌          ❌

`locked` = chỉ đọc.

------------------------------------------------------------------------

## topic_count

Định nghĩa:

> Số Topic đang hiển thị.

Ví dụ:

-   active
-   active
-   locked
-   hidden

thì:

``` text
topic_count = 3
```

------------------------------------------------------------------------

# 4. Topic

## Schema cuối

``` text
forum_topics
------------------------
id
category_id

title
content

image_id
image_path

allow_replies
allow_reactions
allow_attachments
allow_accept_answers

state
is_pinned

post_count

actor_type
actor_id

created_at
updated_at
deleted_at
```

## State

``` text
active
locked
hidden
```

### Active

-   xem
-   tạo post
-   reply
-   reaction
-   attachment
-   accept answer (nếu đủ điều kiện)

### Locked

Vẫn xem được nhưng:

-   không tạo post
-   không reply
-   không reaction
-   không upload
-   không accept answer

### Hidden

Ẩn hoàn toàn với người dùng.

------------------------------------------------------------------------

## is_pinned

Pin Topic trong Category.

Không liên quan đến state.

------------------------------------------------------------------------

## post_count

Định nghĩa:

> Tổng số Post đang hiển thị (bao gồm root và reply).

Ví dụ:

``` text
Topic
├── Post A
│   ├── Reply A1
│   └── Reply A2
└── Post B
```

`post_count = 4`

------------------------------------------------------------------------

# 5. Post

Không tách Question/Answer/Reply.

Tất cả đều là Post.

## Schema

``` text
forum_posts
------------------------
id
topic_id
parent_id

content

state
is_pinned
is_accepted_answer

reply_count

actor_type
actor_id

created_at
updated_at
deleted_at
```

------------------------------------------------------------------------

# 6. Reply

Reply chỉ là Post có:

``` text
parent_id != NULL
```

Ví dụ:

``` text
Post A
├── Reply A1
├── Reply A2
└── Reply A3
```

Không cần bảng Reply riêng.

## Nested Reply

Được phép.

``` text
Post A
└── Reply A1
    └── Reply A1.1
```

`reply_count` chỉ đếm **direct child**.

Ví dụ:

``` text
Post A
├── Reply A1
│   └── Reply A1.1
└── Reply A2
```

Kết quả:

``` text
A.reply_count = 2
A1.reply_count = 1
```

------------------------------------------------------------------------

# 7. Post State

Chỉ có:

``` text
active
hidden
```

Không cần `locked`.

Muốn khóa toàn bộ discussion thì dùng:

``` text
topic.state = locked
```

------------------------------------------------------------------------

# 8. Pin Post

`is_pinned` trên Post chỉ có tác dụng trong Topic.

Ví dụ:

``` text
Topic
├── 📌 Hướng dẫn làm bài
├── Câu hỏi
└── Thảo luận
```

------------------------------------------------------------------------

# 9. Q&A

Q&A chỉ là:

``` text
allow_accept_answers = true
```

Không có:

-   forum_topic_answers
-   accepted_post_id

Post có:

``` text
is_accepted_answer
```

## Một Topic có nhiều đáp án đúng

Ví dụ:

``` text
Question
├── Answer A ⭐
├── Answer B ⭐
├── Answer C
└── Answer D ⭐
```

Được phép nhiều `is_accepted_answer = true`.

## Rule

Backend phải validate:

-   Topic bật `allow_accept_answers`
-   Chỉ direct child của Question mới được đánh dấu đáp án.

------------------------------------------------------------------------

# 10. Reaction

Reaction là **interaction event**.

Mỗi lần click tạo một record.

Ví dụ:

``` text
A 👍
A 👍
A ❤️
```

Kết quả:

``` text
like = 2
love = 1
```

Không có:

-   unique theo user
-   toggle
-   unreact

------------------------------------------------------------------------

# 11. forum_post_reactions

``` text
forum_post_reactions
------------------------
id
post_id

reaction_type

actor_type
actor_id

created_at
```

Ví dụ:

  post   type   actor
  ------ ------ ----------
  100    like   customer
  100    like   customer
  100    love   teacher

------------------------------------------------------------------------

# 12. Reaction Type

Lưu dạng string.

Ví dụ:

``` text
like
love
laugh
wow
sad
angry
fire
```

Không tạo cột:

``` text
like_count
love_count
...
```

------------------------------------------------------------------------

# 13. Reaction Stats

## Schema

``` text
forum_post_reaction_stats
------------------------
id
post_id
reaction_type
count

created_at
updated_at
```

Constraint:

``` text
UNIQUE(post_id, reaction_type)
```

Ví dụ:

  type    count
  ------- -------
  like    125
  love    31
  laugh   7

## Flow

``` text
User
 ↓
Click 👍
 ↓
INSERT reaction
 ↓
UPDATE reaction_stats
```

------------------------------------------------------------------------

# 14. Actor

Customer và Teacher là hai entity riêng.

Forum chỉ lưu:

``` text
actor_type
actor_id
```

Ví dụ:

``` text
customer
teacher
```

Không tạo bảng `users`.

------------------------------------------------------------------------

## Dùng ở đâu?

-   Category
-   Topic
-   Post
-   Reaction
-   Attachment

Đều dùng chung:

``` text
actor_type
actor_id
```

------------------------------------------------------------------------

# 15. Actor khác Permission

`actor_type = teacher`

không đồng nghĩa có quyền moderate.

Permission vẫn do LMS quyết định.

------------------------------------------------------------------------

# 16. Authorization

Forum không quản lý:

-   Enrollment
-   Teacher Assignment
-   Membership

LMS sẽ quyết định Actor có quyền vào context đó hay không.

------------------------------------------------------------------------

# 17. Permission Matrix

  Chức năng       Teacher   Customer
  --------------- --------- ----------
  Xem Category    ✅        ✅
  Xem Topic       ✅        ✅
  Tạo Topic       ✅        ✅
  Reply           ✅        ✅
  Edit bài mình   ✅        ✅
  Xóa bài mình    ✅        ✅
  Reaction        ✅        ✅
  Attachment      ✅        ✅
  Pin Topic       ✅        ❌
  Lock Topic      ✅        ❌
  Hide Topic      ✅        ❌
  Hide Post       ✅        ❌
  Accept Answer   ✅        ❌

------------------------------------------------------------------------

# 18. Attachment

Forum chỉ lưu metadata.

Binary thuộc File Service.

## Schema

``` text
forum_attachments
------------------------
id

attachable_type
attachable_id

file_id
file_name
file_size
file_path
mime_type

actor_type
actor_id

created_at
updated_at
```

## attachable_type

Chỉ còn:

``` text
category
topic
post
```

------------------------------------------------------------------------

# 19. Ảnh Category và Topic

Đây là thay đổi cuối cùng.

Không tạo record trong `forum_attachments`.

Category và Topic lưu trực tiếp:

``` text
image_id
image_path
```

Ví dụ:

### Category

``` text
id
name
image_id
image_path
```

### Topic

``` text
id
title
image_id
image_path
```

Như vậy:

-   đơn giản hơn
-   không cần `category_image`
-   không cần `topic_image`

------------------------------------------------------------------------

# 20. Attachment khác Image

### Attachment

Lưu trong:

``` text
forum_attachments
```

Dùng cho:

-   PDF
-   DOCX
-   ZIP
-   File đính kèm

### Image

Lưu trực tiếp trên:

-   `forum_categories`
-   `forum_topics`

------------------------------------------------------------------------

# 21. Hệ thống thống kê

``` text
Category
└── topic_count

Topic
└── post_count

Post
├── reply_count
└── reaction_stats
```

Ý nghĩa:

  Stats            Định nghĩa
  ---------------- -------------------------------
  topic_count      số Topic visible
  post_count       số Post visible
  reply_count      số direct child visible
  reaction_stats   số interaction theo từng type

------------------------------------------------------------------------

# 22. Hidden ảnh hưởng Stats

Ví dụ:

``` text
Topic
├── Post A
├── Post B
└── Post C
```

Nếu:

``` text
Post B → hidden
```

thì:

``` text
post_count -= 1
```

Nếu Reply bị hidden:

``` text
reply_count -= 1
post_count -= 1
```

Reaction cũ vẫn giữ.

------------------------------------------------------------------------

# 23. Quy tắc cập nhật Counter

Không update trực tiếp ở Controller.

Nên gom trong Service.

Ví dụ:

``` text
Create Topic
→ topic_count++

Create Post
→ post_count++

Create Reply
→ post_count++
→ parent.reply_count++

Reaction
→ reaction_stats++
```

Các thao tác:

-   hide
-   unhide
-   delete
-   restore

đều phải đi qua cùng một logic.

------------------------------------------------------------------------

# 24. Kafka

Flow đề xuất:

``` text
DB Transaction
│
├── forum_post_reactions
├── forum_post_reaction_stats
└── outbox_events
        │
        ▼
Kafka
```

Downstream:

-   Analytics
-   Notification
-   Recommendation
-   Gamification

------------------------------------------------------------------------

# 25. Ranh giới Service

## Forum sở hữu

-   Category
-   Topic
-   Post
-   Reaction
-   Reaction Stats
-   Attachment metadata

## Forum không sở hữu

-   Tenant
-   Program
-   Course
-   Lesson
-   Customer
-   Teacher
-   Enrollment
-   Authentication
-   File binary

------------------------------------------------------------------------

# 26. ERD cuối cùng

``` text
Category
    │
    ▼
Topic
    │
    ▼
Post
 ├── Reply (self reference)
 ├── Reaction
 ├── Reaction Stats
 └── Attachment

Category.image_id
Category.image_path

Topic.image_id
Topic.image_path
```

------------------------------------------------------------------------

# 27. Checklist quyết định cuối

  Quyết định                                Kết quả
  ----------------------------------------- ----------------------
  topic_type                                ❌
  Discussion/Q&A entity riêng               ❌
  Reply entity                              ❌
  Answer entity                             ❌
  parent_id                                 ✅
  Nested Reply                              ✅
  Nhiều Accepted Answer                     ✅
  is_accepted_answer                        ✅
  Reaction là event                         ✅
  Duplicate reaction                        Cho phép
  Reaction Stats riêng                      ✅
  actor_type + actor_id                     ✅
  State Category                            active/locked/hidden
  State Topic                               active/locked/hidden
  State Post                                active/hidden
  Pin Topic                                 ✅
  Pin Post                                  ✅
  topic_count                               ✅
  post_count                                ✅
  reply_count                               ✅
  Image lưu trực tiếp trên Category/Topic   ✅
  category_image attachable_type            ❌
  topic_image attachable_type               ❌
  File binary do Forum quản lý              ❌
  Kafka                                     Khuyến nghị
  Outbox                                    Khuyến nghị

------------------------------------------------------------------------

# 28. Kiến trúc cuối

``` text
            LMS
             │
             ▼
      Authorization
             │
             ▼
        Forum Service
             │
     ┌───────┴────────┐
     │                │
  Category          Topic
                     │
             capability + state
                     │
                     ▼
                   Post
         ├──────────┼──────────┐
         │          │          │
      Reply     Reaction   Attachment
                    │
                    ▼
              Reaction Stats
```

## Kết luận

Thiết kế cuối cùng giữ domain rất **phẳng (flat)** và đủ linh hoạt để hỗ
trợ:

-   Discussion
-   Q&A
-   Announcement
-   Reply nhiều tầng
-   Nhiều Accepted Answer
-   Reaction theo từng loại
-   Pin
-   Moderation
-   Attachment
-   Ảnh Category/Topic
-   Multi-tenant
-   Customer/Teacher độc lập
-   Authorization ngoài
-   File Service ngoài
-   Kafka event-driven

Forum Service chỉ tập trung vào **quản lý nội dung và tương tác**, còn
Identity, Authorization và File Storage được giao cho các service chuyên
trách.
