# Higher-Order Components (HOC) 🔄

Higher-Order Components are a pattern in React where a function takes a component and returns a new enhanced component with additional props, behavior, or capabilities. This pattern enables code reuse, logic abstraction, and composition.

## 📋 Table of Contents

- [Core Concepts](#core-concepts)
- [Implementation Approaches](#implementation-approaches)
- [Benefits](#benefits)
- [Implementation Guide](#implementation-guide)
- [Examples](#examples)
- [Common Pitfalls](#common-pitfalls)
- [Resources](#resources)

## 🎯 Core Concepts

Higher-Order Components revolve around these key concepts:

1. **Component Transformation** - A function that takes a component and returns a new component
2. **Logic Reuse** - Extracting common functionality into reusable wrappers
3. **Separation of Concerns** - Dividing responsibilities between display and behavior
4. **Composition** - Building complex behavior through component composition
5. **Props Proxy** - Manipulating props before passing them to wrapped components

## 🔄 Implementation Approaches

There are several approaches to implementing HOCs:

### 1. Props Proxy Pattern
- Renders the wrapped component with modified props
- Simplest implementation
- Minimal impact on component lifecycle

### 2. Inheritance Inversion
- Extends the wrapped component
- Provides more control over render method and lifecycle
- More complex and potentially fragile

### 3. Render Hijacking
- Controls what is rendered by the wrapped component
- Can conditionally render or modify output
- Useful for conditional rendering patterns

### 4. Compose / Pipe
- Combining multiple HOCs together
- Creates a pipeline of transformations
- Improves readability of multiple HOC applications

## 🌟 Benefits

- **Code Reuse** - Extract common logic from multiple components
- **Separation of Concerns** - Split cross-cutting concerns from presentation
- **Composability** - Combine multiple HOCs for complex behavior
- **Unobtrusive** - Add functionality without modifying original components
- **Testability** - Test HOCs and wrapped components separately
- **Dynamic Enhancement** - Apply enhancements conditionally at runtime

## 🚀 Implementation Guide

### Creating a Basic HOC

1. **Identify Reusable Logic**
   - Find common patterns across components
   - Consider cross-cutting concerns (logging, authentication, etc.)

2. **Create HOC Function**
   - Write a function that accepts a component
   - Return a new component that renders the original

3. **Handle Props Correctly**
   - Forward all relevant props to the wrapped component
   - Add new props as needed

4. **Preserve Component Name and displayName**
   - Maintain component identity for debugging
   - Set proper displayName for DevTools

5. **Forward Refs**
   - Use React.forwardRef for HOCs
   - Ensure ref accessibility to wrapped component

6. **Structure Your Files**
   ```
   src/
   ├── hocs/
   │   ├── withAuth.tsx       # Authentication HOC
   │   ├── withLogging.tsx    # Logging HOC
   │   ├── withData.tsx       # Data fetching HOC
   │   └── index.ts           # Exports all HOCs
   ```

## 💻 Examples

### Authentication HOC

```tsx
// hocs/withAuth.tsx
import React, { ComponentType } from 'react';
import { useAuth } from '../hooks/useAuth';
import { LoginPage } from '../pages/LoginPage';

interface WithAuthProps {
  requiredRole?: string;
}

// HOC that redirects to login if user is not authenticated
export function withAuth<P extends object>(
  WrappedComponent: ComponentType<P>,
  { requiredRole }: WithAuthProps = {}
) {
  const displayName = WrappedComponent.displayName || WrappedComponent.name || 'Component';
  
  // Create the new component
  const WithAuth = (props: P) => {
    const { user, isLoading } = useAuth();
    
    // Show loading state
    if (isLoading) {
      return <div>Loading authentication status...</div>;
    }
    
    // Redirect to login if not authenticated
    if (!user) {
      return <LoginPage />;
    }
    
    // Check for required role
    if (requiredRole && !user.roles.includes(requiredRole)) {
      return <div>Access denied. You need {requiredRole} role to view this page.</div>;
    }
    
    // Render the wrapped component with all props
    return <WrappedComponent {...props} />;
  };
  
  // Set display name for debugging
  WithAuth.displayName = `withAuth(${displayName})`;
  
  return WithAuth;
}

// Usage example
// const ProtectedComponent = withAuth(UserProfile, { requiredRole: 'admin' });
```

### Data Fetching HOC

```tsx
// hocs/withData.tsx
import React, { ComponentType, useState, useEffect } from 'react';

interface WithDataProps<T> {
  url: string;
  initialData?: T;
}

interface InjectedProps<T> {
  data: T | null;
  isLoading: boolean;
  error: Error | null;
  refetch: () => void;
}

export function withData<T, P extends object>(
  WrappedComponent: ComponentType<P & InjectedProps<T>>,
  { url, initialData = null }: WithDataProps<T>
) {
  const displayName = WrappedComponent.displayName || WrappedComponent.name || 'Component';
  
  const WithData = (props: P) => {
    const [data, setData] = useState<T | null>(initialData);
    const [isLoading, setIsLoading] = useState(true);
    const [error, setError] = useState<Error | null>(null);
    
    const fetchData = async () => {
      setIsLoading(true);
      setError(null);
      
      try {
        const response = await fetch(url);
        
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
        
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err instanceof Error ? err : new Error('An unknown error occurred'));
      } finally {
        setIsLoading(false);
      }
    };
    
    useEffect(() => {
      fetchData();
    }, [url]);
    
    // Inject our props and pass through the original props
    return (
      <WrappedComponent
        {...props}
        data={data}
        isLoading={isLoading}
        error={error}
        refetch={fetchData}
      />
    );
  };
  
  WithData.displayName = `withData(${displayName})`;
  
  return WithData;
}

// Usage example:
// interface User {
//   id: number;
//   name: string;
//   email: string;
// }
// 
// interface UserListProps {
//   title: string;
//   data: User[] | null;
//   isLoading: boolean;
//   error: Error | null;
//   refetch: () => void;
// }
// 
// function UserList({ title, data, isLoading, error, refetch }: UserListProps) {
//   return (
//     <div>
//       <h1>{title}</h1>
//       <button onClick={refetch}>Refresh</button>
//       {isLoading && <p>Loading...</p>}
//       {error && <p>Error: {error.message}</p>}
//       {data && (
//         <ul>
//           {data.map(user => (
//             <li key={user.id}>{user.name} ({user.email})</li>
//           ))}
//         </ul>
//       )}
//     </div>
//   );
// }
// 
// const UsersWithData = withData<User[], { title: string }>(
//   UserList, 
//   { url: 'https://api.example.com/users' }
// );
// 
// // Then use it as:
// <UsersWithData title="User Directory" />
```

### HOC with Forwarded Ref

```tsx
// hocs/withForwardedRef.tsx
import React, { forwardRef, ComponentType, ForwardedRef, PropsWithoutRef } from 'react';

export function withForwardedRef<T, P extends { ref?: React.Ref<T> }>(
  WrappedComponent: ComponentType<P>
) {
  const displayName = WrappedComponent.displayName || WrappedComponent.name || 'Component';
  
  // Create a new component that forwards the ref
  const WithForwardedRef = (
    props: PropsWithoutRef<P>, 
    ref: ForwardedRef<T>
  ) => {
    return <WrappedComponent {...props as P} ref={ref} />;
  };
  
  // Set display name for debugging
  WithForwardedRef.displayName = `withForwardedRef(${displayName})`;
  
  // Return the component with forwarded ref
  return forwardRef(WithForwardedRef);
}

// Usage example:
// class MyInput extends React.Component<{ value: string }> {
//   focus() {
//     // focus implementation
//   }
//   render() {
//     return <input value={this.props.value} />;
//   }
// }
// 
// const ForwardedInput = withForwardedRef<HTMLInputElement, { value: string }>(MyInput);
// 
// function Form() {
//   const inputRef = useRef<HTMLInputElement>(null);
//   
//   const focusInput = () => {
//     inputRef.current?.focus();
//   };
//   
//   return (
//     <div>
//       <ForwardedInput ref={inputRef} value="Hello" />
//       <button onClick={focusInput}>Focus Input</button>
//     </div>
//   );
// }
```

### Composing Multiple HOCs

```tsx
// hocs/compose.ts
import { ComponentType } from 'react';

// Helper to compose multiple HOCs from right to left
export function compose<T>(...funcs: Function[]) {
  return funcs.reduce((a, b) => (...args: any) => a(b(...args)), (arg: T) => arg) as T;
}

// Usage example:
// const EnhancedComponent = compose(
//   withAuth({ requiredRole: 'admin' }),
//   withLogging({ level: 'info' }),
//   withData<User>({ url: '/api/user' })
// )(BaseComponent);
```

## ⚠️ Common Pitfalls

- **Wrapper Hell** - Too many nested HOCs making debugging difficult
- **Props Collision** - Multiple HOCs injecting props with the same name
- **Static Methods Loss** - Static methods from wrapped component not copied to HOC
- **Ref Forwarding Issues** - Refs not properly passed to wrapped components
- **Performance Overhead** - Excessive re-rendering with multiple HOC layers
- **Conditional Rendering Bugs** - HOC conditionally rendering can lead to React reconciliation issues
- **TypeScript Complexity** - Complex generic types for proper type inference

## 🌟 Best Practices

- **Use Composition** - Compose multiple HOCs with a utility function
- **Forward All Props** - Pass through props you don't use
- **Forward Refs** - Use forwardRef for DOM components
- **Preserve Static Methods** - Copy static methods from the wrapped component
- **Add Displayname** - Set proper displayName for debugging
- **Wrap Only at Export** - Apply HOCs when exporting components
- **Consider Alternatives** - Use hooks when appropriate

## 🔄 Comparison with Other Patterns

| Pattern | Advantages | Disadvantages |
|---------|------------|---------------|
| HOCs | Clear separation of concerns, works with class components | Wrapper hell, props collision, static methods loss |
| Render Props | More explicit data flow, no naming collisions | Can lead to callback hell, potential performance issues |
| Custom Hooks | Simpler syntax, easier composition, less boilerplate | Limited to function components, rules of hooks constraints |
| Context API | Avoids prop drilling, centralized state | Can be overkill for simple sharing, potential performance concerns |

## 📚 Resources

- [React Documentation: Higher-Order Components](https://reactjs.org/docs/higher-order-components.html)
- [Dan Abramov: Mixins Considered Harmful](https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html)
- [Recompose](https://github.com/acdlite/recompose) - A React utility belt for function components and HOCs
- [React Patterns](https://reactpatterns.com/#higher-order-component) - HOC patterns and examples

---

💡 **Pro Tip:** With the introduction of Hooks in React, many use cases for HOCs can now be solved with custom hooks in a more straightforward way. Consider whether a custom hook might be a better solution before implementing an HOC.

```tsx
// Example of replacing an HOC with a custom hook
// Instead of:
// const UserWithData = withData(UserProfile, { url: '/api/user' });

// You could use:
function UserProfile() {
  const { data, isLoading, error, refetch } = useData('/api/user');
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!data) return <div>No data</div>;
  
  return (
    <div>
      <h1>{data.name}</h1>
      <button onClick={refetch}>Refresh</button>
      {/* Rest of the component */}
    </div>
  );
}
```
