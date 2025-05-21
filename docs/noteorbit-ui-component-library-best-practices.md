# NoteOrbit UI Component Library & Best Practices

## 1. Design Philosophy

NoteOrbit follows a minimal, functional design philosophy inspired by Clerk's UI with an emphasis on:

1. **Simplicity**: Clean interfaces with minimal visual noise
2. **Functionality**: Practical components that prioritize usability
3. **Consistency**: Uniform styling and behavior across the application
4. **Accessibility**: Compliance with WCAG 2.1 AA standards
5. **Performance**: Lightweight components that load quickly

## 2. Color System

### 2.1 Primary Colors

The primary color palette is based on Tailwind's indigo scale, which provides a modern, professional look:

| Name | Hex Code | Tailwind Class | Usage |
|------|----------|---------------|-------|
| Primary-50 | #EEF2FF | bg-indigo-50 | Subtle backgrounds, hover states |
| Primary-100 | #E0E7FF | bg-indigo-100 | Active state backgrounds |
| Primary-200 | #C7D2FE | bg-indigo-200 | Borders for primary elements |
| Primary-300 | #A5B4FC | bg-indigo-300 | Secondary buttons |
| Primary-400 | #818CF8 | bg-indigo-400 | Highlights |
| Primary-500 | #6366F1 | bg-indigo-500 | Focus states |
| Primary-600 | #4F46E5 | bg-indigo-600 | Primary buttons, active links |
| Primary-700 | #4338CA | bg-indigo-700 | Hover states for primary buttons |
| Primary-800 | #3730A3 | bg-indigo-800 | Heavy emphasis elements |
| Primary-900 | #312E81 | bg-indigo-900 | High contrast texts |
| Primary-950 | #1E1B4B | bg-indigo-950 | Very high contrast elements |

### 2.2 Neutral Colors

Neutral colors are used for most UI elements to maintain a clean, distraction-free interface:

| Name | Hex Code | Tailwind Class | Usage |
|------|----------|---------------|-------|
| White | #FFFFFF | bg-white | Backgrounds, cards |
| Gray-50 | #F9FAFB | bg-gray-50 | Alternate backgrounds |
| Gray-100 | #F3F4F6 | bg-gray-100 | Dividers, secondary backgrounds |
| Gray-200 | #E5E7EB | bg-gray-200 | Borders, dividers |
| Gray-300 | #D1D5DB | bg-gray-300 | Disabled elements |
| Gray-400 | #9CA3AF | bg-gray-400 | Placeholder text |
| Gray-500 | #6B7280 | bg-gray-500 | Secondary text |
| Gray-600 | #4B5563 | bg-gray-600 | Tertiary text |
| Gray-700 | #374151 | bg-gray-700 | Secondary headings |
| Gray-800 | #1F2937 | bg-gray-800 | Primary headings |
| Gray-900 | #111827 | bg-gray-900 | Primary text |

### 2.3 Semantic Colors

Colors that convey specific meanings:

| Name | Hex Code | Tailwind Class | Usage |
|------|----------|---------------|-------|
| Success-50 | #F0FDF4 | bg-green-50 | Success backgrounds |
| Success-500 | #22C55E | bg-green-500 | Success indicators |
| Success-700 | #15803D | bg-green-700 | Success buttons |
| Warning-50 | #FFFBEB | bg-amber-50 | Warning backgrounds |
| Warning-500 | #F59E0B | bg-amber-500 | Warning indicators |
| Warning-700 | #B45309 | bg-amber-700 | Warning actions |
| Error-50 | #FEF2F2 | bg-red-50 | Error backgrounds |
| Error-500 | #EF4444 | bg-red-500 | Error indicators |
| Error-700 | #B91C1C | bg-red-700 | Error actions |
| Info-50 | #EFF6FF | bg-blue-50 | Info backgrounds |
| Info-500 | #3B82F6 | bg-blue-500 | Info indicators |
| Info-700 | #1D4ED8 | bg-blue-700 | Info actions |

## 3. Typography

### 3.1 Font Family

NoteOrbit uses the Inter font family for its modern, legible appearance that works well across devices.

```css
font-family: 'Inter', ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
```

### 3.2 Font Sizes

| Name | Size | Line Height | Tailwind Class | Usage |
|------|------|------------|---------------|-------|
| xs | 0.75rem (12px) | 1rem (16px) | text-xs | Fine print, captions |
| sm | 0.875rem (14px) | 1.25rem (20px) | text-sm | Secondary text, form labels |
| base | 1rem (16px) | 1.5rem (24px) | text-base | Body text |
| lg | 1.125rem (18px) | 1.75rem (28px) | text-lg | Emphasis text, important content |
| xl | 1.25rem (20px) | 1.75rem (28px) | text-xl | Small headings |
| 2xl | 1.5rem (24px) | 2rem (32px) | text-2xl | Medium headings |
| 3xl | 1.875rem (30px) | 2.25rem (36px) | text-3xl | Large headings |
| 4xl | 2.25rem (36px) | 2.5rem (40px) | text-4xl | Primary headings |

### 3.3 Font Weights

| Name | Weight | Tailwind Class | Usage |
|------|--------|---------------|-------|
| Light | 300 | font-light | Decorative headings |
| Normal | 400 | font-normal | Body text |
| Medium | 500 | font-medium | Semi-emphasized text, buttons |
| Semibold | 600 | font-semibold | Emphasized text, small headings |
| Bold | 700 | font-bold | Headings, important emphasis |
| Extrabold | 800 | font-extrabold | Very strong emphasis |

### 3.4 Typography Rules

1. Use no more than 2-3 font sizes per page
2. Maintain consistent font weights for similar elements
3. Ensure sufficient contrast (4.5:1 minimum)
4. Keep line lengths between 50-75 characters for readability
5. Use appropriate line height (generally 1.5-1.7 for body text)

## 4. Spacing System

NoteOrbit uses Tailwind's spacing scale for consistency across the application:

| Name | Size | Tailwind Class | Usage |
|------|------|---------------|-------|
| px | 1px | p-px, m-px | Thin borders, small details |
| 0.5 | 0.125rem (2px) | p-0.5, m-0.5 | Very small spacing |
| 1 | 0.25rem (4px) | p-1, m-1 | Small spacing |
| 2 | 0.5rem (8px) | p-2, m-2 | Tight spacing |
| 3 | 0.75rem (12px) | p-3, m-3 | Close content |
| 4 | 1rem (16px) | p-4, m-4 | Standard spacing |
| 6 | 1.5rem (24px) | p-6, m-6 | Medium spacing |
| 8 | 2rem (32px) | p-8, m-8 | Generous spacing |
| 12 | 3rem (48px) | p-12, m-12 | Section spacing |
| 16 | 4rem (64px) | p-16, m-16 | Large section spacing |

### 4.1 Spacing Rules

1. Use consistent spacing for similar elements
2. Maintain visual hierarchy with appropriate spacing
3. Use larger spacing between sections than between related elements
4. Keep spacing proportional to the size of the elements

## 5. Core UI Components

### 5.1 Button

Buttons are one of the most frequently used components and follow a consistent design pattern.

#### 5.1.1 Button Variants

1. **Primary**: High-emphasis actions
   ```jsx
   <Button variant="primary">Create Note</Button>
   ```
   
2. **Secondary**: Medium-emphasis actions
   ```jsx
   <Button variant="secondary">Preview</Button>
   ```
   
3. **Outline**: Low-emphasis actions
   ```jsx
   <Button variant="outline">Cancel</Button>
   ```
   
4. **Ghost**: Very low-emphasis actions
   ```jsx
   <Button variant="ghost">View Details</Button>
   ```

5. **Link**: Navigation actions that look like text
   ```jsx
   <Button variant="link">Learn more</Button>
   ```

6. **Danger**: Destructive actions
   ```jsx
   <Button variant="danger">Delete</Button>
   ```

#### 5.1.2 Button Sizes

1. **Small**: For compact UIs
   ```jsx
   <Button size="sm">Small</Button>
   ```

2. **Medium** (default): For standard use
   ```jsx
   <Button size="md">Medium</Button>
   ```

3. **Large**: For emphasis
   ```jsx
   <Button size="lg">Large</Button>
   ```

4. **Icon**: For buttons with just an icon
   ```jsx
   <Button size="icon" aria-label="Add item">
     <PlusIcon className="h-5 w-5" />
   </Button>
   ```

#### 5.1.3 Button States

- **Default**: Normal state
- **Hover**: Visual feedback on hover
- **Active**: Visual feedback when clicked
- **Focus**: Visual feedback when focused via keyboard
- **Disabled**: Indicates the button is not interactive

```jsx
<Button disabled>Cannot Submit</Button>
```

#### 5.1.4 Button with Icons

```jsx
<Button leftIcon={<PlusIcon className="h-5 w-5" />}>
  Add Item
</Button>

<Button rightIcon={<ArrowRightIcon className="h-5 w-5" />}>
  Continue
</Button>
```

### 5.2 Input

Text inputs for collecting user data.

#### 5.2.1 Basic Input

```jsx
<Input
  label="Email"
  type="email"
  placeholder="Enter your email"
  required
/>
```

#### 5.2.2 Input with Error

```jsx
<Input
  label="Username"
  value={username}
  onChange={(e) => setUsername(e.target.value)}
  error="Username is already taken"
/>
```

#### 5.2.3 Input with Icons

```jsx
<Input
  label="Search"
  placeholder="Search notes..."
  leftIcon={<SearchIcon className="h-5 w-5" />}
  rightIcon={
    <button onClick={clearSearch}>
      <XIcon className="h-5 w-5" />
    </button>
  }
/>
```

#### 5.2.4 Disabled Input

```jsx
<Input
  label="Organization"
  value="Acme Corp"
  disabled
/>
```

### 5.3 Select

Dropdown selection component.

```jsx
<Select
  label="Access Level"
  value={accessLevel}
  onChange={(e) => setAccessLevel(e.target.value)}
>
  <option value="public">Public</option>
  <option value="shared">Shared</option>
  <option value="private">Private</option>
</Select>
```

### 5.4 Checkbox

```jsx
<Checkbox
  id="remember-me"
  checked={rememberMe}
  onChange={(e) => setRememberMe(e.target.checked)}
  label="Remember me"
/>
```

### 5.5 Radio

```jsx
<Radio
  id="edit-everyone"
  name="editPermission"
  value="everyone"
  checked={editPermission === 'everyone'}
  onChange={() => setEditPermission('everyone')}
  label="Everyone can edit"
/>
```

### 5.6 Card

Container for related information.

```jsx
<Card>
  <Card.Header>
    <Card.Title>Team Settings</Card.Title>
    <Card.Description>Manage your team preferences.</Card.Description>
  </Card.Header>
  
  <Card.Content>
    {/* Form fields or content */}
  </Card.Content>
  
  <Card.Footer>
    <Button variant="outline">Cancel</Button>
    <Button variant="primary">Save Changes</Button>
  </Card.Footer>
</Card>
```

### 5.7 Alert

Information notifications.

```jsx
<Alert
  variant="success"
  title="Note Created"
  onClose={() => setShowAlert(false)}
>
  Your note has been created successfully.
</Alert>
```

Variants: `info`, `success`, `warning`, `error`

### 5.8 Modal

Focused interaction dialogs.

```jsx
<Modal
  isOpen={isOpen}
  onClose={() => setIsOpen(false)}
  title="Delete Note"
  description="Are you sure you want to delete this note? This action cannot be undone."
>
  <div className="mt-6 flex justify-end space-x-3">
    <Button variant="outline" onClick={() => setIsOpen(false)}>
      Cancel
    </Button>
    <Button variant="danger" onClick={handleDelete}>
      Delete
    </Button>
  </div>
</Modal>
```

### 5.9 Badge

Visual indicators for status or categories.

```jsx
<Badge variant="purple">Admin</Badge>
<Badge variant="gray">Member</Badge>
<Badge variant="green">Active</Badge>
<Badge variant="yellow">Pending</Badge>
<Badge variant="red">Error</Badge>
```

### 5.10 Tooltip

Contextual information on hover.

```jsx
<Tooltip content="View more details about this note">
  <Button variant="ghost" size="icon" aria-label="Information">
    <InformationCircleIcon className="h-5 w-5" />
  </Button>
</Tooltip>
```

## 6. Layout Components

### 6.1 Container

```jsx
<Container size="md">
  <h1>Page Content</h1>
  {/* Other content */}
</Container>
```

Sizes: `sm` (640px), `md` (768px), `lg` (1024px), `xl` (1280px), `2xl` (1536px)

### 6.2 Grid

```jsx
<Grid cols={3} gap={6}>
  <Card>Item 1</Card>
  <Card>Item 2</Card>
  <Card>Item 3</Card>
</Grid>
```

### 6.3 Flex

```jsx
<Flex direction="row" justify="between" align="center">
  <div>Left Content</div>
  <div>Right Content</div>
</Flex>
```

### 6.4 Sidebar

```jsx
<Sidebar>
  <Sidebar.Header>
    <Logo />
  </Sidebar.Header>
  <Sidebar.Content>
    <Sidebar.NavLink href="/dashboard" icon={HomeIcon}>
      Dashboard
    </Sidebar.NavLink>
    <Sidebar.NavLink href="/dashboard/notes" icon={DocumentTextIcon}>
      Notes
    </Sidebar.NavLink>
    <Sidebar.NavLink href="/org" icon={UserGroupIcon}>
      Organization
    </Sidebar.NavLink>
  </Sidebar.Content>
  <Sidebar.Footer>
    <UserMenu />
  </Sidebar.Footer>
</Sidebar>
```

### 6.5 Header

```jsx
<Header>
  <Header.Left>
    <Logo />
  </Header.Left>
  <Header.Center>
    <SearchBar />
  </Header.Center>
  <Header.Right>
    <OrganizationSwitcher />
    <UserButton />
  </Header.Right>
</Header>
```

## 7. Data Display Components

### 7.1 Table

```jsx
<Table>
  <Table.Header>
    <Table.Row>
      <Table.Head>Member</Table.Head>
      <Table.Head>Role</Table.Head>
      <Table.Head>Status</Table.Head>
      <Table.Head>Actions</Table.Head>
    </Table.Row>
  </Table.Header>
  <Table.Body>
    {members.map((member) => (
      <Table.Row key={member.id}>
        <Table.Cell>
          <UserInfo user={member} />
        </Table.Cell>
        <Table.Cell>
          <RoleBadge role={member.role} />
        </Table.Cell>
        <Table.Cell>
          <StatusBadge status={member.status} />
        </Table.Cell>
        <Table.Cell>
          <TableActions member={member} />
        </Table.Cell>
      </Table.Row>
    ))}
  </Table.Body>
</Table>
```

### 7.2 Pagination

```jsx
<Pagination
  currentPage={currentPage}
  totalPages={totalPages}
  onPageChange={setCurrentPage}
/>
```

### 7.3 Empty State

```jsx
<EmptyState
  title="No notes found"
  description="Create your first note to get started."
  icon={DocumentTextIcon}
  action={{
    label: "Create Note",
    href: "/dashboard/notes/new"
  }}
/>
```

### 7.4 Skeleton Loader

```jsx
<Skeleton className="h-12 w-full rounded-md" />
<div className="mt-4 space-y-2">
  <Skeleton className="h-4 w-full" />
  <Skeleton className="h-4 w-3/4" />
  <Skeleton className="h-4 w-5/6" />
</div>
```

## 8. Form Patterns

### 8.1 Basic Form Layout

```jsx
<form onSubmit={handleSubmit} className="space-y-6">
  <Input
    label="Title"
    value={title}
    onChange={(e) => setTitle(e.target.value)}
    required
  />
  
  <Textarea
    label="Description"
    value={description}
    onChange={(e) => setDescription(e.target.value)}
    rows={4}
  />
  
  <Select
    label="Category"
    value={category}
    onChange={(e) => setCategory(e.target.value)}
  >
    <option value="">Select a category</option>
    <option value="work">Work</option>
    <option value="personal">Personal</option>
    <option value="project">Project</option>
  </Select>
  
  <div className="flex justify-end space-x-3">
    <Button variant="outline" type="button" onClick={onCancel}>
      Cancel
    </Button>
    <Button variant="primary" type="submit">
      Submit
    </Button>
  </div>
</form>
```

### 8.2 Form Sections

```jsx
<form onSubmit={handleSubmit} className="space-y-8">
  <FormSection
    title="Basic Information"
    description="Enter the basic details for this note."
  >
    <Input label="Title" value={title} onChange={(e) => setTitle(e.target.value)} required />
    <Textarea label="Content" value={content} onChange={(e) => setContent(e.target.value)} rows={6} />
  </FormSection>
  
  <FormSection
    title="Access Control"
    description="Control who can view and edit this note."
  >
    <Select
      label="Access Level"
      value={accessLevel}
      onChange={(e) => setAccessLevel(e.target.value)}
    >
      <option value="public">Public (All organization members)</option>
      <option value="shared">Shared (Specific members only)</option>
      <option value="private">Private (Only you)</option>
    </Select>
    
    {accessLevel === 'shared' && (
      <MemberSelector
        label="Who can view this note?"
        selectedMembers={viewMembers}
        onChange={setViewMembers}
      />
    )}
  </FormSection>
  
  <div className="flex justify-end space-x-3">
    <Button variant="outline" type="button" onClick={onCancel}>
      Cancel
    </Button>
    <Button variant="primary" type="submit">
      Save Note
    </Button>
  </div>
</form>
```

### 8.3 Form Validation

```jsx
function validateForm() {
  const errors = {};
  
  if (!title.trim()) {
    errors.title = 'Title is required';
  }
  
  if (accessLevel === 'shared' && viewMembers.length === 0) {
    errors.viewMembers = 'Select at least one member';
  }
  
  setFormErrors(errors);
  return Object.keys(errors).length === 0;
}

function handleSubmit(e) {
  e.preventDefault();
  
  if (!validateForm()) {
    return;
  }
  
  // Proceed with form submission
}
```

### 8.4 Inline Validation

```jsx
function validateTitle(value) {
  if (!value.trim()) {
    return 'Title is required';
  }
  if (value.length > 100) {
    return 'Title must be less than 100 characters';
  }
  return null;
}

// In the component
const [title, setTitle] = useState('');
const [titleError, setTitleError] = useState(null);

function handleTitleChange(e) {
  const value = e.target.value;
  setTitle(value);
  setTitleError(validateTitle(value));
}

// In the JSX
<Input
  label="Title"
  value={title}
  onChange={handleTitleChange}
  onBlur={() => setTitleError(validateTitle(title))}
  error={titleError}
  required
/>
```

## 9. UI Patterns

### 9.1 Loading States

```jsx
<Button variant="primary" disabled={isLoading}>
  {isLoading ? (
    <>
      <Spinner className="mr-2 h-4 w-4" />
      Loading...
    </>
  ) : (
    'Submit'
  )}
</Button>
```

### 9.2 Error Handling

```jsx
{error && (
  <Alert variant="error" title="An error occurred" onClose={() => setError(null)}>
    {error}
  </Alert>
)}

{formErrors.title && (
  <p className="mt-1 text-sm text-red-600">{formErrors.title}</p>
)}
```

### 9.3 Success Feedback

```jsx
{success && (
  <Alert variant="success" title="Success" onClose={() => setSuccess(null)}>
    {success}
  </Alert>
)}
```

### 9.4 Confirmation Dialogs

```jsx
function handleDelete() {
  setIsConfirmOpen(true);
}

<ConfirmationDialog
  isOpen={isConfirmOpen}
  onClose={() => setIsConfirmOpen(false)}
  title="Delete Note"
  description="Are you sure you want to delete this note? This action cannot be undone."
  confirmLabel="Delete"
  confirmVariant="danger"
  onConfirm={confirmDelete}
/>
```

### 9.5 Toast Notifications

```jsx
function showSuccessToast(message) {
  toast({
    title: 'Success',
    description: message,
    variant: 'success',
    duration: 3000,
  });
}

function handleSubmit(e) {
  e.preventDefault();
  
  try {
    // Submit form
    showSuccessToast('Note created successfully');
  } catch (error) {
    toast({
      title: 'Error',
      description: error.message,
      variant: 'error',
      duration: 5000,
    });
  }
}
```

## 10. Responsive Design

### 10.1 Viewport Breakpoints

| Breakpoint | Size | Tailwind Class | Description |
|------------|------|---------------|-------------|
| xs | < 640px | (default) | Mobile phones |
| sm | ≥ 640px | sm: | Large phones, small tablets |
| md | ≥ 768px | md: | Tablets, small laptops |
| lg | ≥ 1024px | lg: | Laptops, desktops |
| xl | ≥ 1280px | xl: | Large desktops |
| 2xl | ≥ 1536px | 2xl: | Extra large screens |

### 10.2 Responsive Layout Patterns

**Responsive Container:**
```jsx
<div className="container mx-auto px-4 sm:px-6 lg:px-8">
  {/* Content */}
</div>
```

**Responsive Grid:**
```jsx
<div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
  {/* Grid items */}
</div>
```

**Responsive Flex Layout:**
```jsx
<div className="flex flex-col space-y-4 sm:flex-row sm:space-y-0 sm:space-x-4">
  <div className="w-full sm:w-1/3">Sidebar</div>
  <div className="w-full sm:w-2/3">Main Content</div>
</div>
```

**Responsive Typography:**
```jsx
<h1 className="text-2xl sm:text-3xl md:text-4xl font-bold">
  Page Title
</h1>
```

**Responsive Visibility:**
```jsx
<div className="hidden md:block">
  {/* Only visible on medium screens and up */}
</div>
<div className="block md:hidden">
  {/* Only visible on small screens */}
</div>
```

### 10.3 Responsive UI Components

**Responsive Button:**
```jsx
<Button
  size={{
    base: "sm",
    md: "md",
    lg: "lg"
  }}
  className="w-full sm:w-auto"
>
  Save Changes
</Button>
```

**Responsive Input:**
```jsx
<div className="flex flex-col sm:flex-row sm:items-center sm:space-x-4">
  <Input
    label="Search"
    className="w-full sm:w-64"
    leftIcon={<SearchIcon className="h-5 w-5" />}
  />
  <Button className="mt-2 sm:mt-0">Search</Button>
</div>
```

**Responsive Card:**
```jsx
<Card className="p-4 sm:p-6">
  <div className="flex flex-col sm:flex-row sm:items-center sm:justify-between">
    <h3 className="text-lg font-medium">Card Title</h3>
    <Button size="sm" className="mt-4 sm:mt-0">Action</Button>
  </div>
  <div className="mt-4">{/* Content */}</div>
</Card>
```

**Responsive Table:**
```jsx
{/* Card view for small screens */}
<div className="grid grid-cols-1 gap-4 sm:hidden">
  {data.map((item) => (
    <Card key={item.id}>
      <div className="space-y-2">
        <div className="flex justify-between">
          <span className="font-medium">Name</span>
          <span>{item.name}</span>
        </div>
        <div className="flex justify-between">
          <span className="font-medium">Role</span>
          <span>{item.role}</span>
        </div>
        <div className="flex justify-between">
          <span className="font-medium">Status</span>
          <span>{item.status}</span>
        </div>
      </div>
    </Card>
  ))}
</div>

{/* Table view for larger screens */}
<div className="hidden sm:block">
  <Table>{/* ... */}</Table>
</div>
```

## 11. Accessibility Guidelines

### 11.1 Core Principles

1. **Perceivable**: Information must be presentable in ways all users can perceive
2. **Operable**: UI components must be operable by all users
3. **Understandable**: Content and operation must be understandable
4. **Robust**: Content must be robust enough to work with various user agents

### 11.2 Implementation Guidelines

#### 11.2.1 Semantic HTML

Use proper HTML elements for their intended purpose.

```jsx
// Good
<button onClick={handleClick}>Click Me</button>

// Avoid
<div onClick={handleClick} className="button">Click Me</div>
```

#### 11.2.2 Keyboard Navigation

Ensure all interactive elements are keyboard accessible.

```jsx
// Add proper focus styles
<button className="px-4 py-2 bg-indigo-600 text-white rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">
  Submit
</button>
```

#### 11.2.3 Screen Reader Support

Provide text alternatives for non-text content.

```jsx
<img src="/logo.png" alt="NoteOrbit Logo" />

<button
  aria-label="Close dialog"
  onClick={closeDialog}
>
  <XIcon className="h-5 w-5" />
</button>
```

#### 11.2.4 Form Accessibility

Associate labels with form controls and provide clear error messages.

```jsx
<div>
  <label htmlFor="email-input" className="block text-sm font-medium text-gray-700">
    Email
  </label>
  <input
    id="email-input"
    type="email"
    aria-describedby="email-error"
    className={`mt-1 block w-full ${errors.email ? 'border-red-300' : 'border-gray-300'}`}
  />
  {errors.email && (
    <p id="email-error" className="mt-1 text-sm text-red-600">
      {errors.email}
    </p>
  )}
</div>
```

#### 11.2.5 ARIA Attributes

Use ARIA attributes appropriately to enhance accessibility.

```jsx
<div
  role="alert"
  aria-live="assertive"
  className="rounded-md bg-red-50 p-4"
>
  <p className="text-sm text-red-700">
    Please correct the errors in the form.
  </p>
</div>
```

#### 11.2.6 Color and Contrast

Ensure sufficient color contrast and don't rely solely on color to convey information.

```jsx
{/* Good - Uses both color and icon */}
<div className="flex items-center text-red-600">
  <ExclamationIcon className="mr-1 h-5 w-5" />
  <span>Error message</span>
</div>
```

## 12. Best Practices

### 12.1 Component Design

1. **Single Responsibility**: Each component should do one thing well
2. **Composition**: Build complex components by composing simpler ones
3. **Reusability**: Design components for reuse across the application
4. **Progressive Enhancement**: Start with basic functionality and enhance as needed
5. **Accessibility First**: Design with accessibility in mind from the start

### 12.2 Naming Conventions

1. **Descriptive Names**: Use clear, descriptive names for components and props
2. **Consistent Casing**: Use PascalCase for components, camelCase for props and variables
3. **Prefix Booleans with 'is', 'has', or 'should'**: `isOpen`, `hasError`, `shouldAutoFocus`
4. **Consistent Terminology**: Use the same terms throughout the UI
5. **Avoid Abbreviations**: Write out full words for clarity

### 12.3 Performance Tips

1. **Memoization**: Use `useMemo` and `useCallback` for expensive calculations
2. **Code Splitting**: Split code into smaller chunks for faster loading
3. **Lazy Loading**: Load components only when needed
4. **Efficient Rendering**: Minimize unnecessary re-renders
5. **Image Optimization**: Use optimized images and webp format

### 12.4 Error Handling

1. **Graceful Degradation**: Show fallback UI when errors occur
2. **Clear Error Messages**: Provide helpful error messages to users
3. **Error Boundaries**: Use React error boundaries to catch and handle errors
4. **Form Validation**: Validate user input and provide clear feedback
5. **Network Error Handling**: Handle API errors gracefully

## 13. Documentation Template

### 13.1 Component Documentation

```markdown
# Button Component

A versatile button component with multiple variants and sizes.

## Usage

```jsx
import { Button } from '@/components/ui/button';

<Button variant="primary" size="md">
  Click Me
</Button>
```

## Props

| Name | Type | Default | Description |
|------|------|---------|-------------|
| variant | 'primary' \| 'secondary' \| 'outline' \| 'ghost' \| 'link' \| 'danger' | 'primary' | The style variant of the button |
| size | 'sm' \| 'md' \| 'lg' \| 'icon' | 'md' | The size of the button |
| leftIcon | ReactNode | undefined | Icon to display before the button text |
| rightIcon | ReactNode | undefined | Icon to display after the button text |
| href | string | undefined | If provided, the button renders as an anchor tag |
| external | boolean | false | If true and href is provided, opens in new tab |

## Examples

### Button Variants

```jsx
<Button variant="primary">Primary</Button>
<Button variant="secondary">Secondary</Button>
<Button variant="outline">Outline</Button>
<Button variant="ghost">Ghost</Button>
<Button variant="link">Link</Button>
<Button variant="danger">Danger</Button>
```

### Button Sizes

```jsx
<Button size="sm">Small</Button>
<Button size="md">Medium</Button>
<Button size="lg">Large</Button>
<Button size="icon">
  <PlusIcon className="h-5 w-5" />
</Button>
```

### Button with Icons

```jsx
<Button leftIcon={<PlusIcon className="h-5 w-5" />}>
  Add Item
</Button>

<Button rightIcon={<ArrowRightIcon className="h-5 w-5" />}>
  Continue
</Button>
```

### Button as Link

```jsx
<Button href="/dashboard" variant="primary">
  Go to Dashboard
</Button>

<Button href="https://example.com" external variant="link">
  External Link
</Button>
```
```

## 14. Design-to-Code Workflow

### 14.1 Design Handoff

1. **Design System Reference**: Maintain a living design system document
2. **Component Specifications**: Document sizes, spacing, colors, and behavior
3. **Design Tokens**: Export design tokens for consistent implementation
4. **Responsive Behaviors**: Document how components respond to different screen sizes
5. **Interaction States**: Document all states (default, hover, active, focus, disabled)

### 14.2 Implementation Process

1. **Review Design**: Understand the design and its responsive behavior
2. **Identify Components**: Break down the design into reusable components
3. **Implement Structure**: Build the HTML structure with semantic elements
4. **Style with Tailwind**: Apply Tailwind classes for styling
5. **Add Interactions**: Implement interactive behaviors
6. **Test Accessibility**: Ensure accessibility for all users
7. **Review and Refine**: Compare with design and refine as needed

### 14.3 Design-to-Code Checklist

- [ ] Component matches design visually
- [ ] All interaction states implemented
- [ ] Responsive behavior matches design specs
- [ ] Accessibility requirements met
- [ ] Code is clean and maintainable
- [ ] Performance optimizations applied
- [ ] Documentation provided