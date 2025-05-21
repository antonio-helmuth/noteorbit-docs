# NoteOrbit System Architecture

## 1. System Overview and Architecture Principles

### 1.1 Core Architecture

NoteOrbit is a multi-tenant note-taking SaaS application built on a modern full-stack architecture. The system follows these core architectural principles:

1. **Modularity First**: Each feature is isolated into its own module with clear boundaries
2. **Clean Separation of Concerns**: UI, business logic, and data access are distinctly separated
3. **Domain-Driven Design**: Code organization reflects business domains (notes, organizations, etc.)
4. **API-First Approach**: All client-server communication happens through well-defined API endpoints
5. **Authentication as Infrastructure**: Auth is treated as cross-cutting infrastructure rather than a feature
6. **Stateless Backend**: Server maintains no user session state outside the auth system
7. **Progressive Enhancement**: Core functionality works without JS, enhanced with client-side features

### 1.2 Technology Stack Rationale

| Technology | Purpose | Rationale |
|------------|---------|-----------|
| **Next.js (App Router)** | Full-stack React framework | Server components reduce client bundle, API Routes simplify backend development |
| **TypeScript** | Type-safe JavaScript | Prevents common errors, improves maintainability and IDE support |
| **TailwindCSS** | Utility-first CSS framework | Rapid UI development without context switching, standardized design system |
| **Clerk** | Authentication & Organization management | Reduces auth complexity, provides multi-tenant capabilities out of the box |
| **PostgreSQL** | Relational database | ACID compliance, robust for structured data with relations between entities |
| **Prisma** | ORM | Type-safe database access, schema migrations, clean separation from application code |
| **React Query** | Data fetching, caching & mutations | Simplifies data lifecycle, provides caching and optimistic updates |
| **react-markdown-editor-lite** | Markdown editing | Lightweight, focused editor with the features we need and nothing more |

### 1.3 High-Level System Diagram

```
┌─────────────────────────────────────────┐
│                                         │
│              Client Browser              │
│  ┌─────────────┐       ┌─────────────┐  │
│  │   React UI   │◄────►│ Query Cache  │  │
│  └─────────────┘       └─────────────┘  │
│          ▲                              │
└──────────┼──────────────────────────────┘
           │
           │ HTTP/API Requests
           ▼
┌─────────────────────────────────────────┐
│                                         │
│              Next.js Server             │
│                                         │
│  ┌─────────────┐       ┌─────────────┐  │
│  │ API Routes  │◄────►│Clerk Auth MW │  │
│  └─────────────┘       └─────────────┘  │
│          ▲                    ▲         │
│          │                    │         │
│          ▼                    ▼         │
│  ┌─────────────┐       ┌─────────────┐  │
│  │Business Logic│      │ Clerk API   │  │
│  └─────────────┘       └─────────────┘  │
│          ▲                              │
└──────────┼──────────────────────────────┘
           │
           │ Database Queries (Prisma)
           ▼
┌─────────────────────────────────────────┐
│                                         │
│            PostgreSQL Database          │
│                                         │
│  ┌─────────────────────────────────────┐│
│  │              Tables                 ││
│  │  - Notes                            ││
│  │  - Organizations (from Clerk)       ││
│  │  - Users (from Clerk)               ││
│  │  - Locks                            ││
│  └─────────────────────────────────────┘│
│                                         │
└─────────────────────────────────────────┘
```

### 1.4 Data Flow Architecture

The application follows a unidirectional data flow pattern:

1. **UI Event** → User interacts with the application
2. **Client Action** → UI triggers client-side action (hook, mutation, query)
3. **API Request** → Client sends HTTP request to API endpoint
4. **Server Validation** → API validates request data and user permission
5. **Business Logic** → Server executes business logic
6. **Database Operation** → Server performs database operations via Prisma
7. **Response Generation** → Server generates API response
8. **Client Processing** → Client processes API response data
9. **State Update** → Query cache is updated, UI reactively renders new data
10. **UI Update** → User sees the updated information

This pattern ensures predictable data flow, simplifies debugging, and enables clean separation between components.

## 2. Project Structure and Organization

### 2.1 Directory Structure

```
/
├── app/                           # Next.js App Router pages
│   ├── api/                       # API Routes
│   │   ├── notes/                 # Notes API endpoints
│   │   │   ├── route.ts           # GET/POST /api/notes
│   │   │   ├── [noteId]/          # Note-specific endpoints
│   │   │   │   ├── route.ts       # GET/PATCH/DELETE /api/notes/[noteId]
│   │   │   │   ├── lock/          # Note locking endpoints
│   │   │   │   │   └── route.ts   # POST/DELETE /api/notes/[noteId]/lock
│   │   │   │   └── access/        # Access control endpoints
│   │   │   │       └── route.ts   # PATCH /api/notes/[noteId]/access
│   │   ├── org/                   # Organization API endpoints
│   │   │   ├── route.ts           # GET/PATCH /api/org
│   │   │   ├── members/           # Organization members endpoints
│   │   │   │   └── route.ts       # GET/POST /api/org/members
│   │   │   └── invite/            # Organization invite endpoints
│   │   │       └── route.ts       # POST /api/org/invite
│   ├── dashboard/                 # Dashboard pages
│   │   ├── page.tsx               # Dashboard home page
│   │   ├── layout.tsx             # Dashboard layout with navigation
│   │   └── notes/                 # Notes pages
│   │       ├── page.tsx           # Notes list page
│   │       ├── new/               # New note page
│   │       │   └── page.tsx       # Create new note
│   │       └── [noteId]/          # Note-specific pages
│   │           ├── page.tsx       # Note view page
│   │           └── edit/          # Note edit page
│   │               └── page.tsx   # Edit note
│   ├── auth/                      # Auth pages
│   │   ├── sign-in/               # Sign in page
│   │   │   └── page.tsx           # Sign in form
│   │   └── sign-up/               # Sign up page
│   │       └── page.tsx           # Sign up form
│   ├── org/                       # Organization pages
│   │   ├── page.tsx               # Organization management
│   │   └── invite/                # Invite members page
│   │       └── page.tsx           # Invite form
│   ├── layout.tsx                 # Root layout
│   └── page.tsx                   # Home/landing page
├── lib/                           # Business logic and utilities
│   ├── auth/                      # Authentication utilities
│   │   ├── clerk-client.ts        # Clerk client configuration
│   │   ├── clerk-server.ts        # Clerk server utilities
│   │   ├── get-current-user.ts    # Get current user utility
│   │   ├── get-current-org.ts     # Get current organization utility
│   │   ├── middleware.ts          # Auth middleware configurations
│   │   └── types.ts               # Auth-related type definitions
│   ├── notes/                     # Notes module
│   │   ├── api/                   # API handlers for notes
│   │   │   ├── create-note.ts     # Create note handler
│   │   │   ├── delete-note.ts     # Delete note handler
│   │   │   ├── get-note.ts        # Get note handler
│   │   │   ├── get-notes.ts       # Get notes list handler
│   │   │   ├── update-note.ts     # Update note handler
│   │   │   ├── lock-note.ts       # Lock note handler
│   │   │   └── update-access.ts   # Update note access handler
│   │   ├── hooks/                 # React hooks for notes
│   │   │   ├── use-notes.ts       # Hook for notes list
│   │   │   ├── use-note.ts        # Hook for single note
│   │   │   ├── use-create-note.ts # Hook for note creation
│   │   │   ├── use-update-note.ts # Hook for note updates
│   │   │   ├── use-delete-note.ts # Hook for note deletion
│   │   │   ├── use-note-lock.ts   # Hook for note locking
│   │   │   └── use-note-access.ts # Hook for note access control
│   │   ├── components/            # Note-specific components
│   │   │   ├── note-card.tsx      # Note card component
│   │   │   ├── note-list.tsx      # Notes list component
│   │   │   ├── note-editor.tsx    # Note editor component
│   │   │   ├── note-viewer.tsx    # Note viewer component
│   │   │   ├── access-control.tsx # Access control component
│   │   │   └── lock-indicator.tsx # Lock status indicator
│   │   ├── utils/                 # Note utilities
│   │   │   ├── access-checks.ts   # Access control utilities
│   │   │   ├── lock-utils.ts      # Lock utilities
│   │   │   └── markdown-utils.ts  # Markdown processing utilities
│   │   └── types.ts               # Note type definitions
│   ├── org/                       # Organization module
│   │   ├── api/                   # API handlers for organizations
│   │   │   ├── get-org.ts         # Get organization handler
│   │   │   ├── update-org.ts      # Update organization handler
│   │   │   ├── get-members.ts     # Get members handler
│   │   │   ├── update-member.ts   # Update member handler
│   │   │   └── invite-member.ts   # Invite member handler
│   │   ├── hooks/                 # React hooks for organizations
│   │   │   ├── use-org.ts         # Hook for organization data
│   │   │   ├── use-members.ts     # Hook for members list
│   │   │   ├── use-update-org.ts  # Hook for updating organization
│   │   │   └── use-invite.ts      # Hook for sending invitations
│   │   ├── components/            # Organization-specific components
│   │   │   ├── org-profile.tsx    # Organization profile component
│   │   │   ├── members-list.tsx   # Members list component
│   │   │   ├── invite-form.tsx    # Invitation form component
│   │   │   └── role-badge.tsx     # Role indicator badge
│   │   ├── utils/                 # Organization utilities
│   │   │   ├── role-checks.ts     # Role validation utilities
│   │   │   └── email-validators.ts # Email validation utilities
│   │   └── types.ts               # Organization type definitions
│   ├── db/                        # Database utilities
│   │   ├── prisma.ts              # Prisma client
│   │   └── seed.ts                # Database seeding utilities
│   ├── api/                       # API utilities
│   │   ├── client.ts              # API client for frontend
│   │   ├── middleware.ts          # API middleware
│   │   ├── response.ts            # API response utilities
│   │   └── error.ts               # API error handling
│   ├── logger/                    # Logging utilities
│   │   ├── server-logger.ts       # Server-side logging
│   │   └── client-logger.ts       # Client-side logging
│   ├── utils/                     # General utilities
│   │   ├── date.ts                # Date utilities
│   │   ├── validation.ts          # Input validation utilities
│   │   └── ids.ts                 # ID generation/handling utilities
│   └── types/                     # Shared type definitions
│       ├── api.ts                 # API types
│       ├── common.ts              # Common type definitions
│       └── index.ts               # Type export index
├── components/                    # Shared UI components
│   ├── ui/                        # Basic UI components
│   │   ├── button.tsx             # Button component
│   │   ├── input.tsx              # Input component
│   │   ├── select.tsx             # Select component
│   │   ├── textarea.tsx           # Textarea component
│   │   ├── card.tsx               # Card component
│   │   ├── modal.tsx              # Modal component
│   │   └── toast.tsx              # Toast notification component
│   ├── layout/                    # Layout components
│   │   ├── header.tsx             # Header component
│   │   ├── sidebar.tsx            # Sidebar component
│   │   ├── footer.tsx             # Footer component
│   │   └── container.tsx          # Container component
│   ├── data/                      # Data display components
│   │   ├── table.tsx              # Table component
│   │   ├── pagination.tsx         # Pagination component
│   │   └── empty-state.tsx        # Empty state component
│   └── feedback/                  # Feedback components
│       ├── loading.tsx            # Loading indicator
│       ├── error.tsx              # Error display
│       └── success.tsx            # Success message
├── prisma/                        # Prisma ORM
│   ├── schema.prisma              # Prisma schema
│   ├── migrations/                # Database migrations
│   └── seed.ts                    # Database seed script
├── public/                        # Static files
├── styles/                        # Global styles
│   └── globals.css                # Global CSS
├── middleware.ts                  # Next.js middleware (auth)
├── next.config.js                 # Next.js configuration
├── tailwind.config.js             # TailwindCSS configuration
├── tsconfig.json                  # TypeScript configuration
├── docker-compose.yml             # Docker compose configuration
└── package.json                   # NPM dependencies and scripts
```

### 2.2 Module Organization

Each module follows a consistent structure:

1. **API Layer**: Handles HTTP requests and responses, input validation, and permissions
2. **Service Layer**: Contains business logic, separated from API concerns
3. **Data Access Layer**: Handles database interactions via Prisma
4. **Hooks Layer**: Provides React hooks for data fetching and mutations
5. **Component Layer**: Contains UI components specific to the module
6. **Utility Layer**: Provides utility functions specific to the module
7. **Types**: Defines TypeScript types and interfaces for the module

### 2.3 Naming Conventions

1. **Files and Directories**:
   - Use kebab-case for files and directories: `note-editor.tsx`, `access-control.tsx`
   - Use PascalCase for React components: `NoteEditor`, `AccessControl`
   - Use camelCase for variables, functions, and methods: `getNotes`, `useCurrentUser`

2. **Components**:
   - Prefix hooks with `use`: `useNotes`, `useOrganization`
   - Suffix context providers with `Provider`: `AuthProvider`, `NotesProvider`
   - Suffix higher-order components with `HOC`: `withAuth`, `withOrganization`

3. **API Endpoints**:
   - Use RESTful conventions: `/api/notes`, `/api/notes/[noteId]`
   - Use HTTP verbs appropriately: GET, POST, PATCH, DELETE
   - For sub-resources, nest appropriately: `/api/notes/[noteId]/lock`

4. **Database**:
   - Use snake_case for database columns: `created_at`, `organization_id`
   - Use singular nouns for table names: `note`, `organization`, `user`
   - Prefix foreign keys with related table: `user_id`, `organization_id`

### 2.4 Code Organization Rules

1. **No UI Logic in API Routes**: API routes should only handle HTTP concerns, delegating business logic to service functions
2. **No Direct Database Access in Components**: Components should access data only through hooks
3. **No Business Logic in UI Components**: UI components should focus on rendering and user interaction
4. **Shared Components in /components**: Only truly shared components go in the root `/components` directory
5. **Module-Specific Components Stay in Module**: Components specific to a module remain in that module's directory
6. **Consistent Export Pattern**: Each module exports a clear public API through an index.ts file
7. **Centralized Type Definitions**: Types used across modules go in `/lib/types`

## 3. Authentication and Authorization Architecture

### 3.1 Clerk Integration

Clerk provides the core authentication and organization management infrastructure. It's integrated at multiple levels:

1. **Frontend Components**: Using Clerk's React components for sign-in, sign-up, and organization management
2. **API Authentication**: Using Clerk's middleware to authenticate API requests
3. **Server-Side Authentication**: Using Clerk's server-side SDK to authenticate server-side requests
4. **Organization Management**: Using Clerk's Organizations API for multi-tenancy

#### 3.1.1 Auth Flow Diagram

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │
│  Sign Up    │────►│  Create or  │────►│ Dashboard   │
│  Page       │     │  Join Org   │     │ Page        │
│             │     │             │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
      ▲                                        ▲
      │                                        │
      │                                        │
      │           ┌─────────────┐              │
      │           │             │              │
      └───────────┤  Sign In    │──────────────┘
                  │  Page       │
                  │             │
                  └─────────────┘
                        ▲
                        │
                        │
                  ┌─────────────┐
                  │             │
                  │  Landing    │
                  │  Page       │
                  │             │
                  └─────────────┘
```

### 3.2 Multi-Tenancy Model

Multi-tenancy is implemented through Clerk's Organizations API:

1. **Organization Creation**: Users can create organizations during sign-up or later
2. **Organization Membership**: Users can be members of multiple organizations
3. **Organization Context**: Current organization is maintained in URL/state
4. **Organization Switching**: Users can switch between organizations they belong to

### 3.3 Role-Based Access Control

Roles are defined at the organization level:

1. **Admin**: Full control over organization settings, members, and all notes
2. **Member**: Can create notes and access notes based on permission settings

Role checks occur at multiple levels:

1. **UI Level**: Hiding/showing UI elements based on role
2. **API Level**: Validating permissions before processing requests
3. **Database Level**: Queries filter by organization and permission settings

### 3.4 Permission Model

Permissions follow a layered approach:

1. **Organization Membership**: Base requirement for accessing any notes
2. **Note Access Level**: Controls visibility within the organization
   - **Public**: Visible to all organization members
   - **Shared**: Visible to members in the view access list
   - **Private**: Visible only to the creator and explicitly permitted users
3. **Edit Permission**: Controls who can edit the note
   - **Everyone**: Any organization member who can view can also edit
   - **Access List**: Only users in the edit access list can edit
   - **Deny List**: Everyone except users in the deny list can edit
   - **No One**: No one can edit (read-only)

### 3.5 Auth Implementation Details

#### 3.5.1 Clerk Setup

```typescript
// lib/auth/clerk-client.ts
import { ClerkProvider } from '@clerk/nextjs/client';
import { dark } from '@clerk/themes';

export const ClerkProviderWithTheme = ({ children }: { children: React.ReactNode }) => (
  <ClerkProvider
    appearance={{
      baseTheme: dark,
      variables: {
        colorPrimary: '#4f46e5', // Indigo-600 from Tailwind
        colorText: '#0f172a',    // Slate-900 from Tailwind
      }
    }}
  >
    {children}
  </ClerkProvider>
);
```

#### 3.5.2 Middleware for Protected Routes

```typescript
// middleware.ts
import { authMiddleware, clerkClient, createRouteMatcher } from '@clerk/nextjs/server';

const publicRoutes = [
  '/',
  '/auth/sign-in(.*)',
  '/auth/sign-up(.*)',
  '/api/health',
  '/api/webhook/clerk',
];

const isPublic = createRouteMatcher(publicRoutes);

export default authMiddleware({
  publicRoutes: isPublic,
  ignoredRoutes: [
    '/(api|trpc)(.*)',
    '/_next(.*)',
    '/favicon.ico',
  ],
  afterAuth: async (auth, req, evt) => {
    // Redirect logged in users from auth pages
    if (auth.userId && req.nextUrl.pathname.startsWith('/auth')) {
      const url = new URL('/dashboard', req.url);
      return Response.redirect(url);
    }
    
    // Redirect logged out users to sign-in page
    if (!auth.userId && !isPublic(req.nextUrl.pathname)) {
      const url = new URL('/auth/sign-in', req.url);
      url.searchParams.set('redirect_url', req.nextUrl.pathname);
      return Response.redirect(url);
    }
    
    // Check if user has an organization
    if (auth.userId && !auth.orgId && req.nextUrl.pathname.startsWith('/dashboard')) {
      const user = await clerkClient.users.getUser(auth.userId);
      const orgs = await clerkClient.users.getOrganizationMembershipList({ userId: auth.userId });
      
      if (orgs.length === 0) {
        // User needs to create an organization
        const url = new URL('/org/create', req.url);
        return Response.redirect(url);
      } else {
        // Redirect to first organization
        const url = new URL(req.nextUrl.pathname, req.url);
        url.searchParams.set('org_id', orgs[0].organization.id);
        return Response.redirect(url);
      }
    }
  }
});

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

#### 3.5.3 Server-Side Auth Utilities

```typescript
// lib/auth/get-current-user.ts
import { currentUser } from '@clerk/nextjs/server';
import { redirect } from 'next/navigation';

export async function getCurrentUser() {
  const user = await currentUser();
  
  if (!user) {
    redirect('/auth/sign-in');
  }
  
  return {
    id: user.id,
    email: user.emailAddresses[0]?.emailAddress,
    name: `${user.firstName} ${user.lastName}`,
    imageUrl: user.imageUrl,
  };
}
```

```typescript
// lib/auth/get-current-org.ts
import { auth, clerkClient } from '@clerk/nextjs/server';
import { redirect } from 'next/navigation';

export async function getCurrentOrg() {
  const { userId, orgId } = auth();
  
  if (!userId) {
    redirect('/auth/sign-in');
  }
  
  if (!orgId) {
    const orgs = await clerkClient.users.getOrganizationMembershipList({ userId });
    
    if (orgs.length === 0) {
      redirect('/org/create');
    }
    
    redirect(`/dashboard?org_id=${orgs[0].organization.id}`);
  }
  
  const org = await clerkClient.organizations.getOrganization({ organizationId: orgId });
  const membership = await clerkClient.organizations.getOrganizationMembership({
    organizationId: orgId,
    userId
  });
  
  return {
    id: org.id,
    name: org.name,
    slug: org.slug,
    imageUrl: org.imageUrl,
    role: membership.role,
    permissions: membership.permissions,
    isAdmin: membership.role === 'admin',
  };
}
```

#### 3.5.4 Client-Side Auth Hooks

```typescript
// lib/auth/hooks/use-auth.ts
import { useUser, useOrganization } from '@clerk/nextjs/client';
import { useMemo } from 'react';

export function useAuth() {
  const { user, isLoaded: userLoaded } = useUser();
  const { organization, membership, isLoaded: orgLoaded } = useOrganization();
  
  const isAdmin = useMemo(() => membership?.role === 'admin', [membership]);
  
  const isLoaded = userLoaded && orgLoaded;
  
  return {
    user,
    organization,
    isAdmin,
    membership,
    isLoaded
  };
}
```

## 4. Database Schema and ORM Integration

### 4.1. Prisma Schema

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Note {
  id                String    @id @default(uuid())
  title             String
  content           String
  created_by_user_id String
  organization_id   String
  created_at        DateTime  @default(now())
  updated_at        DateTime  @updatedAt
  
  access_level      String    // 'public' | 'shared' | 'private'
  can_edit          String    // 'everyone' | 'accessList' | 'denyList' | 'noOne'
  
  // Stored as comma-separated lists of IDs
  edit_access_list  String?   
  view_access_list  String?
  
  locked_by_user_id String?
  locked_at         DateTime?
  
  @@index([organization_id])
  @@index([created_by_user_id])
  @@index([organization_id, access_level])
}

model Audit {
  id               String   @id @default(uuid())
  action           String   // 'create' | 'update' | 'delete' | 'lock' | 'unlock' | 'changeAccess'
  resource_type    String   // 'note' | 'organization'
  resource_id      String
  user_id          String
  organization_id  String
  details          Json?    // Additional details about the action
  created_at       DateTime @default(now())
  
  @@index([resource_type, resource_id])
  @@index([organization_id])
  @@index([user_id])
}
```

### 4.2. Prisma Client Setup

```typescript
// lib/db/prisma.ts
import { PrismaClient } from '@prisma/client';

declare global {
  var prisma: PrismaClient | undefined;
}

const prismaClient = globalThis.prisma || new PrismaClient();

if (process.env.NODE_ENV !== 'production') {
  globalThis.prisma = prismaClient;
}

export const prisma = prismaClient;
```

### 4.3. Seeding Script

```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  // Create sample notes for testing
  await prisma.note.createMany({
    data: [
      {
        id: 'note-1',
        title: 'Getting Started with NoteOrbit',
        content: '# Welcome to NoteOrbit\n\nThis is your first note. Edit it to get started!',
        created_by_user_id: 'seed-user-1',
        organization_id: 'seed-org-1',
        access_level: 'public',
        can_edit: 'everyone',
      },
      {
        id: 'note-2',
        title: 'Private Note Example',
        content: 'This is a private note example.',
        created_by_user_id: 'seed-user-1',
        organization_id: 'seed-org-1',
        access_level: 'private',
        can_edit: 'noOne',
      },
      {
        id: 'note-3',
        title: 'Shared Note Example',
        content: 'This is a shared note example.',
        created_by_user_id: 'seed-user-1',
        organization_id: 'seed-org-1',
        access_level: 'shared',
        can_edit: 'accessList',
        view_access_list: 'seed-user-1,seed-user-2',
        edit_access_list: 'seed-user-1',
      },
    ],
  });

  // Create audit log entries
  await prisma.audit.createMany({
    data: [
      {
        action: 'create',
        resource_type: 'note',
        resource_id: 'note-1',
        user_id: 'seed-user-1',
        organization_id: 'seed-org-1',
        details: { title: 'Getting Started with NoteOrbit' },
      },
    ],
  });
}

main()
  .then(async () => {
    await prisma.$disconnect();
  })
  .catch(async (e) => {
    console.error(e);
    await prisma.$disconnect();
    process.exit(1);
  });
```

### 4.4. Database Access Patterns

#### 4.4.1. Repository Pattern for Notes

```typescript
// lib/notes/repositories/note-repository.ts
import { prisma } from '@/lib/db/prisma';
import { Note, NoteAccessLevel, NoteEditPermission } from '@/lib/notes/types';

export const NoteRepository = {
  // Get all notes for an organization with filtering
  async getOrganizationNotes(
    organizationId: string,
    userId: string,
    filters?: {
      accessLevel?: NoteAccessLevel;
      createdBy?: string;
      search?: string;
    }
  ): Promise<Note[]> {
    // Base query conditions
    const where: any = {
      organization_id: organizationId,
    };
    
    // Apply access level filter if provided
    if (filters?.accessLevel) {
      where.access_level = filters.accessLevel;
    }
    
    // Apply creator filter if provided
    if (filters?.createdBy) {
      where.created_by_user_id = filters.createdBy;
    }
    
    // Apply search filter if provided
    if (filters?.search) {
      where.OR = [
        { title: { contains: filters.search, mode: 'insensitive' } },
        { content: { contains: filters.search, mode: 'insensitive' } },
      ];
    }
    
    // Get all notes that match the criteria
    const notes = await prisma.note.findMany({
      where,
      orderBy: { updated_at: 'desc' },
    });
    
    // Filter by access permission
    return notes.filter(note => {
      // Public notes are accessible to all organization members
      if (note.access_level === 'public') {
        return true;
      }
      
      // Private notes are only accessible to the creator
      if (note.access_level === 'private' && note.created_by_user_id === userId) {
        return true;
      }
      
      // Shared notes require checking the access list
      if (note.access_level === 'shared') {
        // Creator always has access
        if (note.created_by_user_id === userId) {
          return true;
        }
        
        // Check if the user is in the view access list
        const viewAccessList = note.view_access_list?.split(',') || [];
        return viewAccessList.includes(userId);
      }
      
      return false;
    });
  },
  
  // Get a single note by ID
  async getNoteById(noteId: string, userId: string, organizationId: string): Promise<Note | null> {
    const note = await prisma.note.findUnique({
      where: { id: noteId },
    });
    
    if (!note) {
      return null;
    }
    
    // Check if the note belongs to the correct organization
    if (note.organization_id !== organizationId) {
      return null;
    }
    
    // Check if the user has access to the note
    if (note.access_level === 'public') {
      return note;
    }
    
    if (note.access_level === 'private' && note.created_by_user_id !== userId) {
      return null;
    }
    
    if (note.access_level === 'shared') {
      // Creator always has access
      if (note.created_by_user_id === userId) {
        return note;
      }
      
      // Check if the user is in the view access list
      const viewAccessList = note.view_access_list?.split(',') || [];
      if (!viewAccessList.includes(userId)) {
        return null;
      }
    }
    
    return note;
  },
  
  // Create a new note
  async createNote(
    data: {
      title: string;
      content: string;
      createdByUserId: string;
      organizationId: string;
      accessLevel: NoteAccessLevel;
      canEdit: NoteEditPermission;
      editAccessList?: string[];
      viewAccessList?: string[];
    }
  ): Promise<Note> {
    return prisma.note.create({
      data: {
        title: data.title,
        content: data.content,
        created_by_user_id: data.createdByUserId,
        organization_id: data.organizationId,
        access_level: data.accessLevel,
        can_edit: data.canEdit,
        edit_access_list: data.editAccessList?.join(','),
        view_access_list: data.viewAccessList?.join(','),
      },
    });
  },
  
  // Update an existing note
  async updateNote(
    noteId: string,
    data: {
      title?: string;
      content?: string;
    }
  ): Promise<Note> {
    return prisma.note.update({
      where: { id: noteId },
      data: {
        title: data.title,
        content: data.content,
        updated_at: new Date(),
      },
    });
  },
  
  // Update note access settings
  async updateNoteAccess(
    noteId: string,
    data: {
      accessLevel?: NoteAccessLevel;
      canEdit?: NoteEditPermission;
      editAccessList?: string[];
      viewAccessList?: string[];
    }
  ): Promise<Note> {
    return prisma.note.update({
      where: { id: noteId },
      data: {
        access_level: data.accessLevel,
        can_edit: data.canEdit,
        edit_access_list: data.editAccessList?.join(','),
        view_access_list: data.viewAccessList?.join(','),
        updated_at: new Date(),
      },
    });
  },
  
  // Delete a note
  async deleteNote(noteId: string): Promise<void> {
    await prisma.note.delete({
      where: { id: noteId },
    });
  },
  
  // Lock a note for editing
  async lockNote(
    noteId: string,
    userId: string
  ): Promise<Note> {
    return prisma.note.update({
      where: { id: noteId },
      data: {
        locked_by_user_id: userId,
        locked_at: new Date(),
      },
    });
  },
  
  // Unlock a note
  async unlockNote(noteId: string): Promise<Note> {
    return prisma.note.update({
      where: { id: noteId },
      data: {
        locked_by_user_id: null,
        locked_at: null,
      },
    });
  },
  
  // Check if user can edit note
  async canUserEditNote(noteId: string, userId: string): Promise<boolean> {
    const note = await prisma.note.findUnique({
      where: { id: noteId },
    });
    
    if (!note) {
      return false;
    }
    
    // Check locking status
    if (note.locked_by_user_id && note.locked_by_user_id !== userId) {
      // Check if lock has expired (15 minutes)
      const lockExpirationTime = new Date(Date.now() - 15 * 60 * 1000);
      if (note.locked_at && note.locked_at > lockExpirationTime) {
        return false;
      }
    }
    
    // Creator can always edit
    if (note.created_by_user_id === userId) {
      return true;
    }
    
    // Check edit permission
    switch (note.can_edit) {
      case 'everyone':
        return true;
        
      case 'accessList':
        const editAccessList = note.edit_access_list?.split(',') || [];
        return editAccessList.includes(userId);
        
      case 'denyList':
        const denyList = note.edit_access_list?.split(',') || [];
        return !denyList.includes(userId);
        
      case 'noOne':
        return note.created_by_user_id === userId; // Only creator can edit
        
      default:
        return false;
    }
  }
};
```

#### 4.4.2. Audit Log Repository

```typescript
// lib/audit/repositories/audit-repository.ts
import { prisma } from '@/lib/db/prisma';
import { AuditEvent } from '@/lib/audit/types';

export const AuditRepository = {
  // Log an audit event
  async logEvent(event: {
    action: string;
    resourceType: string;
    resourceId: string;
    userId: string;
    organizationId: string;
    details?: Record<string, any>;
  }): Promise<void> {
    await prisma.audit.create({
      data: {
        action: event.action,
        resource_type: event.resourceType,
        resource_id: event.resourceId,
        user_id: event.userId,
        organization_id: event.organizationId,
        details: event.details,
      },
    });
  },
  
  // Get audit events for a resource
  async getResourceEvents(
    resourceType: string,
    resourceId: string,
    organizationId: string
  ): Promise<AuditEvent[]> {
    return prisma.audit.findMany({
      where: {
        resource_type: resourceType,
        resource_id: resourceId,
        organization_id: organizationId,
      },
      orderBy: {
        created_at: 'desc',
      },
    });
  },
  
  // Get recent audit events for an organization
  async getOrganizationEvents(
    organizationId: string,
    limit = 50
  ): Promise<AuditEvent[]> {
    return prisma.audit.findMany({
      where: {
        organization_id: organizationId,
      },
      orderBy: {
        created_at: 'desc',
      },
      take: limit,
    });
  },
};
```

## 5. Docker Setup

### 5.1. Docker Compose Configuration

```yaml
# docker-compose.yml
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
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U noteorbit"]
      interval: 5s
      timeout: 5s
      retries: 5

  pgadmin:
    image: dpage/pgadmin4
    restart: always
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@noteorbit.local
      PGADMIN_DEFAULT_PASSWORD: noteorbit
    ports:
      - "5050:80"
    depends_on:
      - db

volumes:
  db_data:
```

### 5.2. Database Initialization Script

```sql
-- init.sql (mounted in docker-compose)
-- Create extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Create indexes for performance
CREATE INDEX IF NOT EXISTS note_organization_id_idx ON note(organization_id);
CREATE INDEX IF NOT EXISTS note_created_by_user_id_idx ON note(created_by_user_id);
CREATE INDEX IF NOT EXISTS note_access_level_idx ON note(access_level);
CREATE INDEX IF NOT EXISTS note_organization_access_idx ON note(organization_id, access_level);
CREATE INDEX IF NOT EXISTS audit_resource_idx ON audit(resource_type, resource_id);
CREATE INDEX IF NOT EXISTS audit_organization_id_idx ON audit(organization_id);
CREATE INDEX IF NOT EXISTS audit_user_id_idx ON audit(user_id);
```

### 5.3. Environment Configuration

```env
# .env.example
# Database
DATABASE_URL=postgresql://noteorbit:noteorbit@localhost:5432/noteorbit_db

# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/auth/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/auth/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard
```