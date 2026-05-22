# Sprint Plan — Francais.vn MVP

**Companion to:** `prd.md` v2.0 · `prd-features.md` v2.0
**Date:** 2026-05-14
**Sprint length:** 2 weeks each (assumed — confirm in OD-11)
**Team assumption:** 1 FE, 1 BE/CMS, 1 Designer (part-time), 1 PM/Founder

---

## 0. Sprint Overview

| Sprint | Theme | Outcome | Demo-able |
|---|---|---|---|
| 1 | **Core Learning Loop** | User có thể: register → vào Dashboard → vào Lesson → làm Exercise → thấy progress | ✅ Internal alpha |
| 2 | **Notes + Dictation + Funnel** | Quick Note + Notes Page + Dictation + Guest Trial + Google OAuth + Bookmark + Onboarding | ✅ Closed beta |
| 3 | **Retention + Skill Practice** | Shadowing + Saved Mistakes + Streak + Polish | ✅ Public MVP launch |
| 4+ | **Exam Prep** | Mock Test + Timer + Placement Test | Phase 2 |

---

## 1. Sprint 1 — Core Learning Loop

**Goal:** End-to-end learner happy path bằng email auth, không cần guest/OAuth/note/dictation. Admin có thể tạo lesson + exercise.

| # | Epic | Story | Feature IDs | Est. (pts) | Owner |
|---|---|---|---|---|---|
| S1-01 | Auth | Đăng ký email + password | AUTH-01, AUTH-02, AUTH-06, AUTH-09 | 5 | BE |
| S1-02 | Auth | Reset password flow | AUTH-04 | 3 | BE |
| S1-03 | Auth | Đăng xuất + session expiry | AUTH-05 | 2 | FE |
| S1-04 | Auth | Admin login tách biệt | AUTH-08 | 3 | BE |
| S1-05 | Data | Schema: USER, LESSON, EXERCISE, QUESTION, PROGRESS, EXERCISE_ATTEMPT | — | 5 | BE |
| S1-06 | Data | Seed dữ liệu: 5 lessons + 5 exercises để dev | — | 2 | BE |
| S1-07 | Admin | CRUD Lesson (markdown editor + preview) | ADM-02 | 8 | BE+FE |
| S1-08 | Admin | CRUD Exercise: MC + T/F + Fill | ADM-03 | 8 | BE+FE |
| S1-09 | Admin | Publish/Unpublish state machine | ADM-06 | 3 | BE |
| S1-10 | Admin | LP management: Level, Topic Group, reorder | LPM-01..05 | 5 | BE+FE |
| S1-11 | Dashboard | Card "Học tiếp" 4 states (new/in-progress/next/done) | DASH-01, 02, 03 | 5 | FE |
| S1-12 | Dashboard | Tiến độ cơ bản + link LP + link Notes (stub) | DASH-04, 05, 06 | 3 | FE |
| S1-13 | LP | Hiển thị Khởi đầu + A1 grouped, trạng thái 3 states | LP-01, 02, 03, 06, 07 | 8 | FE |
| S1-14 | LP | Highlight bài "nên học tiếp", không khóa bài | LP-04, 05 | 3 | FE |
| S1-15 | Lesson | Render markdown + callouts + CTAs | LES-01, 02, 03, 04, 05, 06 | 5 | FE |
| S1-16 | Lesson | Auto-mark `in_progress`, breadcrumb, mobile responsive | LES-09, 10, 11 | 3 | FE |
| S1-17 | Exercise | MC + T/F + Fill rendering | EX-01, 02, 03 | 5 | FE |
| S1-18 | Exercise | Feedback ngay + Kết quả cuối bài + Retry | EX-04, 05, 07 | 5 | FE+BE |
| S1-19 | Exercise | Fill compare rules (case-insensitive, accent) | EX-03 (OD-02) | 3 | BE |
| S1-20 | Progress | Lưu lesson state + exercise attempts + % per level | PT-01..04 | 5 | BE |
| S1-21 | Infra | Hosting setup: Vercel + Supabase + Strapi | NFR-05, 06 | 5 | BE |
| S1-22 | Infra | Sentry + analytics events scaffolding | NFR-10 | 3 | BE |
| S1-23 | QA | E2E happy path (register → lesson → exercise → complete) | — | 3 | QA |

**Total Sprint 1: ~100 pts**

**Definition of Done — Sprint 1:**
- [x] User register → Dashboard → vào Lesson → làm Exercise → thấy progress cập nhật
- [x] Admin tạo & publish 5 lesson + 5 exercise hoạt động trên prod env
- [x] Mobile (iPhone 12 / Android Chrome) render đúng
- [x] LCP < 3s trên Lighthouse Mobile

---

## 2. Sprint 2 — Notes + Dictation + Funnel

**Goal:** Bật Quick Note (USP), Dictation (must-have feature), Guest Trial (funnel), Google OAuth, Onboarding, Bookmark.

| # | Epic | Story | Feature IDs | Est. | Owner |
|---|---|---|---|---|---|
| S2-01 | Auth | Google OAuth | AUTH-03 | 5 | BE |
| S2-02 | Guest Trial | Landing CTA + Guest Dashboard | GT-01, 02 | 5 | FE |
| S2-03 | Guest Trial | Guest lesson + exercise mẫu (read-only mode) | GT-03, 04 | 5 | FE |
| S2-04 | Guest Trial | Soft register banner + hard modal at limit | GT-06, 07, 10 | 5 | FE |
| S2-05 | Guest Trial | localStorage progress + merge on register | GT-08, AUTH-10 | 5 | BE+FE |
| S2-06 | Guest Trial | CMS flag `is_sample` for 1 lesson + 1 exercise | ADM-11 | 2 | BE |
| S2-07 | Quick Note | Floating button + slide-over panel (desktop + mobile) | QN-01, 02 | 5 | FE |
| S2-08 | Quick Note | CRUD note + tạo từ lesson + chọn note cũ | QN-03..06 | 5 | FE+BE |
| S2-09 | Quick Note | Auto-save + rich text (bold/italic/bullet) | QN-09, 10 | 5 | FE |
| S2-10 | Quick Note | Hide trên Mock Test (placeholder for future) + toast | QN-08 | 2 | FE |
| S2-11 | Notes Page | List + search + filter by lesson | NP-01..05 | 5 | FE+BE |
| S2-12 | Dictation | Schema + admin CRUD + audio upload | ADM-04, ADM-07 | 8 | BE+FE |
| S2-13 | Dictation | Learner UI: audio player + textarea + replay | DIC-01, 02, 06 | 5 | FE |
| S2-14 | Dictation | Token diff compare + highlight + result | DIC-03, 04, 05, 07 | 8 | BE+FE |
| S2-15 | Dictation | Save attempt + progress update | DIC-attempt, PT-05 | 3 | BE |
| S2-16 | Bookmark | Bookmark lesson + question + Saved Items page | BM-01..05 | 5 | FE+BE |
| S2-17 | Onboarding | Goal modal sau register + USER_GOAL schema | ONB-01..04 | 5 | FE+BE |
| S2-18 | Countdown | Set exam_date + Dashboard card | CD-01..04 | 3 | FE+BE |
| S2-19 | Dashboard | Hoạt động gần nhất + link Saved Items | DASH-07, 08 | 3 | FE |
| S2-20 | QA | E2E: Guest → Register → Note → Dictation | — | 3 | QA |

**Total Sprint 2: ~92 pts**

**Definition of Done — Sprint 2:**
- [x] Guest có thể trial → register → progress migrate đúng
- [x] User tạo Quick Note trong lesson + xem trong Notes Page
- [x] User làm Dictation, accuracy hiển thị đúng theo compare rules
- [x] Google OAuth hoạt động trên prod
- [x] ≥ 5 dictation bài published

---

## 3. Sprint 3 — Retention + Skill Practice + Polish

**Goal:** Bật retention features (Streak, Saved Mistakes), Shadowing, polish UX cho public launch.

| # | Epic | Story | Feature IDs | Est. | Owner |
|---|---|---|---|---|---|
| S3-01 | Shadowing | Schema + admin CRUD | ADM-05 | 5 | BE |
| S3-02 | Shadowing | Audio playback + transcript display | SH-01, 02 | 3 | FE |
| S3-03 | Shadowing | MediaRecorder integration | SH-03, 04 | 5 | FE |
| S3-04 | Shadowing | Progress state + Luyện lại | SH-05, 06 | 3 | FE+BE |
| S3-05 | Saved Mistakes | Auto-collect wrong answers | SM-01 | 5 | BE |
| S3-06 | Saved Mistakes | Saved Mistakes page + filter | SM-02 | 3 | FE |
| S3-07 | Saved Mistakes | "Luyện lại" mini-exercise | SM-03, 04 | 5 | FE+BE |
| S3-08 | Streak | Đếm ngày học liên tục | ST-01 | 3 | BE |
| S3-09 | Streak | Dashboard badge | ST-02 | 2 | FE |
| S3-10 | Polish | Performance audit: LCP, TTI optimize | NFR-02 | 5 | FE |
| S3-11 | Polish | A11y audit: contrast, keyboard nav | NFR-04 | 3 | FE |
| S3-12 | Polish | SEO: landing meta + sitemap + OG | NFR-03 | 3 | FE |
| S3-13 | Polish | Loading states, error states, empty states audit | — | 5 | FE |
| S3-14 | Analytics | Funnel dashboard PostHog: guest → register → first lesson | — | 3 | BE |
| S3-15 | Content | Founder produce 20+ lessons + 10+ dictation | — | — | Founder |
| S3-16 | QA | Full regression + cross-browser | — | 5 | QA |
| S3-17 | Launch | Marketing site copy + onboarding emails | — | 5 | Founder |

**Total Sprint 3: ~63 pts (lower — content production + QA heavy)**

**Definition of Done — Sprint 3 / Public MVP:**
- [x] ≥ 20 lessons + ≥ 10 dictation published
- [x] Shadowing available (Could-have working)
- [x] Streak counter visible
- [x] LCP < 3s p75 trên Real User Monitoring
- [x] Public landing page với SEO

---

## 4. Phase 2 — Sprint 4+ (Post-MVP)

| # | Theme | Features | When |
|---|---|---|---|
| Sprint 4 | Mock Test foundation | Schema, Builder, Timer logic | After MVP validation |
| Sprint 5 | Mock Test learner UI | Full mock, submit, review | — |
| Sprint 6 | Placement Test | Initial assessment | — |
| Later | AI Correction (pronunciation, free text) | — | Premium phase |
| Later | Payment / Subscription | Premium content, Stripe/SePay | After value validated |
| Later | Community / Forum | Discussion threads | — |

---

## 5. Cross-Sprint Tracking

### 5.1 Feature → Sprint Map (Reverse Index)
| Feature | Sprint |
|---|---|
| Auth basic | 1 |
| Auth Google OAuth | 2 |
| Dashboard | 1 (core) + 2 (recent activity) |
| Learning Path | 1 |
| Lesson Detail | 1 |
| Exercise | 1 |
| Progress | 1 (lesson/exercise) + 2 (dictation) + 3 (streak) |
| Quick Note | 2 |
| Notes Page | 2 |
| Dictation | 2 |
| Shadowing | 3 |
| Guest Trial | 2 |
| Bookmark | 2 |
| Onboarding | 2 |
| Countdown | 2 |
| Saved Mistakes | 3 |
| Streak | 3 |
| Admin Lesson/Exercise | 1 |
| Admin Dictation | 2 |
| Admin Shadowing | 3 |
| Mock Test | 4+ |

### 5.2 Dependency Critical Path
```
S1-05 (schema) ─┬─ S1-07 (Admin Lesson) ─ S1-13 (LP) ─ S1-15 (Lesson UI) ─ S1-17 (Exercise) ─ S1-20 (Progress)
                ├─ S1-11 (Dashboard card)
                └─ S1-01 (Auth) ─ S1-04 (Admin login)

S2-02 (Guest dash) ── needs S1-13, S1-15
S2-12 (Dictation admin) ── needs S1-07 pattern
S2-07 (QN floating) ── needs S1-15 layout

S3-01 (Shadowing) ── needs S2-12 audio pattern
S3-05 (Saved Mistakes) ── needs S1-17, S1-18 exercise flow
```

### 5.3 Risk Register
| Risk | Sprint | Mitigation |
|---|---|---|
| Strapi vs Supabase tech stack chưa chốt | 1 | Lock decision in OD-04 before S1-05 |
| Audio upload UX phức tạp | 2 | Spike S2-12 đầu sprint, fallback `<input type=file>` |
| MediaRecorder cross-browser | 3 | Test sớm trên iOS Safari, có fallback |
| Content production chậm | 1–3 | Founder track parallel với dev sprint |
| Compare rules dictation gây hiểu lầm | 2 | UX test với 3 users trước S2-14 |

---

## 6. Estimation Notes

- Points dùng Fibonacci (1, 2, 3, 5, 8) — relative complexity, not hours.
- 1 sprint cap ~80–100 pts cho 3-người-team.
- Sprint 3 nhẹ tay vì cần buffer cho content production + launch prep.

---

> **Status:** Draft v1.0 — Cần founder confirm sprint length + team composition (OD-11).
