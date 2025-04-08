# Render Props 🎭

The Render Props pattern is a technique in React for sharing code between components using a prop whose value is a function. This function returns a React element, allowing components to share code, logic, and state while giving consumers control over rendering.

## 📋 Table of Contents

- [Core Concepts](#core-concepts)
- [Implementation Approaches](#implementation-approaches)
- [Benefits](#benefits)
- [Implementation Guide](#implementation-guide)
- [Examples](#examples)
- [Common Pitfalls](#common-pitfalls)
- [Resources](#resources)

## 🎯 Core Concepts

The Render Props pattern is built around these key concepts:

1. **Function as a Prop** - A component receives a function prop that returns React elements
2. **Logic Encapsulation** - The component handles behavior and state management
3. **Rendering Delegation** - The consumer controls what gets rendered
4. **Inversion of Control** - Parent component provides structure, child determines content
5. **Dynamic Composition** - Consumers can compose UI based on props and state

## 🔄 Implementation Approaches

There are several ways to implement the Render Props pattern:

### 1. Render Prop via Function Property
- Uses a prop named `render` that is a function
- The function accepts data and returns JSX
- Most common implementation

### 2. Children as a Function
- Uses the standard `children` prop as a function
- Cleaner syntax with single render prop
- More intuitive JSX nesting

### 3. Component Prop
- Passes a component instead of a render function
- Component receives necessary props
- Useful for more complex rendering scenarios

### 4. Headless Component
- Component with no UI, only behavior
- Implements render props to let consumers control all visual aspects
- Focuses purely on functionality

## 🌟 Benefits

- **Code Reuse** - Share logic between components without inheritance
- **Separation of Concerns** - Split behavior from presentation
- **Flexibility** - Customize rendering based on state or props
- **Composition** - Compose components in different ways
- **Testability** - Test logic and rendering separately
- **Reduced HOC Problems** - Avoids naming collisions and "wrapper hell"
- **Type Safety** - Works well with TypeScript for prop typing

## 🚀 Implementation Guide

### Creating a Component with Render Props

1. **Identify Reusable Logic**
   - Determine what behavior/state should be extracted
   - Identify what data will be exposed to consumers

2. **Create Component Structure**
   - Build component that manages state and behavior
   - Accept render prop or children as function

3. **Expose Data and Functions**
   - Pass state and handler functions to render prop
   - Consider what API will be most useful to consumers

4. **Document Usage**
   - Clearly document what values are provided to the render function
   - Provide examples of common usage patterns

## 💻 Examples

### Mouse Tracker Example

```tsx
// components/MouseTracker.tsx
import React, { useState, useEffect } from 'react';

interface MousePosition {
  x: number;
  y: number;
}

interface MouseTrackerProps {
  render: (position: MousePosition) => React.ReactNode;
}

export function MouseTracker({ render }: MouseTrackerProps) {
  const [position, setPosition] = useState<MousePosition>({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (event: MouseEvent) => {
      setPosition({ x: event.clientX, y: event.clientY });
    };
    
    window.addEventListener('mousemove', handleMouseMove);
    
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []);
  
  return <>{render(position)}</>;
}

// Usage example
// <MouseTracker
//   render={({ x, y }) => (
//     <div>
//       <h1>Mouse Position</h1>
//       <p>X: {x}, Y: {y}</p>
//     </div>
//   )}
// />
```

### Children as Function Example

```tsx
// components/Toggle.tsx
import React, { useState } from 'react';

interface ToggleState {
  on: boolean;
  toggle: () => void;
}

interface ToggleProps {
  children: (state: ToggleState) => React.ReactNode;
  initialOn?: boolean;
}

export function Toggle({ children, initialOn = false }: ToggleProps) {
  const [on, setOn] = useState(initialOn);
  
  const toggle = () => {
    setOn(prevOn => !prevOn);
  };
  
  return <>{children({ on, toggle })}</>;
}

// Usage example
// <Toggle>
//   {({ on, toggle }) => (
//     <div>
//       <button onClick={toggle}>{on ? 'Turn Off' : 'Turn On'}</button>
//       <div>{on ? 'The light is on!' : 'The light is off.'}</div>
//     </div>
//   )}
// </Toggle>
```

### Fetch Data Example

```tsx
// components/DataFetcher.tsx
import React, { useState, useEffect } from 'react';

interface DataFetcherState<T> {
  data: T | null;
  isLoading: boolean;
  error: Error | null;
  refetch: () => void;
}

interface DataFetcherProps<T> {
  url: string;
  render: (state: DataFetcherState<T>) => React.ReactNode;
}

export function DataFetcher<T>({ url, render }: DataFetcherProps<T>) {
  const [data, setData] = useState<T | null>(null);
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
  
  return <>{render({ data, isLoading, error, refetch: fetchData })}</>;
}

// Usage example
// <DataFetcher
//   url="https://api.example.com/users"
//   render={({ data, isLoading, error, refetch }) => (
//     <div>
//       {isLoading && <p>Loading...</p>}
//       {error && <p>Error: {error.message}</p>}
//       {data && (
//         <ul>
//           {data.map(user => (
//             <li key={user.id}>{user.name}</li>
//           ))}
//         </ul>
//       )}
//       <button onClick={refetch}>Reload</button>
//     </div>
//   )}
// />
```

### Composition with Multiple Render Props

```tsx
// Combining multiple render props components
// <MouseTracker
//   render={mousePosition => (
//     <Toggle>
//       {({ on, toggle }) => (
//         <div>
//           <button onClick={toggle}>Toggle Tracking</button>
//           {on && (
//             <p>
//               Mouse position: {mousePosition.x}, {mousePosition.y}
//             </p>
//           )}
//         </div>
//       )}
//     </Toggle>
//   )}
// />
```

## ⚠️ Common Pitfalls

- **Performance Concerns** - Creating functions in render causes rerenders
- **Callback Hell** - Nesting multiple render props can reduce readability
- **Unnecessary Complexity** - Using render props when simpler patterns would suffice
- **Poor Type Definitions** - Inadequate TypeScript types leading to errors
- **Incomplete Documentation** - Not clearly explaining what data is passed to the render function
- **Inconsistent API** - Using different patterns (render vs children) in related components

## 🌟 Best Practices

- **Memoize Render Functions** - Use React.useCallback for render props when appropriate
- **Consistent Naming** - Use consistent prop names (render or children) across your codebase
- **Extract Complex Renders** - Move complex render functions to separate components
- **Provide Defaults** - Include fallback rendering when render prop isn't provided
- **Compose with Hooks** - Consider using custom hooks alongside render props
- **Clear Documentation** - Document all values passed to render functions

## 🔄 Comparison with Other Patterns

| Pattern | Advantages | Disadvantages |
|---------|------------|---------------|
| Render Props | Flexible rendering, clear data flow | Can lead to callback hell, potential performance issues |
| HOCs | Clean component tree, reusable logic | Naming collisions, "wrapper hell", static typing issues |
| Hooks | Simpler code, stateful logic reuse | Limited to function components, rules of hooks constraints |
| Compound Components | Great component composition, declarative API | More complex implementation, potentially tighter coupling |

## 📚 Resources

- [React Documentation: Render Props](https://reactjs.org/docs/render-props.html)
- [Kent C. Dodds: When to Use Render Props](https://kentcdodds.com/blog/when-to-use-render-props)
- [Ryan Florence: Render Props Talk](https://www.youtube.com/watch?v=BcVAq3YFiuc)
- [Downshift](https://github.com/downshift-js/downshift) - A library that uses render props effectively

---

💡 **Pro Tip:** Consider creating a custom hook that encapsulates the same logic as your render props component. This gives consumers the flexibility to choose between the render props pattern or hooks based on their needs.

```tsx
// Using both patterns for the same functionality
function useToggle(initialOn = false) {
  const [on, setOn] = useState(initialOn);
  const toggle = () => setOn(prevOn => !prevOn);
  return { on, toggle };
}

function Toggle({ children, initialOn = false }) {
  const { on, toggle } = useToggle(initialOn);
  return <>{children({ on, toggle })}</>;
}
```
