# React Copilot Instructions

These instructions define React-specific standards and best practices for React applications.

## Component Architecture

### Component Structure
- Prefer function components over class components.
- Keep components focused on a single responsibility.
- Extract complex logic into custom hooks.
- Co-locate component files with their styles and tests when using component folders.

**Example - Component Organization:**
```
src/
  components/
    UserProfile/
      UserProfile.tsx
      UserProfile.module.css
      UserProfile.test.tsx
      index.ts
    Button/
      Button.tsx
      Button.module.css
      Button.test.tsx
      index.ts
```

### Props Design
- Define explicit prop interfaces for all components.
- Avoid prop drilling beyond 2-3 levels; use composition or context instead.
- Use discriminated unions for variant props.
- Prefer object props over many individual props when passing related data.

**Example - Discriminated Union Props:**
```tsx
// ❌ Bad: Boolean flags for variants
interface ButtonProps {
  isPrimary?: boolean;
  isSecondary?: boolean;
  isDanger?: boolean;
}

// ✅ Good: Discriminated union
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  children: React.ReactNode;
}
```

**Example - Avoiding Prop Drilling:**
```tsx
// ❌ Bad: Prop drilling through multiple levels
function App() {
  const user = useUser();
  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {
  return <UserMenu user={user} />;
}

// ✅ Good: Use context for deep data
const UserContext = createContext<User | null>(null);

function App() {
  const user = useUser();
  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
}

function UserMenu() {
  const user = useContext(UserContext);
  return <div>{user?.name}</div>;
}
```

## Hooks Best Practices

### Custom Hooks
- Prefix all custom hooks with `use`.
- Extract reusable stateful logic into custom hooks.
- Keep hooks focused on a single concern.
- Return objects for multiple values instead of arrays when order isn't obvious.

**Example - Custom Hook Design:**
```tsx
// ❌ Bad: Hook does too much
function useUserData() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [comments, setComments] = useState([]);
  // ... complex logic for all three
}

// ✅ Good: Focused hooks
function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [userId]);
  
  return { user, loading, error };
}

function useUserPosts(userId: string) {
  // Focused on posts only
}
```

### Hook Dependencies
- Always include all dependencies in useEffect/useCallback/useMemo arrays.
- Use ESLint `exhaustive-deps` rule and fix warnings, don't disable.
- Prefer useCallback for functions passed as props to memoized children.
- Use useMemo sparingly; only for expensive computations.

**Example - Proper Dependencies:**
```tsx
// ❌ Bad: Missing dependencies
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    searchApi(query).then(setResults);
  }, []); // Missing 'query' dependency
}

// ✅ Good: Complete dependencies
function SearchResults({ query }: { query: string }) {
  const [results, setResults] = useState<Result[]>([]);
  
  useEffect(() => {
    let cancelled = false;
    
    searchApi(query).then(data => {
      if (!cancelled) setResults(data);
    });
    
    return () => { cancelled = true; };
  }, [query]);
  
  return <ResultList results={results} />;
}
```

## State Management

### Local vs Global State
- Prefer local component state when data is only needed in one component tree.
- Use Context API for app-wide concerns (theme, auth, i18n).
- Consider external state management (Zustand, Jotai, Redux) for complex shared state.
- Keep server state separate from client state using data fetching libraries.

### State Updates
- Use functional updates when new state depends on previous state.
- Batch related state updates in the same component.
- Avoid storing derived values in state; compute them during render.

**Example - Functional Updates:**
```tsx
// ❌ Bad: Direct state reference
function Counter({ step }) {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    setCount(count + step); // Stale closure issue
  };
}

// ✅ Good: Functional update
function Counter({ step }: { step: number }) {
  const [count, setCount] = useState(0);
  
  const increment = useCallback(() => {
    setCount(prev => prev + step);
  }, [step]);
  
  return <button onClick={increment}>{count}</button>;
}
```

**Example - Avoid Derived State:**
```tsx
// ❌ Bad: Storing derived value
function ProductList({ products, searchTerm }) {
  const [filteredProducts, setFilteredProducts] = useState([]);
  
  useEffect(() => {
    setFilteredProducts(
      products.filter(p => p.name.includes(searchTerm))
    );
  }, [products, searchTerm]);
}

// ✅ Good: Compute during render
function ProductList({ 
  products, 
  searchTerm 
}: { 
  products: Product[]; 
  searchTerm: string;
}) {
  const filteredProducts = products.filter(p => 
    p.name.toLowerCase().includes(searchTerm.toLowerCase())
  );
  
  return <div>{filteredProducts.map(...)}</div>;
}
```

## Performance Optimization

### Memoization
- Use React.memo for expensive components that render often with same props.
- Don't memoize everything; measure first.
- Ensure memo comparisons are shallow or provide custom comparison.
- Use useMemo for expensive calculations, not cheap ones.

**Example - Effective Memoization:**
```tsx
// ❌ Bad: Unnecessary memo
const SimpleButton = React.memo(({ label }: { label: string }) => (
  <button>{label}</button>
));

// ✅ Good: Memo for expensive render
const ExpensiveChart = React.memo(({ 
  data 
}: { 
  data: number[] 
}) => {
  // Complex D3 visualization
  return <svg>...</svg>;
}, (prev, next) => {
  // Custom comparison for deep equality if needed
  return areArraysEqual(prev.data, next.data);
});
```

### Code Splitting
- Use React.lazy for route-based code splitting.
- Wrap lazy components with Suspense boundaries.
- Consider splitting large feature modules.
- Preload critical routes when likely to be needed.

**Example - Code Splitting:**
```tsx
// ✅ Good: Route-based splitting
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));
const Profile = lazy(() => import('./pages/Profile'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </Suspense>
  );
}
```

## Event Handling

### Event Handlers
- Use TypeScript event types (React.MouseEvent, React.ChangeEvent, etc.).
- Define handlers inside components or use useCallback for handlers passed as props.
- Avoid inline arrow functions in JSX for handlers when performance matters.
- Prevent default and stop propagation explicitly when needed.

**Example - Typed Event Handlers:**
```tsx
// ❌ Bad: Untyped events
function LoginForm() {
  const handleSubmit = (e) => {
    e.preventDefault();
    // ...
  };
}

// ✅ Good: Properly typed events
function LoginForm() {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    // ...
  };
  
  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    // ...
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="email" onChange={handleInputChange} />
    </form>
  );
}
```

## Data Fetching

### Server State Management
- Use data fetching libraries (TanStack Query, SWR, Apollo) for server state.
- Handle loading, error, and success states explicitly.
- Implement proper error boundaries for component tree isolation.
- Consider optimistic updates for better UX.

**Example - Data Fetching Pattern:**
```tsx
// ✅ Good: Using data fetching library (TanStack Query example)
function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });
  
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return <NotFound />;
  
  return <ProfileView user={user} />;
}
```

## Forms

### Form Handling
- Use controlled components for form inputs requiring validation or transformation.
- Consider form libraries (React Hook Form, Formik) for complex forms.
- Validate on blur for better UX; show errors after user interaction.
- Use proper input types and accessibility attributes.

**Example - Controlled Form:**
```tsx
interface FormData {
  email: string;
  password: string;
}

interface FormErrors {
  email?: string;
  password?: string;
}

function LoginForm() {
  const [formData, setFormData] = useState<FormData>({
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState<FormErrors>({});
  const [touched, setTouched] = useState<Set<string>>(new Set());
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleBlur = (e: React.FocusEvent<HTMLInputElement>) => {
    const { name } = e.target;
    setTouched(prev => new Set(prev).add(name));
    validateField(name, formData[name as keyof FormData]);
  };
  
  const validateField = (name: string, value: string) => {
    // Validation logic
  };
  
  return (
    <form>
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
        onBlur={handleBlur}
        aria-invalid={touched.has('email') && !!errors.email}
        aria-describedby={errors.email ? 'email-error' : undefined}
      />
      {touched.has('email') && errors.email && (
        <span id="email-error" role="alert">{errors.email}</span>
      )}
    </form>
  );
}
```

## Accessibility

### ARIA and Semantic HTML
- Use semantic HTML elements (button, nav, main, article, etc.) over divs.
- Add ARIA labels for interactive elements without visible text.
- Ensure keyboard navigation works for all interactive elements.
- Use appropriate ARIA roles and states.
- Test with screen readers.

**Example - Accessible Components:**
```tsx
// ❌ Bad: Div buttons without accessibility
function IconButton({ onClick }) {
  return <div onClick={onClick}>×</div>;
}

// ✅ Good: Proper button with ARIA
function IconButton({ 
  onClick, 
  label 
}: { 
  onClick: () => void; 
  label: string;
}) {
  return (
    <button
      onClick={onClick}
      aria-label={label}
      type="button"
    >
      <CloseIcon aria-hidden="true" />
    </button>
  );
}
```

## TypeScript Integration

### Component Typing
- Use React.FC sparingly; prefer explicit prop interfaces with function components.
- Type children explicitly when needed.
- Use generics for reusable components with varying data types.

**Example - Component Typing:**
```tsx
// ❌ Avoid: React.FC (implicit children, less flexible)
const Button: React.FC<{ label: string }> = ({ label, children }) => (
  <button>{label}</button>
);

// ✅ Good: Explicit typing
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary';
  onClick?: () => void;
  children?: React.ReactNode;
}

function Button({ label, variant = 'primary', onClick, children }: ButtonProps) {
  return (
    <button className={variant} onClick={onClick}>
      {label}
      {children}
    </button>
  );
}

// ✅ Good: Generic components
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map(item => (
        <li key={keyExtractor(item)}>
          {renderItem(item)}
        </li>
      ))}
    </ul>
  );
}
```

## Testing React Components

### Component Testing Strategy
- Test user behavior, not implementation details.
- Use React Testing Library for component tests.
- Query by accessible roles and labels, not test IDs when possible.
- Test integration of components together, not in isolation always.

**Example - Component Tests:**
```tsx
// ✅ Good: Testing user behavior
describe('LoginForm', () => {
  it('should submit form with email and password', async () => {
    const handleSubmit = jest.fn();
    render(<LoginForm onSubmit={handleSubmit} />);
    
    // Query by labels (accessible)
    const emailInput = screen.getByLabelText(/email/i);
    const passwordInput = screen.getByLabelText(/password/i);
    const submitButton = screen.getByRole('button', { name: /log in/i });
    
    // Simulate user interaction
    await userEvent.type(emailInput, 'user@example.com');
    await userEvent.type(passwordInput, 'password123');
    await userEvent.click(submitButton);
    
    // Assert behavior
    expect(handleSubmit).toHaveBeenCalledWith({
      email: 'user@example.com',
      password: 'password123'
    });
  });
  
  it('should show validation error for invalid email', async () => {
    render(<LoginForm onSubmit={jest.fn()} />);
    
    const emailInput = screen.getByLabelText(/email/i);
    await userEvent.type(emailInput, 'invalid-email');
    await userEvent.tab(); // Trigger blur
    
    expect(await screen.findByRole('alert'))
      .toHaveTextContent(/valid email/i);
  });
});
```

## Error Boundaries

### Error Handling
- Wrap feature sections with error boundaries.
- Provide meaningful fallback UI.
- Log errors to monitoring service.
- Don't catch errors in event handlers with error boundaries (use try/catch).

**Example - Error Boundary:**
```tsx
interface ErrorBoundaryProps {
  children: React.ReactNode;
  fallback?: React.ReactNode;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends React.Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    this.props.onError?.(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div role="alert">
          <h2>Something went wrong</h2>
          <details>
            <summary>Error details</summary>
            <pre>{this.state.error?.message}</pre>
          </details>
        </div>
      );
    }
    
    return this.props.children;
  }
}
```

## Project Structure

### File Organization
- Group by feature/domain, not by file type.
- Keep shared/common components separate from feature components.
- Co-locate related files (components, hooks, utils, types).

**Example - Feature-Based Structure:**
```
src/
  features/
    auth/
      components/
        LoginForm.tsx
        SignupForm.tsx
      hooks/
        useAuth.ts
        useSession.ts
      types/
        auth.types.ts
      authService.ts
      index.ts
    products/
      components/
        ProductList.tsx
        ProductCard.tsx
      hooks/
        useProducts.ts
      types/
        product.types.ts
      productService.ts
      index.ts
  shared/
    components/
      Button/
      Input/
      Modal/
    hooks/
      useLocalStorage.ts
      useDebounce.ts
    utils/
      formatting.ts
      validation.ts
```

## Styling

### CSS Approaches
- Choose one styling approach and be consistent (CSS Modules, CSS-in-JS, utility-first).
- Avoid inline styles except for dynamic values.
- Use CSS custom properties for theming.
- Keep style co-located with components when using CSS Modules or CSS-in-JS.

**Example - CSS Modules:**
```tsx
// Button.module.css
.button {
  padding: var(--spacing-md);
  border-radius: var(--radius-sm);
}

.primary {
  background: var(--color-primary);
  color: white;
}

// Button.tsx
import styles from './Button.module.css';

function Button({ variant }: { variant: 'primary' | 'secondary' }) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      Click me
    </button>
  );
}
```

## Build and Development

### Development Tools
- Use React DevTools for debugging component trees and performance.
- Enable React Strict Mode to catch potential issues.
- Use Fast Refresh/HMR for better development experience.
- Configure ESLint with React-specific rules.

**Example - Strict Mode:**
```tsx
// ✅ Good: Enable Strict Mode
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root')!);
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

