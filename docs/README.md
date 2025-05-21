# NoteOrbit Build Prompt

@docs https://clerk.com/docs  
@docs https://www.npmjs.com/package/react-markdown-editor-lite  
@docs https://tailwindcss.com/docs

## 🔧 Project Overview

Build a **multi-tenant note-taking SaaS app** using the following base:

- Framework: `Next.js` (App Router or Pages – choose what works best)
- UI: TailwindCSS (already installed, use **no fancy UI** — minimal, Clerk-style colors)
- Auth: Clerk (use official React SDK + Org API)
- DB: PostgreSQL via Docker Compose
- ORM: Use a clean, modern ORM like Prisma (ONLY if it's modular, no crap inside core logic)
- Editor: Use `react-markdown-editor-lite` ONLY
- Modularity: Use **modular architecture**: keep logic in `lib/`, split by feature (auth, notes, orgs, etc)

The project is already initialized. Continue building from the current state.

---

## ✅ Core Features and What to Learn

| Feature | What to Learn |
|--------|----------------|
| ✅ Sign up / Sign in | Use Clerk's `<SignIn />`, `<SignUp />` React components |
| ✅ Multi-tenancy | Clerk's Organizations API – users must belong to an org |
| ✅ Authenticated routes | Use `withServerSideAuth()` and `useUser()` hooks |
| ✅ CRUD for notes | Fullstack Next.js API + client logic |
| ✅ Org-level access | Notes visible/accessible only inside the org they’re created in |
| ✅ Fine-grained access control | Public/Shared/Private modes with allow/deny edit/view lists |
| ✅ Collaborative locking | Only 1 user can edit a note at a time (locking mechanism) |
| ✅ Invite to org | Users with "admin" role can invite others |
| ✅ Roles (admin vs member) | Add role system scoped to org context |
| ✅ Markdown editor | Use `react-markdown-editor-lite` only – setup basic config for inline editing |
| ✅ Logging | Log all actions clearly – both backend and UI events |
| ✅ Caching & Mutations | Use `useQuery`/`useMutation`, handle cache invalidation per mutation |
| ✅ Docker-based Postgres | Include Docker Compose setup with seeded schema |
| ✅ Tailwind minimal UI | Use Clerk UI colors, keep everything *ultra minimal*

---

## 🗂️ Project Structure

Please structure it cleanly as:

```

/app
/dashboard
/notes
\[noteId]/edit.tsx
new\.tsx
index.tsx
/auth
sign-in.tsx
sign-up.tsx
/org
index.tsx
invite.tsx
/lib
/auth
/notes
/org
/db
/types
/utils
/prisma
schema.prisma

````

---

## 📝 Notes: CRUD and Access Logic

Each **note belongs to an org** and has the following fields:

```ts
id
title
content
createdByUserId
organizationId
accessLevel: 'public' | 'shared' | 'private'
canEdit: 'everyone' | 'accessList' | 'denyList' | 'noOne'
editAccessList: userId[]
viewAccessList: userId[]
lockedByUserId: userId | null
lockedAt: timestamp | null
````

### Rules:

* All org members can view **public** notes
* Shared notes use `viewAccessList` and `editAccessList`
* Private notes are only visible to the creator unless shared
* If `lockedByUserId` is set and not expired, only that user can edit the note
* If user tries to edit while locked, show a message and disable editor

---

## 🧑‍🤝‍🧑 Multi-Tenancy via Clerk

Use **Clerk Organizations API**:

* Users must belong to one organization
* Use `<OrganizationSwitcher />` and `<OrganizationProfile />`
* Each org should see only its own notes and members

Roles:

* `admin`: can invite users, change roles, manage all notes
* `member`: can create/edit based on access

Invite flow:

* Admin sends invite via email
* Clerk handles invitation → user joins via magic link
* Once accepted, user is added to org with “member” role

---

## 🔐 Auth & Route Protection

* Use `useUser()` on client components
* Use `withServerSideAuth()` or middleware to protect server routes
* Redirect unauthenticated users to sign-in page

Pages needing auth:

* `/dashboard`
* `/org`
* `/dashboard/notes`

Use Clerk's official middleware docs — do not overcomplicate.

---

## 🖋️ Markdown Editor (react-markdown-editor-lite)

Use this npm package:

[https://www.npmjs.com/package/react-markdown-editor-lite](https://www.npmjs.com/package/react-markdown-editor-lite)

* Simple, live preview markdown editor
* Load/save content from note model
* Disable all advanced plugins and keep UI minimal

---

## 📦 Database Setup

Use **Docker Compose** for local development:

```yaml
version: '3'
services:
  db:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_USER: noteorbit
      POSTGRES_PASSWORD: noteorbit
      POSTGRES_DB: noteorbit_db
    volumes:
      - db_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
volumes:
  db_data:
```

* Use Prisma as ORM (or alternative that keeps schema & logic outside app code)
* Seed script to create a test org, test users, and sample notes
* No auth logic in DB — all org/user data from Clerk

---

## ⚠️ Important Rules

* 🧱 Be **modular** — keep notes, orgs, auth, etc., in clean `lib/` folders
* 🧼 Don’t add garbage logic inside UI files
* 🔁 Invalidate queries after each mutation
* 🔒 Only 1 user can edit note at a time – handle optimistic lock
* 🧾 Keep logs – client actions, note edits, org invites
* 📚 If stuck — reread docs mentioned with `@docs`
* 🧠 Don’t be smart — don’t use complex UI, animations, or unnecessary dependencies
* 🎨 Stick to Clerk’s design — subtle, clean, no colors beyond Tailwind defaults + Clerk brand tones
* 🛑 Absolutely no smart auto abstractions — just clean separation of concerns
* ✍️ Keep markdown editor clean — no overconfig, just WYSIWYG markdown with preview

---

## 🧭 Pages Overview

| Route                            | Purpose                          |
| -------------------------------- | -------------------------------- |
| `/sign-in`                       | Clerk SignIn page                |
| `/sign-up`                       | Clerk SignUp page                |
| `/dashboard`                     | List of notes for the org        |
| `/dashboard/notes/new`           | Create new note                  |
| `/dashboard/notes/[noteId]/edit` | Edit note if you have permission |
| `/org`                           | Manage organization info         |
| `/org/invite`                    | Invite users (admin only)        |

---

## 🧪 Bonus

* Optional: `useLock()` hook for note locking logic
* Optional: Real-time editor state (basic WebSocket or polling)
* Optional: Audit trail for edits

---

@docs [https://clerk.com/docs](https://clerk.com/docs)
@docs [https://www.npmjs.com/package/react-markdown-editor-lite](https://www.npmjs.com/package/react-markdown-editor-lite)
@docs [https://tailwindcss.com/docs](https://tailwindcss.com/docs)

Recheck docs **every time there's confusion**. Keep repeating and re-reading until implementation is aligned. Keep each module clean. No smart guessing. Stick to this spec. GO.