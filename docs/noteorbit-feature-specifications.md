# NoteOrbit Feature Specifications

## 1. Authentication and User Management

### 1.1 Sign Up Process

#### 1.1.1 Description
The sign-up process allows new users to create an account in the NoteOrbit application using Clerk's authentication system.

#### 1.1.2 UI Components
- Sign-up form with email, password, and name fields
- Email verification step
- Organization creation step
- Success/error messages

#### 1.1.3 Data Requirements
- User email (required, must be valid email format)
- User password (required, minimum 8 characters, must include letters and numbers)
- First name (required)
- Last name (required)
- Organization name (required during onboarding)

#### 1.1.4 Implementation Details

**Client-Side Implementation:**
```tsx
// app/auth/sign-up/page.tsx
import { SignUp } from '@clerk/nextjs';
import Link from 'next/link';

export default function SignUpPage() {
  return (
    <div className="flex min-h-screen flex-col items-center justify-center bg-gray-50 px-4 py-12">
      <div className="w-full max-w-md">
        <div className="mb-8 text-center">
          <h1 className="text-3xl font-bold text-gray-900">Create your account</h1>
          <p className="mt-2 text-sm text-gray-600">
            Already have an account?{' '}
            <Link href="/auth/sign-in" className="font-medium text-indigo-600 hover:text-indigo-500">
              Sign in
            </Link>
          </p>
        </div>
        
        <div className="rounded-lg bg-white p-8 shadow">
          <SignUp 
            path="/auth/sign-up"
            routing="path"
            signInUrl="/auth/sign-in"
            redirectUrl="/onboarding"
            appearance={{
              elements: {
                formButtonPrimary: 
                  'bg-indigo-600 hover:bg-indigo-700 text-white rounded-md px-4 py-2 w-full',
                card: 'bg-white shadow-none',
                headerTitle: 'text-2xl font-semibold text-gray-900',
                headerSubtitle: 'text-gray-600',
              }
            }}
          />
        </div>
      </div>
    </div>
  );
}
```

**Onboarding Flow:**
```tsx
// app/onboarding/page.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { useUser, useClerk } from '@clerk/nextjs';

export default function OnboardingPage() {
  const { user, isLoaded } = useUser();
  const { createOrganization } = useClerk();
  const router = useRouter();
  const [orgName, setOrgName] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  if (!isLoaded) {
    return <div className="flex min-h-screen items-center justify-center">Loading...</div>;
  }
  
  if (!user) {
    router.push('/auth/sign-in');
    return null;
  }
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);
    setError(null);
    
    try {
      // Create organization
      const organization = await createOrganization({ name: orgName });
      
      // Redirect to dashboard
      router.push(`/dashboard?org_id=${organization.id}`);
    } catch (error) {
      setError('Failed to create organization. Please try again.');
      console.error('Organization creation error:', error);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <div className="flex min-h-screen flex-col items-center justify-center bg-gray-50 px-4 py-12">
      <div className="w-full max-w-md">
        <div className="mb-8 text-center">
          <h1 className="text-3xl font-bold text-gray-900">Create your workspace</h1>
          <p className="mt-2 text-sm text-gray-600">
            Let's set up your organization to get started with NoteOrbit.
          </p>
        </div>
        
        <div className="rounded-lg bg-white p-8 shadow">
          <form onSubmit={handleSubmit}>
            <div className="mb-6">
              <label htmlFor="orgName" className="block text-sm font-medium text-gray-700">
                Organization Name
              </label>
              <input
                id="orgName"
                type="text"
                value={orgName}
                onChange={(e) => setOrgName(e.target.value)}
                className="mt-1 block w-full rounded-md border border-gray-300 px-3 py-2 shadow-sm focus:border-indigo-500 focus:outline-none focus:ring-indigo-500"
                placeholder="Your Company Name"
                required
              />
            </div>
            
            {error && (
              <div className="mb-4 rounded-md bg-red-50 p-4">
                <p className="text-sm text-red-700">{error}</p>
              </div>
            )}
            
            <button
              type="submit"
              className="w-full rounded-md bg-indigo-600 px-4 py-2 text-white hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:bg-indigo-400"
              disabled={isSubmitting || !orgName.trim()}
            >
              {isSubmitting ? 'Creating...' : 'Create Organization'}
            </button>
          </form>
        </div>
      </div>
    </div>
  );
}
```

#### 1.1.5 Error Handling
- Email already in use: Show Clerk error message and suggest sign-in
- Invalid email format: Show validation error
- Weak password: Show password strength requirements
- Failed organization creation: Show error message with retry option

#### 1.1.6 Testing Requirements
- Test valid sign-up flow
- Test sign-up with existing email
- Test password validation
- Test organization creation
- Test redirect to dashboard after successful sign-up

### 1.2 Sign In Process

#### 1.2.1 Description
The sign-in process allows existing users to authenticate and access the NoteOrbit application.

#### 1.2.2 UI Components
- Sign-in form with email and password fields
- "Remember me" option
- Password reset link
- Sign-up link
- Organization switcher after sign-in

#### 1.2.3 Data Requirements
- User email (required, must be valid email format)
- User password (required)

#### 1.2.4 Implementation Details

**Client-Side Implementation:**
```tsx
// app/auth/sign-in/page.tsx
import { SignIn } from '@clerk/nextjs';
import Link from 'next/link';

export default function SignInPage() {
  return (
    <div className="flex min-h-screen flex-col items-center justify-center bg-gray-50 px-4 py-12">
      <div className="w-full max-w-md">
        <div className="mb-8 text-center">
          <h1 className="text-3xl font-bold text-gray-900">Welcome back</h1>
          <p className="mt-2 text-sm text-gray-600">
            Don't have an account?{' '}
            <Link href="/auth/sign-up" className="font-medium text-indigo-600 hover:text-indigo-500">
              Sign up
            </Link>
          </p>
        </div>
        
        <div className="rounded-lg bg-white p-8 shadow">
          <SignIn 
            path="/auth/sign-in"
            routing="path"
            signUpUrl="/auth/sign-up"
            redirectUrl="/dashboard"
            appearance={{
              elements: {
                formButtonPrimary: 
                  'bg-indigo-600 hover:bg-indigo-700 text-white rounded-md px-4 py-2 w-full',
                card: 'bg-white shadow-none',
                headerTitle: 'text-2xl font-semibold text-gray-900',
                headerSubtitle: 'text-gray-600',
              }
            }}
          />
        </div>
      </div>
    </div>
  );
}
```

**Organization Selection After Sign-In:**
```tsx
// lib/components/organization-switcher.tsx
'use client';

import { OrganizationSwitcher } from '@clerk/nextjs';
import { useRouter } from 'next/navigation';

export function CustomOrganizationSwitcher() {
  const router = useRouter();
  
  return (
    <OrganizationSwitcher
      appearance={{
        elements: {
          rootBox: 'w-full',
          organizationSwitcherTrigger: 'w-full flex justify-between items-center rounded-md border border-gray-300 px-3 py-2 text-left shadow-sm focus:border-indigo-500 focus:outline-none focus:ring-indigo-500',
        }
      }}
      hidePersonal
      createOrganizationUrl="/org/create"
      afterCreateOrganizationUrl="/dashboard"
      afterSelectOrganizationUrl="/dashboard"
      afterLeaveOrganizationUrl="/org/create"
    />
  );
}
```

#### 1.2.5 Error Handling
- Invalid credentials: Show Clerk error message
- Account not found: Suggest sign-up
- User has no organizations: Redirect to organization creation
- Server errors: Show general error message with retry option

#### 1.2.6 Testing Requirements
- Test valid sign-in flow
- Test sign-in with invalid credentials
- Test sign-in with non-existent account
- Test "Remember me" functionality
- Test organization switching after sign-in

### 1.3 Password Reset

#### 1.3.1 Description
The password reset flow allows users to recover access to their accounts by setting a new password.

#### 1.3.2 UI Components
- Password reset request form
- Email verification
- New password form
- Success/error messages

#### 1.3.3 Data Requirements
- User email (for reset request)
- New password (for password update)

#### 1.3.4 Implementation Details

**Password Reset Request:**
```tsx
// app/auth/reset-password/page.tsx
import { useState } from 'react';
import { useSignIn } from '@clerk/nextjs';
import Link from 'next/link';

export default function ResetPasswordPage() {
  const { signIn, isLoaded } = useSignIn();
  const [email, setEmail] = useState('');
  const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
  const [errorMessage, setErrorMessage] = useState('');
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!isLoaded) return;
    
    setStatus('loading');
    setErrorMessage('');
    
    try {
      await signIn.create({
        strategy: 'reset_password_email_code',
        identifier: email,
      });
      setStatus('success');
    } catch (error) {
      console.error('Password reset error:', error);
      setStatus('error');
      setErrorMessage('Failed to request password reset. Please try again.');
    }
  };
  
  return (
    <div className="flex min-h-screen flex-col items-center justify-center bg-gray-50 px-4 py-12">
      <div className="w-full max-w-md">
        <div className="mb-8 text-center">
          <h1 className="text-3xl font-bold text-gray-900">Reset your password</h1>
          <p className="mt-2 text-sm text-gray-600">
            Enter your email address and we'll send you a link to reset your password.
          </p>
        </div>
        
        <div className="rounded-lg bg-white p-8 shadow">
          {status === 'success' ? (
            <div className="text-center">
              <div className="mb-4 text-green-600">
                <CheckCircleIcon className="mx-auto h-12 w-12" />
              </div>
              <h2 className="text-lg font-medium text-gray-900">Check your email</h2>
              <p className="mt-2 text-sm text-gray-600">
                We've sent a password reset link to your email address.
              </p>
              <Link href="/auth/sign-in" className="mt-4 block text-sm font-medium text-indigo-600 hover:text-indigo-500">
                Back to sign in
              </Link>
            </div>
          ) : (
            <form onSubmit={handleSubmit}>
              <div className="mb-6">
                <label htmlFor="email" className="block text-sm font-medium text-gray-700">
                  Email address
                </label>
                <input
                  id="email"
                  type="email"
                  value={email}
                  onChange={(e) => setEmail(e.target.value)}
                  className="mt-1 block w-full rounded-md border border-gray-300 px-3 py-2 shadow-sm focus:border-indigo-500 focus:outline-none focus:ring-indigo-500"
                  placeholder="you@example.com"
                  required
                />
              </div>
              
              {status === 'error' && (
                <div className="mb-4 rounded-md bg-red-50 p-4">
                  <p className="text-sm text-red-700">{errorMessage}</p>
                </div>
              )}
              
              <button
                type="submit"
                className="w-full rounded-md bg-indigo-600 px-4 py-2 text-white hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:bg-indigo-400"
                disabled={status === 'loading' || !email.trim()}
              >
                {status === 'loading' ? 'Sending...' : 'Reset Password'}
              </button>
              
              <div className="mt-4 text-center">
                <Link href="/auth/sign-in" className="text-sm font-medium text-indigo-600 hover:text-indigo-500">
                  Back to sign in
                </Link>
              </div>
            </form>
          )}
        </div>
      </div>
    </div>
  );
}
```

**Reset Password Completion:**
```tsx
// app/auth/reset-password/[token]/page.tsx
import { useState } from 'react';
import { useSignIn } from '@clerk/nextjs';
import { useRouter } from 'next/navigation';

export default function ResetPasswordCompletePage({
  params,
  searchParams,
}: {
  params: { token: string };
  searchParams: { email: string };
}) {
  const { signIn, isLoaded } = useSignIn();
  const router = useRouter();
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
  const [errorMessage, setErrorMessage] = useState('');
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!isLoaded) return;
    
    if (password !== confirmPassword) {
      setErrorMessage('Passwords do not match');
      return;
    }
    
    setStatus('loading');
    setErrorMessage('');
    
    try {
      const result = await signIn.attemptFirstFactor({
        strategy: 'reset_password_email_code',
        code: params.token,
        password,
      });
      
      if (result.status === 'complete') {
        setStatus('success');
        // Redirect to sign-in page after a short delay
        setTimeout(() => {
          router.push('/auth/sign-in');
        }, 2000);
      }
    } catch (error) {
      console.error('Password reset error:', error);
      setStatus('error');
      setErrorMessage('Failed to reset password. Please try again or request a new link.');
    }
  };
  
  return (
    <div className="flex min-h-screen flex-col items-center justify-center bg-gray-50 px-4 py-12">
      <div className="w-full max-w-md">
        <div className="mb-8 text-center">
          <h1 className="text-3xl font-bold text-gray-900">Set new password</h1>
          <p className="mt-2 text-sm text-gray-600">
            Enter your new password to reset your account.
          </p>
        </div>
        
        <div className="rounded-lg bg-white p-8 shadow">
          {status === 'success' ? (
            <div className="text-center">
              <div className="mb-4 text-green-600">
                <CheckCircleIcon className="mx-auto h-12 w-12" />
              </div>
              <h2 className="text-lg font-medium text-gray-900">Password reset complete</h2>
              <p className="mt-2 text-sm text-gray-600">
                Your password has been reset successfully. You'll be redirected to sign in.
              </p>
            </div>
          ) : (
            <form onSubmit={handleSubmit}>
              <div className="mb-4">
                <label htmlFor="password" className="block text-sm font-medium text-gray-700">
                  New Password
                </label>
                <input
                  id="password"
                  type="password"
                  value={password}
                  onChange={(e) => setPassword(e.target.value)}
                  className="mt-1 block w-full rounded-md border border-gray-300 px-3 py-2 shadow-sm focus:border-indigo-500 focus:outline-none focus:ring-indigo-500"
                  required
                  minLength={8}
                />
              </div>
              
              <div className="mb-6">
                <label htmlFor="confirmPassword" className="block text-sm font-medium text-gray-700">
                  Confirm Password
                </label>
                <input
                  id="confirmPassword"
                  type="password"
                  value={confirmPassword}
                  onChange={(e) => setConfirmPassword(e.target.value)}
                  className="mt-1 block w-full rounded-md border border-gray-300 px-3 py-2 shadow-sm focus:border-indigo-500 focus:outline-none focus:ring-indigo-500"
                  required
                  minLength={8}
                />
              </div>
              
              {errorMessage && (
                <div className="mb-4 rounded-md bg-red-50 p-4">
                  <p className="text-sm text-red-700">{errorMessage}</p>
                </div>
              )}
              
              <button
                type="submit"
                className="w-full rounded-md bg-indigo-600 px-4 py-2 text-white hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:bg-indigo-400"
                disabled={status === 'loading' || !password || !confirmPassword}
              >
                {status === 'loading' ? 'Resetting...' : 'Reset Password'}
              </button>
            </form>
          )}
        </div>
      </div>
    </div>
  );
}
```

#### 1.3.5 Error Handling
- Email not found: Show error message
- Invalid or expired token: Show error with option to request new link
- Password requirements not met: Show validation error
- Passwords don't match: Show validation error

#### 1.3.6 Testing Requirements
- Test password reset request
- Test invalid email handling
- Test token validation
- Test password requirements validation
- Test successful password reset

### 1.4 User Profile Management

#### 1.4.1 Description
User profile management allows users to update their personal information, including name, email, and password.

#### 1.4.2 UI Components
- Profile information form
- Avatar upload
- Email management
- Password update form

#### 1.4.3 Data Requirements
- User name (first and last)
- User email
- User password (for changes)
- User avatar image

#### 1.4.4 Implementation Details

**Profile Page:**
```tsx
// app/profile/page.tsx
import { UserProfile } from '@clerk/nextjs';
import { PageTitle } from '@/components/layout/page-title';

export default function ProfilePage() {
  return (
    <div className="container mx-auto px-4 py-8">
      <PageTitle title="Your Profile" />
      
      <div className="mt-8">
        <UserProfile
          path="/profile"
          routing="path"
          appearance={{
            elements: {
              card: 'rounded-lg border border-gray-200 bg-white shadow-sm',
              navbar: 'bg-white border-b border-gray-200',
              navbarButton: 'text-gray-700 hover:text-indigo-600',
              navbarButtonActive: 'text-indigo-600 border-b-2 border-indigo-600',
              formButtonPrimary: 'bg-indigo-600 hover:bg-indigo-700 text-white',
              formButtonReset: 'border border-gray-300 text-gray-700 hover:bg-gray-50',
              headerTitle: 'text-xl font-semibold text-gray-900',
              headerSubtitle: 'text-gray-600',
            }
          }}
        />
      </div>
    </div>
  );
}
```

#### 1.4.5 Error Handling
- Email already in use: Show error message
- Invalid current password: Show error message
- Weak password: Show password requirements
- Avatar upload failure: Show error with retry option

#### 1.4.6 Testing Requirements
- Test profile information update
- Test email update
- Test password update
- Test avatar upload

## 2. Organization Management

### 2.1 Organization Creation

#### 2.1.1 Description
Organization creation allows users to create a new organization within the NoteOrbit application, which acts as a workspace for notes and collaborators.

#### 2.1.2 UI Components
- Organization creation form
- Organization logo upload
- Organization settings section

#### 2.1.3 Data Requirements
- Organization name (required)
- Organization logo (optional)
- Organization settings (optional)

#### 2.1.4 Implementation Details

**Organization Creation Page:**
```tsx
// app/org/create/page.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { useClerk } from '@clerk/nextjs';
import { PageTitle } from '@/components/layout/page-title';

export default function CreateOrganizationPage() {
  const { createOrganization } = useClerk();
  const router = useRouter();
  const [name, setName] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!name.trim()) {
      return;
    }
    
    setIsSubmitting(true);
    setError(null);
    
    try {
      const organization = await createOrganization({ name });
      router.push(`/dashboard?org_id=${organization.id}`);
    } catch (error) {
      console.error('Organization creation error:', error);
      setError('Failed to create organization. Please try again.');
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <div className="container mx-auto px-4 py-8">
      <PageTitle title="Create Organization" />
      
      <div className="mx-auto mt-8 max-w-2xl">
        <div className="rounded-lg border border-gray-200 bg-white p-8 shadow-sm">
          <form onSubmit={handleSubmit}>
            <div className="mb-6">
              <label htmlFor="name" className="block text-sm font-medium text-gray-700">
                Organization Name
              </label>
              <input
                id="name"
                type="text"
                value={name}
                onChange={(e) => setName(e.target.value)}
                className="mt-1 block w-full rounded-md border border-gray-300 px-3 py-2 shadow-sm focus:border-indigo-500 focus:outline-none focus:ring-indigo-500"
                placeholder="Your Company Name"
                required
              />
            </div>
            
            {error && (
              <div className="mb-4 rounded-md bg-red-50 p-4">
                <p className="text-sm text-red-700">{error}</p>
              </div>
            )}
            
            <button
              type="submit"
              className="w-full rounded-md bg-indigo-600 px-4 py-2 text-white hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:bg-indigo-400"
              disabled={isSubmitting || !name.trim()}
            >
              {isSubmitting ? 'Creating...' : 'Create Organization'}
            </button>
          </form>
        </div>
      </div>
    </div>
  );
}
```

#### 2.1.5 Error Handling
- Name validation: Ensure name is not empty
- Organization creation failure: Show error message with retry option
- Duplicate organization name: Show error message with suggestion to use a different name

#### 2.1.6 Testing Requirements
- Test organization creation with valid name
- Test organization creation with empty name
- Test logo upload functionality
- Test redirection to dashboard after creation

### 2.2 Organization Settings Management

#### 2.2.1 Description
Organization settings management allows organization administrators to modify organization details and settings.

#### 2.2.2 UI Components
- Organization profile form
- Organization logo management
- Organization settings tabs
- Save/cancel buttons

#### 2.2.3 Data Requirements
- Organization name
- Organization logo
- Organization settings

#### 2.2.4 Implementation Details

**Organization Settings Page:**
```tsx
// app/org/page.tsx
import { OrganizationProfile } from '@clerk/nextjs';
import { PageTitle } from '@/components/layout/page-title';

export default function OrganizationSettingsPage() {
  return (
    <div className="container mx-auto px-4 py-8">
      <PageTitle title="Organization Settings" />
      
      <div className="mt-8">
        <OrganizationProfile
          appearance={{
            elements: {
              card: 'rounded-lg border border-gray-200 bg-white shadow-sm',
              navbar: 'bg-white border-b border-gray-200',
              navbarButton: 'text-gray-700 hover:text-indigo-600',
              navbarButtonActive: 'text-indigo-600 border-b-2 border-indigo-600',
              formButtonPrimary: 'bg-indigo-600 hover:bg-indigo-700 text-white',
              formButtonReset: 'border border-gray-300 text-gray-700 hover:bg-gray-50',
              headerTitle: 'text-xl font-semibold text-gray-900',
              headerSubtitle: 'text-gray-600',
            }
          }}
        />
      </div>
    </div>
  );
}
```

#### 2.2.5 Error Handling
- Name validation: Ensure name is not empty
- Organization update failure: Show error message with retry option
- Logo upload failure: Show error message with retry option

#### 2.2.6 Testing Requirements
- Test organization name update
- Test organization logo update
- Test settings update
- Test admin-only access restriction

### 2.3 Member Management

#### 2.3.1 Description
Member management allows organization administrators to view, invite, and manage organization members.

#### 2.3.2 UI Components
- Members list with role and status
- Role selector for each member
- Invite form
- Remove member button
- Pagination for large member lists

#### 2.3.3 Data Requirements
- Member information (name, email, role, status)
- Invitee email and role
- Pagination parameters

#### 2.3.4 Implementation Details

**Members List Component:**
```tsx
// lib/org/components/members-list.tsx
'use client';

import { useState, useEffect } from 'react';
import { useOrganization, useClerk } from '@clerk/nextjs';
import { RoleSelector } from './role-selector';
import { Avatar } from '@/components/ui/avatar';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Pagination } from '@/components/data/pagination';
import { EmptyState } from '@/components/data/empty-state';
import { Skeleton } from '@/components/ui/skeleton';

export function MembersList() {
  const { organization, membershipList, isLoaded } = useOrganization({
    membershipList: true,
  });
  const [currentPage, setCurrentPage] = useState(1);
  const [itemsPerPage] = useState(10);
  
  if (!isLoaded) {
    return <MembersListSkeleton />;
  }
  
  if (!organization || !membershipList) {
    return (
      <EmptyState
        title="No organization selected"
        description="Please select an organization to view its members."
        icon={UserGroupIcon}
      />
    );
  }
  
  if (membershipList.length === 0) {
    return (
      <EmptyState
        title="No members yet"
        description="Invite team members to collaborate in this organization."
        icon={UserGroupIcon}
        action={{
          label: 'Invite Members',
          href: '/org/invite',
        }}
      />
    );
  }
  
  // Calculate pagination
  const indexOfLastItem = currentPage * itemsPerPage;
  const indexOfFirstItem = indexOfLastItem - itemsPerPage;
  const currentItems = membershipList.slice(indexOfFirstItem, indexOfLastItem);
  const totalPages = Math.ceil(membershipList.length / itemsPerPage);
  
  return (
    <div className="space-y-6">
      <div className="overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider text-gray-500">
                Member
              </th>
              <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider text-gray-500">
                Role
              </th>
              <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider text-gray-500">
                Status
              </th>
              <th scope="col" className="px-6 py-3 text-right text-xs font-medium uppercase tracking-wider text-gray-500">
                Actions
              </th>
            </tr>
          </thead>
          <tbody className="divide-y divide-gray-200 bg-white">
            {currentItems.map((membership) => (
              <MemberRow
                key={membership.id}
                membership={membership}
                isCurrentUserAdmin={
                  organization.membership?.role === 'admin'
                }
              />
            ))}
          </tbody>
        </table>
      </div>
      
      {totalPages > 1 && (
        <Pagination
          currentPage={currentPage}
          totalPages={totalPages}
          onPageChange={setCurrentPage}
        />
      )}
    </div>
  );
}

function MemberRow({ membership, isCurrentUserAdmin }) {
  const { organization } = useOrganization();
  const [isUpdating, setIsUpdating] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const handleRoleChange = async (newRole: 'admin' | 'member') => {
    if (!isCurrentUserAdmin || !organization) return;
    
    setIsUpdating(true);
    setError(null);
    
    try {
      await organization.updateMembership({
        userId: membership.publicUserData.userId,
        role: newRole,
      });
    } catch (error) {
      console.error('Role update error:', error);
      setError('Failed to update role');
    } finally {
      setIsUpdating(false);
    }
  };
  
  const handleRemoveMember = async () => {
    if (!isCurrentUserAdmin || !organization) return;
    
    if (!confirm('Are you sure you want to remove this member?')) {
      return;
    }
    
    setIsUpdating(true);
    
    try {
      await organization.removeMember(membership.publicUserData.userId);
    } catch (error) {
      console.error('Member removal error:', error);
      setError('Failed to remove member');
    } finally {
      setIsUpdating(false);
    }
  };
  
  return (
    <tr>
      <td className="whitespace-nowrap px-6 py-4">
        <div className="flex items-center">
          <div className="h-10 w-10 flex-shrink-0">
            <Avatar
              src={membership.publicUserData.imageUrl}
              alt={membership.publicUserData.firstName || ''}
              size="md"
            />
          </div>
          <div className="ml-4">
            <div className="text-sm font-medium text-gray-900">
              {membership.publicUserData.firstName} {membership.publicUserData.lastName}
            </div>
            <div className="text-sm text-gray-500">
              {membership.publicUserData.identifier}
            </div>
          </div>
        </div>
      </td>
      <td className="whitespace-nowrap px-6 py-4">
        {isCurrentUserAdmin ? (
          <RoleSelector
            value={membership.role}
            onChange={handleRoleChange}
            disabled={isUpdating}
          />
        ) : (
          <Badge variant={membership.role === 'admin' ? 'purple' : 'gray'}>
            {membership.role === 'admin' ? 'Admin' : 'Member'}
          </Badge>
        )}
      </td>
      <td className="whitespace-nowrap px-6 py-4">
        <Badge variant={membership.pending ? 'yellow' : 'green'}>
          {membership.pending ? 'Pending' : 'Active'}
        </Badge>
      </td>
      <td className="whitespace-nowrap px-6 py-4 text-right text-sm font-medium">
        {isCurrentUserAdmin && !isUpdating && (
          <Button
            variant="text"
            color="danger"
            onClick={handleRemoveMember}
            size="sm"
          >
            Remove
          </Button>
        )}
        {isUpdating && <span className="text-sm text-gray-500">Updating...</span>}
        {error && <span className="text-sm text-red-500">{error}</span>}
      </td>
    </tr>
  );
}

function MembersListSkeleton() {
  return (
    <div className="space-y-6">
      <div className="overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider text-gray-500">
                Member
              </th>
              <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider text-gray-500">
                Role
              </th>
              <th scope="col" className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider text-gray-500">
                Status
              </th>
              <th scope="col" className="px-6 py-3 text-right text-xs font-medium uppercase tracking-wider text-gray-500">
                Actions
              </th>
            </tr>
          </thead>
          <tbody className="divide-y divide-gray-200 bg-white">
            {Array.from({ length: 5 }).map((_, index) => (
              <tr key={index}>
                <td className="whitespace-nowrap px-6 py-4">
                  <div className="flex items-center">
                    <div className="h-10 w-10 flex-shrink-0">
                      <Skeleton className="h-10 w-10 rounded-full" />
                    </div>
                    <div className="ml-4">
                      <Skeleton className="h-4 w-32" />
                      <Skeleton className="mt-1 h-3 w-40" />
                    </div>
                  </div>
                </td>
                <td className="whitespace-nowrap px-6 py-4">
                  <Skeleton className="h-6 w-20" />
                </td>
                <td className="whitespace-nowrap px-6 py-4">
                  <Skeleton className="h-6 w-16" />
                </td>
                <td className="whitespace-nowrap px-6 py-4 text-right">
                  <Skeleton className="ml-auto h-6 w-16" />
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

**Invite Form Component:**
```tsx
// lib/org/components/invite-form.tsx
'use client';

import { useState } from 'react';
import { useOrganization } from '@clerk/nextjs';
import { useRouter } from 'next/navigation';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Select } from '@/components/ui/select';
import { Alert } from '@/components/ui/alert';
import { validateEmail } from '@/lib/utils/validation';

export function InviteForm() {
  const { organization, isLoaded } = useOrganization();
  const router = useRouter();
  const [email, setEmail] = useState('');
  const [role, setRole] = useState<'admin' | 'member'>('member');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [success, setSuccess] = useState<string | null>(null);
  
  if (!isLoaded) {
    return <div>Loading...</div>;
  }
  
  if (!organization) {
    return <div>No organization selected</div>;
  }
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!validateEmail(email)) {
      setError('Please enter a valid email address.');
      return;
    }
    
    setIsSubmitting(true);
    setError(null);
    setSuccess(null);
    
    try {
      await organization.inviteMember({
        emailAddress: email,
        role,
      });
      
      setSuccess(`Invitation sent to ${email}`);
      setEmail('');
    } catch (error) {
      console.error('Invitation error:', error);
      setError('Failed to send invitation. Please try again.');
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit} className="space-y-6">
      {error && (
        <Alert variant="error" title="Error" onClose={() => setError(null)}>
          {error}
        </Alert>
      )}
      
      {success && (
        <Alert variant="success" title="Success" onClose={() => setSuccess(null)}>
          {success}
        </Alert>
      )}
      
      <div>
        <label htmlFor="email" className="block text-sm font-medium text-gray-700">
          Email Address
        </label>
        <Input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="colleague@example.com"
          required
        />
      </div>
      
      <div>
        <label htmlFor="role" className="block text-sm font-medium text-gray-700">
          Role
        </label>
        <Select
          id="role"
          value={role}
          onChange={(e) => setRole(e.target.value as 'admin' | 'member')}
        >
          <option value="member">Member</option>
          <option value="admin">Admin</option>
        </Select>
        <p className="mt-1 text-xs text-gray-500">
          Admins can manage organization settings and members. Members can create and edit notes based on permissions.
        </p>
      </div>
      
      <Button
        type="submit"
        variant="primary"
        disabled={isSubmitting || !email.trim()}
      >
        {isSubmitting ? 'Sending Invitation...' : 'Send Invitation'}
      </Button>
    </form>
  );
}
```

**Invite Page:**
```tsx
// app/org/invite/page.tsx
import { PageTitle } from '@/components/layout/page-title';
import { Card } from '@/components/ui/card';
import { InviteForm } from '@/lib/org/components/invite-form';

export default function InvitePage() {
  return (
    <div className="container mx-auto px-4 py-8">
      <PageTitle title="Invite Team Members" />
      
      <div className="mx-auto mt-8 max-w-2xl">
        <Card>
          <div className="px-6 py-5 sm:px-8">
            <h2 className="text-lg font-medium text-gray-900">Invite new members</h2>
            <p className="mt-1 text-sm text-gray-600">
              Send invitations to your team members to collaborate in this organization.
            </p>
          </div>
          
          <div className="border-t border-gray-200 px-6 py-5 sm:px-8">
            <InviteForm />
          </div>
        </Card>
      </div>
    </div>
  );
}
```

#### 2.3.5 Error Handling
- Invalid email: Show validation error
- Invitation failure: Show error message with retry option
- Role update failure: Show error message
- Remove member failure: Show error message
- Permission denied: Show error if non-admin attempts administrative actions

#### 2.3.6 Testing Requirements
- Test members list display
- Test member role update
- Test member removal
- Test invitation sending
- Test pagination
- Test admin-only access restriction

### 2.4 Role Management

#### 2.4.1 Description
Role management allows defining and assigning roles to organization members, controlling their permissions within the organization.

#### 2.4.2 UI Components
- Role selector
- Role indicator badges
- Permission explanation tooltips

#### 2.4.3 Data Requirements
- Role information (admin or member)
- User membership information

#### 2.4.4 Implementation Details

**Role Selector Component:**
```tsx
// lib/org/components/role-selector.tsx
import { useRef, useState } from 'react';
import { Select } from '@/components/ui/select';

interface RoleSelectorProps {
  value: string;
  onChange: (role: 'admin' | 'member') => void;
  disabled?: boolean;
}

export function RoleSelector({ value, onChange, disabled = false }: RoleSelectorProps) {
  const selectRef = useRef<HTMLSelectElement>(null);
  const [isChanging, setIsChanging] = useState(false);
  
  const handleChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    const newRole = e.target.value as 'admin' | 'member';
    
    if (newRole === 'admin') {
      // Confirm before making someone an admin
      if (window.confirm('Are you sure you want to make this user an admin? Admins can manage the organization and all its members.')) {
        setIsChanging(true);
        onChange(newRole);
      } else {
        // Reset the select value if canceled
        if (selectRef.current) {
          selectRef.current.value = value;
        }
      }
    } else {
      // Confirm before downgrading an admin
      if (value === 'admin' && !window.confirm('Are you sure you want to remove admin privileges from this user?')) {
        // Reset the select value if canceled
        if (selectRef.current) {
          selectRef.current.value = value;
        }
        return;
      }
      
      setIsChanging(true);
      onChange(newRole);
    }
  };
  
  return (
    <div className="relative inline-block w-32">
      <Select
        ref={selectRef}
        value={value}
        onChange={handleChange}
        disabled={disabled || isChanging}
        className={isChanging ? 'opacity-50' : ''}
      >
        <option value="admin">Admin</option>
        <option value="member">Member</option>
      </Select>
      
      {isChanging && (
        <div className="absolute right-2 top-2 h-4 w-4 animate-spin rounded-full border-2 border-gray-300 border-t-indigo-600"></div>
      )}
    </div>
  );
}
```

**Role Badge Component:**
```tsx
// lib/org/components/role-badge.tsx
import { Badge, BadgeProps } from '@/components/ui/badge';
import { Tooltip } from '@/components/ui/tooltip';

interface RoleBadgeProps {
  role: string;
  showTooltip?: boolean;
}

export function RoleBadge({ role, showTooltip = true }: RoleBadgeProps) {
  const variant: BadgeProps['variant'] = role === 'admin' ? 'purple' : 'gray';
  const label = role === 'admin' ? 'Admin' : 'Member';
  
  const tooltipContent = role === 'admin'
    ? 'Admins can manage organization settings, members, and all notes.'
    : 'Members can create and edit notes based on permissions.';
  
  const badge = (
    <Badge variant={variant}>{label}</Badge>
  );
  
  if (!showTooltip) {
    return badge;
  }
  
  return (
    <Tooltip content={tooltipContent}>
      {badge}
    </Tooltip>
  );
}
```

#### 2.4.5 Error Handling
- Role update failure: Show error message
- Permission denied: Show error if non-admin attempts to change roles
- Optimistic UI updates with rollback on failure

#### 2.4.6 Testing Requirements
- Test role selector UI
- Test role update functionality
- Test confirmation dialogs
- Test admin permission enforcement

## 3. Notes Management

### 3.1 Notes List

#### 3.1.1 Description
The notes list page displays all accessible notes for the current organization, with filtering and sorting options.

#### 3.1.2 UI Components
- Notes grid/list view
- Note cards with preview
- Filtering controls (by access level, creator, etc.)
- Search input
- Sorting options
- Create note button
- Pagination for large lists

#### 3.1.3 Data Requirements
- Notes collection (filtered by organization and access permissions)
- User permissions
- Organization information
- Filter and sort parameters

#### 3.1.4 Implementation Details

**Notes List Page:**
```tsx
// app/dashboard/notes/page.tsx
import { Suspense } from 'react';
import { PageTitle } from '@/components/layout/page-title';
import { NotesList } from '@/lib/notes/components/notes-list';
import { NotesListSkeleton } from '@/lib/notes/components/notes-list-skeleton';
import { CreateNoteButton } from '@/lib/notes/components/create-note-button';
import { NotesFilters } from '@/lib/notes/components/notes-filters';
import { getCurrentOrg } from '@/lib/auth/get-current-org';

export default async function NotesPage({
  searchParams,
}: {
  searchParams: { [key: string]: string | string[] | undefined };
}) {
  const org = await getCurrentOrg();
  
  // Parse search parameters
  const filters = {
    search: typeof searchParams.search === 'string' ? searchParams.search : undefined,
    accessLevel: typeof searchParams.access === 'string' ? searchParams.access : undefined,
    createdBy: typeof searchParams.creator === 'string' ? searchParams.creator : undefined,
  };
  
  return (
    <div className="container mx-auto px-4 py-8">
      <div className="mb-8 flex items-center justify-between">
        <PageTitle title="Notes" />
        <CreateNoteButton />
      </div>
      
      <NotesFilters initialFilters={filters} />
      
      <Suspense fallback={<NotesListSkeleton />}>
        <NotesList
          organizationId={org.id}
          filters={filters}
        />
      </Suspense>
    </div>
  );
}
```

**Notes List Component:**
```tsx
// lib/notes/components/notes-list.tsx
import { getNotes } from '@/lib/notes/api/get-notes';
import { NoteCard } from './note-card';
import { EmptyState } from '@/components/data/empty-state';
import { Pagination } from '@/components/data/pagination';
import { NoteAccessLevel } from '@/lib/notes/types';

interface NotesListProps {
  organizationId: string;
  filters?: {
    search?: string;
    accessLevel?: string;
    createdBy?: string;
  };
  page?: number;
  perPage?: number;
}

export async function NotesList({
  organizationId,
  filters,
  page = 1,
  perPage = 12,
}: NotesListProps) {
  const { notes, total } = await getNotes({
    organizationId,
    page,
    perPage,
    accessLevel: filters?.accessLevel as NoteAccessLevel | undefined,
    createdBy: filters?.createdBy,
    search: filters?.search,
  });
  
  if (notes.length === 0) {
    return (
      <EmptyState
        title="No notes found"
        description={
          Object.values(filters || {}).some(Boolean)
            ? "No notes match your filter criteria. Try adjusting your filters."
            : "You don't have any notes yet. Create your first note to get started."
        }
        icon={DocumentTextIcon}
        action={{
          label: 'Create Note',
          href: '/dashboard/notes/new',
        }}
      />
    );
  }
  
  const totalPages = Math.ceil(total / perPage);
  
  return (
    <div className="space-y-6">
      <div className="grid grid-cols-1 gap-6 sm:grid-cols-2 lg:grid-cols-3">
        {notes.map((note) => (
          <NoteCard key={note.id} note={note} />
        ))}
      </div>
      
      {totalPages > 1 && (
        <Pagination
          currentPage={page}
          totalPages={totalPages}
          baseUrl="/dashboard/notes"
          queryParams={filters}
        />
      )}
    </div>
  );
}
```

**Note Card Component:**
```tsx
// lib/notes/components/note-card.tsx
import Link from 'next/link';
import { formatDate } from '@/lib/utils/date';
import { truncateText } from '@/lib/utils/string';
import { AccessLevelBadge } from './access-level-badge';
import { Note } from '@/lib/notes/types';

interface NoteCardProps {
  note: Note;
}

export function NoteCard({ note }: NoteCardProps) {
  return (
    <Link
      href={`/dashboard/notes/${note.id}`}
      className="block overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm transition hover:shadow"
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
          {truncateText(note.content.replace(/#+\s|[*_~`]/g, ''), 120)}
        </div>
        
        <div className="mt-4 flex items-center justify-between text-xs text-gray-500">
          <span>Updated {formatDate(note.updatedAt)}</span>
          
          {note.lockedByUserId && (
            <span className="flex items-center text-amber-600">
              <LockClosedIcon className="mr-1 h-3 w-3" />
              Locked
            </span>
          )}
        </div>
      </div>
    </Link>
  );
}
```

**Notes Filters Component:**
```tsx
// lib/notes/components/notes-filters.tsx
'use client';

import { useEffect, useState } from 'react';
import { useRouter, usePathname } from 'next/navigation';
import { useDebounce } from '@/lib/hooks/use-debounce';
import { Input } from '@/components/ui/input';
import { Select } from '@/components/ui/select';
import { Button } from '@/components/ui/button';
import { useOrganization, useUser } from '@clerk/nextjs';

interface NotesFiltersProps {
  initialFilters?: {
    search?: string;
    accessLevel?: string;
    createdBy?: string;
  };
}

export function NotesFilters({ initialFilters = {} }: NotesFiltersProps) {
  const router = useRouter();
  const pathname = usePathname();
  const { organization, membershipList } = useOrganization({ membershipList: true });
  const { user } = useUser();
  
  const [search, setSearch] = useState(initialFilters.search || '');
  const [accessLevel, setAccessLevel] = useState(initialFilters.accessLevel || '');
  const [createdBy, setCreatedBy] = useState(initialFilters.createdBy || '');
  
  const debouncedSearch = useDebounce(search, 300);
  
  useEffect(() => {
    const params = new URLSearchParams();
    
    if (debouncedSearch) {
      params.set('search', debouncedSearch);
    }
    
    if (accessLevel) {
      params.set('access', accessLevel);
    }
    
    if (createdBy) {
      params.set('creator', createdBy);
    }
    
    const queryString = params.toString();
    router.push(`${pathname}${queryString ? `?${queryString}` : ''}`);
  }, [debouncedSearch, accessLevel, createdBy, pathname, router]);
  
  const resetFilters = () => {
    setSearch('');
    setAccessLevel('');
    setCreatedBy('');
  };
  
  const hasActiveFilters = search || accessLevel || createdBy;
  
  return (
    <div className="mb-6 space-y-4">
      <div className="flex flex-col space-y-4 sm:flex-row sm:space-x-4 sm:space-y-0">
        <div className="flex-1">
          <Input
            type="search"
            placeholder="Search notes..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="w-full"
            leftIcon={SearchIcon}
          />
        </div>
        
        <div className="w-full sm:w-48">
          <Select
            value={accessLevel}
            onChange={(e) => setAccessLevel(e.target.value)}
            placeholder="Access Level"
          >
            <option value="">All Access Levels</option>
            <option value="public">Public</option>
            <option value="shared">Shared</option>
            <option value="private">Private</option>
          </Select>
        </div>
        
        <div className="w-full sm:w-48">
          <Select
            value={createdBy}
            onChange={(e) => setCreatedBy(e.target.value)}
            placeholder="Created By"
          >
            <option value="">All Members</option>
            {user && (
              <option value={user.id}>My Notes</option>
            )}
            {membershipList?.map((membership) => (
              membership.publicUserData.userId !== user?.id && (
                <option 
                  key={membership.publicUserData.userId} 
                  value={membership.publicUserData.userId}
                >
                  {membership.publicUserData.firstName} {membership.publicUserData.lastName}
                </option>
              )
            ))}
          </Select>
        </div>
        
        {hasActiveFilters && (
          <div>
            <Button variant="outline" onClick={resetFilters} size="icon">
              <XMarkIcon className="h-5 w-5" />
            </Button>
          </div>
        )}
      </div>
    </div>
  );
}
```

#### 3.1.5 Error Handling
- Loading errors: Show error message with retry option
- Empty states: Show appropriate empty state based on filters
- Pagination errors: Handle gracefully with fallback
- Permission errors: Show appropriate messaging

#### 3.1.6 Testing Requirements
- Test notes list rendering
- Test filtering functionality
- Test search functionality
- Test pagination
- Test empty states
- Test permission-based filtering

### 3.2 Note Creation

#### 3.2.1 Description
The note creation feature allows users to create new notes within an organization, setting title, content, and access permissions.

#### 3.2.2 UI Components
- Note creation form
- Markdown editor
- Access control settings
- Save/cancel buttons
- Preview toggle

#### 3.2.3 Data Requirements
- Note title
- Note content (markdown)
- Access level
- Edit permissions
- Access lists (if applicable)

#### 3.2.4 Implementation Details

**Create Note Page:**
```tsx
// app/dashboard/notes/new/page.tsx
import { PageTitle } from '@/components/layout/page-title';
import { NoteForm } from '@/lib/notes/components/note-form';
import { getCurrentUser } from '@/lib/auth/get-current-user';
import { getCurrentOrg } from '@/lib/auth/get-current-org';
import { redirect } from 'next/navigation';

export default async function CreateNotePage() {
  const user = await getCurrentUser();
  const org = await getCurrentOrg();
  
  if (!user || !org) {
    redirect('/auth/sign-in');
  }
  
  const initialNote = {
    title: '',
    content: '',
    accessLevel: 'public' as const,
    canEdit: 'everyone' as const,
    editAccessList: [],
    viewAccessList: [],
  };
  
  return (
    <div className="container mx-auto px-4 py-8">
      <PageTitle title="Create New Note" />
      
      <div className="mx-auto mt-8 max-w-4xl">
        <NoteForm
          initialNote={initialNote}
          userId={user.id}
          organizationId={org.id}
          mode="create"
        />
      </div>
    </div>
  );
}
```

**Note Form Component:**
```tsx
// lib/notes/components/note-form.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import MarkdownEditor from 'react-markdown-editor-lite';
import 'react-markdown-editor-lite/lib/index.css';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Card } from '@/components/ui/card';
import { AccessControlSettings } from './access-control-settings';
import { useCreateNote } from '@/lib/notes/hooks/use-create-note';
import { useUpdateNote } from '@/lib/notes/hooks/use-update-note';
import { NoteAccessLevel, NoteEditPermission, NoteFormData } from '@/lib/notes/types';
import { logger } from '@/lib/logger/client-logger';

interface NoteFormProps {
  initialNote: {
    id?: string;
    title: string;
    content: string;
    accessLevel: NoteAccessLevel;
    canEdit: NoteEditPermission;
    editAccessList: string[];
    viewAccessList: string[];
  };
  userId: string;
  organizationId: string;
  mode: 'create' | 'edit';
}

export function NoteForm({
  initialNote,
  userId,
  organizationId,
  mode,
}: NoteFormProps) {
  const router = useRouter();
  
  const [formData, setFormData] = useState<NoteFormData>({
    title: initialNote.title,
    content: initialNote.content,
    accessLevel: initialNote.accessLevel,
    canEdit: initialNote.canEdit,
    editAccessList: initialNote.editAccessList,
    viewAccessList: initialNote.viewAccessList,
  });
  
  const { mutate: createNote, isPending: isCreating } = useCreateNote();
  const { mutate: updateNote, isPending: isUpdating } = useUpdateNote();
  
  const isPending = isCreating || isUpdating;
  
  const handleTitleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setFormData((prev) => ({ ...prev, title: e.target.value }));
  };
  
  const handleEditorChange = ({ text }: { text: string }) => {
    setFormData((prev) => ({ ...prev, content: text }));
  };
  
  const handleAccessSettingsChange = (settings: {
    accessLevel: NoteAccessLevel;
    canEdit: NoteEditPermission;
    editAccessList: string[];
    viewAccessList: string[];
  }) => {
    setFormData((prev) => ({
      ...prev,
      ...settings,
    }));
  };
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    if (formData.title.trim() === '') {
      return;
    }
    
    if (mode === 'create') {
      logger.info('Creating new note', { title: formData.title });
      
      createNote(
        {
          title: formData.title,
          content: formData.content,
          accessLevel: formData.accessLevel,
          canEdit: formData.canEdit,
          editAccessList: formData.editAccessList,
          viewAccessList: formData.viewAccessList,
          createdByUserId: userId,
          organizationId,
        },
        {
          onSuccess: (note) => {
            logger.info('Note created successfully', { noteId: note.id });
            router.push(`/dashboard/notes/${note.id}`);
          },
          onError: (error) => {
            logger.error('Failed to create note', { error });
          },
        }
      );
    } else if (initialNote.id) {
      logger.info('Updating note', { noteId: initialNote.id });
      
      updateNote(
        {
          id: initialNote.id,
          title: formData.title,
          content: formData.content,
          accessLevel: formData.accessLevel,
          canEdit: formData.canEdit,
          editAccessList: formData.editAccessList,
          viewAccessList: formData.viewAccessList,
        },
        {
          onSuccess: (note) => {
            logger.info('Note updated successfully', { noteId: note.id });
            router.push(`/dashboard/notes/${note.id}`);
          },
          onError: (error) => {
            logger.error('Failed to update note', { error });
          },
        }
      );
    }
  };
  
  const handleCancel = () => {
    if (mode === 'edit' && initialNote.id) {
      router.push(`/dashboard/notes/${initialNote.id}`);
    } else {
      router.push('/dashboard/notes');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <Card>
        <div className="px-6 py-5 sm:px-8">
          <div className="mb-4">
            <label htmlFor="title" className="block text-sm font-medium text-gray-700">
              Title
            </label>
            <Input
              id="title"
              type="text"
              value={formData.title}
              onChange={handleTitleChange}
              placeholder="Note title"
              required
              className="mt-1"
            />
          </div>
          
          <div>
            <label htmlFor="content" className="block text-sm font-medium text-gray-700">
              Content
            </label>
            <div className="mt-1 rounded-md border border-gray-300">
              <MarkdownEditor
                id="content"
                value={formData.content}
                onChange={handleEditorChange}
                style={{ height: '400px' }}
                renderHTML={(text) => Promise.resolve(text)}
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
                }}
              />
            </div>
          </div>
        </div>
        
        <div className="border-t border-gray-200 px-6 py-5 sm:px-8">
          <AccessControlSettings
            accessLevel={formData.accessLevel}
            canEdit={formData.canEdit}
            editAccessList={formData.editAccessList}
            viewAccessList={formData.viewAccessList}
            onChange={handleAccessSettingsChange}
          />
        </div>
        
        <div className="flex items-center justify-end space-x-3 border-t border-gray-200 bg-gray-50 px-6 py-4 sm:px-8">
          <Button
            type="button"
            variant="outline"
            onClick={handleCancel}
            disabled={isPending}
          >
            Cancel
          </Button>
          <Button
            type="submit"
            variant="primary"
            disabled={isPending || formData.title.trim() === ''}
          >
            {isPending
              ? mode === 'create' ? 'Creating...' : 'Updating...'
              : mode === 'create' ? 'Create Note' : 'Update Note'}
          </Button>
        </div>
      </Card>
    </form>
  );
}
```

**Access Control Settings Component:**
```tsx
// lib/notes/components/access-control-settings.tsx
'use client';

import { useState, useEffect } from 'react';
import { useOrganization } from '@clerk/nextjs';
import { Select } from '@/components/ui/select';
import { Radio } from '@/components/ui/radio';
import { NoteAccessLevel, NoteEditPermission } from '@/lib/notes/types';
import { MemberSelector } from './member-selector';

interface AccessControlSettingsProps {
  accessLevel: NoteAccessLevel;
  canEdit: NoteEditPermission;
  editAccessList: string[];
  viewAccessList: string[];
  onChange: (settings: {
    accessLevel: NoteAccessLevel;
    canEdit: NoteEditPermission;
    editAccessList: string[];
    viewAccessList: string[];
  }) => void;
}

export function AccessControlSettings({
  accessLevel,
  canEdit,
  editAccessList,
  viewAccessList,
  onChange,
}: AccessControlSettingsProps) {
  const { organization, membershipList, isLoaded } = useOrganization({
    membershipList: true,
  });
  
  const [localAccessLevel, setLocalAccessLevel] = useState<NoteAccessLevel>(accessLevel);
  const [localCanEdit, setLocalCanEdit] = useState<NoteEditPermission>(canEdit);
  const [localEditAccessList, setLocalEditAccessList] = useState<string[]>(editAccessList);
  const [localViewAccessList, setLocalViewAccessList] = useState<string[]>(viewAccessList);
  
  // Update parent component when local state changes
  useEffect(() => {
    onChange({
      accessLevel: localAccessLevel,
      canEdit: localCanEdit,
      editAccessList: localEditAccessList,
      viewAccessList: localViewAccessList,
    });
  }, [localAccessLevel, localCanEdit, localEditAccessList, localViewAccessList, onChange]);
  
  if (!isLoaded || !organization || !membershipList) {
    return <div>Loading...</div>;
  }
  
  return (
    <div className="space-y-6">
      <div>
        <h3 className="text-base font-medium text-gray-900">Access Control</h3>
        <p className="mt-1 text-sm text-gray-500">
          Configure who can view and edit this note.
        </p>
      </div>
      
      <div>
        <label className="block text-sm font-medium text-gray-700">
          Access Level
        </label>
        <Select
          value={localAccessLevel}
          onChange={(e) => setLocalAccessLevel(e.target.value as NoteAccessLevel)}
          className="mt-1"
        >
          <option value="public">Public (All organization members)</option>
          <option value="shared">Shared (Specific members only)</option>
          <option value="private">Private (Only you)</option>
        </Select>
      </div>
      
      {localAccessLevel === 'shared' && (
        <div>
          <label className="block text-sm font-medium text-gray-700">
            Who can view this note?
          </label>
          <div className="mt-1">
            <MemberSelector
              membershipList={membershipList}
              selectedUserIds={localViewAccessList}
              onChange={setLocalViewAccessList}
            />
          </div>
        </div>
      )}
      
      <div>
        <label className="block text-sm font-medium text-gray-700">
          Edit Permissions
        </label>
        <div className="mt-2 space-y-3">
          <Radio
            id="edit-everyone"
            name="canEdit"
            value="everyone"
            checked={localCanEdit === 'everyone'}
            onChange={() => setLocalCanEdit('everyone')}
            label="Everyone who can view can also edit"
          />
          
          <Radio
            id="edit-access-list"
            name="canEdit"
            value="accessList"
            checked={localCanEdit === 'accessList'}
            onChange={() => setLocalCanEdit('accessList')}
            label="Only specific members can edit"
          />
          
          <Radio
            id="edit-deny-list"
            name="canEdit"
            value="denyList"
            checked={localCanEdit === 'denyList'}
            onChange={() => setLocalCanEdit('denyList')}
            label="Everyone except specific members can edit"
          />
          
          <Radio
            id="edit-no-one"
            name="canEdit"
            value="noOne"
            checked={localCanEdit === 'noOne'}
            onChange={() => setLocalCanEdit('noOne')}
            label="No one can edit (read-only)"
          />
        </div>
      </div>
      
      {(localCanEdit === 'accessList' || localCanEdit === 'denyList') && (
        <div>
          <label className="block text-sm font-medium text-gray-700">
            {localCanEdit === 'accessList'
              ? 'Who can edit this note?'
              : 'Who cannot edit this note?'}
          </label>
          <div className="mt-1">
            <MemberSelector
              membershipList={membershipList}
              selectedUserIds={localEditAccessList}
              onChange={setLocalEditAccessList}
            />
          </div>
        </div>
      )}
    </div>
  );
}
```

**Member Selector Component:**
```tsx
// lib/notes/components/member-selector.tsx
'use client';

import { useState } from 'react';
import { useUser } from '@clerk/nextjs';
import { Checkbox } from '@/components/ui/checkbox';
import { Avatar } from '@/components/ui/avatar';
import { Input } from '@/components/ui/input';

interface MemberSelectorProps {
  membershipList: any[];
  selectedUserIds: string[];
  onChange: (selectedUserIds: string[]) => void;
}

export function MemberSelector({
  membershipList,
  selectedUserIds,
  onChange,
}: MemberSelectorProps) {
  const { user } = useUser();
  const [search, setSearch] = useState('');
  
  const handleToggleUser = (userId: string) => {
    const isSelected = selectedUserIds.includes(userId);
    
    if (isSelected) {
      onChange(selectedUserIds.filter((id) => id !== userId));
    } else {
      onChange([...selectedUserIds, userId]);
    }
  };
  
  const filteredMembers = membershipList.filter((membership) => {
    if (!search) return true;
    
    const fullName = `${membership.publicUserData.firstName} ${membership.publicUserData.lastName}`.toLowerCase();
    const email = membership.publicUserData.identifier.toLowerCase();
    const query = search.toLowerCase();
    
    return fullName.includes(query) || email.includes(query);
  });
  
  return (
    <div className="space-y-3">
      <Input
        type="search"
        placeholder="Search members..."
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        leftIcon={SearchIcon}
      />
      
      <div className="max-h-64 overflow-y-auto rounded-md border border-gray-300 bg-white">
        {filteredMembers.length === 0 ? (
          <div className="p-4 text-center text-sm text-gray-500">
            No members found
          </div>
        ) : (
          <ul className="divide-y divide-gray-200">
            {filteredMembers.map((membership) => (
              <li key={membership.publicUserData.userId}>
                <div className="flex items-center px-4 py-3">
                  <div className="mr-3">
                    <Checkbox
                      id={`user-${membership.publicUserData.userId}`}
                      checked={selectedUserIds.includes(membership.publicUserData.userId)}
                      onChange={() => handleToggleUser(membership.publicUserData.userId)}
                    />
                  </div>
                  
                  <div className="mr-3">
                    <Avatar
                      src={membership.publicUserData.imageUrl}
                      alt={membership.publicUserData.firstName || ''}
                      size="sm"
                    />
                  </div>
                  
                  <div className="min-w-0 flex-1">
                    <label
                      htmlFor={`user-${membership.publicUserData.userId}`}
                      className="cursor-pointer"
                    >
                      <div className="truncate text-sm font-medium text-gray-900">
                        {membership.publicUserData.firstName} {membership.publicUserData.lastName}
                        {membership.publicUserData.userId === user?.id && ' (You)'}
                      </div>
                      <div className="truncate text-xs text-gray-500">
                        {membership.publicUserData.identifier}
                      </div>
                    </label>
                  </div>
                </div>
              </li>
            ))}
          </ul>
        )}
      </div>
    </div>
  );
}
```

**Create Note Hook:**
```tsx
// lib/notes/hooks/use-create-note.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { createNote } from '@/lib/notes/api/create-note';
import { Note, NoteAccessLevel, NoteEditPermission } from '@/lib/notes/types';

interface CreateNoteParams {
  title: string;
  content: string;
  accessLevel: NoteAccessLevel;
  canEdit: NoteEditPermission;
  editAccessList: string[];
  viewAccessList: string[];
  createdByUserId: string;
  organizationId: string;
}

export function useCreateNote() {
  const queryClient = useQueryClient();
  
  return useMutation<Note, Error, CreateNoteParams>({
    mutationFn: createNote,
    onSuccess: () => {
      // Invalidate notes list query
      queryClient.invalidateQueries({ queryKey: ['notes'] });
    },
  });
}
```

**Create Note API Function:**
```typescript
// lib/notes/api/create-note.ts
import { Note, NoteAccessLevel, NoteEditPermission } from '@/lib/notes/types';

interface CreateNoteParams {
  title: string;
  content: string;
  accessLevel: NoteAccessLevel;
  canEdit: NoteEditPermission;
  editAccessList: string[];
  viewAccessList: string[];
  createdByUserId: string;
  organizationId: string;
}

export async function createNote(params: CreateNoteParams): Promise<Note> {
  const response = await fetch('/api/notes', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(params),
  });
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'Failed to create note');
  }
  
  return response.json();
}
```

**Create Note API Route:**
```typescript
// app/api/notes/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@clerk/nextjs/server';
import { NoteRepository } from '@/lib/notes/repositories/note-repository';
import { AuditRepository } from '@/lib/audit/repositories/audit-repository';
import { validateCreateNoteInput } from '@/lib/notes/validators';
import { logger } from '@/lib/logger/server-logger';

export async function POST(req: NextRequest) {
  try {
    const { userId, orgId } = auth();
    
    if (!userId || !orgId) {
      return NextResponse.json(
        { message: 'Unauthorized' },
        { status: 401 }
      );
    }
    
    const body = await req.json();
    
    const validation = validateCreateNoteInput(body);
    if (!validation.success) {
      return NextResponse.json(
        { message: 'Invalid input', errors: validation.errors },
        { status: 400 }
      );
    }
    
    // Organization ID from auth context must match the one in the request
    if (body.organizationId !== orgId) {
      return NextResponse.json(
        { message: 'Organization mismatch' },
        { status: 403 }
      );
    }
    
    // Create the note
    const note = await NoteRepository.createNote({
      title: body.title,
      content: body.content,
      createdByUserId: userId,
      organizationId: orgId,
      accessLevel: body.accessLevel,
      canEdit: body.canEdit,
      editAccessList: body.editAccessList,
      viewAccessList: body.viewAccessList,
    });
    
    // Log the action
    await AuditRepository.logEvent({
      action: 'create',
      resourceType: 'note',
      resourceId: note.id,
      userId,
      organizationId: orgId,
      details: {
        title: note.title,
        accessLevel: note.access_level,
      },
    });
    
    logger.info('Note created', {
      noteId: note.id,
      userId,
      organizationId: orgId,
    });
    
    return NextResponse.json(note, { status: 201 });
  } catch (error) {
    logger.error('Failed to create note', { error });
    
    return NextResponse.json(
      { message: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

#### 3.2.5 Error Handling
- Validation errors: Show inline validation messages
- Submission errors: Show error message with details
- Permission errors: Show access denied message
- Network errors: Show connection error with retry option

#### 3.2.6 Testing Requirements
- Test form validation
- Test markdown editor functionality
- Test access control settings
- Test note creation API
- Test successful creation and redirect
- Test error handling

### 3.3 Note Editing

#### 3.3.1 Description
The note editing feature allows users with appropriate permissions to modify existing notes, including content and access settings.

#### 3.3.2 UI Components
- Note edit form (reused from creation)
- Lock status indicator
- Version history (optional)
- Save/cancel buttons

#### 3.3.3 Data Requirements
- Existing note data
- User permissions
- Lock status
- Updated note data

#### 3.3.4 Implementation Details

**Edit Note Page:**
```tsx
// app/dashboard/notes/[noteId]/edit/page.tsx
import { redirect } from 'next/navigation';
import { PageTitle } from '@/components/layout/page-title';
import { NoteForm } from '@/lib/notes/components/note-form';
import { getNoteById } from '@/lib/notes/api/get-note-by-id';
import { getCurrentUser } from '@/lib/auth/get-current-user';
import { getCurrentOrg } from '@/lib/auth/get-current-org';
import { LockWarning } from '@/lib/notes/components/lock-warning';
import { canUserEditNote } from '@/lib/notes/utils/access-checks';

interface EditNotePageProps {
  params: { noteId: string };
}

export default async function EditNotePage({ params }: EditNotePageProps) {
  const user = await getCurrentUser();
  const org = await getCurrentOrg();
  
  if (!user || !org) {
    redirect('/auth/sign-in');
  }
  
  const note = await getNoteById(params.noteId);
  
  if (!note) {
    redirect('/dashboard/notes');
  }
  
  // Check if user has permission to edit the note
  const canEdit = await canUserEditNote({
    note,
    userId: user.id,
    organizationId: org.id,
    userRole: org.role,
  });
  
  if (!canEdit) {
    redirect(`/dashboard/notes/${note.id}`);
  }
  
  // Check if note is locked by someone else
  const isLockedByOtherUser = 
    note.lockedByUserId && 
    note.lockedByUserId !== user.id &&
    note.lockedAt && 
    new Date(note.lockedAt).getTime() > Date.now() - 15 * 60 * 1000;
  
  return (
    <div className="container mx-auto px-4 py-8">
      <PageTitle title="Edit Note" />
      
      {isLockedByOtherUser && (
        <div className="mb-6">
          <LockWarning note={note} />
        </div>
      )}
      
      <div className="mx-auto mt-8 max-w-4xl">
        <NoteForm
          initialNote={{
            id: note.id,
            title: note.title,
            content: note.content,
            accessLevel: note.accessLevel,
            canEdit: note.canEdit,
            editAccessList: note.editAccessList.split(',').filter(Boolean),
            viewAccessList: note.viewAccessList.split(',').filter(Boolean),
          }}
          userId={user.id}
          organizationId={org.id}
          mode="edit"
        />
      </div>
    </div>
  );
}
```

**Lock Warning Component:**
```tsx
// lib/notes/components/lock-warning.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { Alert } from '@/components/ui/alert';
import { Button } from '@/components/ui/button';
import { useOrganization } from '@clerk/nextjs';
import { formatDistanceToNow } from 'date-fns';
import { useTakeLock } from '@/lib/notes/hooks/use-take-lock';

interface LockWarningProps {
  note: {
    id: string;
    lockedByUserId: string;
    lockedAt: string;
  };
}

export function LockWarning({ note }: LockWarningProps) {
  const router = useRouter();
  const { membershipList, isLoaded } = useOrganization({ membershipList: true });
  const [isTakingLock, setIsTakingLock] = useState(false);
  
  const { mutate: takeLock } = useTakeLock();
  
  if (!isLoaded) {
    return null;
  }
  
  const lockedByUser = membershipList?.find(
    (member) => member.publicUserData.userId === note.lockedByUserId
  );
  
  const lockedAt = new Date(note.lockedAt);
  const lockExpiry = new Date(lockedAt.getTime() + 15 * 60 * 1000);
  const isExpired = lockExpiry < new Date();
  
  const timeRemaining = isExpired
    ? 'expired'
    : formatDistanceToNow(lockExpiry, { addSuffix: true });
  
  const handleView = () => {
    router.push(`/dashboard/notes/${note.id}`);
  };
  
  const handleTakeLock = () => {
    setIsTakingLock(true);
    
    takeLock(note.id, {
      onSuccess: () => {
        router.refresh();
      },
      onError: (error) => {
        console.error('Failed to take lock:', error);
        setIsTakingLock(false);
      },
    });
  };
  
  return (
    <Alert
      variant="warning"
      title="This note is currently being edited"
      icon={LockClosedIcon}
    >
      <p className="mb-4">
        {lockedByUser
          ? `This note is currently being edited by ${lockedByUser.publicUserData.firstName} ${lockedByUser.publicUserData.lastName}.`
          : 'This note is locked for editing by another user.'}
        {' '}
        The lock will expire {timeRemaining}.
      </p>
      
      <div className="flex space-x-3">
        <Button variant="outline" onClick={handleView}>
          View Note
        </Button>
        
        {isExpired && (
          <Button
            variant="primary"
            onClick={handleTakeLock}
            disabled={isTakingLock}
          >
            {isTakingLock ? 'Taking Lock...' : 'Take Lock & Edit'}
          </Button>
        )}
      </div>
    </Alert>
  );
}
```

**Update Note Hook:**
```typescript
// lib/notes/hooks/use-update-note.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { updateNote } from '@/lib/notes/api/update-note';
import { Note, NoteAccessLevel, NoteEditPermission } from '@/lib/notes/types';

interface UpdateNoteParams {
  id: string;
  title?: string;
  content?: string;
  accessLevel?: NoteAccessLevel;
  canEdit?: NoteEditPermission;
  editAccessList?: string[];
  viewAccessList?: string[];
}

export function useUpdateNote() {
  const queryClient = useQueryClient();
  
  return useMutation<Note, Error, UpdateNoteParams>({
    mutationFn: updateNote,
    onSuccess: (updatedNote) => {
      // Update the note in the cache
      queryClient.setQueryData(['note', updatedNote.id], updatedNote);
      
      // Invalidate the notes list
      queryClient.invalidateQueries({ queryKey: ['notes'] });
    },
  });
}
```

**Update Note API Function:**
```typescript
// lib/notes/api/update-note.ts
import { Note, NoteAccessLevel, NoteEditPermission } from '@/lib/notes/types';

interface UpdateNoteParams {
  id: string;
  title?: string;
  content?: string;
  accessLevel?: NoteAccessLevel;
  canEdit?: NoteEditPermission;
  editAccessList?: string[];
  viewAccessList?: string[];
}

export async function updateNote(params: UpdateNoteParams): Promise<Note> {
  const { id, ...data } = params;
  
  const response = await fetch(`/api/notes/${id}`, {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(data),
  });
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'Failed to update note');
  }
  
  return response.json();
}
```

**Update Note API Route:**
```typescript
// app/api/notes/[noteId]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@clerk/nextjs/server';
import { NoteRepository } from '@/lib/notes/repositories/note-repository';
import { AuditRepository } from '@/lib/audit/repositories/audit-repository';
import { validateUpdateNoteInput } from '@/lib/notes/validators';
import { logger } from '@/lib/logger/server-logger';

export async function PATCH(
  req: NextRequest,
  { params }: { params: { noteId: string } }
) {
  try {
    const { userId, orgId } = auth();
    
    if (!userId || !orgId) {
      return NextResponse.json(
        { message: 'Unauthorized' },
        { status: 401 }
      );
    }
    
    const noteId = params.noteId;
    const body = await req.json();
    
    const validation = validateUpdateNoteInput(body);
    if (!validation.success) {
      return NextResponse.json(
        { message: 'Invalid input', errors: validation.errors },
        { status: 400 }
      );
    }
    
    // Check if the note exists and belongs to the organization
    const note = await NoteRepository.getNoteById(noteId, userId, orgId);
    
    if (!note) {
      return NextResponse.json(
        { message: 'Note not found' },
        { status: 404 }
      );
    }
    
    // Check if the user can edit the note
    const canEdit = await NoteRepository.canUserEditNote(noteId, userId);
    
    if (!canEdit) {
      return NextResponse.json(
        { message: 'You do not have permission to edit this note' },
        { status: 403 }
      );
    }
    
    // Check if the note is locked by another user
    if (
      note.locked_by_user_id &&
      note.locked_by_user_id !== userId &&
      note.locked_at &&
      new Date(note.locked_at).getTime() > Date.now() - 15 * 60 * 1000
    ) {
      return NextResponse.json(
        { 
          message: 'Note is locked by another user',
          lockedByUserId: note.locked_by_user_id,
          lockedAt: note.locked_at 
        },
        { status: 423 }
      );
    }
    
    // Update the note content if provided
    let updatedNote = note;
    if (body.title !== undefined || body.content !== undefined) {
      updatedNote = await NoteRepository.updateNote(noteId, {
        title: body.title,
        content: body.content,
      });
      
      // Log content update
      await AuditRepository.logEvent({
        action: 'update',
        resourceType: 'note',
        resourceId: noteId,
        userId,
        organizationId: orgId,
        details: {
          title: body.title !== undefined ? body.title : note.title,
        },
      });
    }
    
    // Update the note access settings if provided
    if (
      body.accessLevel !== undefined ||
      body.canEdit !== undefined ||
      body.editAccessList !== undefined ||
      body.viewAccessList !== undefined
    ) {
      updatedNote = await NoteRepository.updateNoteAccess(noteId, {
        accessLevel: body.accessLevel,
        canEdit: body.canEdit,
        editAccessList: body.editAccessList,
        viewAccessList: body.viewAccessList,
      });
      
      // Log access update
      await AuditRepository.logEvent({
        action: 'changeAccess',
        resourceType: 'note',
        resourceId: noteId,
        userId,
        organizationId: orgId,
        details: {
          accessLevel: body.accessLevel,
          canEdit: body.canEdit,
        },
      });
    }
    
    logger.info('Note updated', {
      noteId,
      userId,
      organizationId: orgId,
    });
    
    return NextResponse.json(updatedNote);
  } catch (error) {
    logger.error('Failed to update note', { error });
    
    return NextResponse.json(
      { message: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

#### 3.3.5 Error Handling
- Validation errors: Show inline validation messages
- Lock conflicts: Show warning with options to view or take lock (if expired)
- Permission errors: Redirect to view-only mode with message
- Submission errors: Show error message with retry option
- Optimistic updates with rollback on error

#### 3.3.6 Testing Requirements
- Test lock mechanism
- Test permission checks
- Test form validation
- Test successful updates
- Test conflict resolution
- Test error handling

### 3.4 Note Viewing

#### 3.4.1 Description
The note viewing feature allows users to read notes they have access to, with appropriate rendering of markdown content.

#### 3.4.2 UI Components
- Note title display
- Rendered markdown content
- Access information
- Edit button (if permitted)
- Lock status indicator
- Sharing options

#### 3.4.3 Data Requirements
- Note data
- User permissions
- Lock status
- Organization member information

#### 3.4.4 Implementation Details

**View Note Page:**
```tsx
// app/dashboard/notes/[noteId]/page.tsx
import { redirect } from 'next/navigation';
import { PageTitle } from '@/components/layout/page-title';
import { NoteViewer } from '@/lib/notes/components/note-viewer';
import { NoteActions } from '@/lib/notes/components/note-actions';
import { NoteAccessInfo } from '@/lib/notes/components/note-access-info';
import { getNoteById } from '@/lib/notes/api/get-note-by-id';
import { getCurrentUser } from '@/lib/auth/get-current-user';
import { getCurrentOrg } from '@/lib/auth/get-current-org';
import { canUserViewNote, canUserEditNote } from '@/lib/notes/utils/access-checks';

interface ViewNotePageProps {
  params: { noteId: string };
}

export default async function ViewNotePage({ params }: ViewNotePageProps) {
  const user = await getCurrentUser();
  const org = await getCurrentOrg();
  
  if (!user || !org) {
    redirect('/auth/sign-in');
  }
  
  const note = await getNoteById(params.noteId);
  
  if (!note) {
    redirect('/dashboard/notes');
  }
  
  // Check if user has permission to view the note
  const canView = await canUserViewNote({
    note,
    userId: user.id,
    organizationId: org.id,
    userRole: org.role,
  });
  
  if (!canView) {
    redirect('/dashboard/notes');
  }
  
  // Check if user has permission to edit the note
  const canEdit = await canUserEditNote({
    note,
    userId: user.id,
    organizationId: org.id,
    userRole: org.role,
  });
  
  return (
    <div className="container mx-auto px-4 py-8">
      <div className="mb-8 flex flex-col space-y-4 sm:flex-row sm:items-center sm:justify-between sm:space-y-0">
        <PageTitle title={note.title} />
        <NoteActions note={note} canEdit={canEdit} />
      </div>
      
      <div className="mx-auto max-w-4xl">
        <div className="overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm">
          <div className="border-b border-gray-200 bg-gray-50 px-6 py-4">
            <NoteAccessInfo note={note} isCreator={note.createdByUserId === user.id} />
          </div>
          
          <div className="px-6 py-6">
            <NoteViewer content={note.content} />
          </div>
        </div>
      </div>
    </div>
  );
}
```

**Note Viewer Component:**
```tsx
// lib/notes/components/note-viewer.tsx
'use client';

import { useEffect, useState } from 'react';
import ReactMarkdown from 'react-markdown';
import rehypeRaw from 'rehype-raw';
import rehypeSanitize from 'rehype-sanitize';
import remarkGfm from 'remark-gfm';
import { Skeleton } from '@/components/ui/skeleton';

interface NoteViewerProps {
  content: string;
}

export function NoteViewer({ content }: NoteViewerProps) {
  const [isClient, setIsClient] = useState(false);
  
  useEffect(() => {
    setIsClient(true);
  }, []);
  
  if (!isClient) {
    return <NoteViewerSkeleton />;
  }
  
  return (
    <div className="prose prose-indigo max-w-none">
      <ReactMarkdown
        remarkPlugins={[remarkGfm]}
        rehypePlugins={[rehypeRaw, rehypeSanitize]}
      >
        {content}
      </ReactMarkdown>
    </div>
  );
}

function NoteViewerSkeleton() {
  return (
    <div className="space-y-4">
      <Skeleton className="h-6 w-3/4" />
      <Skeleton className="h-4 w-full" />
      <Skeleton className="h-4 w-5/6" />
      <Skeleton className="h-4 w-full" />
      <Skeleton className="h-4 w-4/5" />
      <Skeleton className="h-4 w-full" />
      <Skeleton className="h-4 w-3/4" />
      <Skeleton className="h-4 w-full" />
    </div>
  );
}
```

**Note Actions Component:**
```tsx
// lib/notes/components/note-actions.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { Button } from '@/components/ui/button';
import { DropdownMenu } from '@/components/ui/dropdown-menu';
import { useLockNote } from '@/lib/notes/hooks/use-lock-note';
import { useDeleteNote } from '@/lib/notes/hooks/use-delete-note';
import { Note } from '@/lib/notes/types';
import { logger } from '@/lib/logger/client-logger';

interface NoteActionsProps {
  note: Note;
  canEdit: boolean;
}

export function NoteActions({ note, canEdit }: NoteActionsProps) {
  const router = useRouter();
  const [isDeleting, setIsDeleting] = useState(false);
  
  const { mutate: lockNote, isPending: isLocking } = useLockNote();
  const { mutate: deleteNote, isPending: isPendingDelete } = useDeleteNote();
  
  const handleEdit = () => {
    if (!canEdit) return;
    
    lockNote(note.id, {
      onSuccess: () => {
        logger.info('Note locked for editing', { noteId: note.id });
        router.push(`/dashboard/notes/${note.id}/edit`);
      },
      onError: (error) => {
        logger.error('Failed to lock note', { noteId: note.id, error });
      },
    });
  };
  
  const handleDelete = () => {
    if (!confirm('Are you sure you want to delete this note? This action cannot be undone.')) {
      return;
    }
    
    setIsDeleting(true);
    
    deleteNote(note.id, {
      onSuccess: () => {
        logger.info('Note deleted', { noteId: note.id });
        router.push('/dashboard/notes');
      },
      onError: (error) => {
        logger.error('Failed to delete note', { noteId: note.id, error });
        setIsDeleting(false);
      },
    });
  };
  
  return (
    <div className="flex space-x-2">
      {canEdit && (
        <Button
          variant="primary"
          onClick={handleEdit}
          disabled={isLocking}
          leftIcon={PencilIcon}
        >
          {isLocking ? 'Preparing Edit...' : 'Edit'}
        </Button>
      )}
      
      <DropdownMenu>
        <DropdownMenu.Trigger asChild>
          <Button variant="outline" icon={EllipsisVerticalIcon} />
        </DropdownMenu.Trigger>
        <DropdownMenu.Content>
          <DropdownMenu.Item
            icon={ArrowTopRightOnSquareIcon}
            onClick={() => window.open(`/dashboard/notes/${note.id}`, '_blank')}
          >
            Open in new tab
          </DropdownMenu.Item>
          
          {canEdit && (
            <DropdownMenu.Item
              icon={PencilIcon}
              onClick={handleEdit}
              disabled={isLocking}
            >
              Edit note
            </DropdownMenu.Item>
          )}
          
          <DropdownMenu.Item
            icon={DocumentDuplicateIcon}
            onClick={() => navigator.clipboard.writeText(window.location.href)}
          >
            Copy link
          </DropdownMenu.Item>
          
          <DropdownMenu.Separator />
          
          {canEdit && (
            <DropdownMenu.Item
              icon={TrashIcon}
              onClick={handleDelete}
              disabled={isDeleting || isPendingDelete}
              className="text-red-600 hover:bg-red-50 hover:text-red-700"
            >
              Delete note
            </DropdownMenu.Item>
          )}
        </DropdownMenu.Content>
      </DropdownMenu>
    </div>
  );
}
```

**Note Access Info Component:**
```tsx
// lib/notes/components/note-access-info.tsx
import { useOrganization } from '@clerk/nextjs';
import { Badge } from '@/components/ui/badge';
import { Tooltip } from '@/components/ui/tooltip';
import { AccessLevelBadge } from './access-level-badge';
import { Note } from '@/lib/notes/types';

interface NoteAccessInfoProps {
  note: Note;
  isCreator: boolean;
}

export function NoteAccessInfo({ note, isCreator }: NoteAccessInfoProps) {
  const { organization, membershipList, isLoaded } = useOrganization({
    membershipList: true,
  });
  
  if (!isLoaded || !organization || !membershipList) {
    return (
      <div className="flex items-center space-x-2">
        <AccessLevelBadge accessLevel={note.accessLevel} />
      </div>
    );
  }
  
  const getAccessDescription = () => {
    switch (note.accessLevel) {
      case 'public':
        return 'Visible to all organization members';
      case 'shared':
        return 'Shared with specific members';
      case 'private':
        return isCreator ? 'Only visible to you' : 'Only visible to the creator and specific members';
      default:
        return '';
    }
  };
  
  const getEditDescription = () => {
    switch (note.canEdit) {
      case 'everyone':
        return 'Anyone who can view can also edit';
      case 'accessList':
        return 'Only specific members can edit';
      case 'denyList':
        return 'Everyone except specific members can edit';
      case 'noOne':
        return isCreator ? 'Only you can edit' : 'This note is read-only';
      default:
        return '';
    }
  };
  
  const creatorUser = membershipList.find(
    (member) => member.publicUserData.userId === note.createdByUserId
  );
  
  const creatorName = creatorUser
    ? `${creatorUser.publicUserData.firstName} ${creatorUser.publicUserData.lastName}`
    : 'Unknown user';
  
  return (
    <div className="flex flex-wrap items-center gap-2 text-sm text-gray-500">
      <AccessLevelBadge accessLevel={note.accessLevel} />
      
      <Tooltip content={getAccessDescription()}>
        <div className="flex cursor-help items-center">
          <EyeIcon className="mr-1 h-4 w-4" />
          <span>{getAccessDescription()}</span>
        </div>
      </Tooltip>
      
      <span className="text-gray-300">•</span>
      
      <Tooltip content={getEditDescription()}>
        <div className="flex cursor-help items-center">
          <PencilIcon className="mr-1 h-4 w-4" />
          <span>{getEditDescription()}</span>
        </div>
      </Tooltip>
      
      <span className="text-gray-300">•</span>
      
      <div className="flex items-center">
        <UserIcon className="mr-1 h-4 w-4" />
        <span>Created by {creatorName}</span>
      </div>
    </div>
  );
}
```

**Get Note API Function:**
```typescript
// lib/notes/api/get-note-by-id.ts
import { Note } from '@/lib/notes/types';

export async function getNoteById(noteId: string): Promise<Note | null> {
  try {
    const response = await fetch(`/api/notes/${noteId}`, {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
      },
    });
    
    if (!response.ok) {
      if (response.status === 404) {
        return null;
      }
      
      throw new Error('Failed to fetch note');
    }
    
    return response.json();
  } catch (error) {
    console.error('Error fetching note:', error);
    return null;
  }
}
```

**Get Note API Route:**
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
        { message: 'Unauthorized' },
        { status: 401 }
      );
    }
    
    const noteId = params.noteId;
    
    // Get the note
    const note = await NoteRepository.getNoteById(noteId, userId, orgId);
    
    if (!note) {
      return NextResponse.json(
        { message: 'Note not found' },
        { status: 404 }
      );
    }
    
    logger.info('Note accessed', {
      noteId,
      userId,
      organizationId: orgId,
    });
    
    return NextResponse.json(note);
  } catch (error) {
    logger.error('Failed to get note', { error });
    
    return NextResponse.json(
      { message: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

#### 3.4.5 Error Handling
- Note not found: Redirect to notes list with message
- Permission errors: Redirect to notes list with access denied message
- Load errors: Show error state with retry option
- Organization access errors: Redirect to organization selection

#### 3.4.6 Testing Requirements
- Test markdown rendering
- Test permission-based access control
- Test note actions based on permissions
- Test loading states
- Test error states

### 3.5 Note Locking Mechanism

#### 3.5.1 Description
The note locking mechanism prevents concurrent editing of the same note by multiple users, avoiding conflicts and data loss.

#### 3.5.2 UI Components
- Lock status indicator
- Lock expiration information
- Take lock button (if lock is expired)
- Lock warning messages

#### 3.5.3 Data Requirements
- Note lock status
- Lock owner information
- Lock timestamp
- User permissions

#### 3.5.4 Implementation Details

**Note Lock Hook:**
```typescript
// lib/notes/hooks/use-lock-note.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { lockNote } from '@/lib/notes/api/lock-note';
import { Note } from '@/lib/notes/types';

export function useLockNote() {
  const queryClient = useQueryClient();
  
  return useMutation<Note, Error, string>({
    mutationFn: lockNote,
    onSuccess: (updatedNote) => {
      // Update the note in the cache
      queryClient.setQueryData(['note', updatedNote.id], updatedNote);
    },
  });
}
```

**Lock Note API Function:**
```typescript
// lib/notes/api/lock-note.ts
import { Note } from '@/lib/notes/types';

export async function lockNote(noteId: string): Promise<Note> {
  const response = await fetch(`/api/notes/${noteId}/lock`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
  });
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'Failed to lock note');
  }
  
  return response.json();
}
```

**Lock Note API Route:**
```typescript
// app/api/notes/[noteId]/lock/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@clerk/nextjs/server';
import { NoteRepository } from '@/lib/notes/repositories/note-repository';
import { AuditRepository } from '@/lib/audit/repositories/audit-repository';
import { logger } from '@/lib/logger/server-logger';

export async function POST(
  req: NextRequest,
  { params }: { params: { noteId: string } }
) {
  try {
    const { userId, orgId } = auth();
    
    if (!userId || !orgId) {
      return NextResponse.json(
        { message: 'Unauthorized' },
        { status: 401 }
      );
    }
    
    const noteId = params.noteId;
    
    // Check if the note exists and belongs to the organization
    const note = await NoteRepository.getNoteById(noteId, userId, orgId);
    
    if (!note) {
      return NextResponse.json(
        { message: 'Note not found' },
        { status: 404 }
      );
    }
    
    // Check if the user can edit the note
    const canEdit = await NoteRepository.canUserEditNote(noteId, userId);
    
    if (!canEdit) {
      return NextResponse.json(
        { message: 'You do not have permission to edit this note' },
        { status: 403 }
      );
    }
    
    // Check if the note is already locked by another user
    if (
      note.locked_by_user_id &&
      note.locked_by_user_id !== userId &&
      note.locked_at &&
      new Date(note.locked_at).getTime() > Date.now() - 15 * 60 * 1000
    ) {
      return NextResponse.json(
        { 
          message: 'Note is locked by another user',
          lockedByUserId: note.locked_by_user_id,
          lockedAt: note.locked_at
        },
        { status: 423 }
      );
    }
    
    // Lock the note
    const lockedNote = await NoteRepository.lockNote(noteId, userId);
    
    // Log the action
    await AuditRepository.logEvent({
      action: 'lock',
      resourceType: 'note',
      resourceId: noteId,
      userId,
      organizationId: orgId,
      details: {
        lockTime: new Date(),
      },
    });
    
    logger.info('Note locked', {
      noteId,
      userId,
      organizationId: orgId,
    });
    
    return NextResponse.json(lockedNote);
  } catch (error) {
    logger.error('Failed to lock note', { error });
    
    return NextResponse.json(
      { message: 'Internal server error' },
      { status: 500 }
    );
  }
}

export async function DELETE(
  req: NextRequest,
  { params }: { params: { noteId: string } }
) {
  try {
    const { userId, orgId } = auth();
    
    if (!userId || !orgId) {
      return NextResponse.json(
        { message: 'Unauthorized' },
        { status: 401 }
      );
    }
    
    const noteId = params.noteId;
    
    // Check if the note exists and belongs to the organization
    const note = await NoteRepository.getNoteById(noteId, userId, orgId);
    
    if (!note) {
      return NextResponse.json(
        { message: 'Note not found' },
        { status: 404 }
      );
    }
    
    // Only the lock owner or an admin can unlock the note
    if (
      note.locked_by_user_id &&
      note.locked_by_user_id !== userId
      // We could check for admin role here as well
    ) {
      return NextResponse.json(
        { message: 'You do not have permission to unlock this note' },
        { status: 403 }
      );
    }
    
    // Unlock the note
    const unlockedNote = await NoteRepository.unlockNote(noteId);
    
    // Log the action
    await AuditRepository.logEvent({
      action: 'unlock',
      resourceType: 'note',
      resourceId: noteId,
      userId,
      organizationId: orgId,
    });
    
    logger.info('Note unlocked', {
      noteId,
      userId,
      organizationId: orgId,
    });
    
    return NextResponse.json(unlockedNote);
  } catch (error) {
    logger.error('Failed to unlock note', { error });
    
    return NextResponse.json(
      { message: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

**Use Lock Hook:**
```typescript
// lib/notes/hooks/use-note-lock.ts
import { useState, useEffect } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { lockNote } from '@/lib/notes/api/lock-note';
import { unlockNote } from '@/lib/notes/api/unlock-note';
import { getNoteById } from '@/lib/notes/api/get-note-by-id';
import { logger } from '@/lib/logger/client-logger';

export function useNoteLock(noteId: string, userId: string) {
  const queryClient = useQueryClient();
  const [refreshInterval, setRefreshInterval] = useState<number | false>(false);
  
  // Get note data
  const { data: note, isLoading } = useQuery({
    queryKey: ['note', noteId],
    queryFn: () => getNoteById(noteId),
    enabled: !!noteId,
  });
  
  // Lock mutations
  const lockMutation = useMutation({
    mutationFn: lockNote,
    onSuccess: (updatedNote) => {
      queryClient.setQueryData(['note', noteId], updatedNote);
      setRefreshInterval(30000); // Refresh every 30 seconds to keep lock active
    },
  });
  
  const unlockMutation = useMutation({
    mutationFn: unlockNote,
    onSuccess: (updatedNote) => {
      queryClient.setQueryData(['note', noteId], updatedNote);
      setRefreshInterval(false);
    },
  });
  
  // Determine lock status
  const isLocked = note?.lockedByUserId && note.lockedAt;
  const isLockedByMe = isLocked && note.lockedByUserId === userId;
  const isLockedByOther = isLocked && note.lockedByUserId !== userId;
  
  // Check if lock is expired (15 minutes)
  const isLockExpired = isLocked && note.lockedAt && 
    new Date(note.lockedAt).getTime() < Date.now() - 15 * 60 * 1000;
  
  // Handle lock refresh
  useEffect(() => {
    if (refreshInterval && isLockedByMe && !isLockExpired) {
      const intervalId = setInterval(() => {
        lockMutation.mutate(noteId);
        logger.info('Refreshing note lock', { noteId });
      }, refreshInterval);
      
      return () => clearInterval(intervalId);
    }
  }, [refreshInterval, isLockedByMe, isLockExpired, noteId, lockMutation]);
  
  // Clean up lock on unmount
  useEffect(() => {
    return () => {
      if (isLockedByMe) {
        unlockMutation.mutate(noteId);
        logger.info('Releasing note lock on unmount', { noteId });
      }
    };
  }, [isLockedByMe, noteId, unlockMutation]);
  
  // Actions
  const acquireLock = () => {
    if (isLockedByOther && !isLockExpired) {
      return false;
    }
    
    lockMutation.mutate(noteId);
    return true;
  };
  
  const releaseLock = () => {
    if (!isLockedByMe) {
      return false;
    }
    
    unlockMutation.mutate(noteId);
    return true;
  };
  
  // Calculate time remaining on lock
  const lockExpiryTime = isLocked && note.lockedAt
    ? new Date(new Date(note.lockedAt).getTime() + 15 * 60 * 1000)
    : null;
  
  return {
    isLoading,
    isLocked,
    isLockedByMe,
    isLockedByOther,
    isLockExpired,
    lockOwner: note?.lockedByUserId,
    lockTime: note?.lockedAt,
    lockExpiryTime,
    acquireLock,
    releaseLock,
    isAcquiringLock: lockMutation.isPending,
    isReleasingLock: unlockMutation.isPending,
  };
}
```

#### 3.5.5 Error Handling
- Lock acquisition failure: Show error message with retry option
- Lock conflict: Show warning with view-only option or take lock (if expired)
- Lock release failure: Show error message but allow user to continue
- Lock timeout: Warn user before lock expires, with option to extend

#### 3.5.6 Testing Requirements
- Test lock acquisition
- Test lock conflict handling
- Test lock expiration
- Test lock release
- Test auto lock refresh
- Test lock takeover

### 3.6 Note Deletion

#### 3.6.1 Description
The note deletion feature allows users with appropriate permissions to permanently remove notes from the system.

#### 3.6.2 UI Components
- Delete button
- Confirmation dialog
- Success/error notifications

#### 3.6.3 Data Requirements
- Note ID
- User permissions
- Organization ID

#### 3.6.4 Implementation Details

**Delete Note Hook:**
```typescript
// lib/notes/hooks/use-delete-note.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { deleteNote } from '@/lib/notes/api/delete-note';

export function useDeleteNote() {
  const queryClient = useQueryClient();
  
  return useMutation<void, Error, string>({
    mutationFn: deleteNote,
    onSuccess: (_, noteId) => {
      // Remove the note from the cache
      queryClient.removeQueries({ queryKey: ['note', noteId] });
      
      // Invalidate the notes list
      queryClient.invalidateQueries({ queryKey: ['notes'] });
    },
  });
}
```

**Delete Note API Function:**
```typescript
// lib/notes/api/delete-note.ts
export async function deleteNote(noteId: string): Promise<void> {
  const response = await fetch(`/api/notes/${noteId}`, {
    method: 'DELETE',
    headers: {
      'Content-Type': 'application/json',
    },
  });
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'Failed to delete note');
  }
}
```

**Delete Note API Route:**
```typescript
// app/api/notes/[noteId]/route.ts (DELETE method)
import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@clerk/nextjs/server';
import { NoteRepository } from '@/lib/notes/repositories/note-repository';
import { AuditRepository } from '@/lib/audit/repositories/audit-repository';
import { logger } from '@/lib/logger/server-logger';

export async function DELETE(
  req: NextRequest,
  { params }: { params: { noteId: string } }
) {
  try {
    const { userId, orgId } = auth();
    
    if (!userId || !orgId) {
      return NextResponse.json(
        { message: 'Unauthorized' },
        { status: 401 }
      );
    }
    
    const noteId = params.noteId;
    
    // Check if the note exists and belongs to the organization
    const note = await NoteRepository.getNoteById(noteId, userId, orgId);
    
    if (!note) {
      return NextResponse.json(
        { message: 'Note not found' },
        { status: 404 }
      );
    }
    
    // Only the creator or an admin can delete the note
    // We would need to check the user's role in the organization
    if (note.created_by_user_id !== userId) {
      // Check if user is admin
      const isAdmin = false; // Replace with actual role check
      
      if (!isAdmin) {
        return NextResponse.json(
          { message: 'You do not have permission to delete this note' },
          { status: 403 }
        );
      }
    }
    
    // Delete the note
    await NoteRepository.deleteNote(noteId);
    
    // Log the action
    await AuditRepository.logEvent({
      action: 'delete',
      resourceType: 'note',
      resourceId: noteId,
      userId,
      organizationId: orgId,
      details: {
        title: note.title,
      },
    });
    
    logger.info('Note deleted', {
      noteId,
      userId,
      organizationId: orgId,
    });
    
    return new NextResponse(null, { status: 204 });
  } catch (error) {
    logger.error('Failed to delete note', { error });
    
    return NextResponse.json(
      { message: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

#### 3.6.5 Error Handling
- Deletion failure: Show error message with retry option
- Permission errors: Show access denied message
- Network errors: Show connection error with retry option
- Optimistic deletion with rollback on error

#### 3.6.6 Testing Requirements
- Test deletion confirmation
- Test successful deletion
- Test permission-based deletion
- Test error handling

## 4. UI Component Library

### 4.1 Design System

#### 4.1.1 Description
The design system provides consistent styling and components throughout the application, following Clerk's minimal design aesthetic.

#### 4.1.2 Components Structure
- Core UI components (buttons, inputs, etc.)
- Layout components (containers, grids, etc.)
- Data display components (tables, cards, etc.)
- Feedback components (alerts, toasts, etc.)
- Navigation components (tabs, breadcrumbs, etc.)

#### 4.1.3 Design Tokens
- Colors: Clerk-style color palette with indigo as primary color
- Typography: Sans-serif fonts with clear hierarchy
- Spacing: Consistent spacing scale
- Shadows: Minimal shadow system for depth
- Borders: Subtle border system for containment

#### 4.1.4 Implementation Details

**TailwindCSS Configuration:**
```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
    './lib/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        indigo: {
          50: '#eef2ff',
          100: '#e0e7ff',
          200: '#c7d2fe',
          300: '#a5b4fc',
          400: '#818cf8',
          500: '#6366f1',
          600: '#4f46e5',
          700: '#4338ca',
          800: '#3730a3',
          900: '#312e81',
          950: '#1e1b4b',
        },
      },
      fontFamily: {
        sans: [
          'Inter',
          'ui-sans-serif',
          'system-ui',
          '-apple-system',
          'BlinkMacSystemFont',
          '"Segoe UI"',
          'Roboto',
          '"Helvetica Neue"',
          'Arial',
          '"Noto Sans"',
          'sans-serif',
          '"Apple Color Emoji"',
          '"Segoe UI Emoji"',
          '"Segoe UI Symbol"',
          '"Noto Color Emoji"',
        ],
      },
      boxShadow: {
        sm: '0 1px 2px 0 rgba(0, 0, 0, 0.05)',
        DEFAULT: '0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06)',
        md: '0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06)',
        lg: '0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05)',
        xl: '0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04)',
        none: 'none',
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms')({
      strategy: 'class',
    }),
    require('@tailwindcss/typography'),
  ],
};
```

**Button Component:**
```tsx
// components/ui/button.tsx
import { forwardRef } from 'react';
import Link from 'next/link';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils/cn';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        primary: 'bg-indigo-600 text-white hover:bg-indigo-700 focus-visible:ring-indigo-500',
        secondary: 'bg-indigo-100 text-indigo-900 hover:bg-indigo-200 focus-visible:ring-indigo-500',
        outline: 'border border-gray-300 bg-transparent hover:bg-gray-50 focus-visible:ring-gray-500',
        ghost: 'bg-transparent hover:bg-gray-100 focus-visible:ring-gray-500',
        link: 'bg-transparent text-indigo-600 underline-offset-4 hover:underline focus-visible:ring-indigo-500 p-0 h-auto',
        danger: 'bg-red-600 text-white hover:bg-red-700 focus-visible:ring-red-500',
        text: 'bg-transparent text-gray-700 hover:bg-gray-100 hover:text-gray-900 focus-visible:ring-gray-500',
      },
      size: {
        sm: 'h-8 px-3 text-xs',
        md: 'h-10 px-4',
        lg: 'h-12 px-6 text-base',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
  href?: string;
  external?: boolean;
}

const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, leftIcon, rightIcon, href, external, children, ...props }, ref) => {
    const Comp = href ? Link : 'button';
    const linkProps = href
      ? {
          href,
          ...(external ? { target: '_blank', rel: 'noopener noreferrer' } : {}),
        }
      : {};
    
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...linkProps}
        {...props}
      >
        {leftIcon && <span className="mr-2">{leftIcon}</span>}
        {children}
        {rightIcon && <span className="ml-2">{rightIcon}</span>}
      </Comp>
    );
  }
);

Button.displayName = 'Button';

export { Button, buttonVariants };
```

**Input Component:**
```tsx
// components/ui/input.tsx
import { forwardRef } from 'react';
import { cn } from '@/lib/utils/cn';

export interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ className, label, error, leftIcon, rightIcon, id, ...props }, ref) => {
    const inputId = id || `input-${Math.random().toString(36).substring(2, 9)}`;
    
    return (
      <div className="w-full">
        {label && (
          <label
            htmlFor={inputId}
            className="mb-1 block text-sm font-medium text-gray-700"
          >
            {label}
          </label>
        )}
        
        <div className="relative">
          {leftIcon && (
            <div className="pointer-events-none absolute inset-y-0 left-0 flex items-center pl-3 text-gray-500">
              {leftIcon}
            </div>
          )}
          
          <input
            id={inputId}
            className={cn(
              'block w-full rounded-md border border-gray-300 bg-white px-3 py-2 shadow-sm placeholder:text-gray-400 focus:border-indigo-500 focus:outline-none focus:ring-indigo-500 disabled:cursor-not-allowed disabled:bg-gray-50 disabled:text-gray-500',
              leftIcon && 'pl-10',
              rightIcon && 'pr-10',
              error && 'border-red-300 focus:border-red-500 focus:ring-red-500',
              className
            )}
            ref={ref}
            {...props}
          />
          
          {rightIcon && (
            <div className="absolute inset-y-0 right-0 flex items-center pr-3 text-gray-500">
              {rightIcon}
            </div>
          )}
        </div>
        
        {error && (
          <p className="mt-1 text-sm text-red-600">{error}</p>
        )}
      </div>
    );
  }
);

Input.displayName = 'Input';

export { Input };
```

### 4.2 Component Documentation

#### 4.2.1 Button Component

- **Purpose**: Provide a consistent button style throughout the application
- **Variants**: primary, secondary, outline, ghost, link, danger, text
- **Sizes**: sm, md, lg, icon
- **Props**:
  - `variant`: Button style variant
  - `size`: Button size
  - `leftIcon`: Icon to display before button text
  - `rightIcon`: Icon to display after button text
  - `href`: Optional URL (transforms button into a link)
  - `external`: If true and href is provided, opens in new tab
  - Standard HTML button attributes

#### 4.2.2 Input Component

- **Purpose**: Provide a consistent input style throughout the application
- **Props**:
  - `label`: Optional label text
  - `error`: Optional error message
  - `leftIcon`: Icon to display at the left side of the input
  - `rightIcon`: Icon to display at the right side of the input
  - Standard HTML input attributes

#### 4.2.3 Card Component

- **Purpose**: Provide a consistent card container style
- **Props**:
  - `title`: Optional card title
  - `description`: Optional card description
  - `footer`: Optional card footer content
  - `className`: Additional CSS classes

#### 4.2.4 Alert Component

- **Purpose**: Display alert messages to users
- **Variants**: info, success, warning, error
- **Props**:
  - `variant`: Alert style variant
  - `title`: Optional alert title
  - `icon`: Optional custom icon
  - `onClose`: Optional callback for dismissible alerts
  - `className`: Additional CSS classes

#### 4.2.5 Modal Component

- **Purpose**: Display modal dialogs
- **Props**:
  - `isOpen`: Controls modal visibility
  - `onClose`: Callback when modal is closed
  - `title`: Optional modal title
  - `description`: Optional modal description
  - `footer`: Optional modal footer content
  - `size`: Modal size (sm, md, lg, xl, full)
  - `className`: Additional CSS classes

## 5. Testing Strategy

### 5.1 Testing Approach

#### 5.1.1 Unit Testing
- Test isolated components and utility functions
- Focus on business logic and complex calculations
- Use Jest and React Testing Library
- Aim for high coverage of critical paths

#### 5.1.2 Integration Testing
- Test interactions between components
- Test API routes with mock requests
- Verify data flow through the application
- Focus on user workflows

#### 5.1.3 End-to-End Testing
- Test complete user journeys
- Use Cypress or Playwright
- Cover critical user flows
- Include authentication and multi-user scenarios

### 5.2 Test Structure

#### 5.2.1 Unit Tests

**Utility Function Tests:**
```typescript
// lib/utils/access-checks.test.ts
import { canUserViewNote, canUserEditNote } from '../access-checks';

describe('Access Check Utilities', () => {
  describe('canUserViewNote', () => {
    test('admin can view any note', () => {
      const note = {
        id: 'note-1',
        accessLevel: 'private',
        createdByUserId: 'user-2',
        viewAccessList: '',
      };
      
      const result = canUserViewNote({
        note,
        userId: 'user-1',
        organizationId: 'org-1',
        userRole: 'admin',
      });
      
      expect(result).toBe(true);
    });
    
    test('creator can view their private note', () => {
      const note = {
        id: 'note-1',
        accessLevel: 'private',
        createdByUserId: 'user-1',
        viewAccessList: '',
      };
      
      const result = canUserViewNote({
        note,
        userId: 'user-1',
        organizationId: 'org-1',
        userRole: 'member',
      });
      
      expect(result).toBe(true);
    });
    
    test('non-creator cannot view private note', () => {
      const note = {
        id: 'note-1',
        accessLevel: 'private',
        createdByUserId: 'user-2',
        viewAccessList: '',
      };
      
      const result = canUserViewNote({
        note,
        userId: 'user-1',
        organizationId: 'org-1',
        userRole: 'member',
      });
      
      expect(result).toBe(false);
    });
    
    test('any organization member can view public note', () => {
      const note = {
        id: 'note-1',
        accessLevel: 'public',
        createdByUserId: 'user-2',
        viewAccessList: '',
      };
      
      const result = canUserViewNote({
        note,
        userId: 'user-1',
        organizationId: 'org-1',
        userRole: 'member',
      });
      
      expect(result).toBe(true);
    });
    
    test('user in view access list can view shared note', () => {
      const note = {
        id: 'note-1',
        accessLevel: 'shared',
        createdByUserId: 'user-2',
        viewAccessList: 'user-1,user-3',
      };
      
      const result = canUserViewNote({
        note,
        userId: 'user-1',
        organizationId: 'org-1',
        userRole: 'member',
      });
      
      expect(result).toBe(true);
    });
    
    test('user not in view access list cannot view shared note', () => {
      const note = {
        id: 'note-1',
        accessLevel: 'shared',
        createdByUserId: 'user-2',
        viewAccessList: 'user-3,user-4',
      };
      
      const result = canUserViewNote({
        note,
        userId: 'user-1',
        organizationId: 'org-1',
        userRole: 'member',
      });
      
      expect(result).toBe(false);
    });
  });
  
  // Similar tests for canUserEditNote
});
```

**React Component Tests:**
```tsx
// components/ui/button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './button';

describe('Button Component', () => {
  test('renders button with text', () => {
    render(<Button>Click me</Button>);
    
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });
  
  test('applies variant class', () => {
    render(<Button variant="secondary">Secondary</Button>);
    
    const button = screen.getByRole('button', { name: /secondary/i });
    expect(button).toHaveClass('bg-indigo-100');
  });
  
  test('applies size class', () => {
    render(<Button size="lg">Large</Button>);
    
    const button = screen.getByRole('button', { name: /large/i });
    expect(button).toHaveClass('h-12');
  });
  
  test('renders as link when href is provided', () => {
    render(<Button href="/test">Link</Button>);
    
    const link = screen.getByRole('link', { name: /link/i });
    expect(link).toHaveAttribute('href', '/test');
  });
  
  test('handles click events', async () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await userEvent.click(screen.getByRole('button', { name: /click me/i }));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  test('renders with left icon', () => {
    render(<Button leftIcon={<span data-testid="left-icon" />}>With Icon</Button>);
    
    expect(screen.getByTestId('left-icon')).toBeInTheDocument();
  });
  
  test('renders with right icon', () => {
    render(<Button rightIcon={<span data-testid="right-icon" />}>With Icon</Button>);
    
    expect(screen.getByTestId('right-icon')).toBeInTheDocument();
  });
  
  test('applies disabled state', () => {
    render(<Button disabled>Disabled</Button>);
    
    expect(screen.getByRole('button', { name: /disabled/i })).toBeDisabled();
  });
});
```

#### 5.2.2 Integration Tests

**API Route Tests:**
```typescript
// app/api/notes/route.test.ts
import { createMocks } from 'node-mocks-http';
import { POST } from './route';
import { NoteRepository } from '@/lib/notes/repositories/note-repository';
import { AuditRepository } from '@/lib/audit/repositories/audit-repository';

// Mock dependencies
jest.mock('@clerk/nextjs/server', () => ({
  auth: jest.fn(() => ({
    userId: 'test-user-id',
    orgId: 'test-org-id',
  })),
}));

jest.mock('@/lib/notes/repositories/note-repository');
jest.mock('@/lib/audit/repositories/audit-repository');
jest.mock('@/lib/logger/server-logger');

describe('Notes API Routes', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  describe('POST /api/notes', () => {
    test('creates a new note successfully', async () => {
      // Mock request
      const { req, res } = createMocks({
        method: 'POST',
        body: {
          title: 'Test Note',
          content: 'Test content',
          accessLevel: 'public',
          canEdit: 'everyone',
          organizationId: 'test-org-id',
        },
      });
      
      // Mock repository response
      NoteRepository.createNote.mockResolvedValue({
        id: 'test-note-id',
        title: 'Test Note',
        content: 'Test content',
        access_level: 'public',
        can_edit: 'everyone',
        created_by_user_id: 'test-user-id',
        organization_id: 'test-org-id',
      });
      
      AuditRepository.logEvent.mockResolvedValue(undefined);
      
      // Call the route handler
      await POST(req, res);
      
      // Assertions
      expect(res._getStatusCode()).toBe(201);
      expect(JSON.parse(res._getData())).toHaveProperty('id', 'test-note-id');
      expect(NoteRepository.createNote).toHaveBeenCalledWith(
        expect.objectContaining({
          title: 'Test Note',
          content: 'Test content',
          accessLevel: 'public',
          canEdit: 'everyone',
          createdByUserId: 'test-user-id',
          organizationId: 'test-org-id',
        })
      );
      expect(AuditRepository.logEvent).toHaveBeenCalledWith(
        expect.objectContaining({
          action: 'create',
          resourceType: 'note',
          resourceId: 'test-note-id',
          userId: 'test-user-id',
          organizationId: 'test-org-id',
        })
      );
    });
    
    test('returns 400 for invalid input', async () => {
      // Mock request with invalid input
      const { req, res } = createMocks({
        method: 'POST',
        body: {
          // Missing required fields
          content: 'Test content',
        },
      });
      
      // Call the route handler
      await POST(req, res);
      
      // Assertions
      expect(res._getStatusCode()).toBe(400);
      expect(JSON.parse(res._getData())).toHaveProperty('message', 'Invalid input');
    });
    
    test('returns 401 when not authenticated', async () => {
      // Mock auth to return no user
      require('@clerk/nextjs/server').auth.mockReturnValueOnce({
        userId: null,
        orgId: null,
      });
      
      // Mock request
      const { req, res } = createMocks({
        method: 'POST',
        body: {
          title: 'Test Note',
          content: 'Test content',
          accessLevel: 'public',
          canEdit: 'everyone',
          organizationId: 'test-org-id',
        },
      });
      
      // Call the route handler
      await POST(req, res);
      
      // Assertions
      expect(res._getStatusCode()).toBe(401);
      expect(JSON.parse(res._getData())).toHaveProperty('message', 'Unauthorized');
    });
    
    test('returns 403 when organization ID does not match', async () => {
      // Mock request with mismatched org ID
      const { req, res } = createMocks({
        method: 'POST',
        body: {
          title: 'Test Note',
          content: 'Test content',
          accessLevel: 'public',
          canEdit: 'everyone',
          organizationId: 'different-org-id',
        },
      });
      
      // Call the route handler
      await POST(req, res);
      
      // Assertions
      expect(res._getStatusCode()).toBe(403);
      expect(JSON.parse(res._getData())).toHaveProperty('message', 'Organization mismatch');
    });
  });
});
```

**React Component Integration Tests:**
```tsx
// lib/notes/components/note-form.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { NoteForm } from './note-form';

// Mock dependencies
jest.mock('next/navigation', () => ({
  useRouter: () => ({
    push: jest.fn(),
  }),
}));

jest.mock('@/lib/notes/hooks/use-create-note', () => ({
  useCreateNote: () => ({
    mutate: jest.fn((data, options) => {
      options.onSuccess({ id: 'new-note-id', ...data });
    }),
    isPending: false,
  }),
}));

jest.mock('@/lib/notes/hooks/use-update-note', () => ({
  useUpdateNote: () => ({
    mutate: jest.fn((data, options) => {
      options.onSuccess({ id: data.id, ...data });
    }),
    isPending: false,
  }),
}));

jest.mock('react-markdown-editor-lite', () => {
  return {
    __esModule: true,
    default: ({ value, onChange }) => (
      <div data-testid="markdown-editor">
        <textarea
          data-testid="markdown-textarea"
          value={value}
          onChange={(e) => onChange({ text: e.target.value })}
        />
      </div>
    ),
  };
});

describe('NoteForm Component', () => {
  const queryClient = new QueryClient();
  
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  function setup(props) {
    return render(
      <QueryClientProvider client={queryClient}>
        <NoteForm {...props} />
      </QueryClientProvider>
    );
  }
  
  test('renders form fields correctly for new note', () => {
    const initialNote = {
      title: '',
      content: '',
      accessLevel: 'public',
      canEdit: 'everyone',
      editAccessList: [],
      viewAccessList: [],
    };
    
    setup({
      initialNote,
      userId: 'user-1',
      organizationId: 'org-1',
      mode: 'create',
    });
    
    expect(screen.getByLabelText(/title/i)).toBeInTheDocument();
    expect(screen.getByTestId('markdown-editor')).toBeInTheDocument();
    expect(screen.getByText(/access control/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /create note/i })).toBeInTheDocument();
  });
  
  test('fills form fields correctly for existing note', () => {
    const initialNote = {
      id: 'note-1',
      title: 'Test Note',
      content: 'Test content',
      accessLevel: 'shared',
      canEdit: 'accessList',
      editAccessList: ['user-2'],
      viewAccessList: ['user-2', 'user-3'],
    };
    
    setup({
      initialNote,
      userId: 'user-1',
      organizationId: 'org-1',
      mode: 'edit',
    });
    
    expect(screen.getByLabelText(/title/i)).toHaveValue('Test Note');
    expect(screen.getByTestId('markdown-textarea')).toHaveValue('Test content');
    expect(screen.getByRole('button', { name: /update note/i })).toBeInTheDocument();
  });
  
  test('validates title field', async () => {
    const initialNote = {
      title: '',
      content: '',
      accessLevel: 'public',
      canEdit: 'everyone',
      editAccessList: [],
      viewAccessList: [],
    };
    
    setup({
      initialNote,
      userId: 'user-1',
      organizationId: 'org-1',
      mode: 'create',
    });
    
    const createButton = screen.getByRole('button', { name: /create note/i });
    expect(createButton).toBeDisabled();
    
    await userEvent.type(screen.getByLabelText(/title/i), 'New Test Note');
    
    expect(createButton).not.toBeDisabled();
  });
  
  test('submits form data correctly for new note', async () => {
    const initialNote = {
      title: '',
      content: '',
      accessLevel: 'public',
      canEdit: 'everyone',
      editAccessList: [],
      viewAccessList: [],
    };
    
    const createNoteMock = require('@/lib/notes/hooks/use-create-note').useCreateNote().mutate;
    
    setup({
      initialNote,
      userId: 'user-1',
      organizationId: 'org-1',
      mode: 'create',
    });
    
    await userEvent.type(screen.getByLabelText(/title/i), 'New Test Note');
    await userEvent.type(screen.getByTestId('markdown-textarea'), 'New content');
    
    await userEvent.click(screen.getByRole('button', { name: /create note/i }));
    
    expect(createNoteMock).toHaveBeenCalledWith(
      expect.objectContaining({
        title: 'New Test Note',
        content: 'New content',
        accessLevel: 'public',
        canEdit: 'everyone',
        createdByUserId: 'user-1',
        organizationId: 'org-1',
      }),
      expect.any(Object)
    );
  });
});
```

#### 5.2.3 End-to-End Tests

**Cypress Test for Note Creation:**
```typescript
// cypress/e2e/notes.spec.ts
describe('Notes Functionality', () => {
  beforeEach(() => {
    // Login and set up auth state
    cy.login('test-user@example.com', 'password');
    cy.visit('/dashboard/notes');
  });
  
  it('should create a new note', () => {
    // Click on create note button
    cy.findByRole('button', { name: /create note/i }).click();
    
    // Fill in note details
    cy.findByLabelText(/title/i).type('Cypress Test Note');
    
    // Type in the markdown editor
    cy.get('[data-testid="markdown-editor"] textarea').type('# Test Content\n\nThis is a test note created by Cypress.');
    
    // Submit the form
    cy.findByRole('button', { name: /create note/i }).click();
    
    // Verify redirect to note view
    cy.url().should('include', '/dashboard/notes/');
    
    // Verify note content is displayed
    cy.findByRole('heading', { name: 'Cypress Test Note' }).should('exist');
    cy.findByRole('heading', { name: 'Test Content' }).should('exist');
    cy.findByText('This is a test note created by Cypress.').should('exist');
  });
  
  it('should edit an existing note', () => {
    // Create a note first (or use a fixture)
    cy.createNote('Cypress Edit Test');
    
    // Open the note
    cy.findByText('Cypress Edit Test').click();
    
    // Click edit button
    cy.findByRole('button', { name: /edit/i }).click();
    
    // Update the note
    cy.findByLabelText(/title/i).clear().type('Updated Note Title');
    cy.get('[data-testid="markdown-editor"] textarea').clear().type('# Updated Content\n\nThis note has been updated.');
    
    // Save changes
    cy.findByRole('button', { name: /update note/i }).click();
    
    // Verify updated content
    cy.findByRole('heading', { name: 'Updated Note Title' }).should('exist');
    cy.findByRole('heading', { name: 'Updated Content' }).should('exist');
    cy.findByText('This note has been updated.').should('exist');
  });
  
  it('should handle note locking', () => {
    // Create a test note
    cy.createNote('Lock Test Note');
    
    // Open the note in one browser
    cy.findByText('Lock Test Note').click();
    cy.findByRole('button', { name: /edit/i }).click();
    
    // Open the same note in another browser/session
    cy.session('second-user', () => {
      cy.login('second-user@example.com', 'password');
    });
    
    cy.session('second-user', () => {
      cy.visit('/dashboard/notes');
      cy.findByText('Lock Test Note').click();
      
      // Try to edit
      cy.findByRole('button', { name: /edit/i }).click();
      
      // Should see lock warning
      cy.findByText(/note is currently being edited/i).should('exist');
      cy.findByText(/test-user@example.com/i).should('exist');
      
      // Should have view option
      cy.findByRole('button', { name: /view note/i }).should('exist');
    });
  });
});
```

### 5.3 Test Coverage Goals

#### 5.3.1 Unit Test Coverage
- Business logic: 95%+ coverage
- Utility functions: 90%+ coverage
- UI components: 80%+ coverage

#### 5.3.2 Integration Test Coverage
- API routes: 85%+ coverage
- Form submissions: 80%+ coverage
- Data fetching: 80%+ coverage

#### 5.3.3 End-to-End Test Coverage
- Critical user flows: 100% coverage
- Happy paths: 90%+ coverage
- Edge cases: 70%+ coverage

## 6. Deployment and CI/CD

### 6.1 CI/CD Pipeline

#### 6.1.1 GitHub Actions Workflow

```yaml
# .github/workflows/main.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
  
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: noteorbit
          POSTGRES_PASSWORD: noteorbit
          POSTGRES_DB: noteorbit_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:ci
        env:
          DATABASE_URL: postgresql://noteorbit:noteorbit@localhost:5432/noteorbit_test
  
  build:
    runs-on: ubuntu-latest
    needs: [lint, test]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
  
  deploy:
    runs-on: ubuntu-latest
    needs: [build]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to production
        # Add deployment steps here (e.g., Vercel, AWS, etc.)
```

### 6.2 Infrastructure

#### 6.2.1 Hosting
- **Frontend/API**: Vercel (Next.js optimized)
- **Database**: AWS RDS (PostgreSQL)
- **Authentication**: Clerk (managed service)

#### 6.2.2 Environment Setup

```bash
# .env.production
# Database
DATABASE_URL=postgresql://username:password@hostname:5432/database_name

# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_live_...
CLERK_SECRET_KEY=sk_live_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/auth/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/auth/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard

# Monitoring
NEXT_PUBLIC_SENTRY_DSN=https://...

# Other
NODE_ENV=production
```

#### 6.2.3 Database Migration Strategy

```bash
# package.json scripts
{
  "scripts": {
    // ...
    "db:generate": "prisma generate",
    "db:migrate": "prisma migrate deploy",
    "db:push": "prisma db push",
    "db:seed": "prisma db seed"
  },
  "prisma": {
    "seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
  }
}
```

## 7. Performance Optimization

### 7.1 Frontend Optimization

#### 7.1.1 Code Splitting
- Use dynamic imports for large components
- Split routes for better initial load time
- Lazy load below-the-fold content

#### 7.1.2 Asset Optimization
- Use Next.js Image component for optimized images
- Minimize CSS with Tailwind's purge option
- Use font display swap for better text rendering

#### 7.1.3 Caching Strategy
- Cache API responses with React Query
- Implement stale-while-revalidate pattern
- Use appropriate cache headers for static assets

### 7.2 Backend Optimization

#### 7.2.1 Database Optimization
- Use appropriate indexes for frequent queries
- Implement connection pooling
- Use pagination for large result sets

#### 7.2.2 API Optimization
- Use Edge functions for global performance
- Implement request throttling for rate limiting
- Cache frequently accessed data

## 8. Scaling Considerations

### 8.1 Horizontal Scaling
- Stateless design enables easy replication
- Database read replicas for scaling reads
- Distribute API load across multiple regions

### 8.2 Vertical Scaling
- Optimize database queries for performance
- Monitor memory usage and optimize where needed
- Use appropriate instance sizes for workload

### 8.3 Feature Scaling
- Implement usage quotas per organization
- Add tiered access based on subscription level
- Design with future enterprise features in mind