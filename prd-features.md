# Feature Requirements — Francais.vn

**Companion to:** `prd.md` v2.0
**Version:** 2.0 (Consolidated / Implementation-Ready)
**Date:** 2026-05-14

> Mỗi feature gồm: **Mục tiêu · Functional Requirements · States/Rules · Edge cases · Dependencies · Acceptance Criteria**.
> Các ID requirement giữ nguyên prefix từ v1 để không vỡ traceability; bổ sung mới sẽ có suffix `-Nx`.

---

## Index

| § | Feature | Priority | Sprint |
|---|---|---|---|
| 5.0 | Guest Trial Flow | Must | 1 |
| 5.1 | Authentication & Registration | Must | 1 |
| 5.2 | Dashboard | Must | 1 |
| 5.3 | Learning Path | Must | 1 |
| 5.4 | Lesson Detail | Must | 1 |
| 5.5 | Exercise Basic | Must | 1 |
| 5.6 | Progress Tracking | Must | 1 |
| 5.7 | Quick Note + Notes Page | Must | 1–2 |
| 5.8 | Dictation Practice | Must | 2 |
| 5.9 | Shadowing Practice | Could | 3 |
| 5.10 | Bookmark / Saved Items | Should | 2 |
| 5.11 | Onboarding (Goal Selection) | Should | 2 |
| 5.12 | Countdown ngày thi | Should | 2 |
| 5.13 | Admin CMS — Content | Must | 1–2 |
| 5.14 | Admin CMS — Learning Path management | Must | 1 |
| 5.15 | Saved Mistakes | Could | 3 |
| 5.16 | Streak / Study Streak | Could | 3 |

---

## 5.0 Guest Trial Flow (NEW)

**Mục tiêu:** Cho user trải nghiệm giá trị trước khi bị chặn login. Tăng conversion guest → register.

### Functional Requirements
| ID | Requirement | Logic | Priority |
|---|---|---|---|
| GT-01 | Landing có CTA "Thử ngay miễn phí" | Click → `/guest/dashboard` không cần auth | Must |
| GT-02 | Guest Dashboard | Hiển thị 1 bài mẫu của lộ trình Khởi đầu + tóm tắt LP read-only | Must |
| GT-03 | Guest có thể vào Lesson mẫu | Lesson được mark `is_sample=true` trong CMS | Must |
| GT-04 | Guest làm Exercise mẫu | 2–3 câu, full feedback inline | Must |
| GT-05 | Quick Note in-memory cho guest | Cho thử nhưng KHÔNG persist (lost on refresh) — tooltip cảnh báo | Must |
| GT-06 | Sau Exercise mẫu → Soft register banner | Modal nhẹ: "Đăng ký để học toàn bộ lộ trình & lưu tiến độ" | Must |
| GT-07 | Banner có 2 CTA: Đăng ký / Tiếp tục dùng thử | Skip → vẫn được dùng nhưng có nudge banner persistent ở header | Must |
| GT-08 | Guest progress lưu trong `localStorage` | Khi register → merge vào account (lesson_id viewed, exercise score) | Must |
| GT-09 | Guest KHÔNG access Dictation / Shadowing / Notes Page | Click vào → redirect tới Sign Up | Must |
| GT-10 | Guest limit = 1 lesson + 1 exercise (default; xem OD-01) | Cố vào lesson thứ 2 → modal "Đăng ký để mở khóa" | Must |

### States
| State | Trigger | Behavior |
|---|---|---|
| `guest_fresh` | Vào landing lần đầu | Hiển thị "Thử ngay" |
| `guest_browsing` | Click "Thử ngay" | Render guest dashboard |
| `guest_tasted` | Hoàn thành 1 lesson + 1 exercise | Persistent register banner |
| `guest_at_limit` | Cố vào lesson thứ 2 | Hard modal register |
| `authenticated` | Sau register/login | Merge progress, drop localStorage |

### Edge Cases
- User clear localStorage giữa session → reset về `guest_fresh`.
- User register → login lần 2 trên thiết bị khác chưa có guest progress → không thấy bài đã thử (chấp nhận).
- Crawler (Google bot) vào guest URLs → render full SSR cho SEO, nhưng analytics filter UA.

### Dependencies
- Auth (5.1) cho register flow.
- Lesson Detail (5.4) + Exercise (5.5) phải support `guest_mode` flag (no progress persist).
- CMS phải có flag `is_sample` cho 1 lesson + 1 exercise.

### Acceptance Criteria
| ID | Criteria |
|---|---|
| GT-AC-01 | Click "Thử ngay" → vào Guest Dashboard, không bị redirect tới login. |
| GT-AC-02 | Guest hoàn thành lesson + exercise mẫu mà không cần đăng ký. |
| GT-AC-03 | Sau exercise mẫu, soft banner hiển thị 1 lần / session. |
| GT-AC-04 | Cố vào lesson thứ 2 → hard modal register. |
| GT-AC-05 | Sau register, progress của lesson mẫu được giữ. |
| GT-AC-06 | Quick Note in guest hiển thị tooltip "Đăng ký để lưu ghi chú". |

---

## 5.1 Authentication & Registration

**Mục tiêu:** User đăng ký, đăng nhập, quản lý session. Hỗ trợ email/password + Google OAuth.

### Functional Requirements
| ID | Requirement | Logic | Priority |
|---|---|---|---|
| AUTH-01 | Đăng ký email + password | Validate email format, password ≥ 8 ký tự, có ≥ 1 chữ số | Must |
| AUTH-02 | Đăng nhập email + password | Trả JWT 30d, redirect → Dashboard | Must |
| AUTH-03 | Google OAuth | Tạo account auto nếu chưa có; merge nếu email trùng | Should (Sprint 2) |
| AUTH-04 | Quên mật khẩu | Email reset link, expire 1h, one-time use | Must |
| AUTH-05 | Đăng xuất | Clear token (cookie + localStorage), redirect Landing | Must |
| AUTH-06 | Persistent session | JWT cookie 30d, refresh token rotation | Must |
| AUTH-07 | Verify email (optional) | Gửi mail xác nhận, KHÔNG block học | Could |
| AUTH-08 | Admin login tách biệt | `/admin/login`, chỉ user có `role=admin` mới vào được | Must |
| AUTH-09 | Rate-limit login | 5 sai trong 15 phút → khóa IP 15 phút | Must |
| AUTH-10 | Merge guest progress | On first register, đọc `localStorage.guestProgress` → upsert vào DB | Must |

### Edge Cases
| Case | Behavior |
|---|---|
| Email đã tồn tại | "Email đã được đăng ký. Đăng nhập?" với link |
| Sai pass 5 lần | "Quá nhiều lần thử, vui lòng đợi 15 phút." |
| Token hết hạn giữa session | Refresh auto; nếu refresh fail → redirect login với `?next=...` |
| OAuth email trùng password account | Merge, ưu tiên giữ profile cũ |
| Guest progress nhưng register thất bại | Giữ nguyên localStorage |

### Acceptance Criteria
| ID | Criteria |
|---|---|
| AUTH-AC-01 | User mới đăng ký thành công → Dashboard. |
| AUTH-AC-02 | Email trùng → error message rõ ràng, có link đăng nhập. |
| AUTH-AC-03 | Sai login → "Email hoặc mật khẩu không đúng" (không tiết lộ email tồn tại hay không). |
| AUTH-AC-04 | Reset link hoạt động trong 1h, one-time. |
| AUTH-AC-05 | Đăng xuất → token bị xóa, refresh page không còn vào được Dashboard. |
| AUTH-AC-06 | Admin user login qua user login bình thường vẫn vào được nhưng KHÔNG được vào `/admin`. |

---

## 5.2 Dashboard

**Mục tiêu:** Trung tâm điều hướng. User vào app phải biết ngay nên học gì tiếp.

### Functional Requirements
| ID | Requirement | Logic | Priority |
|---|---|---|---|
| DASH-01 | Card "Học tiếp" — bài đang dở | `PROGRESS.state=in_progress` mới nhất → show resume CTA | Must |
| DASH-02 | Gợi ý bài đầu tiên (user mới) | Nếu không có progress → lesson đầu tiên trong "Khởi đầu" (order asc) | Must |
| DASH-03 | Gợi ý bài tiếp theo (user cũ) | Bài đầu tiên có `state ≠ completed` theo order trong lộ trình | Must |
| DASH-04 | Tiến độ cơ bản | `completed_count / total_published_lessons` (toàn bộ) + per-level breakdown | Must |
| DASH-05 | Link Learning Path | Nav item / card | Must |
| DASH-06 | Link Notes Page | Nav item / card | Must |
| DASH-07 | Link Saved Items | Nav item / card | Should |
| DASH-08 | Hoạt động gần nhất | List 3–5 lesson/exercise/dictation gần nhất sorted desc | Should |
| DASH-09 | Countdown ngày thi | Card riêng nếu `USER_GOAL.exam_date` set, ẩn nếu không | Should |
| DASH-10 | Streak / XP | Hidden in MVP, infra-ready table | Later |

### State Rules — Card "Học tiếp"
| User state | Card content | CTA |
|---|---|---|
| Chưa học gì | "Bắt đầu lộ trình Khởi đầu" + lesson đầu tiên | "Bắt đầu học" |
| Có bài `in_progress` | "Bạn đang học: [tên bài]" | "Tiếp tục học" |
| Bài gần nhất `completed`, có bài kế | "Bài tiếp theo: [tên bài]" | "Học bài tiếp theo" |
| Đã completed toàn bộ published | "Bạn đã hoàn thành tất cả bài hiện có 🎉" | "Khám phá Learning Path" |

### Edge Cases
- Bài đang dở bị admin unpublish → bỏ qua, dùng rule kế tiếp.
- Có 2 bài `in_progress` (legacy data) → chọn `last_visited` mới nhất.
- Total published = 0 → "Chưa có bài học. Quay lại sau."

### Acceptance Criteria
| ID | Criteria |
|---|---|
| DASH-AC-01 | User mới thấy CTA "Bắt đầu học" với lesson đúng. |
| DASH-AC-02 | User cũ thấy đúng CTA: "Tiếp tục" hoặc "Học bài tiếp theo". |
| DASH-AC-03 | Click CTA mở đúng lesson. |
| DASH-AC-04 | Progress cập nhật ≤ 1s sau khi user hoàn thành bài. |
| DASH-AC-05 | Dashboard ≤ 6 widgets visible above the fold trên mobile. |
| DASH-AC-06 | Countdown chỉ hiển thị khi user đã set exam_date. |

---

## 5.3 Learning Path

**Mục tiêu:** User xem toàn bộ lộ trình Khởi đầu + A1 (có roadmap A2–C2 hiển thị "sắp ra mắt"), học tự do, không khóa bài.

### Functional Requirements
| ID | Requirement | Logic | Priority |
|---|---|---|---|
| LP-01 | Hiển thị lộ trình theo level | Tabs/sections: Khởi đầu, A1 (active), A2–C2 (locked teaser) | Must |
| LP-02 | Phân nhóm theo `topic_group` | Phát âm, Ngữ pháp, Từ vựng, Luyện tập | Must |
| LP-03 | Trạng thái bài | 3 trạng thái: Chưa học (xám) / Đang học (xanh) / Hoàn thành (check) | Must |
| LP-04 | Highlight bài nên học tiếp | Theo logic DASH-03 — viền hoặc badge "Tiếp theo" | Must |
| LP-05 | Không khóa bài | User click bất kỳ bài nào đều mở được | Must |
| LP-06 | % tiến độ theo level | `completed_in_level / published_in_level × 100` | Must |
| LP-07 | Click vào bài → Lesson Detail | — | Must |
| LP-08 | Badge bài có Exercise/Dictation | Icon `📝` / `🎧` nhỏ | Should |
| LP-09 | Filter theo trạng thái | Pill: Tất cả / Đang học / Hoàn thành / Chưa học | Should |
| LP-10 | Sticky progress bar trên top | % của level đang xem | Should |

### Edge Cases
- Lesson `unpublished` giữa lúc user đang xem → hiển thị "Bài học tạm ẩn" nếu đã có progress, ẩn hoàn toàn nếu chưa.
- Topic group rỗng → ẩn nguyên group.

### Acceptance Criteria
| ID | Criteria |
|---|---|
| LP-AC-01 | User thấy toàn bộ lộ trình Khởi đầu + A1. |
| LP-AC-02 | Bài completed có checkmark + đổi màu. |
| LP-AC-03 | Bài "nên học tiếp" được highlight rõ. |
| LP-AC-04 | Click bài bất kỳ đều mở được. |
| LP-AC-05 | % cập nhật real-time sau khi hoàn thành. |
| LP-AC-06 | Mobile: layout grid → list, không vỡ. |

---

## 5.4 Lesson Detail

**Mục tiêu:** Trải nghiệm học chính. Mobile-first, dễ đọc, có flow tự nhiên sang Exercise.

### Functional Requirements
| ID | Requirement | Logic | Priority |
|---|---|---|---|
| LES-01 | Render nội dung markdown bài học | Hỗ trợ headings, bold/italic, callout, code, table | Must |
| LES-02 | Block "Ghi chú quan trọng" | Callout style xanh (`> [!note]`) | Must |
| LES-03 | Block "Lỗi thường gặp" | Callout style đỏ (`> [!warning]`) | Must |
| LES-04 | CTA "Làm bài tập" | Link đến Exercise của lesson hiện tại. Disabled nếu không có exercise | Must |
| LES-05 | CTA "Hoàn thành bài" | Click → set `progress.state=completed`, suggest bài kế tiếp | Must |
| LES-06 | CTA "Học bài tiếp theo" | Xuất hiện sau khi `completed`, link đến lesson kế trong order | Must |
| LES-07 | Quick Note floating button | Bottom-right desktop, bottom-left mobile (tránh đè CTA) | Must |
| LES-08 | Audio inline cho ví dụ | `<audio>` player nhỏ, max 3 audio / lesson | Should |
| LES-09 | Responsive | ≥ 320px width, font ≥ 16px body, line-height 1.6 | Must |
| LES-10 | Breadcrumb | `LP > Topic > Lesson` clickable | Should |
| LES-11 | Auto-mark `in_progress` | Khi user scroll ≥ 30% hoặc dwell ≥ 30s | Must |
| LES-12 | Resume scroll position | Lưu scroll % vào localStorage per lesson, restore khi quay lại | Should |

### State Transitions
| From | Event | To |
|---|---|---|
| `not_started` | Mở Lesson Detail + dwell 30s hoặc scroll 30% | `in_progress` |
| `in_progress` | Click "Hoàn thành bài" | `completed` |
| `in_progress` | Exercise pass ≥ 70% (OD-05) | Suggest `completed` (manual confirm) |
| `completed` | Click "Học lại" (optional) | giữ `completed` nhưng update `last_visited` |

### Edge Cases
- Lesson không có Exercise → ẩn CTA "Làm bài tập", chỉ có "Hoàn thành bài".
- Lesson là cuối cùng trong level → CTA "Bài tiếp theo" thành "Hoàn thành level".
- Audio fail load → fallback text "Không thể tải audio", lesson vẫn đọc được.

### Acceptance Criteria
| ID | Criteria |
|---|---|
| LES-AC-01 | Nội dung render đúng markdown + callouts. |
| LES-AC-02 | CTA "Làm bài tập" mở đúng Exercise. |
| LES-AC-03 | "Hoàn thành bài" → state = completed, Dashboard cập nhật. |
| LES-AC-04 | Quick Note button hiển thị và mở panel đúng. |
| LES-AC-05 | Render tốt trên 375px width (iPhone SE). |
| LES-AC-06 | Auto-mark `in_progress` sau 30s. |

---

## 5.5 Exercise Basic

**Mục tiêu:** Bài tập sau lesson. 3 loại câu hỏi, feedback ngay, retry được.

### Functional Requirements
| ID | Requirement | Logic | Priority |
|---|---|---|---|
| EX-01 | Multiple Choice | 1 đáp án đúng trong 3–4 options. Single-select. | Must |
| EX-02 | True/False | 2 options. | Must |
| EX-03 | Fill in the blank | 1–3 ô input, exact match (case-insensitive, trim, normalize whitespace, **giữ accent**) | Must |
| EX-04 | Feedback ngay sau mỗi câu | Đúng → xanh + "Chính xác". Sai → đỏ + đáp án đúng + giải thích | Must |
| EX-05 | Kết quả cuối bài | Score / total, %, thời gian, list câu sai | Must |
| EX-06 | Gắn exercise với lesson | `exercise.lesson_id` required | Must |
| EX-07 | Retry exercise | Tạo `EXERCISE_ATTEMPT` mới; latest attempt là "current score" | Must |
| EX-08 | Quick Note floating | Hiển thị trong toàn bộ exercise (trừ Mock Test) | Must |
| EX-09 | Progress bar | `Q n / total` ở top | Should |
| EX-10 | Hỗ trợ "Câu trước" / "Câu sau" | Nav cho phép quay lại sửa trước khi submit cuối | Should |
| EX-11 | Skip câu | Cho phép skip, đánh dấu chưa trả lời ở result | Could |

### Fill-in-the-blank Compare Rules (OD-02)
- Lowercase cả 2 vế.
- Trim + collapse whitespace.
- **Bỏ qua dấu câu thừa** (`,.?!;:` ở đầu/cuối).
- **GIỮ accent** (`é ≠ e`, `à ≠ a`) — đây là tiếng Pháp.
- Multiple acceptable answers: admin nhập dạng `je suis | je m'appelle` separator `|`.

### State Rules
| State | Trigger |
|---|---|
| `not_started` | Default |
| `in_progress` | User trả lời câu đầu |
| `submitted` | User submit toàn bài |
| `passed` | score ≥ 70% (OD-05) |
| `failed` | score < 70% |

### Edge Cases
- User refresh giữa bài → lưu draft answers trong sessionStorage, restore.
- User submit không trả lời câu nào → "Bạn chưa trả lời câu nào, chắc chắn submit?"
- Exercise có 0 câu hỏi (admin error) → "Bài tập đang được cập nhật."

### Acceptance Criteria
| ID | Criteria |
|---|---|
| EX-AC-01 | Cả 3 loại câu hoạt động đúng. |
| EX-AC-02 | Feedback hiển thị ngay sau submit câu. |
| EX-AC-03 | Kết quả cuối bài đúng số liệu. |
| EX-AC-04 | Retry → kết quả mới được lưu, attempts history giữ. |
| EX-AC-05 | Fill blank giữ accent (`é` ≠ `e`). |
| EX-AC-06 | Quick Note button hiển thị suốt exercise. |

---

## 5.6 Progress Tracking

**Mục tiêu:** Lưu & expose tiến độ để Dashboard + LP hoạt động.

### Functional Requirements
| ID | Requirement | Logic | Priority |
|---|---|---|---|
| PT-01 | Lưu lesson state | `not_started | in_progress | completed` per (user, lesson) | Must |
| PT-02 | Lưu exercise attempts | Score, total, duration, submitted_at, per-question answers | Must |
| PT-03 | % theo level | `completed_count / published_count_in_level` | Must |
| PT-04 | Xác định bài tiếp theo | Bài first có `state ≠ completed` theo order trong active level | Must |
| PT-05 | Lưu dictation attempts | accuracy %, user_input, submitted_at | Must |
| PT-06 | Lịch sử học | List 30 events gần nhất sorted desc | Should |
| PT-07 | Recompute on unpublish | Nếu lesson unpublished → loại khỏi denominator | Must |

### Events to track
| Event | When | Payload |
|---|---|---|
| `lesson.started` | First visit | user_id, lesson_id |
| `lesson.in_progress` | 30s dwell / 30% scroll | + duration |
| `lesson.completed` | Click "Hoàn thành" | + completed_at |
| `exercise.submitted` | End of exercise | + score |
| `dictation.submitted` | End of dictation | + accuracy |
| `note.created` | New note | — |
| `guest.register` | First auth after guest | + guest_progress merged |

### Acceptance Criteria
| ID | Criteria |
|---|---|
| PT-AC-01 | Hoàn thành bài → state cập nhật Dashboard + LP trong ≤ 1s. |
| PT-AC-02 | % tính đúng và nhất quán giữa các screens. |
| PT-AC-03 | Exercise result xem lại được từ Lesson Detail. |
| PT-AC-04 | Admin unpublish → denominator giảm, % tăng tương ứng. |

---

## 5.7 Quick Note + Notes Page

**Mục tiêu:** UX khác biệt. Ghi chú trong khi học, không phải mở tab khác.

### Functional Requirements — Floating Quick Note
| ID | Requirement | Logic | Priority |
|---|---|---|---|
| QN-01 | Floating button | Bottom-right desktop, bottom-left mobile, z-index trên content, ẩn dưới modal | Must |
| QN-02 | Slide-over panel | Desktop: right slide; Mobile: bottom sheet 80% height | Must |
| QN-03 | Tạo note mới | Default title = tên lesson hiện tại, user editable | Must |
| QN-04 | Chọn note cũ | Dropdown danh sách note (sort by updated_at desc) | Must |
| QN-05 | CRUD note | Create, edit, delete (confirm). Read = list view. | Must |
| QN-06 | Link với lesson | `note.lesson_id` set khi tạo trong lesson; user có thể đổi/clear | Must |
| QN-07 | Notes Page riêng | `/notes` — list all notes, search, filter by lesson | Must |
| QN-08 | Ẩn trong Mock Test | Button không render, click vùng tương đương → toast | Must |
| QN-09 | Rich text cơ bản | bold, italic, bullet list, ordered list (no image/embed) — OD-09 | Should |
| QN-10 | Auto-save | Debounced 1s sau khi ngừng gõ | Must |
| QN-11 | Sync với Notes Page | Lưu xong → notes page reload list (next visit) | Must |
| QN-12 | Note picker on first open | Lần đầu mở panel trong session → hỏi "Tạo mới / Chọn note" | Should |

### Notes Page
| ID | Requirement | Logic | Priority |
|---|---|---|---|
| NP-01 | List tất cả note | Pagination 20/page hoặc infinite scroll | Must |
| NP-02 | Search by title + body | Full-text simple (LIKE) | Must |
| NP-03 | Filter by lesson | Dropdown lesson list | Should |
| NP-04 | Sort | updated_at desc default | Must |
| NP-05 | Click note → mở viewer/editor | Full-page edit | Must |
| NP-06 | Export note | Markdown copy to clipboard | Could |

### Edge Cases
- Lesson bị unpublish → note vẫn giữ, chỉ hiển thị "Bài học không khả dụng" trong note view.
- User xóa note đang mở trên Lesson → panel reset về picker.
- Network fail khi auto-save → giữ trong localStorage, retry on reconnect.
- Conflict edit (2 tabs) → last-write-wins với warning toast.

### Acceptance Criteria
| ID | Criteria |
|---|---|
| QN-AC-01 | Floating button hiển thị trong Lesson/Exercise/Dictation/Shadowing, ẩn trong Mock Test + Admin. |
| QN-AC-02 | Tạo note mới → xuất hiện trong Notes Page. |
| QN-AC-03 | Chọn note cũ → load nội dung đúng. |
| QN-AC-04 | Delete note → biến mất khỏi list. |
| QN-AC-05 | Auto-save 1s, mất kết nối vẫn không mất nội dung (local fallback). |
| QN-AC-06 | Mock Test → click toast "Không có ghi chú trong chế độ thi". |

---

## 5.8 Dictation Practice

**Mục tiêu:** Luyện nghe chủ động — không chỉ play audio.

### Functional Requirements
| ID | Requirement | Logic | Priority |
|---|---|---|---|
| DIC-01 | Phát audio ngắn | Chunk 5–15s (cấu hình per dictation) | Must |
| DIC-02 | Text input area | Textarea, autocomplete OFF, spellcheck OFF | Must |
| DIC-03 | Compare với transcript | Theo rules trong "Compare Rules" dưới | Must |
| DIC-04 | Highlight lỗi sai | Token-level diff: missing (gạch đỏ), wrong (đỏ underline), correct (xanh) | Must |
| DIC-05 | Hiển thị transcript chuẩn | Sau khi submit | Must |
| DIC-06 | Replay audio | Unlimited; counter hiển thị "Đã nghe lại N lần" | Must |
| DIC-07 | Kết quả | accuracy = `matched_tokens / total_tokens`, số từ đúng/sai, thời gian | Must |
| DIC-08 | Gắn lesson optional | `dictation.lesson_id` nullable | Should |
| DIC-09 | Giải thích từ vựng | Tooltip per từ khó (admin nhập) | Should |
| DIC-10 | Slow playback 0.75x | Audio control speed | Should |
| DIC-11 | Per-chunk navigation | "Câu trước / Câu sau" nếu nhiều chunk | Could |

### Compare Rules (OD-02 default)
1. Normalize: lowercase, NFC unicode, collapse whitespace, strip leading/trailing `,.?!;:`.
2. **Giữ accent** (é è ê à â ç …).
3. Token-level diff (split by whitespace).
4. Apostrophe variants `’` ↔ `'` → coi như giống nhau.
5. Hyphens giữ nguyên (`peut-être` ≠ `peut etre`).

### Edge Cases
- Audio fail load → "Không tải được audio. Thử lại?" + retry button.
- User submit empty → "Bạn chưa nhập gì. Submit luôn?"
- Transcript có dấu nháy đặc biệt → normalize.
- Microphone không liên quan ở Dictation (chỉ Shadowing).

### Acceptance Criteria
| ID | Criteria |
|---|---|
| DIC-AC-01 | Audio phát đúng, replay unlimited. |
| DIC-AC-02 | Compare hoạt động theo rules (`é` ≠ `e`, bỏ qua `,.`). |
| DIC-AC-03 | Lỗi sai highlight rõ ở token level. |
| DIC-AC-04 | Accuracy % đúng công thức. |
| DIC-AC-05 | Submit lưu attempt vào DB; xem lại được. |

---

## 5.9 Shadowing Practice (Could-have, Sprint 3)

**Mục tiêu:** Luyện nói/lặp lại. MVP không AI — user tự so sánh.

### Functional Requirements
| ID | Requirement | Logic | Priority |
|---|---|---|---|
| SH-01 | Phát audio mẫu | Audio file + transcript hiển thị kèm | Could |
| SH-02 | Hiển thị transcript | Đồng bộ với audio (highlight word đang đọc nếu có timestamps) | Could |
| SH-03 | Ghi âm user | MediaRecorder API, permission flow | Could |
| SH-04 | Nghe lại bản ghi | Audio player của recording | Could |
| SH-05 | Lưu trạng thái | `shadowing.state=completed` khi user submit "Đã luyện xong" | Could |
| SH-06 | CTA "Luyện lại" | Reset state, không xóa lịch sử | Could |
| SH-07 | Recording NOT uploaded | Giữ in-memory / blob; không lưu server (privacy) | Could |

### Edge Cases
- Browser không support MediaRecorder → "Trình duyệt không hỗ trợ ghi âm. Dùng Chrome/Safari mới nhất."
- User từ chối permission mic → fallback chỉ nghe + đọc theo, không record.
- iOS Safari yêu cầu HTTPS — OK vì NFR-05.

### Acceptance Criteria
| ID | Criteria |
|---|---|
| SH-AC-01 | Audio mẫu phát, transcript hiển thị. |
| SH-AC-02 | Recording hoạt động trên desktop Chrome + iOS Safari. |
| SH-AC-03 | User nghe lại được. |
| SH-AC-04 | "Đã luyện xong" cập nhật progress. |

---

## 5.10 Bookmark / Saved Items

| ID | Requirement | Logic | Priority |
|---|---|---|---|
| BM-01 | Bookmark lesson | Icon star trong Lesson Detail | Should |
| BM-02 | Bookmark question | Trong Exercise Result, đánh dấu câu cần ôn | Should |
| BM-03 | Saved Items page | List bookmark, sort + filter | Should |
| BM-04 | Link từ Dashboard | Shortcut card | Should |
| BM-05 | Unbookmark | Click lại icon | Should |

### Acceptance Criteria
- BM-AC-01: Bookmark → xuất hiện Saved Items.
- BM-AC-02: Unbookmark → biến mất.
- BM-AC-03: Bookmark câu hỏi → click → mở lại exercise scroll tới câu đó.

---

## 5.11 Onboarding — Goal Selection (Should-have, Sprint 2)

| ID | Requirement | Logic | Priority |
|---|---|---|---|
| ONB-01 | Sau register → modal "Mục tiêu của bạn?" | 3 options + "Bỏ qua" (OD-06) | Should |
| ONB-02 | Options: "Từ đầu" / "Ôn A1" / "Luyện thi" | Lưu vào `USER_GOAL.goal` | Should |
| ONB-03 | Nếu chọn "Luyện thi" → hỏi ngày thi | Date picker, save `USER_GOAL.exam_date` | Should |
| ONB-04 | Cho phép sửa goal sau | Settings page | Should |
| ONB-05 | Recommendation dựa trên goal | "Từ đầu" → ưu tiên Phát âm; "Ôn A1" → Ngữ pháp; "Thi" → countdown card | Should |

---

## 5.12 Countdown ngày thi (Should-have, Sprint 2)

| ID | Requirement | Logic | Priority |
|---|---|---|---|
| CD-01 | Set ngày thi | Trong Onboarding hoặc Settings | Should |
| CD-02 | Card trên Dashboard | "Còn N ngày tới [exam_date]" + progress bar so với hôm bắt đầu | Should |
| CD-03 | Ẩn nếu không set | Card không render | Should |
| CD-04 | Khi hết hạn | "Ngày thi đã qua. Cập nhật mục tiêu mới?" | Should |

---

## 5.13 Admin CMS — Content

**Mục tiêu:** 1 admin tạo & vận hành toàn bộ content không cần dev.

### Functional Requirements
| ID | Requirement | Logic | Priority |
|---|---|---|---|
| ADM-01 | Admin login riêng | `/admin/login`, role check | Must |
| ADM-02 | CRUD Lesson | Markdown editor + preview, callout shortcuts | Must |
| ADM-03 | CRUD Exercise | UI tạo MC/T-F/Fill, gắn lesson | Must |
| ADM-04 | CRUD Dictation | Upload audio + transcript + chunk timing optional | Must (Sprint 2) |
| ADM-05 | CRUD Shadowing | Audio + transcript | Could (Sprint 3) |
| ADM-06 | Publish state machine | Draft → Published → Unpublished | Must |
| ADM-07 | Upload audio | Format MP3, ≤ 2MB, ≤ 30s. Validate trước upload. | Must |
| ADM-08 | Preview giống learner | Render đúng như Lesson Detail / Exercise của user | Should |
| ADM-09 | Bulk publish/unpublish | Checkbox + action | Could |
| ADM-10 | Admin Dashboard | KPI: số bài published, user count, hoạt động gần | Should |
| ADM-11 | Mark `is_sample` | Toggle để chọn bài mẫu cho Guest Trial | Must |
| ADM-12 | Acceptable answers cho Fill | Field "Alternate answers" với `|` separator | Must |

### Edge Cases
- Admin xóa lesson có user progress → soft delete (flag), giữ progress.
- Upload audio fail → giữ form data, allow retry.
- Sửa published lesson → warning "Nội dung đang live, thay đổi sẽ ảnh hưởng người học".
- 2 admin login đồng thời edit cùng lesson → last-write-wins + warning.

### Acceptance Criteria
| ID | Criteria |
|---|---|
| ADM-AC-01 | Admin tạo lesson → publish → xuất hiện Learning Path. |
| ADM-AC-02 | Exercise gắn đúng lesson. |
| ADM-AC-03 | Unpublish → ẩn khỏi learner, data giữ. |
| ADM-AC-04 | Upload audio MP3 < 2MB thành công và phát được. |
| ADM-AC-05 | Preview render giống hệt learner view. |

---

## 5.14 Admin CMS — Learning Path Management

| ID | Requirement | Logic | Priority |
|---|---|---|---|
| LPM-01 | CRUD Level | Pre-seeded "Khởi đầu", "A1"; A2–C2 admin tạo sau | Must |
| LPM-02 | CRUD Topic Group | Phát âm, Ngữ pháp, Từ vựng, Luyện tập (pre-seeded) | Must |
| LPM-03 | Reorder lesson | Drag-drop hoặc input `order` integer | Must |
| LPM-04 | Gán lesson vào topic group | Dropdown | Must |
| LPM-05 | Set "is_sample" | Toggle — chỉ 1 lesson được = true / level | Must |

---

## 5.15 Saved Mistakes (Could-have, Sprint 3)

| ID | Requirement | Logic |
|---|---|---|
| SM-01 | Tự động lưu câu sai | Sau mỗi exercise/dictation, câu sai add vào `saved_mistakes` |
| SM-02 | Saved Mistakes page | List câu sai, filter by lesson |
| SM-03 | "Luyện lại" | Tạo mini-exercise từ câu sai |
| SM-04 | Remove từ list | Khi user trả lời đúng 2 lần liên tiếp |

---

## 5.16 Streak / Study Streak (Could-have, Sprint 3)

| ID | Requirement | Logic |
|---|---|---|
| ST-01 | Đếm ngày học liên tục | Hôm nay có ít nhất 1 activity → +1; bỏ 1 ngày → reset |
| ST-02 | Hiển thị streak | Badge trên Dashboard |
| ST-03 | Grace period | Cho phép "freeze" 1 ngày/tuần (optional) |

---

## 6. Non-Functional Requirements
Đã consolidated trong `prd.md` §8. Không lặp.

---

## 7. Cross-Feature Rules

### 7.1 Quick Note Visibility Matrix
Xem `prd.md` §4.2.

### 7.2 Progress Update Triggers
| Source | Effect |
|---|---|
| Lesson Detail dwell 30s / scroll 30% | `lesson.state = in_progress` |
| Click "Hoàn thành bài" | `lesson.state = completed` |
| Exercise submit ≥ 70% | Suggest completed (manual confirm) |
| Dictation submit | `dictation.state = completed` regardless score |
| Admin unpublish | Loại khỏi denominator, giữ user progress trong DB |

### 7.3 Guest → Authenticated Migration
| Item | Migrate? |
|---|---|
| Guest viewed lessons (id list) | ✅ → `PROGRESS.state=in_progress` |
| Guest exercise score | ✅ → `EXERCISE_ATTEMPT` |
| Guest Quick Note (in-memory) | ❌ — lost, user được cảnh báo |
| Guest bookmarks | N/A (guest không có bookmark) |

---

## 8. Sprint Backlog Summary
Xem `sprint-plan.md` cho chi tiết epic → story → estimate.

---

## 9. Glossary
| Term | Meaning |
|---|---|
| Learner | User role = `learner` |
| Guest | User chưa đăng ký, dùng trial |
| Lesson | Đơn vị bài học (markdown content) |
| Exercise | Bộ câu hỏi MC/T-F/Fill gắn với 1 lesson |
| Dictation | Bài nghe chép chính tả |
| Shadowing | Bài luyện nói lặp lại |
| Quick Note | Ghi chú nhanh slide-over |
| Topic Group | Phát âm / Ngữ pháp / Từ vựng / Luyện tập |
| Level | Khởi đầu / A1 / A2 / B1 / B2 / C1 / C2 |
| Mock Test | Đề thi giả lập có timer (Sprint 4+) |

---

> **Status:** Draft v2.0 — Implementation-ready pending §11 Open Decisions in `prd.md`.
