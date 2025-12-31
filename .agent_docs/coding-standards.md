# Coding Standards

Consolidated coding conventions and best practices for the Twenty CRM codebase.

---

## Core Principles

- **Functional components only** (no class components)
- **Named exports only** (no default exports)
- **Types over interfaces** (except when extending third-party interfaces)
- **String literals over enums** (except for GraphQL enums)
- **No 'any' type allowed**
- **Event handlers over useEffect** for state updates

---

## TypeScript Guidelines

### Strict Typing

TypeScript strict mode is enabled. No 'any' type allowed.

```typescript
// Good
function processUser(user: User) {
  return user.name;
}

// Bad
function processUser(user: any) {
  return user.name;
}
```

### Type Definitions

Use `type` for all type definitions. Exception: extending third-party interfaces.

```typescript
// Good
type User = {
  id: string;
  name: string;
  email: string;
};

// Bad
interface User {
  id: string;
  name: string;
  email: string;
}
```

### String Literals over Enums

Use string literal unions instead of enums. Exception: GraphQL enums.

```typescript
// Good
type UserRole = 'admin' | 'user' | 'guest';

// Bad
enum UserRole {
  Admin = 'admin',
  User = 'user',
  Guest = 'guest',
}
```

### Component Props

Suffix component prop types with 'Props'.

```typescript
type ButtonProps = {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
};
```

### Type Inference

Use inference when types are clear; explicitly type when ambiguous.

```typescript
// Clear inference - no annotation needed
const users = ['John', 'Jane'];

// Explicit typing needed
const processUser = (user: User): UserResponse => {
  return response;
};
```

### Generics

Use descriptive generic names.

```typescript
// Good
type ApiResponse<TData> = {
  data: TData;
  status: number;
};

// Bad
type ApiResponse<T> = {
  data: T;
  status: number;
};
```

---

## React Guidelines

### Component Structure

```typescript
export const UserProfile = ({ user, onEdit }: UserProfileProps) => {
  const handleEdit = () => onEdit(user.id);

  return (
    <StyledContainer>
      <h1>{user.name}</h1>
      <Button onClick={handleEdit}>Edit</Button>
    </StyledContainer>
  );
};
```

### Props and Event Handlers

Destructure props. Use event handlers instead of useEffect for state updates.

```typescript
// Good - Event handlers
const UserForm = ({ onSubmit }: UserFormProps) => {
  const handleSubmit = async (data: FormData) => {
    await onSubmit(data);
  };

  return <Form onSubmit={handleSubmit} />;
};
```

### Component Design

- **Small, focused components** - Single responsibility
- **Composition over inheritance** - Combine simple components
- **Extract complex logic** into custom hooks

```typescript
// Composed from smaller components
const UserCard = ({ user }: UserCardProps) => (
  <StyledCard>
    <UserAvatar user={user} />
    <UserInfo user={user} />
    <UserActions user={user} />
  </StyledCard>
);
```

### Performance

```typescript
// Use memo for expensive components only
const ExpensiveChart = memo(({ data }: ChartProps) => {
  return <ComplexChart data={data} />;
});

// Memoize callbacks when needed
const handleUserSelect = useCallback((user: User) => {
  onUserSelect(user);
}, [onUserSelect]);
```

---

## State Management (Recoil)

### Atoms

```typescript
// Atoms for primitive state
export const currentUserState = atom<User | null>({
  key: 'currentUserState',
  default: null,
});
```

### Selectors

```typescript
// Selectors for derived state
export const userDisplayNameSelector = selector({
  key: 'userDisplayNameSelector',
  get: ({ get }) => {
    const user = get(currentUserState);
    return user ? `${user.firstName} ${user.lastName}` : 'Guest';
  },
});
```

### Atom Families

```typescript
// Atom families for dynamic atoms
export const userByIdState = atomFamily<User | null, string>({
  key: 'userByIdState',
  default: null,
});
```

### Local State

```typescript
// Multiple useState for unrelated state
const [isLoading, setIsLoading] = useState(false);
const [error, setError] = useState<string | null>(null);
const [data, setData] = useState<User[]>([]);

// useReducer for complex state logic
type FormAction =
  | { type: 'SET_FIELD'; field: string; value: string }
  | { type: 'RESET' };

const formReducer = (state: FormState, action: FormAction): FormState => {
  switch (action.type) {
    case 'SET_FIELD':
      return { ...state, [action.field]: action.value };
    case 'RESET':
      return initialFormState;
    default:
      return state;
  }
};
```

### Data Flow Rules

- **Props down, events up** - Unidirectional data flow
- **Avoid bidirectional binding** - Use callback functions
- **Normalize complex data** - Use lookup tables over nested objects

---

## Code Style

### Formatting

- **Prettier**: 2-space indentation, single quotes, trailing commas, semicolons
- **Print width**: 80 characters
- **ESLint**: No unused imports, consistent import ordering, prefer const

### Naming Conventions

```typescript
// Variables and functions - camelCase
const userAccountBalance = 1000;
const calculateMonthlyPayment = () => {};

// Constants - SCREAMING_SNAKE_CASE
const API_ENDPOINTS = {
  USERS: '/api/users',
  ORDERS: '/api/orders',
} as const;

// Types and Classes - PascalCase
class UserService {}
type UserAccountData = {};
type ButtonProps = {};

// Files and directories - kebab-case
// user-profile.component.tsx
// user-profile.styles.ts

// NEVER use abbreviations in callbacks
// Bad
const users = data.map((u) => u.name);

// Good
const users = data.map((user) => user.name);
```

### Import Organization

```typescript
// 1. External libraries
import React from 'react';
import styled from 'styled-components';

// 2. Internal modules (absolute paths)
import { Button } from '@/components/ui';
import { UserService } from '@/services';

// 3. Relative imports
import { UserCardProps } from './types';
```

### Comments

Use short `//` comments, NOT JSDoc blocks. Explain WHY, not WHAT.

```typescript
// Good - Explain non-obvious business logic
// Apply 15% discount for premium users with orders > $100
const discount = isPremiumUser && orderTotal > 100 ? 0.15 : 0;

// Bad - Obvious comment
// Get all inline fields dynamically
const { inlineFieldMetadataItems } = useFieldListFieldMetadataItems({...});
```

**Guidelines:**
- DO comment complex business rules or domain-specific logic
- DO add TODOs for future improvements
- DON'T comment obvious variable declarations
- DON'T add comments that repeat what code already says

### Utility Helpers

Use existing utilities instead of manual checks.

```typescript
import { isDefined } from 'twenty-shared/utils';
import { isNonEmptyString, isNonEmptyArray } from '@sniptt/guards';

// Good
const validItems = items.filter(isDefined);
const hasValue = isDefined(value);

// Bad
const validItems = items.filter((item): item is Item => item !== undefined);
```

### Security Patterns

```typescript
// CSV Export: Apply security first, then formatting
const safeValue = formatValueForCSV(sanitizeValueForCSVExport(userInput));

// Input validation before processing
const sanitizedInput = validateAndSanitize(userInput);
const result = processData(sanitizedInput);
```

### Error Handling

```typescript
try {
  const user = await userService.findById(userId);
  if (!user) {
    throw new UserNotFoundError(`User with ID ${userId} not found`);
  }
  return user;
} catch (error) {
  logger.error('Failed to fetch user', { userId, error });
  throw error;
}
```

---

## File Structure

### Directory Organization

```
packages/twenty-front/src/
├── components/      # Reusable UI components
├── pages/          # Route components
├── modules/        # Feature modules
├── hooks/          # Custom hooks
├── services/       # API services
└── types/          # Type definitions

packages/twenty-server/src/
├── modules/        # Feature modules
├── entities/       # Database entities
├── dto/           # Data transfer objects
└── utils/         # Helper functions
```

### File Naming

Use **kebab-case** for all files and directories with descriptive suffixes.

```
user-profile.component.tsx
user-profile.styles.ts
user-profile.test.tsx
user.service.ts
user.entity.ts
create-user.dto.ts
```

### Index Files and Barrel Exports

```typescript
// index.ts
export { UserCard } from './user-card.component';
export { UserList } from './user-list.component';
export type { UserCardProps, UserListProps } from './types';

// Usage
import { UserCard, UserList } from '@/components/user';
```

### File Size Guidelines

- **Components**: Under 300 lines
- **Services**: Under 500 lines
- Extract logic into hooks/utilities when files grow large

---

## Testing Guidelines

### Test Structure (AAA Pattern)

```typescript
describe('UserService', () => {
  describe('when getting user by ID', () => {
    it('should return user data for valid ID', async () => {
      // Arrange
      const userId = '123';
      const expectedUser = { id: '123', name: 'John' };
      mockUserRepository.findById.mockResolvedValue(expectedUser);

      // Act
      const result = await userService.getUserById(userId);

      // Assert
      expect(result).toEqual(expectedUser);
    });
  });
});
```

### Testing Principles

- **Test behavior, not implementation** - Focus on what users see/do
- **Use descriptive test names** - "should [behavior] when [condition]"
- **Query by user-visible elements** - text, roles, labels over test IDs
- **Keep tests isolated** - Independent and repeatable
- **70% unit, 20% integration, 10% E2E** - Test pyramid

### Running Tests

**Single file execution (PREFERRED - fast)**:

```bash
# Frontend tests (use .test.ts extension)
npx jest packages/twenty-front/src/modules/localization/utils/detection/detectNumberFormat.test.ts --config=packages/twenty-front/jest.config.mjs

# Server tests (use .spec.ts extension)
npx jest packages/twenty-server/src/utils/__test__/is-work-email.spec.ts --config=packages/twenty-server/jest.config.mjs

# Watch mode
npx jest path/to/test.test.ts --config=packages/twenty-front/jest.config.mjs --watch
```

**AVOID running all tests** (slow):
```bash
# This runs ALL tests - use sparingly
npx nx test twenty-front
```

### Mocking Patterns

```typescript
const mockEmailService = {
  sendEmail: jest.fn().mockResolvedValue({ success: true }),
  validateEmail: jest.fn().mockReturnValue(true),
};

// Test data factories
const createTestUser = (overrides = {}) => ({
  id: uuid(),
  email: 'test@example.com',
  name: 'Test User',
  ...overrides,
});

beforeEach(() => {
  jest.clearAllMocks();
});
```

---

## Server Development

### TypeORM Migrations

When modifying `*.entity.ts` files, always generate a migration:

```bash
npx nx run twenty-server:typeorm migration:generate src/database/typeorm/core/migrations/common/[name] -d src/database/typeorm/core/core.datasource.ts
```

- Replace `[name]` with a descriptive kebab-case name (e.g., `add-user-role-field`)
- Prefer generated migrations over manual edits
- Ensure both `up` and `down` logic exists for reversibility

---

## Nx Monorepo Commands

### Core Commands

```bash
# Run target for specific project
npx nx run twenty-front:build
npx nx run twenty-server:test

# Lint changed files only (RECOMMENDED - much faster)
npx nx lint:changed twenty-front
npx nx lint:changed twenty-server
npx nx lint:changed twenty-front --configuration=fix  # Auto-fix

# Run target for all projects (slower)
npx nx run-many --target=build --all
```

### Linting Strategy

Always prefer linting only changed files:
- `npx nx lint:changed <project>` - lint files changed vs main branch
- `--configuration=fix` - auto-fix issues
- `--configuration=staged` - lint staged files only

### Dependency Graph

```bash
npx nx graph                           # View project dependencies
npx nx affected --target=test          # Run on affected projects
```

---

## Internationalization (i18n)

### File Structure

```
src/locales/
├── en/                        # English (primary)
│   ├── common.json
│   ├── auth.json
│   └── forms.json
├── fr/                        # French
│   └── ...
└── index.ts                   # i18n configuration
```

### Implementation

```typescript
import { useTranslation } from 'react-i18next';

const LoginForm = () => {
  const { t } = useTranslation('auth');

  return (
    <form>
      <h1>{t('login.title')}</h1>
      <input placeholder={t('login.email')} type="email" />
    </form>
  );
};
```

### Translation Keys

Use nested objects with hierarchical, descriptive keys:

```json
{
  "auth": {
    "login": {
      "title": "Sign In",
      "email": "Email Address",
      "submit": "Sign In"
    }
  }
}
```

### Interpolation and Pluralization

```typescript
// Interpolation
{t('welcome.message', { name: userName })}
// Translation: "Welcome back, {{name}}!"

// Pluralization
{t('items.count', { count })}
// Translation: "count_one": "{{count}} item", "count_other": "{{count}} items"
```
