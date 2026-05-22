# Open Decisions — Francais.vn

**Companion to:** `prd.md` v2.0, `prd-features.md` v2.0, `sprint-plan.md` v1.0
**Date:** 2026-05-14
**Owner of all decisions:** Founder (Phuon)

> Mỗi item có **Default assumption v2** — nếu founder không phản hồi trước sprint kickoff, dev sẽ implement theo default. Một số default đã được dùng để soạn FR; nếu founder đổi, có thể cần rework.

---

## Decision Table

| ID | Decision needed | Default v2 (proposed) | Impact if changed | Blocks |
|---|---|---|---|---|
| **OD-01** | **Guest Trial limits** — Bao nhiêu bài / exercise free trước login wall? | 1 lesson + 1 exercise. Soft banner sau exercise; hard modal khi cố vào bài thứ 2. | Cao — định hình funnel. Nếu lớn hơn (3+1) thì cần thêm logic select sample, conversion sẽ thấp hơn. | S2-02..06 |
| **OD-02** | **Dictation/Fill compare rules** — accent + punctuation behavior | Lowercase, normalize whitespace, **bỏ qua punctuation thừa**, **giữ accent**, apostrophe `'`↔`'` equivalent, hyphen giữ nguyên. | Trung — nếu strict accent (default ON) thì user mới sẽ thất vọng vì gõ thiếu `é`; nếu loose (accent-insensitive) thì giảm giá trị giáo dục. | S1-19, S2-14 |
| **OD-03** | **Shadowing trong MVP** — Sprint 2 Must-have hay Sprint 3 Could-have? | **Sprint 3, Could-have** (downgrade từ PRD v1). | Trung — nếu Must-have, cần đôn lên S2 và cắt feature khác. MediaRecorder + iOS Safari + permission flow tốn 13+ pts. | S2 scope, S3-01..04 |
| **OD-04** | **Tech stack** — Strapi (headless CMS) vs Supabase only (custom admin) vs Payload | **Strapi + Supabase** (Strapi cho admin UI, Supabase cho auth/storage/DB). | Rất cao — đổi sau S1-05 = rework toàn bộ admin + 2 sprint trễ. | S1-05, S1-07, S1-21 |
| **OD-05** | **"Hoàn thành bài"** — manual click hay auto sau Exercise pass? | Manual; sau Exercise ≥ 70% hiển thị suggestion modal "Đánh dấu bài hoàn thành?" | Thấp — purely UX. | S1-15, S1-18, S1-20 |
| **OD-06** | **Onboarding** — Mandatory hay optional skip? | Optional với "Bỏ qua sau". | Trung — mandatory tăng data quality (segmentation) nhưng tăng register friction. | S2-17 |
| **OD-07** | **Countdown UI** — Trên Dashboard? Settings? Cả 2? | Card riêng trên Dashboard, ẩn nếu chưa set. Setup từ Onboarding hoặc Settings. | Thấp — đặt sai chỗ thì user không thấy. | S2-18 |
| **OD-08** | **Audio storage & CDN** — Supabase Storage đủ MVP? hay cần Cloudflare R2? | **Supabase Storage** cho MVP (≤ 50 audio × 2MB = 100MB). CDN sau khi scale. | Thấp — migrate sau dễ. | S1-21, S2-12 |
| **OD-09** | **Quick Note rich-text scope** — Image upload? Code blocks? | Bold / Italic / Bullet / Ordered list. **KHÔNG image, KHÔNG embed, KHÔNG code blocks.** | Thấp — scope creep risk. | S2-09 |
| **OD-10** | **Guest progress migration on register** — Tự động hay user confirm? | Tự động merge, không hỏi. Hiển thị toast "Đã lưu tiến độ thử nghiệm của bạn". | Thấp. | S2-05 |
| **OD-11** | **Sprint length & team composition** | 2 tuần / sprint. Team: 1 FE, 1 BE, 1 Designer part-time. | Cao — định hình estimate. | All sprints |
| **OD-12** | **Levels nào available ở MVP launch?** | Khởi đầu + A1 only (active). A2–C2 hiển thị card "Sắp ra mắt" để tăng kỳ vọng. | Trung — nếu cần A2, content production team tăng tải. | S1-13 |
| **OD-13** | **Email verification** — Bắt buộc trước khi học? | Không bắt buộc MVP. Gửi mail welcome có link verify optional. | Thấp. | S1-01 |
| **OD-14** | **Password complexity** | ≥ 8 ký tự, có ≥ 1 chữ số. Không bắt buộc special char. | Thấp. | S1-01 |
| **OD-15** | **Rate limit policy** | 5 sai login trong 15 phút → khóa IP 15 phút. Không CAPTCHA MVP. | Thấp. | S1-01 |
| **OD-16** | **Domain & branding** — francais.vn confirmed? | Confirmed (theo project name). | — | Launch |
| **OD-17** | **Analytics tool** — PostHog / Plausible / GA4? | **PostHog** (event funnel + session replay free tier). | Thấp. | S1-22 |
| **OD-18** | **Notification strategy** — Email drip cho retention? | Không có ở MVP. Chỉ welcome + reset password. | Thấp — có thể thêm post-launch. | — |
| **OD-19** | **Pass threshold cho Exercise** — 70%? 60%? | 70% | Thấp — config-able trong CMS sau. | S1-18, OD-05 |
| **OD-20** | **Lesson "is_sample" cho Guest Trial** — Lesson nào? | 1 lesson "Bảng chữ cái + Phát âm cơ bản" (Khởi đầu, order=1). Admin set flag. | Thấp. | S2-03, S2-06, ADM-11 |
| **OD-21** | **Mobile app strategy** — PWA ở MVP? Native sau? | Responsive web only ở MVP. PWA install prompt ở Sprint 3 (Could). Native là Phase 2+. | Trung. | NFR-01 |
| **OD-22** | **Content authoring workflow** — Markdown trực tiếp vs WYSIWYG? | WYSIWYG (Strapi rich-text) cho Lesson body. Callout via custom block hoặc shortcut `> [!note]`. | Trung — markdown nhanh hơn, WYSIWYG dễ cho non-tech. | S1-07 |
| **OD-23** | **"Fail" handling** — Khi user score < 70% trong Exercise, lesson có completed không? | Không. Lesson vẫn `in_progress`. Retry exercise. | Trung — strict learning. | S1-18 |
| **OD-24** | **Topic groups** — Cố định 4 (Phát âm/Ngữ pháp/Từ vựng/Luyện tập) hay admin tạo thêm được? | Pre-seeded 4, admin có thể thêm/sửa. | Thấp. | S1-10 |
| **OD-25** | **Pricing model post-MVP** — Subscription vs One-time vs Freemium? | Out of scope MVP. Đề xuất Freemium (free A0/A1 + paid B1+). | — | Later |

---

## Decision Priority

### Need decision BEFORE Sprint 1 kickoff (blocking)
- OD-04 (Tech stack)
- OD-11 (Sprint length & team)
- OD-12 (Levels at launch)
- OD-14, OD-15 (Auth policy)
- OD-22 (Content authoring tool)
- OD-24 (Topic groups seeded)

### Need decision DURING Sprint 1 (before Sprint 2 kickoff)
- OD-01 (Guest trial limits)
- OD-02 (Dictation compare rules)
- OD-06 (Onboarding mandatory?)
- OD-09 (Quick Note scope)
- OD-20 (Sample lesson)

### Can defer to Sprint 2 review
- OD-03 (Shadowing in/out)
- OD-13 (Email verification)
- OD-17 (Analytics tool — but ideally early)
- OD-21 (PWA?)

### Nice-to-decide, low blocker
- OD-05, OD-07, OD-08, OD-10, OD-16, OD-18, OD-19, OD-23, OD-25

---

## How to use this doc
1. Founder reviews each row, marks `Decision:` column when picking final answer.
2. If founder agrees with **Default v2**, write `✅ accepted` and we proceed.
3. If founder picks differently, write the new decision; PM updates `prd.md` / `prd-features.md` accordingly and flags any sprint impact.

---

> **Status:** Open — review pending founder pass.
