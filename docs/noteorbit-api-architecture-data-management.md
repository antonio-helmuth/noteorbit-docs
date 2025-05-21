# NoteOrbit API Architecture & Data Management

## 1. API Architecture

### 1.1 API Design Principles

NoteOrbit follows a RESTful API architecture with the following design principles:

1. **Resource-Oriented**: APIs are organized around resources (notes, organizations, users)
2. **Standard HTTP Methods**: Use appropriate HTTP methods (GET, POST, PATCH, DELETE)
3. **Consistent Response Formats**: Standardized response structures across all endpoints
4. **Proper Status Codes**: Meaningful HTTP status codes for different scenarios
5. **Versioning-Ready**: Structure supports future API versioning if needed
6. **Secure by Default**: Authentication and authorization for all endpoints
7. **Rate Limiting**: Protection against abuse and overuse
8. **Validation**: Thorough validation of all input data

### 1.2 Next.js API Routes Structure

The API is implemented using Next.js API Routes, organized by resource type:

```
/app/api
├── notes
│   ├── route.ts                # GET /api/notes (list), POST /api/notes (create)
│   ├── [noteId]
│   │   ├── route.ts            # GET, PATCH, DELETE /api/notes/[noteId]
│   │   ├── lock
│   │   │   └── route.ts        # POST, DELETE /api/notes/[noteId]/lock
│   │   └── access
│   │       └── route.ts        # PATCH /api/notes/[noteId]/access
├── org
│   ├── route.ts                # GET, PATCH /api/org
│   ├── members
│   │   └── route.ts            # GET, PATCH /api/org/members
│   └── invite
│       └── route.ts            # POST /api/org/invite
└── webhooks
    └── clerk
        └── route.ts            # POST /api/webhooks/clerk
```

### 1.3 API Response Format

All API responses follow a consistent format for predictability and ease of use:

#### Success Response (200/201)

```typescript
interface SuccessResponse<T> {
  data: T;               // Response data payload
  meta?: {               // Optional metadata 
    pagination?: {       // Pagination info if applicable
      page: number;
      perPage: number;
      total: number;
      totalPages: number;
    };
  };
}
```

Example:
```json
{
  "data": {
    "id": "note-123",
    "title": "Meeting Notes",
    "content": "# Meeting Agenda\n\n1. Introductions\n2. Project updates",
    "accessLevel": "shared",
    "canEdit": "accessList",
    "editAccessList": ["user-456", "user-789"],
    "viewAccessList": ["user-456", "user-789", "user-101"],
    "createdByUserId": "user-123",
    "organizationId": "org-123",
    "createdAt": "2025-03-15T14:30:00Z",
    "updatedAt": "2025-03-15T15:45:00Z"
  }
}
```

#### List Response (200)

```json
{
  "data": [
    {
      "id": "note-123",
      "title": "Meeting Notes",
      "content": "# Meeting Agenda\n\n1. Introductions\n2. Project updates",
      "accessLevel": "shared",
      "canEdit": "accessList",
      "editAccessList": ["user-456", "user-789"],
      "viewAccessList": ["user-456", "user-789", "user-101"],
      "createdByUserId": "user-123",
      "organizationId": "org-123",
      "createdAt": "2025-03-15T14:30:00Z",
      "updatedAt": "2025-03-15T15:45:00Z"
    },
    // More items...
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "perPage": 10,
      "total": 25,
      "totalPages": 3
    }
  }
}
```

#### Error Response (4xx/5xx)

```typescript
interface ErrorResponse {
  error: {
    code: string;           // Error code for client handling
    message: string;        // Human-readable error message
    details?: unknown;      // Optional detailed error information
  };
}
```

Example:
```json
{
  "error": {
    "code": "NOTE_NOT_FOUND",
    "message": "The requested note could not be found",
    "details": {
      "noteId": "note-999"
    }
  }
}
```

### 1.4 API Routes Implementation

#### 1.4.1 Notes List API

```typescript
// app/api/notes/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@clerk/nextjs/server';
import { NoteRepository } from '@/lib/notes/repositories/note-repository';
import { validateListNotesQuery } from '@/lib/notes/validators';
import { logger } from '@/lib/logger/server-logger';

export async function GET(req: NextRequest) {
  try {
    const { userId, orgId } = auth();
    
    if (!userId || !orgId) {
      return NextResponse.json(
        {
          error: {
            code: 'UNAUTHORIZED',
            message: 'You must be logged in to access this resource',
          }
        },
        { status: 401 }
      );
    }
    
    const url = new URL(req.url);
    const params = {
      page: parseInt(url.searchParams.get('page') || '1'),
      perPage: parseInt(url.searchParams.get('perPage') || '10'),
      accessLevel: url.searchParams.get('accessLevel') || undefined,
      createdBy: url.searchParams.get('createdBy') || undefined,
      search: url.searchParams.get('search') || undefined,
    };
    
    const validation = validateListNotesQuery(params);
    if (!validation.success) {
      return NextResponse.json(
        {
          error: {
            code: 'INVALID_QUERY',
            message: 'Invalid query parameters',
            details: validation.errors,
          }
        },
        { status: 400 }
      );
    }
    
    const { notes, total } = await NoteRepository.getOrganizationNotes(
      orgId,
      userId,
      {
        page: params.page,
        perPage: params.perPage,
        accessLevel: params.accessLevel,
        createdBy: params.createdBy,
        search: params.search,
      }
    );
    
    logger.info('Notes list fetched', {
      userId,
      organizationId: orgId,
      filters: {
        accessLevel: params.accessLevel,
        createdBy: params.createdBy,
        search: params.search?.substring(0, 20),
      },
      results: notes.length,
      total,
    });
    
    return NextResponse.json({
      data: notes,
      meta: {
        pagination: {
          page: params.page,
          perPage: params.perPage,
          total,
          totalPages: Math.ceil(total / params.perPage),
        }
      }
    });
  } catch (error) {
    logger.error('Failed to fetch notes list', { error });
    
    return NextResponse.json(
      {
        error: {
          code: 'SERVER_ERROR',
          message: 'An unexpected error occurred',
        }
      },
      { status: 500 }
    );
  }
}
```

#### 1.4.2 Note Detail API

```typescript
// app/api/notes/[noteId]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@clerk/nextjs/server';
import { NoteRepository } from '@/lib/notes/repositories/note-repository';
import { logger } from '@/lib/logger/server-logger';

export async function GET(
  req: NextRequest,
  { params }: { params: { noteId: string } }
) {
  try {
    const { userId, orgId } = auth();
    
    if (!userId || !orgId) {
      return NextResponse.json(
        {
          error: {
            code: 'UNAUTHORIZED',
            message: 'You must be logged in to access this resource',
          }
        },
        { status: 401 }
      );
    }
    
    const noteId = params.noteId;
    
    // Get the note
    const note = await NoteRepository.getNoteById(noteId, userId, orgId);
    
    if (!note) {
      return NextResponse.json(
        {
          error: {
            code: 'NOTE_NOT_FOUND',
            message: 'The requested note could not be found',
            details: { noteId },
          }
        },
        { status: 404 }
      );
    }
    
    logger.info('Note accessed', {
      noteId,
      userId,
      organizationId: orgId,
    });
    
    return NextResponse.json({ data: note });
  } catch (error) {
    logger.error('Failed to get note', { error });
    
    return NextResponse.json(
      {
        error: {
          code: 'SERVER_ERROR',
          message: 'An unexpected error occurred',
        }
      },
      { status: 500 }
    );
  }
}
```

### 1.5 API Error Codes

NoteOrbit uses standardized error codes to make error handling consistent:

| Code | Description | HTTP Status |
|------|-------------|-------------|
| `UNAUTHORIZED` | User is not authenticated | 401 |
| `FORBIDDEN` | User doesn't have permission | 403 |
| `NOT_FOUND` | Resource not found | 404 |
| `NOTE_NOT_FOUND` | Specific note not found | 404 |
| `VALIDATION_ERROR` | Invalid input data | 400 |
| `INVALID_QUERY` | Invalid query parameters | 400 |
| `CONFLICT` | Resource conflict | 409 |
| `NOTE_LOCKED` | Note is locked by another user | 423 |
| `RATE_LIMITED` | Too many requests | 429 |
| `SERVER_ERROR` | Unexpected server error | 500 |

### 1.6 API Validation

All API inputs are validated using Zod schemas:

```typescript
// lib/notes/validators.ts
import { z } from 'zod';
import { NoteAccessLevel, NoteEditPermission } from './types';

export const createNoteSchema = z.object({
  title: z.string().min(1).max(100),
  content: z.string(),
  organizationId: z.string().min(1),
  accessLevel: z.enum(['public', 'shared', 'private']),
  canEdit: z.enum(['everyone', 'accessList', 'denyList', 'noOne']),
  editAccessList: z.array(z.string()).optional(),
  viewAccessList: z.array(z.string()).optional(),
});

export const updateNoteSchema = z.object({
  title: z.string().min(1).max(100).optional(),
  content: z.string().optional(),
  accessLevel: z.enum(['public', 'shared', 'private']).optional(),
  canEdit: z.enum(['everyone', 'accessList', 'denyList', 'noOne']).optional(),
  editAccessList: z.array(z.string()).optional(),
  viewAccessList: z.array(z.string()).optional(),
});

export const listNotesQuerySchema = z.object({
  page: z.number().int().positive().default(1),
  perPage: z.number().int().positive().max(100).default(10),
  accessLevel: z.enum(['public', 'shared', 'private']).optional(),
  createdBy: z.string().optional(),
  search: z.string().optional(),
});

export function validateCreateNoteInput(data: unknown) {
  return createNoteSchema.safeParse(data);
}

export function validateUpdateNoteInput(data: unknown) {
  return updateNoteSchema.safeParse(data);
}

export function validateListNotesQuery(data: unknown) {
  return listNotesQuerySchema.safeParse(data);
}
```

### 1.7 API Security Considerations

1. **Authentication**: All API routes are protected by Clerk authentication middleware
2. **Authorization**: Permission checks for all data access operations
3. **CSRF Protection**: Next.js built-in CSRF protection
4. **Rate Limiting**: Limits on API requests per user/organization
5. **Input Validation**: Thorough validation for all inputs
6. **Error Handling**: Secure error responses that don't leak sensitive information

### 1.8 API Integration with React Query

The frontend connects to API endpoints using React Query for caching and state management:

```typescript
// lib/notes/hooks/use-notes.ts
import { useQuery } from '@tanstack/react-query';
import { getNotes } from '../api/get-notes';

interface UseNotesOptions {
  page?: number;
  perPage?: number;
  accessLevel?: string;
  createdBy?: string;
  search?: string;
}

export function useNotes(options: UseNotesOptions = {}) {
  return useQuery({
    queryKey: ['notes', options],
    queryFn: () => getNotes(options),
    keepPreviousData: true,
  });
}
```

```typescript
// lib/notes/api/get-notes.ts
interface GetNotesOptions {
  page?: number;
  perPage?: number;
  accessLevel?: string;
  createdBy?: string;
  search?: string;
}

export async function getNotes(options: GetNotesOptions = {}) {
  const params = new URLSearchParams();
  
  if (options.page) params.set('page', options.page.toString());
  if (options.perPage) params.set('perPage', options.perPage.toString());
  if (options.accessLevel) params.set('accessLevel', options.accessLevel);
  if (options.createdBy) params.set('createdBy', options.createdBy);
  if (options.search) params.set('search', options.search);
  
  const queryString = params.toString();
  const url = `/api/notes${queryString ? `?${queryString}` : ''}`;
  
  const response = await fetch(url);
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Failed to fetch notes');
  }
  
  return response.json();
}
```

## 2. State Management

### 2.1 React Query Implementation

NoteOrbit uses TanStack Query (React Query) for server state management, providing:

1. **Caching**: In-memory caching of API responses
2. **Automatic Refetching**: Background refetching of stale data
3. **Optimistic Updates**: Immediate UI updates with background synchronization
4. **Mutation Handling**: Streamlined API mutation operations
5. **Pagination**: Built-in support for paginated data
6. **Error Handling**: Consistent error handling across API calls
7. **Loading States**: Easy access to loading states for UI feedback

### 2.2 QueryClient Configuration

```typescript
// lib/query-client.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      cacheTime: 1000 * 60 * 30, // 30 minutes
      refetchOnWindowFocus: true,
      refetchOnMount: true,
      refetchOnReconnect: true,
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    },
    mutations: {
      retry: 2,
      retryDelay: 1000,
    },
  },
});
```

### 2.3 Provider Setup

```tsx
// app/providers.tsx
'use client';

import { PropsWithChildren } from 'react';
import { QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { queryClient } from '@/lib/query-client';
import { ClerkProviderWithTheme } from '@/lib/auth/clerk-client';
import { ToastProvider } from '@/components/ui/toast';

export function Providers({ children }: PropsWithChildren) {
  return (
    <QueryClientProvider client={queryClient}>
      <ClerkProviderWithTheme>
        <ToastProvider>
          {children}
        </ToastProvider>
      </ClerkProviderWithTheme>
      {process.env.NODE_ENV !== 'production' && <ReactQueryDevtools />}
    </QueryClientProvider>
  );
}
```

### 2.4 Query Hooks

#### 2.4.1 Notes List Query

```typescript
// lib/notes/hooks/use-notes.ts
import { useQuery } from '@tanstack/react-query';
import { getNotes } from '../api/get-notes';

interface UseNotesOptions {
  page?: number;
  perPage?: number;
  accessLevel?: string;
  createdBy?: string;
  search?: string;
  enabled?: boolean;
}

export function useNotes({
  page = 1,
  perPage = 10,
  accessLevel,
  createdBy,
  search,
  enabled = true,
}: UseNotesOptions = {}) {
  return useQuery({
    queryKey: ['notes', { page, perPage, accessLevel, createdBy, search }],
    queryFn: () => getNotes({ page, perPage, accessLevel, createdBy, search }),
    keepPreviousData: true,
    enabled,
  });
}
```

#### 2.4.2 Single Note Query

```typescript
// lib/notes/hooks/use-note.ts
import { useQuery } from '@tanstack/react-query';
import { getNoteById } from '../api/get-note-by-id';

export function useNote(noteId: string, options = {}) {
  return useQuery({
    queryKey: ['note', noteId],
    queryFn: () => getNoteById(noteId),
    enabled: !!noteId,
    ...options,
  });
}
```

#### 2.4.3 Create Note Mutation

```typescript
// lib/notes/hooks/use-create-note.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { createNote } from '../api/create-note';
import { toast } from '@/components/ui/toast';
import { logger } from '@/lib/logger/client-logger';

export function useCreateNote() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: createNote,
    onSuccess: (data) => {
      queryClient.invalidateQueries({ queryKey: ['notes'] });
      logger.info('Note created successfully', { noteId: data.data.id });
      toast({
        title: 'Success',
        description: 'Note created successfully',
        variant: 'success',
      });
    },
    onError: (error) => {
      logger.error('Failed to create note', { error });
      toast({
        title: 'Error',
        description: error.message || 'Failed to create note',
        variant: 'error',
      });
    },
  });
}
```

#### 2.4.4 Update Note Mutation

```typescript
// lib/notes/hooks/use-update-note.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { updateNote } from '../api/update-note';
import { toast } from '@/components/ui/toast';
import { logger } from '@/lib/logger/client-logger';

export function useUpdateNote() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: updateNote,
    onSuccess: (data) => {
      const noteId = data.data.id;
      
      // Update the note in the cache
      queryClient.setQueryData(['note', noteId], data);
      
      // Invalidate affected queries
      queryClient.invalidateQueries({ queryKey: ['notes'] });
      
      logger.info('Note updated successfully', { noteId });
      
      toast({
        title: 'Success',
        description: 'Note updated successfully',
        variant: 'success',
      });
    },
    onError: (error, variables) => {
      logger.error('Failed to update note', { error, noteId: variables.id });
      
      toast({
        title: 'Error',
        description: error.message || 'Failed to update note',
        variant: 'error',
      });
    },
  });
}
```

### 2.5 Optimistic Updates

For a responsive user experience, optimistic updates are implemented for key mutations:

```typescript
// lib/notes/hooks/use-update-note-with-optimistic-update.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { updateNote } from '../api/update-note';
import { toast } from '@/components/ui/toast';
import { logger } from '@/lib/logger/client-logger';

export function useUpdateNoteWithOptimisticUpdate() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: updateNote,
    onMutate: async (variables) => {
      // Cancel any outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['note', variables.id] });
      
      // Snapshot the previous value
      const previousNote = queryClient.getQueryData(['note', variables.id]);
      
      // Optimistically update to the new value
      queryClient.setQueryData(['note', variables.id], (old) => ({
        ...old,
        data: {
          ...old.data,
          ...variables,
          // Don't include access lists in optimistic update if they're not being changed
          editAccessList: variables.editAccessList || old.data.editAccessList,
          viewAccessList: variables.viewAccessList || old.data.viewAccessList,
        },
      }));
      
      // Return a context object with the snapshotted value
      return { previousNote };
    },
    onError: (err, variables, context) => {
      // If the mutation fails, use the context returned from onMutate to roll back
      if (context?.previousNote) {
        queryClient.setQueryData(['note', variables.id], context.previousNote);
      }
      
      logger.error('Failed to update note', { error: err, noteId: variables.id });
      
      toast({
        title: 'Error',
        description: err.message || 'Failed to update note',
        variant: 'error',
      });
    },
    onSettled: (data, error, variables) => {
      // Always refetch after error or success to ensure cache is in sync with server
      queryClient.invalidateQueries({ queryKey: ['note', variables.id] });
      queryClient.invalidateQueries({ queryKey: ['notes'] });
    },
    onSuccess: (data, variables) => {
      logger.info('Note updated successfully', { noteId: variables.id });
      
      toast({
        title: 'Success',
        description: 'Note updated successfully',
        variant: 'success',
      });
    },
  });
}
```

### 2.6 Infinite Query for Large Data Sets

For efficiently loading large datasets like notes history:

```typescript
// lib/notes/hooks/use-note-history.ts
import { useInfiniteQuery } from '@tanstack/react-query';
import { getNoteHistory } from '../api/get-note-history';

export function useNoteHistory(noteId: string) {
  return useInfiniteQuery({
    queryKey: ['note-history', noteId],
    queryFn: ({ pageParam = 1 }) => getNoteHistory(noteId, { page: pageParam }),
    getNextPageParam: (lastPage) => {
      const { page, totalPages } = lastPage.meta.pagination;
      return page < totalPages ? page + 1 : undefined;
    },
    enabled: !!noteId,
  });
}
```

### 2.7 Local State Management

For UI-specific state that doesn't need to be synchronized with the server, React's built-in state management is used:

```typescript
// Example of local component state
const [isFilterPanelOpen, setIsFilterPanelOpen] = useState(false);
const [selectedView, setSelectedView] = useState<'grid' | 'list'>('grid');
```

For more complex local state, useReducer is used:

```typescript
// lib/notes/hooks/use-filter-state.ts
import { useReducer } from 'react';

type FilterState = {
  search: string;
  accessLevel: string | null;
  createdBy: string | null;
  sortBy: 'updatedAt' | 'createdAt' | 'title';
  sortDirection: 'asc' | 'desc';
};

type FilterAction =
  | { type: 'SET_SEARCH'; payload: string }
  | { type: 'SET_ACCESS_LEVEL'; payload: string | null }
  | { type: 'SET_CREATED_BY'; payload: string | null }
  | { type: 'SET_SORT'; payload: { sortBy: 'updatedAt' | 'createdAt' | 'title'; sortDirection: 'asc' | 'desc' } }
  | { type: 'RESET_FILTERS' };

const initialState: FilterState = {
  search: '',
  accessLevel: null,
  createdBy: null,
  sortBy: 'updatedAt',
  sortDirection: 'desc',
};

function filterReducer(state: FilterState, action: FilterAction): FilterState {
  switch (action.type) {
    case 'SET_SEARCH':
      return { ...state, search: action.payload };
    case 'SET_ACCESS_LEVEL':
      return { ...state, accessLevel: action.payload };
    case 'SET_CREATED_BY':
      return { ...state, createdBy: action.payload };
    case 'SET_SORT':
      return { ...state, ...action.payload };
    case 'RESET_FILTERS':
      return initialState;
    default:
      return state;
  }
}

export function useFilterState() {
  const [state, dispatch] = useReducer(filterReducer, initialState);
  
  return {
    filters: state,
    setSearch: (search: string) => dispatch({ type: 'SET_SEARCH', payload: search }),
    setAccessLevel: (level: string | null) => dispatch({ type: 'SET_ACCESS_LEVEL', payload: level }),
    setCreatedBy: (userId: string | null) => dispatch({ type: 'SET_CREATED_BY', payload: userId }),
    setSort: (sortBy: 'updatedAt' | 'createdAt' | 'title', sortDirection: 'asc' | 'desc') => 
      dispatch({ type: 'SET_SORT', payload: { sortBy, sortDirection } }),
    resetFilters: () => dispatch({ type: 'RESET_FILTERS' }),
  };
}
```

## 3. Data Modeling & Database Schema

### 3.1 Database Schema Design

NoteOrbit's data model is centered around notes, organizations, and users, with Clerk handling identity management.

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
  content           String    @db.Text
  created_by_user_id String
  organization_id   String
  created_at        DateTime  @default(now())
  updated_at        DateTime  @updatedAt
  
  // Access control fields
  access_level      String    // 'public' | 'shared' | 'private'
  can_edit          String    // 'everyone' | 'accessList' | 'denyList' | 'noOne'
  
  // Stored as comma-separated lists of IDs
  edit_access_list  String?   
  view_access_list  String?
  
  // Lock mechanism fields
  locked_by_user_id String?
  locked_at         DateTime?
  
  // Optimized queries
  @@index([organization_id])
  @@index([created_by_user_id])
  @@index([organization_id, access_level])
  @@index([locked_by_user_id, locked_at])
}

model NoteVersion {
  id               String   @id @default(uuid())
  note_id          String
  content          String   @db.Text
  title            String
  created_by_user_id String
  created_at       DateTime @default(now())
  
  @@index([note_id])
  @@index([created_at])
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
  @@index([created_at])
}
```

### 3.2 Type Definitions

TypeScript interfaces that mirror the database schema, with proper naming conventions and transformations:

```typescript
// lib/notes/types.ts
export type NoteAccessLevel = 'public' | 'shared' | 'private';
export type NoteEditPermission = 'everyone' | 'accessList' | 'denyList' | 'noOne';

export interface Note {
  id: string;
  title: string;
  content: string;
  createdByUserId: string;
  organizationId: string;
  createdAt: string;
  updatedAt: string;
  accessLevel: NoteAccessLevel;
  canEdit: NoteEditPermission;
  editAccessList: string[];
  viewAccessList: string[];
  lockedByUserId: string | null;
  lockedAt: string | null;
}

export interface NoteFormData {
  title: string;
  content: string;
  accessLevel: NoteAccessLevel;
  canEdit: NoteEditPermission;
  editAccessList: string[];
  viewAccessList: string[];
}

export interface NoteListItem {
  id: string;
  title: string;
  content: string;
  createdByUserId: string;
  accessLevel: NoteAccessLevel;
  updatedAt: string;
  lockedByUserId: string | null;
}

export interface NoteVersion {
  id: string;
  noteId: string;
  title: string;
  content: string;
  createdByUserId: string;
  createdAt: string;
}
```

### 3.3 Data Transformation

Converting between database models and API models:

```typescript
// lib/notes/mappers/note-mapper.ts
import { Note as PrismaNote } from '@prisma/client';
import { Note, NoteListItem } from '../types';

export function mapPrismaNoteToNote(prismaNote: PrismaNote): Note {
  return {
    id: prismaNote.id,
    title: prismaNote.title,
    content: prismaNote.content,
    createdByUserId: prismaNote.created_by_user_id,
    organizationId: prismaNote.organization_id,
    createdAt: prismaNote.created_at.toISOString(),
    updatedAt: prismaNote.updated_at.toISOString(),
    accessLevel: prismaNote.access_level as Note['accessLevel'],
    canEdit: prismaNote.can_edit as Note['canEdit'],
    editAccessList: prismaNote.edit_access_list?.split(',').filter(Boolean) || [],
    viewAccessList: prismaNote.view_access_list?.split(',').filter(Boolean) || [],
    lockedByUserId: prismaNote.locked_by_user_id || null,
    lockedAt: prismaNote.locked_at?.toISOString() || null,
  };
}

export function mapPrismaNoteToNoteListItem(prismaNote: PrismaNote): NoteListItem {
  return {
    id: prismaNote.id,
    title: prismaNote.title,
    content: prismaNote.content.substring(0, 200), // Truncate content for list view
    createdByUserId: prismaNote.created_by_user_id,
    accessLevel: prismaNote.access_level as NoteListItem['accessLevel'],
    updatedAt: prismaNote.updated_at.toISOString(),
    lockedByUserId: prismaNote.locked_by_user_id || null,
  };
}
```

### 3.4 Database Migrations

Prisma migrations are used to manage database schema changes:

```bash
# Create a new migration
npx prisma migrate dev --name add_note_versions

# Apply migrations in production
npx prisma migrate deploy
```

### 3.5 Database Seeding

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

## 4. Markdown Editor Integration

### 4.1 Editor Component Integration

NoteOrbit uses react-markdown-editor-lite for markdown editing with a clean configuration:

```tsx
// lib/notes/components/markdown-editor.tsx
'use client';

import { useState, useEffect, useRef } from 'react';
import MarkdownEditor from 'react-markdown-editor-lite';
import ReactMarkdown from 'react-markdown';
import remarkGfm from 'remark-gfm';
import rehypeRaw from 'rehype-raw';
import rehypeSanitize from 'rehype-sanitize';
import 'react-markdown-editor-lite/lib/index.css';

interface MarkdownEditorProps {
  initialValue: string;
  onChange: (text: string) => void;
  placeholder?: string;
  readOnly?: boolean;
  height?: number | string;
  onFocus?: () => void;
  onBlur?: () => void;
}

export function NoteMarkdownEditor({
  initialValue,
  onChange,
  placeholder = 'Start writing...',
  readOnly = false,
  height = 400,
  onFocus,
  onBlur,
}: MarkdownEditorProps) {
  const [value, setValue] = useState(initialValue);
  const editorRef = useRef<any>(null);
  
  // Sync with initialValue when it changes
  useEffect(() => {
    setValue(initialValue);
  }, [initialValue]);
  
  // Handle editor change
  const handleEditorChange = ({ text }: { text: string }) => {
    setValue(text);
    onChange(text);
  };
  
  // Custom markdown renderer
  const renderMarkdown = (text: string) => {
    return (
      <ReactMarkdown
        remarkPlugins={[remarkGfm]}
        rehypePlugins={[rehypeRaw, rehypeSanitize]}
        className="prose prose-indigo max-w-none"
      >
        {text}
      </ReactMarkdown>
    );
  };
  
  return (
    <MarkdownEditor
      ref={editorRef}
      value={value}
      onChange={handleEditorChange}
      style={{ height }}
      placeholder={placeholder}
      readOnly={readOnly}
      renderHTML={(text) => Promise.resolve(renderMarkdown(text))}
      onFocus={onFocus}
      onBlur={onBlur}
      config={{
        view: {
          menu: true,
          md: true,
          html: true,
        },
        canView: {
          menu: true,
          md: true,
          html: true,
          fullScreen: true,
          hideMenu: true,
        },
        markdownClass: 'markdown-body',
        htmlClass: 'markdown-html',
        syncScrollMode: ['rightFollowLeft'],
        allowPasteImage: true,
        imageAccept: '.jpg,.jpeg,.png,.gif',
        // Only include basic toolbar items
        toolbar: [
          'bold', 'italic', 'strike-through', 'header', 'list-ul', 'list-ol',
          'link', 'table', 'quote', 'code', 'divider',
        ],
      }}
    />
  );
}
```

### 4.2 Markdown Renderer Component

For viewing markdown content without the editor interface:

```tsx
// lib/notes/components/markdown-renderer.tsx
'use client';

import { useMemo } from 'react';
import ReactMarkdown from 'react-markdown';
import remarkGfm from 'remark-gfm';
import rehypeRaw from 'rehype-raw';
import rehypeSanitize from 'rehype-sanitize';

interface MarkdownRendererProps {
  content: string;
  className?: string;
}

export function MarkdownRenderer({ content, className = '' }: MarkdownRendererProps) {
  // Only re-render when content changes
  const renderedContent = useMemo(() => {
    return (
      <ReactMarkdown
        remarkPlugins={[remarkGfm]}
        rehypePlugins={[rehypeRaw, rehypeSanitize]}
        className={`prose prose-indigo max-w-none ${className}`}
      >
        {content}
      </ReactMarkdown>
    );
  }, [content, className]);
  
  return renderedContent;
}
```

### 4.3 Editor Configuration

Custom configuration to keep the editor minimal and focused:

```typescript
// lib/notes/config/markdown-editor-config.ts
import { markdownToDraft, draftToMarkdown } from 'markdown-draft-js';

export const markdownEditorConfig = {
  // Only include essential toolbar items
  toolbar: [
    'bold', 'italic', 'strike-through', 
    'header', 'list-ul', 'list-ol',
    'link', 'table', 'quote', 'code',
    'divider',
  ],
  
  // Disable all advanced plugins
  pluginsEnabled: false,
  
  // Styling 
  view: {
    menu: true,
    md: true,
    html: true,
  },
  
  // Allowed views
  canView: {
    menu: true,
    md: true,
    html: true,
    fullScreen: true,
    hideMenu: true,
  },
  
  // Sync scrolling
  syncScrollMode: ['rightFollowLeft'],
  
  // Markdown syntax customization
  shortcuts: {
    bold: 'Ctrl-B',
    italic: 'Ctrl-I',
    heading1: 'Ctrl-1',
    heading2: 'Ctrl-2',
    heading3: 'Ctrl-3',
    heading4: 'Ctrl-4',
    heading5: 'Ctrl-5',
    heading6: 'Ctrl-6',
  },
  
  // Custom parser setup for converting between Markdown and HTML
  markdownParser: {
    blockStyles: {
      'header-one': '# ',
      'header-two': '## ',
      'header-three': '### ',
      'header-four': '#### ',
      'header-five': '##### ',
      'header-six': '###### ',
      'unordered-list-item': '- ',
      'ordered-list-item': '1. ',
    },
    blockEntities: {
      codeBlock: {
        start: '```\n',
        end: '\n```',
        type: 'code-block',
      },
    },
    inlineStyles: {
      BOLD: {
        start: '**',
        end: '**',
      },
      ITALIC: {
        start: '_',
        end: '_',
      },
      CODE: {
        start: '`',
        end: '`',
      },
      STRIKETHROUGH: {
        start: '~~',
        end: '~~',
      },
    },
  },
};
```

### 4.4 Editor Utilities

Helper functions for common markdown operations:

```typescript
// lib/notes/utils/markdown-utils.ts
export function extractTitle(markdown: string): string {
  // Try to extract h1 heading as title
  const h1Match = markdown.match(/^#\s+(.+)$/m);
  if (h1Match) {
    return h1Match[1].trim();
  }
  
  // Fall back to first line if no h1
  const firstLine = markdown.split('\n')[0].trim();
  if (firstLine) {
    // Remove markdown syntax from the line
    return firstLine.replace(/^[#*_~`]+\s*/, '').replace(/[*_~`]+$/, '');
  }
  
  return 'Untitled Note';
}

export function generateTitleFromContent(content: string): string {
  // Extract first line and clean it
  const firstLine = content.split('\n')[0].trim();
  
  // Remove markdown syntax
  const cleanTitle = firstLine
    .replace(/^[#*_~`]+\s*/, '')  // Remove leading markdown
    .replace(/[*_~`]+$/, '')      // Remove trailing markdown
    .substring(0, 50);            // Limit length
    
  return cleanTitle || 'Untitled Note';
}

export function generateTableOfContents(markdown: string): { level: number; text: string; slug: string }[] {
  const headingRegex = /^(#{1,6})\s+(.+)$/gm;
  const toc = [];
  let match;
  
  while ((match = headingRegex.exec(markdown)) !== null) {
    const level = match[1].length;
    const text = match[2].trim();
    const slug = text
      .toLowerCase()
      .replace(/[^\w\s-]/g, '')  // Remove non-word chars
      .replace(/\s+/g, '-')      // Replace spaces with hyphens
      .replace(/-+/g, '-');      // Replace multiple hyphens with single
      
    toc.push({ level, text, slug });
  }
  
  return toc;
}

export function sanitizeMarkdown(markdown: string): string {
  // Basic sanitization to prevent XSS
  return markdown
    .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
    .replace(/javascript:/gi, 'removed:')
    .replace(/on\w+=/gi, 'removed=');
}
```

### 4.5 Editor Plugins (Optional)

Custom plugins to extend the editor functionality:

```typescript
// lib/notes/plugins/auto-save-plugin.ts
import { PluginProps } from 'react-markdown-editor-lite';

export function createAutoSavePlugin(
  saveCallback: (content: string) => void,
  debounceTime = 1000
) {
  let timer: NodeJS.Timeout | null = null;
  
  return {
    name: 'auto-save',
    onChange: (state: { text: string }) => {
      if (timer) {
        clearTimeout(timer);
      }
      
      timer = setTimeout(() => {
        saveCallback(state.text);
      }, debounceTime);
    },
    onDestroy: () => {
      if (timer) {
        clearTimeout(timer);
      }
    },
  };
}
```

## 5. Logging and Monitoring

### 5.1 Logging Strategy

NoteOrbit implements a comprehensive logging strategy to track user actions, system events, and errors:

```typescript
// lib/logger/server-logger.ts
import pino from 'pino';

// Configure log levels and formatting
const pinoConfig = {
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label: string) => {
      return { level: label };
    },
  },
  // Redact sensitive information
  redact: {
    paths: ['password', 'email', 'token', 'authorization'],
    censor: '[REDACTED]',
  },
  // Custom timestamp
  timestamp: () => `,"timestamp":"${new Date().toISOString()}"`,
};

// Create logger instance
export const logger = pino(pinoConfig);

// Add specific context types
export const createContextLogger = (context: Record<string, any>) => {
  return logger.child(context);
};
```

```typescript
// lib/logger/client-logger.ts
type LogLevel = 'info' | 'warn' | 'error';

interface LogOptions {
  level: LogLevel;
  message: string;
  data?: Record<string, any>;
  timestamp?: string;
}

// Client-side logger that sends logs to the server
class ClientLogger {
  private enabled: boolean;
  private endpoint: string;
  private queue: LogOptions[] = [];
  private maxQueueSize: number = 10;
  private flushInterval: number = 10000; // 10 seconds
  private timer: NodeJS.Timeout | null = null;
  
  constructor() {
    this.enabled = process.env.NODE_ENV !== 'test';
    this.endpoint = '/api/logs';
    
    if (typeof window !== 'undefined') {
      // Set up flush interval
      this.timer = setInterval(() => this.flush(), this.flushInterval);
      
      // Flush logs before page unload
      window.addEventListener('beforeunload', () => this.flush());
    }
  }
  
  private async flush() {
    if (!this.enabled || this.queue.length === 0) return;
    
    try {
      const logs = [...this.queue];
      this.queue = [];
      
      await fetch(this.endpoint, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ logs }),
        // Use keepalive to ensure logs are sent even during page navigation
        keepalive: true,
      });
    } catch (error) {
      console.error('Failed to send logs to server:', error);
    }
  }
  
  private log(level: LogLevel, message: string, data?: Record<string, any>) {
    if (!this.enabled) return;
    
    const logEntry: LogOptions = {
      level,
      message,
      data,
      timestamp: new Date().toISOString(),
    };
    
    // Add to queue
    this.queue.push(logEntry);
    
    // Flush if queue is full
    if (this.queue.length >= this.maxQueueSize) {
      this.flush();
    }
    
    // Also log to console in development
    if (process.env.NODE_ENV === 'development') {
      console[level](message, data);
    }
  }
  
  info(message: string, data?: Record<string, any>) {
    this.log('info', message, data);
  }
  
  warn(message: string, data?: Record<string, any>) {
    this.log('warn', message, data);
  }
  
  error(message: string, data?: Record<string, any>) {
    this.log('error', message, data);
  }
}

export const logger = new ClientLogger();
```

### 5.2 Audit Logging

Record important user actions for accountability and troubleshooting:

```typescript
// lib/audit/service.ts
import { prisma } from '@/lib/db/prisma';
import { logger } from '@/lib/logger/server-logger';

export interface AuditLogEntry {
  action: string;
  resourceType: string;
  resourceId: string;
  userId: string;
  organizationId: string;
  details?: Record<string, any>;
}

export class AuditService {
  static async log(entry: AuditLogEntry): Promise<void> {
    try {
      await prisma.audit.create({
        data: {
          action: entry.action,
          resource_type: entry.resourceType,
          resource_id: entry.resourceId,
          user_id: entry.userId,
          organization_id: entry.organizationId,
          details: entry.details ? entry.details : undefined,
        },
      });
      
      logger.info('Audit log entry created', {
        action: entry.action,
        resourceType: entry.resourceType,
        resourceId: entry.resourceId,
        userId: entry.userId,
        organizationId: entry.organizationId,
      });
    } catch (error) {
      logger.error('Failed to create audit log entry', { error, entry });
    }
  }
  
  static async getResourceHistory(
    resourceType: string,
    resourceId: string,
    organizationId: string,
    options: { limit?: number; offset?: number } = {}
  ) {
    const { limit = 50, offset = 0 } = options;
    
    try {
      const logs = await prisma.audit.findMany({
        where: {
          resource_type: resourceType,
          resource_id: resourceId,
          organization_id: organizationId,
        },
        orderBy: {
          created_at: 'desc',
        },
        take: limit,
        skip: offset,
      });
      
      const count = await prisma.audit.count({
        where: {
          resource_type: resourceType,
          resource_id: resourceId,
          organization_id: organizationId,
        },
      });
      
      return {
        logs,
        count,
        hasMore: count > offset + limit,
      };
    } catch (error) {
      logger.error('Failed to get resource history', {
        error,
        resourceType,
        resourceId,
        organizationId,
      });
      
      throw error;
    }
  }
  
  static async getUserActivity(
    userId: string,
    organizationId: string,
    options: { limit?: number; offset?: number } = {}
  ) {
    const { limit = 50, offset = 0 } = options;
    
    try {
      const logs = await prisma.audit.findMany({
        where: {
          user_id: userId,
          organization_id: organizationId,
        },
        orderBy: {
          created_at: 'desc',
        },
        take: limit,
        skip: offset,
      });
      
      const count = await prisma.audit.count({
        where: {
          user_id: userId,
          organization_id: organizationId,
        },
      });
      
      return {
        logs,
        count,
        hasMore: count > offset + limit,
      };
    } catch (error) {
      logger.error('Failed to get user activity', {
        error,
        userId,
        organizationId,
      });
      
      throw error;
    }
  }
}
```

### 5.3 Usage Analytics

Track application usage for understanding user behavior and improving the product:

```typescript
// lib/analytics/client.ts
interface AnalyticsEvent {
  name: string;
  properties?: Record<string, any>;
  timestamp?: number;
}

export class Analytics {
  private static instance: Analytics;
  private initialized: boolean = false;
  private userId: string | null = null;
  private organizationId: string | null = null;
  
  private constructor() {
    // Private constructor for singleton
  }
  
  public static getInstance(): Analytics {
    if (!Analytics.instance) {
      Analytics.instance = new Analytics();
    }
    
    return Analytics.instance;
  }
  
  public init(userId: string, organizationId: string): void {
    this.userId = userId;
    this.organizationId = organizationId;
    this.initialized = true;
    
    // Track session start
    this.track('session_start', {
      referrer: document.referrer,
      url: window.location.href,
    });
  }
  
  public track(eventName: string, properties: Record<string, any> = {}): void {
    if (!this.initialized) {
      console.warn('Analytics not initialized');
      return;
    }
    
    const event: AnalyticsEvent = {
      name: eventName,
      properties: {
        ...properties,
        userId: this.userId,
        organizationId: this.organizationId,
        url: window.location.href,
      },
      timestamp: Date.now(),
    };
    
    this.sendEvent(event);
  }
  
  private async sendEvent(event: AnalyticsEvent): Promise<void> {
    try {
      await fetch('/api/analytics/events', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(event),
        // Use keepalive to ensure events are sent even during page navigation
        keepalive: true,
      });
    } catch (error) {
      console.error('Failed to send analytics event:', error);
    }
  }
  
  // Predefined events for consistency
  public pageView(pageName: string, properties: Record<string, any> = {}): void {
    this.track('page_view', { pageName, ...properties });
  }
  
  public noteCreated(noteId: string, properties: Record<string, any> = {}): void {
    this.track('note_created', { noteId, ...properties });
  }
  
  public noteEdited(noteId: string, properties: Record<string, any> = {}): void {
    this.track('note_edited', { noteId, ...properties });
  }
  
  public noteViewed(noteId: string, properties: Record<string, any> = {}): void {
    this.track('note_viewed', { noteId, ...properties });
  }
  
  public noteDeleted(noteId: string, properties: Record<string, any> = {}): void {
    this.track('note_deleted', { noteId, ...properties });
  }
  
  public noteShared(noteId: string, properties: Record<string, any> = {}): void {
    this.track('note_shared', { noteId, ...properties });
  }
  
  public searchPerformed(query: string, results: number, properties: Record<string, any> = {}): void {
    this.track('search_performed', { query, results, ...properties });
  }
}

export const analytics = Analytics.getInstance();
```

### 5.4 Error Monitoring

Comprehensive error tracking and reporting:

```typescript
// lib/errors/error-monitoring.ts
import * as Sentry from '@sentry/nextjs';

// Initialize Sentry in production only
if (process.env.NODE_ENV === 'production') {
  Sentry.init({
    dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
    tracesSampleRate: 0.2,
    environment: process.env.NEXT_PUBLIC_ENVIRONMENT || 'production',
  });
}

interface ErrorWithContext extends Error {
  context?: Record<string, any>;
}

export class ErrorMonitoring {
  static captureException(error: ErrorWithContext, context?: Record<string, any>): void {
    // Always log to console in development
    if (process.env.NODE_ENV === 'development') {
      console.error('Error:', error, 'Context:', context || error.context);
    }
    
    // In production, send to Sentry
    if (process.env.NODE_ENV === 'production') {
      if (context || error.context) {
        Sentry.withScope((scope) => {
          if (context) {
            Object.entries(context).forEach(([key, value]) => {
              scope.setExtra(key, value);
            });
          }
          
          if (error.context) {
            Object.entries(error.context).forEach(([key, value]) => {
              scope.setExtra(key, value);
            });
          }
          
          Sentry.captureException(error);
        });
      } else {
        Sentry.captureException(error);
      }
    }
  }
  
  static setUser(user: { id: string; email?: string; username?: string }): void {
    if (process.env.NODE_ENV === 'production') {
      Sentry.setUser(user);
    }
  }
  
  static clearUser(): void {
    if (process.env.NODE_ENV === 'production') {
      Sentry.setUser(null);
    }
  }
  
  static setExtra(key: string, value: any): void {
    if (process.env.NODE_ENV === 'production') {
      Sentry.setExtra(key, value);
    }
  }
  
  static setTag(key: string, value: string): void {
    if (process.env.NODE_ENV === 'production') {
      Sentry.setTag(key, value);
    }
  }
}
```

## 6. Error Handling

### 6.1 API Error Handling

Centralized API error handling:

```typescript
// lib/api/error-handler.ts
import { NextResponse } from 'next/server';
import { logger } from '@/lib/logger/server-logger';
import { ErrorMonitoring } from '@/lib/errors/error-monitoring';
import { ZodError } from 'zod';

export interface ApiError {
  code: string;
  message: string;
  details?: unknown;
}

export class ApiErrorHandler {
  static handle(error: unknown): NextResponse {
    if (error instanceof ZodError) {
      return ApiErrorHandler.handleValidationError(error);
    }
    
    if (error instanceof DatabaseError) {
      return ApiErrorHandler.handleDatabaseError(error);
    }
    
    if (error instanceof AuthError) {
      return ApiErrorHandler.handleAuthError(error);
    }
    
    if (error instanceof NotFoundError) {
      return ApiErrorHandler.handleNotFoundError(error);
    }
    
    if (error instanceof ForbiddenError) {
      return ApiErrorHandler.handleForbiddenError(error);
    }
    
    if (error instanceof ConflictError) {
      return ApiErrorHandler.handleConflictError(error);
    }
    
    // Default to internal server error
    return ApiErrorHandler.handleInternalServerError(error);
  }
  
  private static handleValidationError(error: ZodError): NextResponse {
    const apiError: ApiError = {
      code: 'VALIDATION_ERROR',
      message: 'Validation error',
      details: error.format(),
    };
    
    logger.warn('Validation error', { error: apiError });
    
    return NextResponse.json({ error: apiError }, { status: 400 });
  }
  
  private static handleDatabaseError(error: DatabaseError): NextResponse {
    const apiError: ApiError = {
      code: error.code || 'DATABASE_ERROR',
      message: 'Database error',
    };
    
    logger.error('Database error', { error });
    ErrorMonitoring.captureException(error);
    
    return NextResponse.json({ error: apiError }, { status: 500 });
  }
  
  private static handleAuthError(error: AuthError): NextResponse {
    const apiError: ApiError = {
      code: error.code || 'UNAUTHORIZED',
      message: error.message || 'Authentication required',
    };
    
    logger.warn('Auth error', { error });
    
    return NextResponse.json({ error: apiError }, { status: 401 });
  }
  
  private static handleNotFoundError(error: NotFoundError): NextResponse {
    const apiError: ApiError = {
      code: error.code || 'NOT_FOUND',
      message: error.message || 'Resource not found',
      details: error.details,
    };
    
    logger.info('Not found error', { error });
    
    return NextResponse.json({ error: apiError }, { status: 404 });
  }
  
  private static handleForbiddenError(error: ForbiddenError): NextResponse {
    const apiError: ApiError = {
      code: error.code || 'FORBIDDEN',
      message: error.message || 'Access denied',
      details: error.details,
    };
    
    logger.warn('Forbidden error', { error });
    
    return NextResponse.json({ error: apiError }, { status: 403 });
  }
  
  private static handleConflictError(error: ConflictError): NextResponse {
    const apiError: ApiError = {
      code: error.code || 'CONFLICT',
      message: error.message || 'Resource conflict',
      details: error.details,
    };
    
    logger.warn('Conflict error', { error });
    
    return NextResponse.json({ error: apiError }, { status: 409 });
  }
  
  private static handleInternalServerError(error: unknown): NextResponse {
    const apiError: ApiError = {
      code: 'SERVER_ERROR',
      message: 'An unexpected error occurred',
    };
    
    logger.error('Internal server error', { error });
    ErrorMonitoring.captureException(error as Error);
    
    return NextResponse.json({ error: apiError }, { status: 500 });
  }
}

// Custom error classes
export class DatabaseError extends Error {
  code?: string;
  
  constructor(message: string, code?: string) {
    super(message);
    this.name = 'DatabaseError';
    this.code = code;
  }
}

export class AuthError extends Error {
  code?: string;
  
  constructor(message: string, code?: string) {
    super(message);
    this.name = 'AuthError';
    this.code = code;
  }
}

export class NotFoundError extends Error {
  code?: string;
  details?: unknown;
  
  constructor(message: string, code?: string, details?: unknown) {
    super(message);
    this.name = 'NotFoundError';
    this.code = code;
    this.details = details;
  }
}

export class ForbiddenError extends Error {
  code?: string;
  details?: unknown;
  
  constructor(message: string, code?: string, details?: unknown) {
    super(message);
    this.name = 'ForbiddenError';
    this.code = code;
    this.details = details;
  }
}

export class ConflictError extends Error {
  code?: string;
  details?: unknown;
  
  constructor(message: string, code?: string, details?: unknown) {
    super(message);
    this.name = 'ConflictError';
    this.code = code;
    this.details = details;
  }
}
```

### 6.2 Client-Side Error Handling

React Error Boundary for catching and displaying errors:

```tsx
// components/error-boundary.tsx
'use client';

import { Component, ErrorInfo, ReactNode } from 'react';
import { ErrorMonitoring } from '@/lib/errors/error-monitoring';
import { Button } from '@/components/ui/button';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    ErrorMonitoring.captureException(error, { errorInfo });
  }
  
  resetErrorBoundary = (): void => {
    this.setState({ hasError: false, error: null });
  };

  render(): ReactNode {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }
      
      return (
        <div className="flex h-[50vh] flex-col items-center justify-center text-center">
          <div className="max-w-md space-y-4 p-4">
            <h2 className="text-2xl font-bold">Something went wrong</h2>
            <p className="text-gray-600">
              We've encountered an error. Our team has been notified.
            </p>
            {process.env.NODE_ENV !== 'production' && this.state.error && (
              <div className="rounded-md bg-red-50 p-4 text-left">
                <p className="text-sm font-medium text-red-800">
                  {this.state.error.toString()}
                </p>
              </div>
            )}
            <div className="flex justify-center space-x-4">
              <Button onClick={() => window.location.reload()}>
                Reload Page
              </Button>
              <Button 
                variant="outline" 
                onClick={this.resetErrorBoundary}
              >
                Try Again
              </Button>
            </div>
          </div>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### 6.3 Global Error Handler

Global error handling for unhandled errors:

```typescript
// lib/errors/global-error-handler.ts
import { ErrorMonitoring } from './error-monitoring';
import { logger } from '@/lib/logger/server-logger';

export function setupGlobalErrorHandlers(): void {
  if (typeof window !== 'undefined') {
    // Browser environment
    window.addEventListener('error', (event) => {
      logger.error('Unhandled error', {
        message: event.error?.message,
        stack: event.error?.stack,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno,
      });
      
      ErrorMonitoring.captureException(event.error || new Error(event.message));
    });
    
    window.addEventListener('unhandledrejection', (event) => {
      const error = event.reason instanceof Error 
        ? event.reason 
        : new Error(String(event.reason));
      
      logger.error('Unhandled promise rejection', {
        message: error.message,
        stack: error.stack,
      });
      
      ErrorMonitoring.captureException(error);
    });
  } else {
    // Node.js environment
    process.on('uncaughtException', (error) => {
      logger.error('Uncaught exception', {
        message: error.message,
        stack: error.stack,
      });
      
      ErrorMonitoring.captureException(error);
    });
    
    process.on('unhandledRejection', (reason) => {
      const error = reason instanceof Error 
        ? reason 
        : new Error(String(reason));
      
      logger.error('Unhandled promise rejection', {
        message: error.message,
        stack: error.stack,
      });
      
      ErrorMonitoring.captureException(error);
    });
  }
}
```

### 6.4 Query Error Handling

React Query error handling:

```typescript
// lib/query-client.ts
import { QueryClient } from '@tanstack/react-query';
import { ErrorMonitoring } from '@/lib/errors/error-monitoring';
import { toast } from '@/components/ui/toast';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      cacheTime: 1000 * 60 * 30, // 30 minutes
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      onError: (error: unknown) => {
        const message = error instanceof Error 
          ? error.message 
          : 'An unexpected error occurred';
        
        // Only show toast for network or server errors
        if (!(error instanceof Error) || !error.message.includes('validation')) {
          toast({
            title: 'Error',
            description: message,
            variant: 'error',
          });
        }
        
        // Log all errors
        ErrorMonitoring.captureException(
          error instanceof Error ? error : new Error(message)
        );
      },
    },
    mutations: {
      retry: 1,
      onError: (error: unknown, variables, context) => {
        const message = error instanceof Error 
          ? error.message 
          : 'An unexpected error occurred';
        
        toast({
          title: 'Error',
          description: message,
          variant: 'error',
        });
        
        ErrorMonitoring.captureException(
          error instanceof Error ? error : new Error(message),
          { mutationVariables: variables, mutationContext: context }
        );
      },
    },
  },
});
```

## 7. Security Considerations

### 7.1 Input Validation

Thorough validation of all input data:

```typescript
// lib/notes/validators.ts
import { z } from 'zod';
import { NoteAccessLevel, NoteEditPermission } from './types';

// Validation schemas for note-related operations
export const createNoteSchema = z.object({
  title: z.string().min(1, 'Title is required').max(100, 'Title is too long'),
  content: z.string(),
  organizationId: z.string().min(1, 'Organization ID is required'),
  accessLevel: z.enum(['public', 'shared', 'private'], {
    errorMap: () => ({ message: 'Invalid access level' }),
  }),
  canEdit: z.enum(['everyone', 'accessList', 'denyList', 'noOne'], {
    errorMap: () => ({ message: 'Invalid edit permission' }),
  }),
  editAccessList: z.array(z.string()).optional(),
  viewAccessList: z.array(z.string()).optional(),
});

export const updateNoteSchema = z.object({
  title: z.string().min(1, 'Title is required').max(100, 'Title is too long').optional(),
  content: z.string().optional(),
  accessLevel: z.enum(['public', 'shared', 'private'], {
    errorMap: () => ({ message: 'Invalid access level' }),
  }).optional(),
  canEdit: z.enum(['everyone', 'accessList', 'denyList', 'noOne'], {
    errorMap: () => ({ message: 'Invalid edit permission' }),
  }).optional(),
  editAccessList: z.array(z.string()).optional(),
  viewAccessList: z.array(z.string()).optional(),
});

// Comprehensive input validation
export function validateCreateNoteInput(data: unknown) {
  return createNoteSchema.safeParse(data);
}

export function validateUpdateNoteInput(data: unknown) {
  return updateNoteSchema.safeParse(data);
}

// Helper to validate specific properties
export function validateNoteTitle(title: string): string | null {
  if (!title.trim()) {
    return 'Title is required';
  }
  
  if (title.length > 100) {
    return 'Title must be less than 100 characters';
  }
  
  return null;
}

// Content sanitation
export function sanitizeNoteContent(content: string): string {
  // Basic sanitation to prevent XSS
  return content
    .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
    .replace(/javascript:/gi, 'removed:')
    .replace(/on\w+=/gi, 'removed=');
}
```

### 7.2 Content Security

Implementing Content Security Policy (CSP) to prevent XSS attacks:

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { authMiddleware } from '@clerk/nextjs/server';

// CSP middleware
function cspMiddleware(request: NextRequest) {
  const nonce = Buffer.from(crypto.randomUUID()).toString('base64');
  
  const cspHeader = `
    default-src 'self';
    script-src 'self' 'nonce-${nonce}' https://cdnjs.cloudflare.com https://clerk.noteorbit.com;
    style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
    img-src 'self' data: https://img.clerk.com https://images.clerk.dev;
    font-src 'self' https://fonts.gstatic.com;
    connect-src 'self' https://api.clerk.dev https://clerk.noteorbit.com https://noteorbit.com;
    frame-src 'self';
    object-src 'none';
    base-uri 'self';
    form-action 'self';
    frame-ancestors 'none';
    block-all-mixed-content;
    upgrade-insecure-requests;
  `.replace(/\s{2,}/g, ' ').trim();
  
  const requestHeaders = new Headers(request.headers);
  requestHeaders.set('x-nonce', nonce);
  requestHeaders.set('Content-Security-Policy', cspHeader);
  
  const response = NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  });
  
  response.headers.set('Content-Security-Policy', cspHeader);
  
  return response;
}

// Combine with Clerk auth middleware
export default authMiddleware({
  publicRoutes: [
    '/',
    '/auth/sign-in(.*)',
    '/auth/sign-up(.*)',
    '/api/health',
    '/api/webhook/clerk',
  ],
  beforeAuth: (req) => {
    return cspMiddleware(req);
  },
  afterAuth: (auth, req) => {
    // Clerk's auth logic
    if (auth.userId && req.nextUrl.pathname.startsWith('/auth')) {
      const url = new URL('/dashboard', req.url);
      return NextResponse.redirect(url);
    }
    
    if (!auth.userId && !req.nextUrl.pathname.startsWith('/auth') && 
        !req.nextUrl.pathname.startsWith('/api/') && 
        req.nextUrl.pathname !== '/') {
      const url = new URL('/auth/sign-in', req.url);
      url.searchParams.set('redirect_url', req.nextUrl.pathname);
      return NextResponse.redirect(url);
    }
    
    return NextResponse.next();
  },
});

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
};
```

### 7.3 API Rate Limiting

Protect API endpoints from abuse:

```typescript
// lib/api/middleware/rate-limit.ts
import { NextFetchEvent, NextRequest, NextResponse } from 'next/server';
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

// Create a Redis client
const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL || '',
  token: process.env.UPSTASH_REDIS_TOKEN || '',
});

// Create a rate limiter that allows 10 requests per minute
const generalLimiter = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, '1 m'),
  analytics: true,
  prefix: 'ratelimit:general',
});

// Create a more restrictive limiter for sensitive operations
const strictLimiter = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(5, '1 m'),
  analytics: true,
  prefix: 'ratelimit:strict',
});

// Middleware function to apply rate limiting
export async function rateLimitMiddleware(
  req: NextRequest,
  event: NextFetchEvent,
  options: { 
    strict?: boolean;
    identifier?: string | (() => string | undefined);
  } = {}
) {
  const { strict = false, identifier } = options;
  
  // Determine the identifier for rate limiting
  const ip = req.ip || '127.0.0.1';
  let id: string;
  
  if (typeof identifier === 'function') {
    const result = identifier();
    id = result || ip;
  } else if (identifier) {
    id = identifier;
  } else {
    id = ip;
  }
  
  // Apply the appropriate rate limiter
  const limiter = strict ? strictLimiter : generalLimiter;
  
  const { success, limit, reset, remaining } = await limiter.limit(id);
  
  // Set rate limit headers
  const res = success
    ? NextResponse.next()
    : NextResponse.json(
        { 
          error: {
            code: 'RATE_LIMITED',
            message: 'Too many requests, please try again later',
          }
        },
        { status: 429 }
      );
  
  res.headers.set('X-RateLimit-Limit', limit.toString());
  res.headers.set('X-RateLimit-Remaining', remaining.toString());
  res.headers.set('X-RateLimit-Reset', reset.toString());
  
  return res;
}
```

### 7.4 Preventing CSRF

Cross-Site Request Forgery protection:

```typescript
// lib/api/middleware/csrf.ts
import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@clerk/nextjs/server';

// CSRF protection middleware
export async function csrfProtection(req: NextRequest) {
  // Skip for GET/HEAD requests as they should be idempotent
  if (['GET', 'HEAD'].includes(req.method)) {
    return NextResponse.next();
  }
  
  const { userId } = auth();
  
  // Require authentication for all state-changing operations
  if (!userId) {
    return NextResponse.json(
      { 
        error: {
          code: 'UNAUTHORIZED',
          message: 'Authentication required',
        }
      },
      { status: 401 }
    );
  }
  
  // Check the origin header
  const origin = req.headers.get('origin');
  const host = req.headers.get('host');
  
  if (!origin || !host) {
    return NextResponse.json(
      { 
        error: {
          code: 'FORBIDDEN',
          message: 'CSRF protection: missing origin or host header',
        }
      },
      { status: 403 }
    );
  }
  
  // Verify that the origin matches the host
  try {
    const url = new URL(origin);
    const originHost = url.host;
    
    if (originHost !== host) {
      return NextResponse.json(
        { 
          error: {
            code: 'FORBIDDEN',
            message: 'CSRF protection: origin does not match host',
          }
        },
        { status: 403 }
      );
    }
  } catch (error) {
    return NextResponse.json(
      { 
        error: {
          code: 'FORBIDDEN',
          message: 'CSRF protection: invalid origin',
        }
      },
      { status: 403 }
    );
  }
  
  // Continue with the request
  return NextResponse.next();
}
```

### 7.5 Data Encryption

Encrypting sensitive data:

```typescript
// lib/security/encryption.ts
import { randomBytes, createCipheriv, createDecipheriv } from 'crypto';

// Get encryption key from environment variables
const ENCRYPTION_KEY = process.env.ENCRYPTION_KEY as string;
if (!ENCRYPTION_KEY || ENCRYPTION_KEY.length !== 32) {
  throw new Error('Invalid encryption key. Must be 32 bytes (256 bits).');
}

// Encrypt data
export function encrypt(text: string): { encryptedData: string; iv: string } {
  const iv = randomBytes(16);
  const cipher = createCipheriv('aes-256-cbc', Buffer.from(ENCRYPTION_KEY), iv);
  
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  return {
    encryptedData: encrypted,
    iv: iv.toString('hex'),
  };
}

// Decrypt data
export function decrypt(encryptedData: string, iv: string): string {
  const decipher = createDecipheriv(
    'aes-256-cbc',
    Buffer.from(ENCRYPTION_KEY),
    Buffer.from(iv, 'hex')
  );
  
  let decrypted = decipher.update(encryptedData, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}
```

## 8. Performance Optimizations

### 8.1 Next.js Optimizations

```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  poweredByHeader: false,
  compress: true,
  
  // Image optimization
  images: {
    domains: ['img.clerk.com', 'images.clerk.dev'],
    formats: ['image/avif', 'image/webp'],
  },
  
  // Enable SWC minification
  swcMinify: true,
  
  // Optimization for third-party dependencies
  experimental: {
    optimizeCss: true,
    optimizePackageImports: [
      'react-markdown-editor-lite',
      'react-markdown',
      'date-fns',
      '@heroicons/react',
    ],
  },
  
  // Content security policy headers
  headers: async () => [
    {
      source: '/(.*)',
      headers: [
        {
          key: 'X-DNS-Prefetch-Control',
          value: 'on',
        },
        {
          key: 'X-XSS-Protection',
          value: '1; mode=block',
        },
        {
          key: 'X-Frame-Options',
          value: 'SAMEORIGIN',
        },
        {
          key: 'X-Content-Type-Options',
          value: 'nosniff',
        },
        {
          key: 'Referrer-Policy',
          value: 'strict-origin-when-cross-origin',
        },
        {
          key: 'Permissions-Policy',
          value: 'camera=(), microphone=(), geolocation=(), interest-cohort=()',
        },
      ],
    },
  ],
};

module.exports = nextConfig;
```

### 8.2 Optimizing Data Fetching

Using React Query with optimized settings:

```typescript
// lib/query-client.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // Only refetch when window gets focus if the data is stale
      refetchOnWindowFocus: 'always',
      
      // Keep data for 5 minutes before considering it stale
      staleTime: 1000 * 60 * 5,
      
      // Keep unused data in cache for 30 minutes
      cacheTime: 1000 * 60 * 30,
      
      // Retry failed queries 3 times with exponential backoff
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      
      // Use optimistic response for certain queries
      optimisticResults: true,
      
      // Structural sharing to minimize re-renders
      structuralSharing: true,
    },
  },
});
```

### 8.3 Component Optimizations

Using React.memo, useMemo, and useCallback to prevent unnecessary rerenders:

```tsx
// lib/notes/components/optimized-note-card.tsx
import { memo, useMemo, useCallback } from 'react';
import Link from 'next/link';
import { formatDistanceToNow } from 'date-fns';
import { AccessLevelBadge } from './access-level-badge';
import { Note } from '@/lib/notes/types';

interface NoteCardProps {
  note: Note;
  onSelect?: (noteId: string) => void;
}

function NoteCardComponent({ note, onSelect }: NoteCardProps) {
  // Memoize derived values
  const formattedDate = useMemo(() => {
    return formatDistanceToNow(new Date(note.updatedAt), { addSuffix: true });
  }, [note.updatedAt]);
  
  const truncatedContent = useMemo(() => {
    // Remove markdown syntax
    const plainText = note.content
      .replace(/#+\s/g, '')        // Remove headings
      .replace(/\*\*|\*|~~|`/g, '') // Remove formatting
      .replace(/\[([^\]]+)\]\([^)]+\)/g, '$1'); // Remove links
    
    // Truncate to reasonable length
    return plainText.length > 150
      ? plainText.substring(0, 150) + '...'
      : plainText;
  }, [note.content]);
  
  // Memoize callback
  const handleClick = useCallback(() => {
    if (onSelect) {
      onSelect(note.id);
    }
  }, [note.id, onSelect]);
  
  return (
    <div 
      className="overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm transition hover:shadow"
      onClick={handleClick}
    >
      <div className="border-b border-gray-200 bg-gray-50 px-4 py-3">
        <div className="flex items-center justify-between">
          <h3 className="truncate text-sm font-medium text-gray-900">
            {note.title}
          </h3>
          <AccessLevelBadge accessLevel={note.accessLevel} />
        </div>
      </div>
      
      <div className="flex h-32 flex-col p-4">
        <div className="flex-1 text-sm text-gray-600">
          {truncatedContent}
        </div>
        
        <div className="mt-4 flex items-center justify-between text-xs text-gray-500">
          <span>{formattedDate}</span>
          
          {note.lockedByUserId && (
            <span className="flex items-center text-amber-600">
              <LockClosedIcon className="mr-1 h-3 w-3" />
              Locked
            </span>
          )}
        </div>
      </div>
    </div>
  );
}

// Memoize the entire component
export const OptimizedNoteCard = memo(NoteCardComponent);
```

### 8.4 Code Splitting and Lazy Loading

Optimizing bundle size with dynamic imports:

```tsx
// app/dashboard/notes/[noteId]/page.tsx
import dynamic from 'next/dynamic';
import { Suspense } from 'react';
import { PageTitle } from '@/components/layout/page-title';
import { LoadingSpinner } from '@/components/ui/loading-spinner';

// Dynamically import the note viewer component
const NoteViewer = dynamic(
  () => import('@/lib/notes/components/note-viewer'),
  {
    loading: () => <LoadingSpinner />,
    ssr: false, // Disable server-side rendering for this component
  }
);

// Dynamically import the note actions component
const NoteActions = dynamic(
  () => import('@/lib/notes/components/note-actions'),
  {
    loading: () => <div className="h-10 w-32 rounded bg-gray-200" />,
    ssr: true, // Enable server-side rendering for this component
  }
);

export default async function NotePage({ params }: { params: { noteId: string } }) {
  // ...page logic
  
  return (
    <div className="container mx-auto px-4 py-8">
      <div className="mb-8 flex flex-col space-y-4 sm:flex-row sm:items-center sm:justify-between sm:space-y-0">
        <PageTitle title={note.title} />
        <Suspense fallback={<div className="h-10 w-32 rounded bg-gray-200" />}>
          <NoteActions note={note} canEdit={canEdit} />
        </Suspense>
      </div>
      
      <div className="mx-auto max-w-4xl">
        <div className="overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm">
          <div className="border-b border-gray-200 bg-gray-50 px-6 py-4">
            {/* Note metadata */}
          </div>
          
          <div className="px-6 py-6">
            <Suspense fallback={<LoadingSpinner />}>
              <NoteViewer content={note.content} />
            </Suspense>
          </div>
        </div>
      </div>
    </div>
  );
}
```

### 8.5 API Response Optimization

Optimizing API responses with efficient data transfer:

```typescript
// lib/notes/repositories/optimized-note-repository.ts
import { prisma } from '@/lib/db/prisma';
import { Note, NoteListItem } from '@/lib/notes/types';
import { mapPrismaNoteToNote, mapPrismaNoteToNoteListItem } from '@/lib/notes/mappers/note-mapper';

export const OptimizedNoteRepository = {
  // Get notes list with pagination and minimal data
  async getOrganizationNotes(
    organizationId: string,
    userId: string,
    options: {
      page?: number;
      perPage?: number;
      accessLevel?: string;
      createdBy?: string;
      search?: string;
    } = {}
  ): Promise<{ notes: NoteListItem[]; total: number }> {
    const {
      page = 1,
      perPage = 10,
      accessLevel,
      createdBy,
      search,
    } = options;
    
    // Build base query
    const where: any = {
      organization_id: organizationId,
    };
    
    // Apply filters
    if (accessLevel) {
      where.access_level = accessLevel;
    }
    
    if (createdBy) {
      where.created_by_user_id = createdBy;
      if (search) {
      where.OR = [
        { title: { contains: search, mode: 'insensitive' } },
        { content: { contains: search, mode: 'insensitive' } },
      ];
    }
    
    // Add access control to query
    where.OR = [
      ...(where.OR || []),
      // Public notes are visible to all organization members
      { access_level: 'public' },
      // Private notes are only visible to creators
      {
        access_level: 'private',
        created_by_user_id: userId,
      },
      // Shared notes require checking access list
      {
        access_level: 'shared',
        OR: [
          { created_by_user_id: userId },
          { view_access_list: { contains: userId } },
        ],
      },
    ];
    
    // Get total count with filters but without pagination
    const total = await prisma.note.count({ where });
    
    // Get paginated notes with minimal data for list view
    const notes = await prisma.note.findMany({
      where,
      select: {
        id: true,
        title: true,
        content: true,
        created_by_user_id: true,
        access_level: true,
        updated_at: true,
        locked_by_user_id: true,
      },
      orderBy: { updated_at: 'desc' },
      skip: (page - 1) * perPage,
      take: perPage,
    });
    
    // Map to list items
    const noteItems = notes.map(note => mapPrismaNoteToNoteListItem(note));
    
    return { notes: noteItems, total };
  },
  
  // Get a single note by ID with full details
  async getNoteById(
    noteId: string,
    userId: string,
    organizationId: string
  ): Promise<Note | null> {
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
    const hasAccess = await this.canUserViewNote(noteId, userId);
    
    if (!hasAccess) {
      return null;
    }
    
    return mapPrismaNoteToNote(note);
  },
  
  // Check if user can view note
  async canUserViewNote(noteId: string, userId: string): Promise<boolean> {
    const note = await prisma.note.findUnique({
      where: { id: noteId },
      select: {
        access_level: true,
        created_by_user_id: true,
        view_access_list: true,
      },
    });
    
    if (!note) {
      return false;
    }
    
    // Public notes are accessible to all organization members
    if (note.access_level === 'public') {
      return true;
    }
    
    // Private notes are only accessible to the creator
    if (note.access_level === 'private' && note.created_by_user_id !== userId) {
      return false;
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
  }
};
```

### 8.6 Database Query Optimization

Optimizing database queries for better performance:

```typescript
// lib/notes/repositories/performance-tuned-repository.ts
import { prisma } from '@/lib/db/prisma';
import { logger } from '@/lib/logger/server-logger';

export const PerformanceTunedRepository = {
  // Efficient batch loading of notes with their metadata
  async batchLoadNotes(noteIds: string[]) {
    if (noteIds.length === 0) return [];
    
    const startTime = performance.now();
    
    // Use a single query with WHERE IN clause
    const notes = await prisma.note.findMany({
      where: {
        id: { in: noteIds },
      },
    });
    
    const endTime = performance.now();
    logger.info('Batch loaded notes', { 
      count: notes.length, 
      duration: endTime - startTime 
    });
    
    // Index by ID for efficient lookup
    const notesById = notes.reduce((acc, note) => {
      acc[note.id] = note;
      return acc;
    }, {} as Record<string, any>);
    
    // Return in the same order as requested
    return noteIds.map(id => notesById[id] || null);
  },
  
  // Efficient search with pagination and filtering
  async searchNotes(params: {
    organizationId: string;
    query: string;
    accessLevel?: string;
    createdBy?: string;
    page?: number;
    perPage?: number;
  }) {
    const {
      organizationId,
      query,
      accessLevel,
      createdBy,
      page = 1,
      perPage = 10,
    } = params;
    
    const startTime = performance.now();
    
    // Build base query
    const where: any = {
      organization_id: organizationId,
    };
    
    // Apply filters
    if (accessLevel) {
      where.access_level = accessLevel;
    }
    
    if (createdBy) {
      where.created_by_user_id = createdBy;
    }
    
    // Search in title and content
    if (query) {
      where.OR = [
        { title: { contains: query, mode: 'insensitive' } },
        { content: { contains: query, mode: 'insensitive' } },
      ];
    }
    
    // First get total count (in parallel with data query)
    const countPromise = prisma.note.count({ where });
    
    // Then get paginated results with only needed fields
    const notesPromise = prisma.note.findMany({
      where,
      select: {
        id: true,
        title: true,
        content: true,
        access_level: true,
        created_by_user_id: true,
        updated_at: true,
      },
      orderBy: { updated_at: 'desc' },
      skip: (page - 1) * perPage,
      take: perPage,
    });
    
    // Execute both queries in parallel
    const [total, notes] = await Promise.all([countPromise, notesPromise]);
    
    const endTime = performance.now();
    logger.info('Searched notes', { 
      query,
      resultCount: notes.length,
      totalResults: total,
      duration: endTime - startTime 
    });
    
    return {
      notes,
      pagination: {
        page,
        perPage,
        total,
        totalPages: Math.ceil(total / perPage),
      },
    };
  }
};
```

### 8.7 Edge Caching Strategy

Implementing edge caching for improved performance:

```typescript
// app/api/notes/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@clerk/nextjs/server';
import { NoteRepository } from '@/lib/notes/repositories/note-repository';
import { validateListNotesQuery } from '@/lib/notes/validators';
import { logger } from '@/lib/logger/server-logger';

export const runtime = 'edge'; // Use Edge runtime for better performance

export async function GET(req: NextRequest) {
  try {
    const { userId, orgId } = auth();
    
    if (!userId || !orgId) {
      return NextResponse.json(
        {
          error: {
            code: 'UNAUTHORIZED',
            message: 'You must be logged in to access this resource',
          }
        },
        { status: 401 }
      );
    }
    
    const url = new URL(req.url);
    const params = {
      page: parseInt(url.searchParams.get('page') || '1'),
      perPage: parseInt(url.searchParams.get('perPage') || '10'),
      accessLevel: url.searchParams.get('accessLevel') || undefined,
      createdBy: url.searchParams.get('createdBy') || undefined,
      search: url.searchParams.get('search') || undefined,
    };
    
    const validation = validateListNotesQuery(params);
    if (!validation.success) {
      return NextResponse.json(
        {
          error: {
            code: 'INVALID_QUERY',
            message: 'Invalid query parameters',
            details: validation.errors,
          }
        },
        { status: 400 }
      );
    }
    
    const { notes, total } = await NoteRepository.getOrganizationNotes(
      orgId,
      userId,
      {
        page: params.page,
        perPage: params.perPage,
        accessLevel: params.accessLevel,
        createdBy: params.createdBy,
        search: params.search,
      }
    );
    
    logger.info('Notes list fetched', {
      userId,
      organizationId: orgId,
      filters: {
        accessLevel: params.accessLevel,
        createdBy: params.createdBy,
        search: params.search?.substring(0, 20),
      },
      results: notes.length,
      total,
    });
    
    // Create response with cache control headers
    const response = NextResponse.json({
      data: notes,
      meta: {
        pagination: {
          page: params.page,
          perPage: params.perPage,
          total,
          totalPages: Math.ceil(total / perPage),
        }
      }
    });
    
    // Cache for 5 minutes on edge, revalidate every 60 seconds
    response.headers.set(
      'Cache-Control',
      'public, s-maxage=300, stale-while-revalidate=60'
    );
    
    return response;
  } catch (error) {
    logger.error('Failed to fetch notes list', { error });
    
    return NextResponse.json(
      {
        error: {
          code: 'SERVER_ERROR',
          message: 'An unexpected error occurred',
        }
      },
      { status: 500 }
    );
  }
}
```

## 9. Scaling Strategies

### 9.1 Horizontal Scaling

```typescript
// lib/db/connection-pool.ts
import { PrismaClient } from '@prisma/client';
import { logger } from '@/lib/logger/server-logger';

// Configure connection pooling
const connectionPoolConfig = {
  min: 5,           // Minimum pool size
  max: 20,          // Maximum pool size
  idleTimeout: 30,  // Seconds before idle connection is closed
  acquireTimeout: 5, // Seconds to acquire a connection
  retryInterval: 0.5, // Seconds between retry attempts
  maxRetries: 3,    // Maximum retry attempts
};

let prismaClient: PrismaClient;

// Singleton pattern for Prisma connection
export function getPrismaClient(): PrismaClient {
  if (!prismaClient) {
    prismaClient = new PrismaClient({
      log: [
        { level: 'query', emit: 'event' },
        { level: 'error', emit: 'event' },
      ],
      datasources: {
        db: {
          url: process.env.DATABASE_URL,
        },
      },
      // Configure connection pooling
      // @ts-ignore - PrismaClient constructor accepts this option
      connectionLimit: connectionPoolConfig,
    });
    
    // Log slow queries
    prismaClient.$on('query', (e: any) => {
      if (e.duration > 200) { // Log queries that take more than 200ms
        logger.warn('Slow query detected', {
          query: e.query,
          params: e.params,
          duration: e.duration,
        });
      }
    });
    
    // Log errors
    prismaClient.$on('error', (e: any) => {
      logger.error('Prisma query error', {
        error: e.error,
        query: e.query,
        params: e.params,
      });
    });
  }
  
  return prismaClient;
}

// Clean up database connections on shutdown
if (process.env.NODE_ENV === 'production') {
  process.on('SIGINT', () => {
    if (prismaClient) {
      logger.info('Shutting down Prisma client');
      prismaClient.$disconnect();
    }
  });
  
  process.on('SIGTERM', () => {
    if (prismaClient) {
      logger.info('Shutting down Prisma client');
      prismaClient.$disconnect();
    }
  });
}
```

### 9.2 Database Scaling

Strategy for database scaling as the application grows:

```typescript
// lib/db/read-replica-manager.ts
import { PrismaClient } from '@prisma/client';
import { logger } from '@/lib/logger/server-logger';

// Primary database for writes
let primaryClient: PrismaClient;

// Read replicas for read operations
const readReplicas: PrismaClient[] = [];
let currentReplicaIndex = 0;

// Initialize database clients
export function initializeDatabaseClients() {
  // Initialize primary client
  primaryClient = new PrismaClient({
    datasources: {
      db: {
        url: process.env.PRIMARY_DATABASE_URL,
      },
    },
  });
  
  // Initialize read replicas if configured
  const replicaUrls = process.env.READ_REPLICA_URLS?.split(',') || [];
  
  for (const url of replicaUrls) {
    if (url) {
      const replicaClient = new PrismaClient({
        datasources: {
          db: {
            url: url.trim(),
          },
        },
      });
      
      readReplicas.push(replicaClient);
      logger.info('Initialized read replica', { url: url.split('@')[1] });
    }
  }
  
  logger.info('Database clients initialized', {
    primaryClient: true,
    readReplicas: readReplicas.length,
  });
}

// Get primary client for write operations
export function getPrimaryClient(): PrismaClient {
  if (!primaryClient) {
    throw new Error('Primary database client not initialized');
  }
  
  return primaryClient;
}

// Get read replica client using round-robin load balancing
export function getReadClient(): PrismaClient {
  // If no replicas configured, use primary
  if (readReplicas.length === 0) {
    return getPrimaryClient();
  }
  
  // Round-robin selection
  const client = readReplicas[currentReplicaIndex];
  currentReplicaIndex = (currentReplicaIndex + 1) % readReplicas.length;
  
  return client;
}

// Clean up database connections
export async function cleanupDatabaseClients() {
  if (primaryClient) {
    await primaryClient.$disconnect();
  }
  
  for (const replica of readReplicas) {
    await replica.$disconnect();
  }
  
  logger.info('Database clients disconnected');
}
```

### 9.3 Caching Layer

Implementation of caching layer for frequently accessed data:

```typescript
// lib/cache/redis-cache.ts
import { Redis } from 'ioredis';
import { logger } from '@/lib/logger/server-logger';

// Configure Redis client
const redis = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT || '6379'),
  password: process.env.REDIS_PASSWORD,
  // Enable TLS in production
  tls: process.env.NODE_ENV === 'production' ? {} : undefined,
  // Reconnect strategy
  retryStrategy: (times) => {
    const delay = Math.min(times * 50, 2000);
    return delay;
  },
});

// Log Redis connection errors
redis.on('error', (error) => {
  logger.error('Redis connection error', { error });
});

// Cache interface
export class Cache {
  static DEFAULT_TTL = 60 * 5; // 5 minutes in seconds
  
  // Get cached value
  static async get<T>(key: string): Promise<T | null> {
    try {
      const value = await redis.get(key);
      
      if (!value) {
        return null;
      }
      
      return JSON.parse(value) as T;
    } catch (error) {
      logger.error('Cache get error', { error, key });
      return null;
    }
  }
  
  // Set cached value
  static async set<T>(key: string, value: T, ttlSeconds = Cache.DEFAULT_TTL): Promise<boolean> {
    try {
      await redis.set(key, JSON.stringify(value), 'EX', ttlSeconds);
      return true;
    } catch (error) {
      logger.error('Cache set error', { error, key });
      return false;
    }
  }
  
  // Delete cached value
  static async delete(key: string): Promise<boolean> {
    try {
      await redis.del(key);
      return true;
    } catch (error) {
      logger.error('Cache delete error', { error, key });
      return false;
    }
  }
  
  // Clear cache by pattern
  static async clearPattern(pattern: string): Promise<boolean> {
    try {
      const keys = await redis.keys(pattern);
      
      if (keys.length > 0) {
        await redis.del(...keys);
      }
      
      logger.info('Cache cleared by pattern', { pattern, keysRemoved: keys.length });
      return true;
    } catch (error) {
      logger.error('Cache clear pattern error', { error, pattern });
      return false;
    }
  }
  
  // Cache with auto-refresh
  static async getOrSet<T>(
    key: string,
    fetchFn: () => Promise<T>,
    ttlSeconds = Cache.DEFAULT_TTL
  ): Promise<T> {
    // Try to get from cache first
    const cachedValue = await Cache.get<T>(key);
    
    if (cachedValue !== null) {
      return cachedValue;
    }
    
    // If not in cache, fetch new value
    const value = await fetchFn();
    
    // Store in cache
    await Cache.set(key, value, ttlSeconds);
    
    return value;
  }
}
```

### 9.4 Message Queue for Asynchronous Processing

Implementing a message queue for background processing:

```typescript
// lib/queue/queue-service.ts
import { Queue, Worker, QueueScheduler } from 'bullmq';
import { logger } from '@/lib/logger/server-logger';

// Redis connection configuration
const redisConnection = {
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT || '6379'),
  password: process.env.REDIS_PASSWORD,
  tls: process.env.NODE_ENV === 'production' ? {} : undefined,
};

// Queue names
export enum QueueName {
  EMAIL = 'email',
  NOTE_PROCESSING = 'note-processing',
  EXPORT = 'export',
  AUDIT = 'audit',
}

// Job types
export enum JobType {
  // Email jobs
  SEND_WELCOME_EMAIL = 'send-welcome-email',
  SEND_INVITE_EMAIL = 'send-invite-email',
  SEND_NOTIFICATION = 'send-notification',
  
  // Note processing jobs
  GENERATE_PREVIEW = 'generate-preview',
  INDEX_NOTE_CONTENT = 'index-note-content',
  GENERATE_THUMBNAILS = 'generate-thumbnails',
  
  // Export jobs
  EXPORT_NOTES = 'export-notes',
  EXPORT_ORGANIZATION_DATA = 'export-organization-data',
  
  // Audit jobs
  RECORD_AUDIT = 'record-audit',
}

// Queue instances
const queues: Record<QueueName, Queue> = {
  [QueueName.EMAIL]: new Queue(QueueName.EMAIL, { connection: redisConnection }),
  [QueueName.NOTE_PROCESSING]: new Queue(QueueName.NOTE_PROCESSING, { connection: redisConnection }),
  [QueueName.EXPORT]: new Queue(QueueName.EXPORT, { connection: redisConnection }),
  [QueueName.AUDIT]: new Queue(QueueName.AUDIT, { connection: redisConnection }),
};

// Queue schedulers for delayed jobs
const schedulers: Record<QueueName, QueueScheduler> = {
  [QueueName.EMAIL]: new QueueScheduler(QueueName.EMAIL, { connection: redisConnection }),
  [QueueName.NOTE_PROCESSING]: new QueueScheduler(QueueName.NOTE_PROCESSING, { connection: redisConnection }),
  [QueueName.EXPORT]: new QueueScheduler(QueueName.EXPORT, { connection: redisConnection }),
  [QueueName.AUDIT]: new QueueScheduler(QueueName.AUDIT, { connection: redisConnection }),
};

// Queue service
export class QueueService {
  // Add a job to the queue
  static async addJob<T>(
    queueName: QueueName,
    jobType: JobType,
    data: T,
    options: {
      priority?: number;
      delay?: number;
      attempts?: number;
      removeOnComplete?: boolean;
    } = {}
  ): Promise<string> {
    const queue = queues[queueName];
    
    const job = await queue.add(jobType, data, {
      priority: options.priority,
      delay: options.delay,
      attempts: options.attempts || 3,
      backoff: {
        type: 'exponential',
        delay: 1000,
      },
      removeOnComplete: options.removeOnComplete !== false,
    });
    
    logger.info('Job added to queue', {
      queueName,
      jobType,
      jobId: job.id,
    });
    
    return job.id;
  }
  
  // Process jobs from a queue
  static processQueue<T, R>(
    queueName: QueueName,
    jobTypes: JobType[],
    processor: (job: { data: T; type: JobType }) => Promise<R>,
    concurrency = 1
  ): Worker<T, R> {
    const worker = new Worker(
      queueName,
      async (job) => {
        logger.info('Processing job', {
          queueName,
          jobType: job.name,
          jobId: job.id,
        });
        
        return processor({ data: job.data, type: job.name as JobType });
      },
      {
        connection: redisConnection,
        concurrency,
        // Only process specified job types
        lockDuration: 30000, // 30 seconds
      }
    );
    
    // Handle job completion
    worker.on('completed', (job) => {
      logger.info('Job completed', {
        queueName,
        jobType: job.name,
        jobId: job.id,
      });
    });
    
    // Handle job failure
    worker.on('failed', (job, error) => {
      logger.error('Job failed', {
        queueName,
        jobType: job?.name,
        jobId: job?.id,
        error,
      });
    });
    
    logger.info('Worker started for queue', {
      queueName,
      jobTypes,
      concurrency,
    });
    
    return worker;
  }
  
  // Get job status
  static async getJob(queueName: QueueName, jobId: string) {
    const queue = queues[queueName];
    return queue.getJob(jobId);
  }
  
  // Remove a job
  static async removeJob(queueName: QueueName, jobId: string): Promise<void> {
    const queue = queues[queueName];
    const job = await queue.getJob(jobId);
    
    if (job) {
      await job.remove();
      
      logger.info('Job removed', {
        queueName,
        jobId,
      });
    }
  }
  
  // Clean up queues
  static async cleanupQueues(): Promise<void> {
    for (const [name, queue] of Object.entries(queues)) {
      await queue.close();
      logger.info('Queue closed', { queueName: name });
    }
    
    for (const [name, scheduler] of Object.entries(schedulers)) {
      await scheduler.close();
      logger.info('Queue scheduler closed', { queueName: name });
    }
  }
}

// Clean up queues on shutdown
if (process.env.NODE_ENV === 'production') {
  process.on('SIGINT', async () => {
    await QueueService.cleanupQueues();
  });
  
  process.on('SIGTERM', async () => {
    await QueueService.cleanupQueues();
  });
}
```

### 9.5 Microservices Architecture (Future)

Planning for future microservices architecture:

```typescript
// docs/architecture/microservices-roadmap.md
/**
# NoteOrbit Microservices Architecture Roadmap

## Current Monolithic Architecture

NoteOrbit currently operates as a monolithic application with the following components:
- Next.js application for UI and API routes
- PostgreSQL database for data storage
- Redis for caching and message queuing
- Clerk for authentication and organization management

## Scaling Challenges

As the application grows, we anticipate the following scaling challenges:
1. Increased database load, especially for read operations
2. Higher concurrent user count causing API bottlenecks
3. Resource-intensive operations blocking the main application
4. Complex deployment and scaling of the monolithic application

## Microservices Transition Plan

### Phase 1: Preparation (Current)
- Ensure clean separation of concerns in the codebase
- Implement comprehensive logging and monitoring
- Add message queue for asynchronous processing
- Set up caching layer for frequently accessed data

### Phase 2: Database Scaling
- Implement read replicas for read-heavy operations
- Set up database sharding for organization data
- Optimize query patterns and add appropriate indexes
- Implement database connection pooling

### Phase 3: Extract First Microservices
1. **Note Processing Service**
   - Handle note indexing, content analysis, and preview generation
   - Process note exports and imports
   - Run independently from the main application

2. **Notification Service**
   - Handle email notifications
   - Manage in-app notifications
   - Process notification preferences

### Phase 4: API Gateway and Service Mesh
- Implement API Gateway for routing requests
- Add service discovery mechanism
- Set up distributed tracing
- Implement circuit breakers and fallbacks

### Phase 5: Core Domain Microservices
- Note Service: CRUD operations for notes
- Organization Service: Organization management
- User Service: User preferences and profiles
- Collaboration Service: Real-time editing and commenting

### Phase 6: Specialized Services
- Search Service: Full-text search capabilities
- Analytics Service: User and system analytics
- Export/Import Service: Data migration utilities
- Integration Service: Third-party integrations

## Service Communication Strategy

- **Synchronous Communication**: REST APIs for request/response patterns
- **Asynchronous Communication**: Message queue for event-driven operations
- **Service Discovery**: Use DNS-based service discovery initially, moving to a dedicated service mesh
- **Data Consistency**: Implement eventual consistency with compensating transactions

## Deployment Strategy

- **Containerization**: Docker containers for all services
- **Orchestration**: Kubernetes for container orchestration
- **CI/CD**: Automated pipelines for building, testing, and deploying services
- **Infrastructure as Code**: Terraform for provisioning infrastructure

## Monitoring and Observability

- **Distributed Tracing**: Implement OpenTelemetry for tracing requests across services
- **Metrics Collection**: Prometheus for collecting service metrics
- **Logs Aggregation**: ELK stack for log collection and analysis
- **Alerting**: Set up alerts for service degradation and failures

## Rollout Timeline

- Phase 1: Q2 2025 (Current)
- Phase 2: Q3 2025
- Phase 3: Q4 2025
- Phase 4: Q1 2026
- Phase 5: Q2-Q3 2026
- Phase 6: Q4 2026

This roadmap will be revised as we learn from each phase and adapt to changing requirements.
*/
```

## 10. Advanced Features

### 10.1 Real-time Collaboration

Implementation of real-time collaboration using WebSockets:

```typescript
// lib/realtime/websocket-server.ts
import { Server as HTTPServer } from 'http';
import { Server as SocketIOServer } from 'socket.io';
import { auth } from '@clerk/nextjs/server';
import { verify } from 'jsonwebtoken';
import { logger } from '@/lib/logger/server-logger';

// Socket.IO server
let io: SocketIOServer;

// Initialize WebSocket server
export function initWebSocketServer(httpServer: HTTPServer) {
  io = new SocketIOServer(httpServer, {
    path: '/api/socket',
    cors: {
      origin: process.env.NEXT_PUBLIC_APP_URL,
      methods: ['GET', 'POST'],
    },
  });
  
  // Authentication middleware
  io.use(async (socket, next) => {
    try {
      const token = socket.handshake.auth.token;
      
      if (!token) {
        return next(new Error('Authentication error'));
      }
      
      // Verify JWT token
      const decoded = verify(token, process.env.JWT_SECRET as string);
      
      if (!decoded) {
        return next(new Error('Invalid token'));
      }
      
      // Attach user data to socket
      socket.data.userId = decoded.sub as string;
      socket.data.orgId = decoded.org as string;
      
      next();
    } catch (error) {
      logger.error('WebSocket authentication error', { error });
      next(new Error('Authentication error'));
    }
  });
  
  // Connection handler
  io.on('connection', (socket) => {
    const { userId, orgId } = socket.data;
    
    logger.info('WebSocket connection established', { userId, orgId, socketId: socket.id });
    
    // Join organization room
    socket.join(`org:${orgId}`);
    
    // Handle note room join
    socket.on('join-note', (noteId) => {
      socket.join(`note:${noteId}`);
      
      logger.info('User joined note room', { userId, orgId, noteId });
      
      // Notify others in the room
      socket.to(`note:${noteId}`).emit('user-joined', {
        userId,
        timestamp: new Date().toISOString(),
      });
    });
    
    // Handle note room leave
    socket.on('leave-note', (noteId) => {
      socket.leave(`note:${noteId}`);
      
      logger.info('User left note room', { userId, orgId, noteId });
      
      // Notify others in the room
      socket.to(`note:${noteId}`).emit('user-left', {
        userId,
        timestamp: new Date().toISOString(),
      });
    });
    
    // Handle note changes
    socket.on('note-change', ({ noteId, changes, version }) => {
      // Broadcast changes to others in the note room
      socket.to(`note:${noteId}`).emit('note-updated', {
        noteId,
        changes,
        version,
        userId,
        timestamp: new Date().toISOString(),
      });
      
      logger.info('Note change broadcasted', { userId, orgId, noteId, version });
    });
    
    // Handle cursor position updates
    socket.on('cursor-position', ({ noteId, position }) => {
      // Broadcast cursor position to others in the note room
      socket.to(`note:${noteId}`).emit('cursor-moved', {
        userId,
        position,
        timestamp: new Date().toISOString(),
      });
    });
    
    // Handle disconnection
    socket.on('disconnect', () => {
      logger.info('WebSocket connection closed', { userId, orgId, socketId: socket.id });
    });
  });
  
  logger.info('WebSocket server initialized');
  
  return io;
}

// Get WebSocket server instance
export function getWebSocketServer() {
  if (!io) {
    throw new Error('WebSocket server not initialized');
  }
  
  return io;
}

// Emit event to a room
export function emitToRoom(room: string, event: string, data: any) {
  if (!io) {
    throw new Error('WebSocket server not initialized');
  }
  
  io.to(room).emit(event, data);
  
  logger.info('Event emitted to room', { room, event });
}

// Emit event to an organization
export function emitToOrganization(orgId: string, event: string, data: any) {
  emitToRoom(`org:${orgId}`, event, data);
}

// Emit event to a note room
export function emitToNote(noteId: string, event: string, data: any) {
  emitToRoom(`note:${noteId}`, event, data);
}

// Get active users in a note room
export async function getActiveUsersInNote(noteId: string) {
  if (!io) {
    throw new Error('WebSocket server not initialized');
  }
  
  const sockets = await io.in(`note:${noteId}`).fetchSockets();
  
  return sockets.map((socket) => socket.data.userId);
}
```

### 10.2 Note Version History

Implementation of note version history:

```typescript
// lib/notes/repositories/note-version-repository.ts
import { prisma } from '@/lib/db/prisma';
import { NoteVersion } from '@/lib/notes/types';

export const NoteVersionRepository = {
  // Create a new version
  async createVersion(params: {
    noteId: string;
    title: string;
    content: string;
    createdByUserId: string;
  }): Promise<NoteVersion> {
    const { noteId, title, content, createdByUserId } = params;
    
    const version = await prisma.noteVersion.create({
      data: {
        note_id: noteId,
        title,
        content,
        created_by_user_id: createdByUserId,
      },
    });
    
    return {
      id: version.id,
      noteId: version.note_id,
      title: version.title,
      content: version.content,
      createdByUserId: version.created_by_user_id,
      createdAt: version.created_at.toISOString(),
    };
  },
  
  // Get versions for a note
  async getVersions(
    noteId: string,
    options: {
      limit?: number;
      offset?: number;
    } = {}
  ): Promise<{ versions: NoteVersion[]; total: number }> {
    const { limit = 10, offset = 0 } = options;
    
    const [versions, total] = await Promise.all([
      prisma.noteVersion.findMany({
        where: { note_id: noteId },
        orderBy: { created_at: 'desc' },
        skip: offset,
        take: limit,
      }),
      prisma.noteVersion.count({
        where: { note_id: noteId },
      }),
    ]);
    
    return {
      versions: versions.map((version) => ({
        id: version.id,
        noteId: version.note_id,
        title: version.title,
        content: version.content,
        createdByUserId: version.created_by_user_id,
        createdAt: version.created_at.toISOString(),
      })),
      total,
    };
  },
  
  // Get a specific version
  async getVersion(versionId: string): Promise<NoteVersion | null> {
    const version = await prisma.noteVersion.findUnique({
      where: { id: versionId },
    });
    
    if (!version) {
      return null;
    }
    
    return {
      id: version.id,
      noteId: version.note_id,
      title: version.title,
      content: version.content,
      createdByUserId: version.created_by_user_id,
      createdAt: version.created_at.toISOString(),
    };
  },
  
  // Restore a note to a previous version
  async restoreVersion(versionId: string, userId: string): Promise<boolean> {
    const version = await prisma.noteVersion.findUnique({
      where: { id: versionId },
      select: {
        note_id: true,
        title: true,
        content: true,
      },
    });
    
    if (!version) {
      return false;
    }
    
    // Update the note with the version's content
    await prisma.note.update({
      where: { id: version.note_id },
      data: {
        title: version.title,
        content: version.content,
        updated_at: new Date(),
      },
    });
    
    // Create a new version entry to track the restoration
    await prisma.noteVersion.create({
      data: {
        note_id: version.note_id,
        title: version.title,
        content: version.content,
        created_by_user_id: userId,
      },
    });
    
    return true;
  },
  
  // Compare two versions
  async compareVersions(versionId1: string, versionId2: string): Promise<{
    version1: NoteVersion | null;
    version2: NoteVersion | null;
  }> {
    const [version1, version2] = await Promise.all([
      this.getVersion(versionId1),
      this.getVersion(versionId2),
    ]);
    
    return { version1, version2 };
  },
};
```

### 10.3 Search Indexing

Implementation of full-text search for notes:

```typescript
// lib/search/search-service.ts
import { Client } from '@elastic/elasticsearch';
import { logger } from '@/lib/logger/server-logger';

// Initialize Elasticsearch client
const client = new Client({
  node: process.env.ELASTICSEARCH_URL || 'http://localhost:9200',
  auth: {
    username: process.env.ELASTICSEARCH_USERNAME || 'elastic',
    password: process.env.ELASTICSEARCH_PASSWORD || 'changeme',
  },
  tls: {
    rejectUnauthorized: process.env.NODE_ENV === 'production',
  },
});

// Index name constants
const NOTES_INDEX = 'notes';

// Initialize search indices
export async function initializeSearchIndices() {
  try {
    // Check if notes index exists
    const indexExists = await client.indices.exists({
      index: NOTES_INDEX,
    });
    
    if (!indexExists) {
      // Create notes index with appropriate mappings
      await client.indices.create({
        index: NOTES_INDEX,
        body: {
          settings: {
            analysis: {
              analyzer: {
                html_strip_analyzer: {
                  tokenizer: 'standard',
                  filter: ['lowercase', 'stop', 'snowball'],
                  char_filter: ['html_strip'],
                },
              },
            },
          },
          mappings: {
            properties: {
              id: { type: 'keyword' },
              title: { 
                type: 'text',
                analyzer: 'standard',
                fields: {
                  keyword: { type: 'keyword' },
                },
              },
              content: { 
                type: 'text',
                analyzer: 'html_strip_analyzer',
              },
              organizationId: { type: 'keyword' },
              createdByUserId: { type: 'keyword' },
              accessLevel: { type: 'keyword' },
              createdAt: { type: 'date' },
              updatedAt: { type: 'date' },
            },
          },
        },
      });
      
      logger.info('Created notes search index');
    }
  } catch (error) {
    logger.error('Failed to initialize search indices', { error });
    throw error;
  }
}

// Search service
export class SearchService {
  // Index a note
  static async indexNote(note: {
    id: string;
    title: string;
    content: string;
    organizationId: string;
    createdByUserId: string;
    accessLevel: string;
    createdAt: string;
    updatedAt: string;
  }) {
    try {
      await client.index({
        index: NOTES_INDEX,
        id: note.id,
        body: {
          id: note.id,
          title: note.title,
          content: note.content,
          organizationId: note.organizationId,
          createdByUserId: note.createdByUserId,
          accessLevel: note.accessLevel,
          createdAt: note.createdAt,
          updatedAt: note.updatedAt,
        },
        refresh: true,
      });
      
      logger.info('Indexed note', { noteId: note.id });
      
      return true;
    } catch (error) {
      logger.error('Failed to index note', { error, noteId: note.id });
      return false;
    }
  }
  
  // Update note index
  static async updateNoteIndex(noteId: string, updates: {
    title?: string;
    content?: string;
    accessLevel?: string;
    updatedAt?: string;
  }) {
    try {
      const updateBody: any = {};
      
      if (updates.title !== undefined) {
        updateBody.title = updates.title;
      }
      
      if (updates.content !== undefined) {
        updateBody.content = updates.content;
      }
      
      if (updates.accessLevel !== undefined) {
        updateBody.accessLevel = updates.accessLevel;
      }
      
      if (updates.updatedAt !== undefined) {
        updateBody.updatedAt = updates.updatedAt;
      }
      
      await client.update({
        index: NOTES_INDEX,
        id: noteId,
        body: {
          doc: updateBody,
        },
        refresh: true,
      });
      
      logger.info('Updated note index', { noteId });
      
      return true;
    } catch (error) {
      logger.error('Failed to update note index', { error, noteId });
      return false;
    }
  }
  
  // Remove note from index
  static async removeNoteFromIndex(noteId: string) {
    try {
      await client.delete({
        index: NOTES_INDEX,
        id: noteId,
        refresh: true,
      });
      
      logger.info('Removed note from index', { noteId });
      
      return true;
    } catch (error) {
      logger.error('Failed to remove note from index', { error, noteId });
      return false;
    }
  }
  
  // Search notes
  static async searchNotes(params: {
    query: string;
    organizationId: string;
    userId: string;
    accessLevels?: string[];
    page?: number;
    perPage?: number;
    sortBy?: string;
    sortDirection?: 'asc' | 'desc';
  }) {
    const {
      query,
      organizationId,
      userId,
      accessLevels = ['public', 'private', 'shared'],
      page = 1,
      perPage = 10,
      sortBy = 'updatedAt',
      sortDirection = 'desc',
    } = params;
    
    try {
      // Build access control filter
      const accessFilter: any[] = [
        // Public notes are visible to all organization members
        { term: { accessLevel: 'public' } },
        // Private notes are only visible to creator
        {
          bool: {
            must: [
              { term: { accessLevel: 'private' } },
              { term: { createdByUserId: userId } },
            ],
          },
        },
        // For shared notes, more complex logic is handled at the application level
        // So we just include them in the results for filtering later
        { term: { accessLevel: 'shared' } },
      ];
      
      // Build search query
      const searchResponse = await client.search({
        index: NOTES_INDEX,
        body: {
          from: (page - 1) * perPage,
          size: perPage,
          query: {
            bool: {
              must: [
                {
                  multi_match: {
                    query,
                    fields: ['title^3', 'content'],
                    fuzziness: 'AUTO',
                  },
                },
                { term: { organizationId } },
              ],
              should: accessFilter,
              minimum_should_match: 1,
              filter: [
                { terms: { accessLevel: accessLevels } },
              ],
            },
          },
          sort: [
            { [sortBy]: { order: sortDirection } },
          ],
          highlight: {
            fields: {
              title: { number_of_fragments: 0 },
              content: { number_of_fragments: 3, fragment_size: 150 },
            },
            pre_tags: ['<mark>'],
            post_tags: ['</mark>'],
          },
        },
      });
      
      const hits = searchResponse.hits.hits;
      const total = searchResponse.hits.total.value;
      
      const results = hits.map(hit => ({
        id: hit._source.id,
        title: hit._source.title,
        content: hit._source.content,
        createdByUserId: hit._source.createdByUserId,
        organizationId: hit._source.organizationId,
        accessLevel: hit._source.accessLevel,
        createdAt: hit._source.createdAt,
        updatedAt: hit._source.updatedAt,
        score: hit._score,
        highlights: hit.highlight || {},
      }));
      
      logger.info('Search notes', {
        query,
        organizationId,
        results: results.length,
        total,
      });
      
      return {
        results,
        pagination: {
          page,
          perPage,
          total,
          totalPages: Math.ceil(total / perPage),
        },
      };
    } catch (error) {
      logger.error('Failed to search notes', { error, query, organizationId });
      throw error;
    }
  }
}

// Initialize indices on startup
if (process.env.NODE_ENV === 'production') {
  initializeSearchIndices().catch(error => {
    logger.error('Failed to initialize search indices', { error });
  });
}
```

### 10.4 Analytics and Reporting

Implementation of analytics and reporting capabilities:

```typescript
// lib/analytics/analytics-service.ts
import { prisma } from '@/lib/db/prisma';
import { format, addDays, startOfDay, endOfDay, subDays } from 'date-fns';
import { logger } from '@/lib/logger/server-logger';

export class AnalyticsService {
  // Get note creation analytics
  static async getNoteCreationAnalytics(params: {
    organizationId: string;
    startDate: Date;
    endDate: Date;
    groupBy: 'day' | 'week' | 'month';
  }) {
    const { organizationId, startDate, endDate, groupBy } = params;
    
    try {
      // Query for note creation counts
      let result;
      
      if (groupBy === 'day') {
        // Group by day
        result = await prisma.$queryRaw`
          SELECT
            DATE(created_at) as date,
            COUNT(*) as count
          FROM "Note"
          WHERE organization_id = ${organizationId}
            AND created_at >= ${startDate}
            AND created_at <= ${endDate}
          GROUP BY DATE(created_at)
          ORDER BY date ASC
        `;
      } else if (groupBy === 'week') {
        // Group by week
        result = await prisma.$queryRaw`
          SELECT
            DATE_TRUNC('week', created_at) as date,
            COUNT(*) as count
          FROM "Note"
          WHERE organization_id = ${organizationId}
            AND created_at >= ${startDate}
            AND created_at <= ${endDate}
          GROUP BY DATE_TRUNC('week', created_at)
          ORDER BY date ASC
        `;
      } else {
        // Group by month
        result = await prisma.$queryRaw`
          SELECT
            DATE_TRUNC('month', created_at) as date,
            COUNT(*) as count
          FROM "Note"
          WHERE organization_id = ${organizationId}
            AND created_at >= ${startDate}
            AND created_at <= ${endDate}
          GROUP BY DATE_TRUNC('month', created_at)
          ORDER BY date ASC
        `;
      }
      
      // Format the results
      const formattedResult = Array.isArray(result) ? result.map((row: any) => ({
        date: row.date instanceof Date ? format(row.date, 'yyyy-MM-dd') : row.date,
        count: Number(row.count),
      })) : [];
      
      return formattedResult;
    } catch (error) {
      logger.error('Failed to get note creation analytics', { error, organizationId });
      throw error;
    }
  }
  
  // Get user activity analytics
  static async getUserActivityAnalytics(params: {
    organizationId: string;
    days?: number;
  }) {
    const { organizationId, days = 30 } = params;
    
    try {
      const startDate = subDays(new Date(), days);
      
      // Query for user activity counts from audit logs
      const result = await prisma.$queryRaw`
        SELECT
          user_id,
          COUNT(*) as activity_count
        FROM "Audit"
        WHERE organization_id = ${organizationId}
          AND created_at >= ${startDate}
        GROUP BY user_id
        ORDER BY activity_count DESC
      `;
      
      // Format the results
      const formattedResult = Array.isArray(result) ? result.map((row: any) => ({
        userId: row.user_id,
        activityCount: Number(row.activity_count),
      })) : [];
      
      return formattedResult;
    } catch (error) {
      logger.error('Failed to get user activity analytics', { error, organizationId });
      throw error;
    }
  }
  
  // Get note access analytics
  static async getNoteAccessAnalytics(params: {
    organizationId: string;
    days?: number;
  }) {
    const { organizationId, days = 30 } = params;
    
    try {
      const startDate = subDays(new Date(), days);
      
      // Query for note access counts from audit logs
      const result = await prisma.$queryRaw`
        SELECT
          resource_id as note_id,
          COUNT(*) as access_count
        FROM "Audit"
        WHERE organization_id = ${organizationId}
          AND resource_type = 'note'
          AND created_at >= ${startDate}
        GROUP BY resource_id
        ORDER BY access_count DESC
        LIMIT 10
      `;
      
      // Format the results
      const formattedResult = Array.isArray(result) ? result.map((row: any) => ({
        noteId: row.note_id,
        accessCount: Number(row.access_count),
      })) : [];
      
      return formattedResult;
    } catch (error) {
      logger.error('Failed to get note access analytics', { error, organizationId });
      throw error;
    }
  }
  
  // Get note access level distribution
  static async getNoteAccessLevelDistribution(params: {
    organizationId: string;
  }) {
    const { organizationId } = params;
    
    try {
      // Query for note access level distribution
      const result = await prisma.$queryRaw`
        SELECT
          access_level,
          COUNT(*) as count
        FROM "Note"
        WHERE organization_id = ${organizationId}
        GROUP BY access_level
      `;
      
      // Format the results
      const formattedResult = Array.isArray(result) ? result.map((row: any) => ({
        accessLevel: row.access_level,
        count: Number(row.count),
      })) : [];
      
      return formattedResult;
    } catch (error) {
      logger.error('Failed to get note access level distribution', { error, organizationId });
      throw error;
    }
  }
  
  // Generate organization report
  static async generateOrganizationReport(params: {
    organizationId: string;
    startDate: Date;
    endDate: Date;
  }) {
    const { organizationId, startDate, endDate } = params;
    
    try {
      // Collect all report data in parallel
      const [
        totalNotes,
        notesCreatedInPeriod,
        totalUsers,
        activeUsers,
        accessLevelDistribution,
        noteCreationTrend,
        mostAccessedNotes,
        mostActiveUsers,
      ] = await Promise.all([
        // Total notes in organization
        prisma.note.count({
          where: { organization_id: organizationId },
        }),
        
        // Notes created in the period
        prisma.note.count({
          where: {
            organization_id: organizationId,
            created_at: {
              gte: startDate,
              lte: endDate,
            },
          },
        }),
        
        // Total users (this would come from Clerk in a real implementation)
        Promise.resolve(0), // Placeholder
        
        // Active users in period (from audit logs)
        prisma.$queryRaw`
          SELECT COUNT(DISTINCT user_id) as count
          FROM "Audit"
          WHERE organization_id = ${organizationId}
            AND created_at >= ${startDate}
            AND created_at <= ${endDate}
        `.then((result: any) => Number(result[0]?.count || 0)),
        
        // Access level distribution
        this.getNoteAccessLevelDistribution({ organizationId }),
        
        // Note creation trend
        this.getNoteCreationAnalytics({
          organizationId,
          startDate,
          endDate,
          groupBy: 'day',
        }),
        
        // Most accessed notes
        this.getNoteAccessAnalytics({
          organizationId,
          days: Math.ceil((endDate.getTime() - startDate.getTime()) / (1000 * 60 * 60 * 24)),
        }),
        
        // Most active users
        this.getUserActivityAnalytics({
          organizationId,
          days: Math.ceil((endDate.getTime() - startDate.getTime()) / (1000 * 60 * 60 * 24)),
        }),
      ]);
      
      // Compile the report
      const report = {
        organizationId,
        period: {
          startDate: format(startDate, 'yyyy-MM-dd'),
          endDate: format(endDate, 'yyyy-MM-dd'),
        },
        summary: {
          totalNotes,
          notesCreatedInPeriod,
          totalUsers,
          activeUsers,
        },
        details: {
          accessLevelDistribution,
          noteCreationTrend,
          mostAccessedNotes,
          mostActiveUsers,
        },
        generatedAt: new Date().toISOString(),
      };
      
      return report;
    } catch (error) {
      logger.error('Failed to generate organization report', { error, organizationId });
      throw error;
    }
  }
}
```

## 11. Deployment and DevOps

### 11.1 Production Dockerfile

```dockerfile
# Dockerfile.production
FROM node:20-alpine AS base

# Install dependencies only when needed
FROM base AS deps
WORKDIR /app

# Copy package.json files for dependency installation
COPY package.json package-lock.json ./

# Install dependencies
RUN npm ci

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Set environment variables for build
ENV NEXT_TELEMETRY_DISABLED 1
ENV NODE_ENV production

# Build the application
RUN npm run build

# Production image, copy all the files and run
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

# Create a non-root user to run the app
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Copy necessary files
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

# Set user to non-root
USER nextjs

# Expose port
EXPOSE 3000

# Set healthcheck endpoint
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 CMD [ "wget", "-qO-", "http://localhost:3000/api/health" ]

# Run the application
CMD ["node", "server.js"]
```

### 11.2 Kubernetes Deployment

```yaml
# kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: noteorbit-app
  namespace: noteorbit
  labels:
    app: noteorbit
    component: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: noteorbit
      component: app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: noteorbit
        component: app
    spec:
      containers:
        - name: app
          image: noteorbit/app:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 3000
              name: http
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: noteorbit-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: noteorbit-secrets
                  key: redis-url
            - name: CLERK_SECRET_KEY
              valueFrom:
                secretKeyRef:
                  name: noteorbit-secrets
                  key: clerk-secret-key
            - name: NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY
              valueFrom:
                configMapKeyRef:
                  name: noteorbit-config
                  key: clerk-publishable-key
            - name: JWT_SECRET
              valueFrom:
                secretKeyRef:
                  name: noteorbit-secrets
                  key: jwt-secret
            - name: NODE_ENV
              value: "production"
          resources:
            requests:
              cpu: 100m
              memory: 256Mi
            limits:
              cpu: 500m
              memory: 512Mi
          livenessProbe:
            httpGet:
              path: /api/health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /api/health
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
            timeoutSeconds: 3
          startupProbe:
            httpGet:
              path: /api/health
              port: http
            failureThreshold: 30
            periodSeconds: 10
      imagePullSecrets:
        - name: dockerhub-secret
```

### 11.3 CI/CD Pipeline

```yaml
# .github/workflows/ci-cd.yml
name: NoteOrbit CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    name: Lint Code
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Lint code
        run: npm run lint

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: noteorbit_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
          
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Generate Prisma client
        run: npx prisma generate
        
      - name: Run tests
        run: npm test
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/noteorbit_test
          
  build:
    name: Build Application
    needs: [lint, test]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
        
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
          
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          file: Dockerfile.production
          push: true
          tags: |
            noteorbit/app:latest
            noteorbit/app:${{ github.sha }}
          cache-from: type=registry,ref=noteorbit/app:buildcache
          cache-to: type=registry,ref=noteorbit/app:buildcache,mode=max
          
  deploy:
    name: Deploy to Production
    needs: [build]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Set up Kubernetes config
        uses: azure/k8s-set-context@v1
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG }}
          
      - name: Update deployment image
        run: |
          kubectl set image deployment/noteorbit-app app=noteorbit/app:${{ github.sha }} -n noteorbit
          
      - name: Wait for deployment
        run: |
          kubectl rollout status deployment/noteorbit-app -n noteorbit --timeout=300s