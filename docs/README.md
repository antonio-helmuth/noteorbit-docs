# NoteOrbit

A modern multi-tenant note-taking SaaS application with collaborative features and fine-grained access control.

## About

NoteOrbit is a powerful collaborative note-taking platform designed for teams and organizations. It enables seamless sharing and collaboration on markdown documents while maintaining robust access control and security.

## Tech Stack

- **Frontend**: Next.js, React, TypeScript, TailwindCSS
- **Backend**: Next.js API Routes, Node.js
- **Database**: PostgreSQL
- **Authentication**: Clerk (with Organizations API)
- **Editor**: react-markdown-editor-lite
- **State Management**: React Query
- **Deployment**: Docker, Kubernetes

## Getting Started

Before working on this project, please thoroughly review the comprehensive documentation:

📚 [NoteOrbit Documentation](https://antonio-helmuth.github.io/noteorbit-docs/#/)

## Main Goals

NoteOrbit was designed to provide a secure, collaborative note-taking environment with:

- **Clean, modular architecture** - Maintainable and scalable codebase
- **Multi-tenant security** - Strict isolation between organizations
- **Intuitive collaboration** - Real-time editing with minimal friction
- **Simple, focused UX** - Clean interface that prioritizes content

## Key Features

- ✅ Multi-tenant workspaces with Clerk Organizations
- ✅ Markdown editing with live preview
- ✅ Fine-grained access control (public/shared/private)
- ✅ Collaborative locking to prevent edit conflicts
- ✅ Role-based permissions (admin/member)
- ✅ Invite system for team collaboration
- ✅ Version history and note audit trail
- ✅ Real-time notifications

## Development

```bash
# Install dependencies
yarn install

# Start dev environment (with Docker)
docker-compose up -d
yarn run dev

# Run tests
yarn test
```

---

Created with ❤️ by Antonio