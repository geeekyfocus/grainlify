# Dispute Resolution Arbiter Interface Specification

**Component:** `frontend/src/features/admin/components/DisputeResolutionPanel.tsx`
**Issue:** #1391
**Date:** 2026-06-25

---

## 1. Overview

When a contributor and a maintainer disagree on whether a bounty was completed, a designated arbiter reviews evidence and approves a release, refund, or split payout. This spec covers the three-step arbiter flow accessible from `AdminPage` and `ProjectDetailPage`.

---

## 2. Three-Step Flow

```
[Detail View] → [Decision Panel] → [Confirmation Screen]
      ↑               ↑
   ← back          ← back
```

---

## 3. Dispute Detail View

### Layout — 1440 px desktop

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Open Dispute · prog-stellar-q1                               [X close] │
│  Implement XLM wallet adapter                                           │
│  Escrow: 1000 XLM                                                       │
├───────────────────────────────────┬─────────────────────────────────────┤
│  CONTRIBUTOR CLAIM                │  MAINTAINER COUNTER-CLAIM           │
│  "PR merged and all criteria…"    │  "Integration tests not included…"  │
│  GCONT...ADDR                     │  GMAIN...ADDR                       │
├───────────────────────────────────┴─────────────────────────────────────┤
│  ATTACHED EVIDENCE                                                      │
│  [PR #42 ↗]  [Test run ↗]                                              │
├─────────────────────────────────────────────────────────────────────────┤
│  TIMELINE                                                               │
│  ●  2026-06-01  GCONT  Opened dispute                                   │
│  │                                                                      │
│  ●  2026-06-02  GMAIN  Submitted counter-claim                          │
├─────────────────────────────────────────────────────────────────────────┤
│                    [ Review as Arbiter → ]                              │
└─────────────────────────────────────────────────────────────────────────┘
```

### Layout — 768 px tablet

- Claim / counter-claim panels stack vertically (grid → 1 col)
- Evidence pills wrap to multiple lines
- Timeline remains single column
- Padding reduces from `p-8` to `p-5`

### Accessibility
- `role="dialog"` `aria-modal="true"` `aria-labelledby="dispute-title"`
- Evidence links: `target="_blank" rel="noopener noreferrer"`
- Timeline rendered as `<ul aria-label="Dispute timeline">`

---

## 4. Arbiter Decision Panel

### Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│  ← Back to dispute                                                      │
│  Arbiter Decision                                                       │
│  Choose how to distribute the escrow of 1000 XLM.                      │
│                                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                 │
│  │  ✓ Full      │  │  ⚖ Split     │  │  ↺ Full      │                 │
│  │   Release    │  │              │  │   Refund     │                 │
│  └──────────────┘  └──────────────┘  └──────────────┘                 │
│  (active: gold border + bg tint)                                        │
│                                                                         │
│  PAYOUT SPLIT                              70% → contributor            │
│  ──────────────────────────────────                                     │
│  [■■■■■■■■■■■■■■■○○○○○○○○○○○○○░░░░]  ← gold fill / neutral empty      │
│                                                                         │
│  ┌─────────────────────┐  ┌─────────────────────┐                      │
│  │  CONTRIBUTOR        │  │  MAINTAINER         │                      │
│  │  700.00 XLM (gold)  │  │  300.00 XLM (red)   │                      │
│  │  GCONT...ADDR       │  │  GMAIN...ADDR        │                      │
│  └─────────────────────┘  └─────────────────────┘                      │
│                                                                         │
│                  [ Review Decision → ]                                  │
└─────────────────────────────────────────────────────────────────────────┘
```

### Split-Payout Slider Spec

| Property | Value |
|---|---|
| Range | 0–100 (integer %) |
| Step | 1 (arrow keys), 10 (PageUp/PageDown) |
| Fill | Gold `#c9983a` left of thumb, neutral right |
| Live preview | Amount updates on every value change |
| `aria-label` | "Contributor payout percentage" |
| `aria-valuetext` | `"{n}% to contributor, {100-n}% to maintainer"` |
| `aria-live` on preview | `polite`, `atomic` |

**Decision button sync:**
- `Full Release` button → sets slider to 100
- `Full Refund` button → sets slider to 0
- `Split` button → sets slider to 50 (if was at 0 or 100)
- Dragging slider → updates `aria-pressed` on the matching button

### Keyboard Operability
- `←` / `→` — change by 1%
- `PageUp` / `PageDown` — change by 10%, clamped to [0, 100]
- `Home` / `End` — jump to 0 or 100 (native browser behavior)
- All three decision buttons reachable via `Tab`; activated with `Space` / `Enter`

---

## 5. Irreversibility Confirmation Screen

### Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│  ← Change decision                                                      │
│                                                                         │
│  ┌── HIGH-EMPHASIS WARNING (glassmorphism) ───────────────────────────┐ │
│  │  ⚠  This action is irreversible                                    │ │
│  │     Submitting this decision will trigger an on-chain transaction  │ │
│  │     on the Stellar network. Once confirmed, funds cannot be        │ │
│  │     recovered or redistributed. Verify all evidence before         │ │
│  │     proceeding.                                                    │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  YOUR DECISION                                                          │
│  ⚖  Split: 70% / 30%                                                   │
│  Contributor: 700.00 XLM        Maintainer: 300.00 XLM                 │
│                                                                         │
│               [ Confirm & Submit On-chain ]  ← red CTA                 │
└─────────────────────────────────────────────────────────────────────────┘
```

### Warning Treatment

| Token | Value |
|---|---|
| Background (dark) | `bg-red-950/60 border-red-500/40` |
| Background (light) | `bg-red-50 border-red-300` |
| `backdrop-filter` | `blur(20px)` |
| Gradient shimmer | `linear-gradient(135deg, rgba(239,68,68,0.06) 0%, transparent 60%)` |
| Icon | `AlertTriangle` in `text-red-400` circle |
| `role` | `alert` |
| `aria-live` | `assertive` |

### Confirm Button

- Background: `bg-red-600 hover:bg-red-700` (deliberately not gold — signals danger)
- Focus outline: `focus:outline-red-400`
- Text: "Confirm & Submit On-chain"

---

## 6. ARIA & Focus Management

| Event | Focus behaviour |
|---|---|
| Panel opens | First focusable element (close button) |
| Step → decide | "Full Release" button receives focus |
| Step → confirm | "← Change decision" link receives focus |
| Slider keyboard | Handled via `onKeyDown`; `PageUp`/`PageDown` intercepted |
| onClose | Focus returns to the element that opened the panel |
| Backdrop click | Calls `onClose` |

---

## 7. Security Notes

- `DisputeResolutionPanel` never submits an on-chain transaction directly. It calls the `onDecide` prop with `{ disputeId, decision, contributorPct }`; the parent is responsible for authentication, signing, and submission.
- `evidence` URLs are rendered as `<a rel="noopener noreferrer">` to prevent tab-napping.
- No user input is echoed unescaped into the DOM; all string fields come from the typed `Dispute` interface.
- `contributorPct` is constrained to integer `[0, 100]` by the `<input type="range">` element.

---

## 8. QA Checklist

- [ ] Slider reachable and operable via keyboard only
- [ ] PageUp/PageDown work and clamp correctly
- [ ] `aria-valuetext` reads both sides at each step
- [ ] Live preview amounts update on every slider move
- [ ] `role="alert"` on irreversibility warning fires SR announcement on step entry
- [ ] `role="dialog"` panel traps focus (Tab / Shift+Tab)
- [ ] Backdrop click calls `onClose`
- [ ] Close button (X) calls `onClose`
- [ ] Confirm button triggers `onDecide` with correct payload
- [ ] WCAG 2.1 AA: 4.5:1 contrast on gold text against dark surface
- [ ] 768 px: panels stack vertically, no overflow
- [ ] 1440 px: claim / counter-claim side by side

---

## 9. Implementation Reference

| File | Purpose |
|---|---|
| `frontend/src/features/admin/components/DisputeResolutionPanel.tsx` | Component |
| `frontend/src/features/admin/__tests__/DisputeResolutionPanel.test.tsx` | 36 tests (all passing) |
| `frontend/vite.config.ts` | Vitest + jsdom configuration |
| `frontend/src/test-setup.ts` | `@testing-library/jest-dom` matchers |
