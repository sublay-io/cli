# v7 Compatibility Issues

Issues discovered by the playground's TypeScript checker after upgrading to v7 packages.
Run `cd playground && npx tsc --noEmit` to reproduce.

---

## Issue 1: `useCommentVotes` no longer exported from `@sublay/react-js`

**Error**: `TS2305 — Module '"@sublay/react-js"' has no exported member 'useCommentVotes'`

The hook was removed or renamed in v7. Components that imported it will fail at runtime.

**Affected files (all 4 variants):**
- `registry/react/comments-social/styled/files/comments-feed/comment/comment.tsx`
- `registry/react/comments-social/tailwind/files/comments-feed/comment/comment.tsx`
- `registry/react/comments-threaded/styled/files/comments-feed/comment-thread/single-comment/vote-buttons.tsx`
- `registry/react/comments-threaded/tailwind/files/comments-feed/comment-thread/single-comment/vote-buttons.tsx`

**Fix:** Find the replacement hook in v7 and update the import + call site in all 4 files.

---

## Issue 2: `comment.user` is now optional in the v7 `Comment` type

**Errors**: `TS18048 — 'comment.user' is possibly 'undefined'` and `TS2345 — 'User | undefined' not assignable to 'Partial<User>'`

The `user` field on `Comment` changed from required to optional in v7. All direct accesses like `comment.user.name` now need null guards.

**Affected files (all 4 variants):**
- `registry/react/comments-social/styled/files/comments-feed/comment/comment.tsx` (lines 120, 124, 125, 156, 160, 161, 173)
- `registry/react/comments-social/tailwind/files/comments-feed/comment/comment.tsx` (lines 110, 114, 115, 135, 139, 140, 147)
- `registry/react/comments-threaded/styled/files/comments-feed/comment-thread/single-comment/single-comment.tsx` (lines 91, 94, 156, 159, 169)
- `registry/react/comments-threaded/tailwind/files/comments-feed/comment-thread/single-comment/single-comment.tsx` (lines 80, 83, 122, 125, 129)

**Fix:** Add optional chaining (`comment.user?.name`) or an early `if (!comment.user) return null` guard at the top of each component.

---

## Issue 3: `@sublay/ui-core-react-js` and `lucide-react` components incompatible with React 19

**Error**: `TS2786 — 'X' cannot be used as a JSX component` (type `FunctionComponent<Props>` is not assignable — `ReactNode | Promise<ReactNode>` vs `ReactNode`)

React 19 tightened its JSX return type. Packages compiled against `@types/react@18` return `ReactNode | Promise<ReactNode>` from their components, which React 19's JSX transformer no longer accepts.

**From `@sublay/ui-core-react-js`:**
- `EllipsisIcon` — comments-social (styled + tailwind)
- `HeartFullIcon`, `HeartIcon` — comments-social (styled + tailwind)
- `Modal` — comments-social (styled + tailwind), comments-threaded (styled + tailwind)
- `FlagIcon` — comments-social (styled + tailwind), comments-threaded (styled + tailwind)
- `Icon` — notifications-control (styled + tailwind)

**From `@sublay/react-js` / `@sublay/core`:**
- `CommentSectionProvider` — comments-social (styled + tailwind), comments-threaded (styled + tailwind)

**From `lucide-react`:**
- `Bell`, `Loader2`, `CheckCheck` — notifications-control (styled + tailwind)

**Fix:** `@sublay/ui-core-react-js` (and `@sublay/core` / `@sublay/react-js`) need to be rebuilt and republished with `@types/react@19`. For `lucide-react`, upgrade to a version that ships React 19–compatible types (v0.469+ added React 19 support).

---

## Issue 4: `initiatorAvatar` property removed from notification data shape

**Error**: `TS2339 — Property 'initiatorAvatar' does not exist on type ...`

The v7 notification data object no longer includes an `initiatorAvatar` field on certain notification subtypes (milestone-type notifications). The component reads it unconditionally.

**Affected files:**
- `registry/react/notifications-control/styled/files/notification-item.tsx` (line 211)
- `registry/react/notifications-control/tailwind/files/notification-item.tsx` (line 116)

**Fix:** Check the v7 `AppNotification` type to see how the initiator avatar is now accessed (likely on a different key or subtype). Update the field access in both files.

---

## Issue 5: `FlagIcon` props changed in `@sublay/ui-core-react-js` v7

**Error**: `TS2322 — Type '{ size: number; className: string; }' is not assignable to type 'IntrinsicAttributes & IconProps'`

The `FlagIcon` component's `IconProps` interface no longer accepts `className` (and/or `color`) as a prop. This is in the tailwind variants only (styled variants pass `size` + `color`, tailwind variants pass `size` + `className`).

**Affected files:**
- `registry/react/comments-social/tailwind/files/modals/comment-menu-modal/report-content.tsx` (line 54)
- `registry/react/comments-threaded/tailwind/files/modals/comment-menu-modal/report-content.tsx` (line 55)

**Fix:** Check the updated `IconProps` interface in v7 and adjust the props passed to `FlagIcon` to match.

---

## Summary Table

| Issue | Root cause | Components affected | Variants |
|---|---|---|---|
| `useCommentVotes` missing | Hook renamed/removed in v7 SDK | comments-social, comments-threaded | styled + tailwind |
| `comment.user` optional | `Comment` type changed in v7 | comments-social, comments-threaded | styled + tailwind |
| React 19 JSX incompatibility | `ui-core-react-js` built with React 18 types | all 3 components | styled + tailwind |
| `initiatorAvatar` missing | Notification data shape changed in v7 | notifications-control | styled + tailwind |
| `FlagIcon` props changed | `IconProps` interface changed in v7 | comments-social, comments-threaded | tailwind only |
