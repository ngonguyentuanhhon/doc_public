# Forum Service — Final Design

> **Status:** Final design after brainstorming  
> **Scope:** Forum Service for LMS  
> **Database:** SQL Server  
> **ORM/Framework:** Laravel  
> **Principle:** Keep the forum domain flat, generic, and extensible.

---

## 1. Domain Overview

Forum Service manages discussion content only. It does not own LMS identity, enrollment, authorization, or binary file storage.

```text
Tenant / LMS Context
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
Reaction   Attachment
   │
   ▼
Reaction Stats
```

The final domain consists of **6 tables**:

```text
forum_categories
forum_topics
forum_posts
forum_post_reactions
forum_post_reaction_stats
forum_attachments
```

There are no separate entities for Discussion, Q&A, Announcement, Question, Answer, or Reply.

---

# 2. Core Design Principles

## 2.1 No `topic_type`

Do not create:

```text
topic_type
discussion
qna
announcement
```

A Topic's behavior is determined by capabilities:

```text
allow_replies
allow_reactions
allow_attachments
allow_accept_answers
```

Example presets:

### Discussion

```text
allow_replies        = true
allow_reactions      = true
allow_attachments    = true
allow_accept_answers = false
```

### Q&A

```text
allow_replies        = true
allow_reactions      = true
allow_attachments    = true
allow_accept_answers = true
```

### Announcement

```text
allow_replies        = false
allow_reactions      = true
allow_attachments    = true
allow_accept_answers = false
```

These are application/UI presets, not database types.

---

## 2.2 Capability != Permission

A capability means:

> The Topic supports this function.

Permission means:

> This Actor is allowed to perform this action.

For example:

```text
allow_accept_answers = true
```

does not mean every user can accept an answer.

The authorization layer must still determine whether the Actor has permission.

---

# 3. Forum Category

A Category is a container for Topics.

A Category can belong to an LMS context through:

```text
context_type
context_id
```

Examples:

```text
context_type = tenant
context_id   = 1
```

or:

```text
context_type = program
context_id   = 100
```

or:

```text
context_type = course
context_id   = 200
```

or:

```text
context_type = lesson
context_id   = 300
```

## Final schema

```text
forum_categories
--------------------------------
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

### Category state

```text
active
locked
hidden
```

Behavior:

| State | View | Create Topic | Interaction |
|---|---:|---:|---:|
| active | Yes | Yes | Yes |
| locked | Yes | No | No |
| hidden | No | No | No |

`locked` means the Category remains visible but is read-only.

`hidden` means it is hidden from normal users.

---

## 3.1 `topic_count`

`topic_count` is a denormalized statistic.

Definition:

> Number of visible Topics in the Category.

A `locked` Topic is still visible and therefore counted.

A `hidden` Topic is not counted.

Example:

```text
Topic A = active
Topic B = active
Topic C = locked
Topic D = hidden
```

Result:

```text
topic_count = 3
```

---

# 4. Topic

A Topic represents a discussion thread.

## Final schema

```text
forum_topics
--------------------------------
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

---

## 4.1 Topic state

```text
active
locked
hidden
```

### `active`

```text
View          Yes
Create Post   Yes
Reply         Yes
Reaction      Yes, if allow_reactions
Attachment    Yes, if allow_attachments
Accept        Yes, if allowed by capability + permission
```

### `locked`

```text
View          Yes

Create Post   No
Reply         No
Reaction      No
Attachment    No
Accept        No
```

A locked Topic is read-only.

### `hidden`

```text
View          No
Interaction   No
```

`deleted_at` remains independent from `state`.

---

## 4.2 Topic pin

```text
is_pinned
```

means the Topic is pinned inside its Category.

Pinning is independent from state.

Valid example:

```text
state      = locked
is_pinned  = true
```

---

## 4.3 `post_count`

`post_count` is a denormalized statistic.

Definition:

> Total number of visible Posts in the Topic, including root Posts and Replies.

Example:

```text
Topic
├── Post A
│   ├── Reply A1
│   └── Reply A2
└── Post B
```

Then:

```text
post_count = 4
```

---

# 5. Post

There are no separate `Question`, `Answer`, or `Reply` entities.

Everything is a Post.

## Final schema

```text
forum_posts
--------------------------------
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

---

# 6. Reply Model

A Reply is simply a Post with:

```text
parent_id != NULL
```

Example:

```text
Post A
├── Reply A1
├── Reply A2
└── Reply A3
```

Database:

```text
Reply A1.parent_id = Post A.id
Reply A2.parent_id = Post A.id
Reply A3.parent_id = Post A.id
```

No separate `forum_replies` table is required.

---

## 6.1 Nested Replies

Nested replies are allowed:

```text
Post A
└── Reply A1
    └── Reply A1.1
        └── Reply A1.1.1
```

`reply_count` counts only direct children.

Example:

```text
Post A
├── Reply A1
│   └── Reply A1.1
└── Reply A2
```

Then:

```text
Post A.reply_count  = 2
Reply A1.reply_count = 1
```

---

# 7. Post State

Post has:

```text
active
hidden
```

There is no `locked` state for Post.

A whole conversation is locked through:

```text
forum_topics.state = locked
```

A single Post can be hidden through:

```text
forum_posts.state = hidden
```

Behavior:

| State | View | Interaction |
|---|---:|---:|
| active | Yes | Yes |
| hidden | No | No |

---

# 8. Post Pin

Post also has:

```text
is_pinned
```

But this has a different meaning from Topic pinning.

```text
forum_topics.is_pinned
```

means:

> Pin the Topic inside a Category.

```text
forum_posts.is_pinned
```

means:

> Pin the Post inside a Topic.

Example:

```text
Topic
├── 📌 Post A
├── Post B
└── Post C
```

---

# 9. Accepted Answers

Q&A is represented by:

```text
allow_accept_answers = true
```

There is no:

```text
forum_topic_answers
```

table.

There is also no:

```text
accepted_post_id
```

on Topic.

Instead, Post has:

```text
is_accepted_answer
```

---

## 9.1 Multiple Accepted Answers

A Topic can have multiple accepted answers.

Example:

```text
Question
├── Answer A ⭐
├── Answer B ⭐
├── Answer C
└── Answer D ⭐
```

Database:

```text
post_id | is_accepted_answer
--------|-------------------
2       | true
3       | true
4       | false
5       | true
```

There is no one-answer-only restriction.

---

## 9.2 Accepted Answer Validation

Backend must validate:

```text
topic.allow_accept_answers = true
```

and the Post must be a valid answer.

Recommended rule for nested replies:

> Only a direct child of the Question Post can be marked as an accepted answer.

Example:

```text
Question
├── Answer A ⭐
│   └── Reply A1       ← cannot be accepted
└── Answer B ⭐
```

This avoids ambiguity between an answer and a discussion reply.

---

# 10. Reaction

Reaction is an interaction event, not a user state.

Each click creates a new record.

Example:

```text
Customer A → 👍
Customer A → 👍
Customer A → 👍
Customer B → 👍
Customer A → ❤️
```

Result:

```text
like = 4
love = 1
```

There is deliberately no:

```text
UNIQUE(post_id, actor_id)
```

constraint.

There is no reaction toggle.

There is no delete/unreact operation in this model.

---

# 11. `forum_post_reactions`

## Final schema

```text
forum_post_reactions
--------------------------------
id
post_id

reaction_type

actor_type
actor_id

created_at
```

Each interaction is an INSERT.

Example:

```text
id | post_id | reaction_type | actor_type | actor_id
---|---------|---------------|------------|---------
1  | 100     | like          | customer   | 10
2  | 100     | like          | customer   | 10
3  | 100     | love          | teacher    | 20
```

---

# 12. Reaction Type

`reaction_type` is stored as a string.

Examples:

```text
like
love
laugh
wow
sad
angry
fire
```

Do not create one database column per reaction:

```text
like_count
love_count
laugh_count
...
```

Adding a new reaction type should not require a schema migration.

---

# 13. Reaction Stats

The Forum needs stats by reaction type.

Use a separate table:

```text
forum_post_reaction_stats
```

## Final schema

```text
forum_post_reaction_stats
--------------------------------
id
post_id
reaction_type
count

created_at
updated_at
```

Unique constraint:

```text
UNIQUE(post_id, reaction_type)
```

Example:

```text
post_id | reaction_type | count
--------|---------------|------
100     | like          | 125
100     | love          | 31
100     | laugh         | 7
```

---

## 13.1 Reaction Flow

```text
User
  │
  │ click 👍
  ▼
Forum Service
  │
  ├── INSERT forum_post_reactions
  │
  └── UPDATE forum_post_reaction_stats
          SET count = count + 1
```

The reaction table is the interaction/event history.

The stats table is the query-optimized aggregate.

---

# 14. Actor Model

Customer and Teacher are separate external entities.

Forum does not create a generic:

```text
users
```

table.

Instead, entities use:

```text
actor_type
actor_id
```

Examples:

```text
actor_type = customer
actor_id   = 123
```

or:

```text
actor_type = teacher
actor_id   = 456
```

---

## 14.1 Where `actor_type` is used

### Category

```text
actor_type
actor_id
```

### Topic

```text
actor_type
actor_id
```

### Post

```text
actor_type
actor_id
```

### Reaction

```text
actor_type
actor_id
```

### Attachment

```text
actor_type
actor_id
```

---

## 14.2 Actor != Permission

`actor_type` only identifies who performed/created an action.

It does not grant permission.

Authorization must still evaluate:

```text
Actor
+
Role
+
Context
+
Permission
```

---

# 15. Authorization

Forum does not own:

```text
Enrollment
Teacher assignment
Customer membership
Program
Course
Lesson
```

The LMS/Authorization layer determines whether an Actor has access to the relevant context.

Example:

```text
Customer
   ↓
Enrollment
   ↓
Program
   ↓
Course
   ↓
Lesson
```

Forum consumes the authorization result.

---

# 16. Permission vs Topic Capability

A valid action requires both.

Example: Customer wants to reply.

```text
Actor has context access?
        ↓
Topic is active?
        ↓
allow_replies = true?
        ↓
Actor has reply permission?
        ↓
YES
```

Example: Teacher wants to accept an answer.

```text
Teacher has context permission?
        ↓
Topic is active?
        ↓
allow_accept_answers = true?
        ↓
Post is a valid answer?
        ↓
YES
```

---

# 17. Basic Permission Matrix

| Action | Teacher | Customer |
|---|---:|---:|
| View Category | Yes | Yes |
| View Topic | Yes | Yes |
| Create Topic | Yes | Yes |
| Create Post / Reply | Yes | Yes |
| Edit own Post | Yes | Yes |
| Delete own Post | Yes | Yes |
| Reaction | Yes | Yes |
| Attachment | Yes | Yes |
| Pin Topic | Yes | No |
| Lock Topic | Yes | No |
| Hide Post | Yes | No |
| Hide Topic | Yes | No |
| Delete other's Post | Yes | No |
| Accept Answer | Yes | No |

Actual authorization also depends on:

```text
context access
+
entity state
+
topic capability
```

---

# 18. Attachment

Forum does not own binary file storage.

The File Service owns the actual file.

Typical flow:

```text
Client
  ↓
File Service
  ↓
Upload
  ↓
file_id
  ↓
Forum Service
```

Forum stores file metadata needed by the Forum domain.

---

# 19. `forum_attachments`

## Final schema

```text
forum_attachments
--------------------------------
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

`file_id` is the identity of the external file.

`file_path`, `file_name`, `file_size`, and `mime_type` are metadata/snapshots required by the Forum.

---

# 20. `attachable_type`

The final values are:

```text
category
topic
post
```

These represent normal file attachments.

There is one important final decision:

> Category and Topic images are **NOT stored as rows in `forum_attachments`**.

Therefore there are **no**:

```text
category_image
topic_image
```

values in `forum_attachments.attachable_type`.

---

# 21. Category / Topic Image

Category and Topic directly store:

```text
image_id
image_path
```

This is sufficient.

## Category

```text
forum_categories
----------------
image_id
image_path
```

## Topic

```text
forum_topics
------------
image_id
image_path
```

The image does not need a corresponding record in:

```text
forum_attachments
```

This intentionally keeps image handling separate from normal Post/Topic/Category attachments.

---

## 21.1 Why this is simpler

For a Category:

```text
forum_categories
----------------
id
image_id
image_path
```

For a Topic:

```text
forum_topics
------------
id
image_id
image_path
```

There is no extra:

```text
forum_attachments
-----------------
attachable_type = category_image
attachable_id   = ...
```

or:

```text
attachable_type = topic_image
```

The Forum only needs the image reference and path directly on the owning entity.

---

# 22. Attachment vs Image

These are two different concepts.

### Normal attachment

Stored in:

```text
forum_attachments
```

Examples:

```text
PDF
DOCX
ZIP
image attached to a Post
other files
```

Relationship:

```text
attachable_type
attachable_id
```

### Category/Topic image

Stored directly on:

```text
forum_categories
forum_topics
```

with:

```text
image_id
image_path
```

No `forum_attachments` record is required.

---

# 23. Statistics

The final statistics hierarchy is:

```text
Category
└── topic_count

Topic
└── post_count

Post
├── reply_count
└── reaction stats
    ├── like
    ├── love
    ├── laugh
    └── ...
```

Definitions:

### `category.topic_count`

Number of visible Topics.

### `topic.post_count`

Number of visible Posts, including root Posts and Replies.

### `post.reply_count`

Number of visible direct child Posts.

### `reaction_stats.count`

Number of reaction events for a specific reaction type.

---

# 24. State and Statistics

State transitions must update denormalized statistics correctly.

Example:

```text
Topic
├── Post A
├── Post B
└── Post C
```

If:

```text
Post B → hidden
```

then:

```text
topic.post_count -= 1
```

For replies:

```text
Post A
├── Reply A1
└── Reply A2
```

If:

```text
Reply A1 → hidden
```

then:

```text
Post A.reply_count -= 1
Topic.post_count     -= 1
```

Existing reaction history does not need to be deleted merely because a Post becomes hidden.

---

# 25. Denormalized Counter Update

Counters should not be updated arbitrarily in Controllers.

Use domain/service operations.

Example:

```text
Create Topic
    ↓
Insert Topic
    ↓
Category.topic_count++

Create Root Post
    ↓
Insert Post
    ↓
Topic.post_count++

Create Reply
    ↓
Insert Post
    ↓
Topic.post_count++
Parent.reply_count++

Reaction
    ↓
Insert Reaction
    ↓
ReactionStats.count++
```

State transitions must use the same controlled logic:

```text
hide
unhide
delete
restore
```

This prevents counter drift.

---

# 26. Kafka / Event Architecture

Reaction is naturally suited to event publishing because every click is already an interaction event.

Recommended flow:

```text
DB Transaction
│
├── forum_post_reactions
├── forum_post_reaction_stats
└── outbox_events
        │
        ▼
Outbox Publisher
        │
        ▼
Kafka
        │
        ├── Analytics
        ├── Recommendation
        ├── Notification
        ├── Gamification
        └── Engagement
```

Forum remains the source of truth for Forum reactions.

---

# 27. Ownership Boundaries

## Forum Service owns

```text
Category
Topic
Post
Reaction
Reaction Stats
Attachment Metadata
```

## Forum Service does NOT own

```text
Tenant
Program
Course
Lesson

Customer
Teacher
Enrollment

Authentication
Identity

Binary File Storage
```

External services:

```text
Auth / SSO
    → identity

LMS / Authorization
    → context access / enrollment / permissions

File Service
    → binary files

Kafka
    → downstream events
```

---

# 28. Final Database Summary

## `forum_categories`

```text
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

## `forum_topics`

```text
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

## `forum_posts`

```text
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

## `forum_post_reactions`

```text
id
post_id

reaction_type

actor_type
actor_id

created_at
```

## `forum_post_reaction_stats`

```text
id
post_id
reaction_type
count

created_at
updated_at
```

Constraint:

```text
UNIQUE(post_id, reaction_type)
```

## `forum_attachments`

```text
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

---

# 29. Final ERD

```text
┌──────────────────────────┐
│    forum_categories      │
├──────────────────────────┤
│ id                       │
│ tenant_id                │
│ context_type             │
│ context_id               │
│ name                     │
│ description              │
│ image_id                 │
│ image_path               │
│ state                    │
│ topic_count              │
│ actor_type               │
│ actor_id                 │
│ created_at               │
│ updated_at               │
│ deleted_at               │
└────────────┬─────────────┘
             │
             │ 1:N
             ▼
┌──────────────────────────┐
│      forum_topics        │
├──────────────────────────┤
│ id                       │
│ category_id              │
│ title                    │
│ content                  │
│ image_id                 │
│ image_path               │
│ allow_replies            │
│ allow_reactions          │
│ allow_attachments        │
│ allow_accept_answers     │
│ state                    │
│ is_pinned                │
│ post_count               │
│ actor_type               │
│ actor_id                 │
│ created_at               │
│ updated_at               │
│ deleted_at               │
└────────────┬─────────────┘
             │
             │ 1:N
             ▼
┌──────────────────────────┐
│       forum_posts        │
├──────────────────────────┤
│ id                       │
│ topic_id                 │
│ parent_id                │◄──── self-reference
│ content                  │
│ state                    │
│ is_pinned                │
│ is_accepted_answer       │
│ reply_count              │
│ actor_type               │
│ actor_id                 │
│ created_at               │
│ updated_at               │
│ deleted_at               │
└───────┬─────────┬────────┘
        │         │
        │         │ 1:N
        │         ▼
        │   ┌──────────────────────────┐
        │   │ forum_post_reactions     │
        │   ├──────────────────────────┤
        │   │ id                       │
        │   │ post_id                  │
        │   │ reaction_type            │
        │   │ actor_type               │
        │   │ actor_id                 │
        │   │ created_at               │
        │   └──────────────────────────┘
        │
        │ 1:N
        ▼
┌───────────────────────────────┐
│ forum_post_reaction_stats     │
├───────────────────────────────┤
│ id                            │
│ post_id                       │
│ reaction_type                 │
│ count                         │
│ created_at                    │
│ updated_at                    │
└───────────────────────────────┘


Normal file attachments:

Category / Topic / Post
          │
          ▼
┌──────────────────────────┐
│   forum_attachments      │
├──────────────────────────┤
│ id                       │
│ attachable_type          │
│ attachable_id            │
│ file_id                  │
│ file_name                │
│ file_size                │
│ file_path                │
│ mime_type                │
│ actor_type               │
│ actor_id                 │
│ created_at               │
│ updated_at               │
└──────────────────────────┘


Category / Topic image:

forum_categories.image_id
forum_categories.image_path

forum_topics.image_id
forum_topics.image_path

(no forum_attachments row required)
```

---

# 30. Final Design Decision Checklist

| Decision | Final |
|---|---|
| Separate Discussion/Q&A/Announcement entities | No |
| `topic_type` | No |
| Topic capabilities | Yes |
| Separate Question entity | No |
| Separate Answer entity | No |
| Separate Reply entity | No |
| `post.parent_id` | Yes |
| Nested replies | Yes |
| Multiple accepted answers | Yes |
| `post.is_accepted_answer` | Yes |
| `topic.accepted_post_id` | No |
| Reaction as user state | No |
| Reaction as event | Yes |
| Reaction duplicate restriction | No |
| Reaction stats per type | Yes |
| Separate reaction stats table | Yes |
| Generic Actor table | No |
| `actor_type + actor_id` | Yes |
| Category/Topic state | active / locked / hidden |
| Post state | active / hidden |
| Topic pin | Yes |
| Post pin | Yes |
| Category topic counter | Yes |
| Topic post counter | Yes |
| Post reply counter | Yes |
| Generic attachment table | Yes |
| Category/Topic image via attachment polymorphism | No |
| Category/Topic `image_id + image_path` | Yes |
| `category_image` attachable type | No |
| `topic_image` attachable type | No |
| File binary owned by Forum | No |
| File binary owned by File Service | Yes |
| Authorization owned by Forum | No |
| Kafka events | Recommended |
| Outbox | Recommended |

---

# 31. Final Entity Model

The final Forum domain is intentionally flat:

```text
Category
   ↓
Topic
   ├── capabilities
   └── state
        ↓
      Post
      ├── parent_id
      ├── state
      ├── is_accepted_answer
      └── reply_count
           │
           ├── Reaction
           │      └── Reaction Stats
           │
           └── Attachment
```

This model avoids unnecessary entity proliferation while still supporting:

- Discussion
- Q&A
- Announcement
- Multiple replies
- Nested replies
- Multiple accepted answers
- Reactions
- Reaction statistics by type
- Pinning
- Moderation
- Attachments
- Category images
- Topic images
- Multi-tenant contexts
- Customer/Teacher actors
- External authorization
- External file storage
- Event-driven downstream processing

The final design therefore keeps the Forum Service focused on its actual responsibility: **managing discussion content and interaction data while delegating identity, authorization, LMS context, and file storage to their respective services.**
