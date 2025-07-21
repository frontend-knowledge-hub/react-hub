# Module Pattern 📦

The Module Pattern is a design pattern that encapsulates code into a self-contained, reusable unit with private and public parts. In React, it's often implemented through well-structured components and hooks that expose a clean interface while hiding implementation details.

## 📋 Table of Contents

- [Module Pattern 📦](#module-pattern-)
  - [📋 Table of Contents](#-table-of-contents)
  - [🎯 Core Concepts](#-core-concepts)
  - [🔄 Implementation Approaches](#-implementation-approaches)
    - [1. File-based Modules](#1-file-based-modules)
    - [2. Custom Hooks](#2-custom-hooks)
    - [3. Component Composition](#3-component-composition)
    - [4. Context Modules](#4-context-modules)
  - [🌟 Benefits](#-benefits)
  - [🚀 Implementation Guide](#-implementation-guide)
    - [Creating a Module with Custom Hooks](#creating-a-module-with-custom-hooks)
  - [💻 Examples](#-examples)
    - [Custom Hook Module Example](#custom-hook-module-example)
    - [Context Module Example](#context-module-example)
  - [⚠️ Common Pitfalls](#️-common-pitfalls)
  - [📚 Resources](#-resources)

## 🎯 Core Concepts

The Module Pattern in React applications is built on these key concepts:

1. **Encapsulation** - Bundling data and methods that operate on that data within a single unit
2. **Information Hiding** - Exposing only necessary APIs while keeping implementation details private
3. **Namespace Management** - Preventing naming collisions by scoping variables to the module
4. **Reusability** - Creating self-contained, portable code units

## 🔄 Implementation Approaches

In React, the Module Pattern can be implemented through several techniques:

### 1. File-based Modules
- Using ES6 modules to organize code
- Export only what should be accessible outside the file
- Keep implementation details private in the file scope

### 2. Custom Hooks
- Encapsulate related logic in a hook
- Expose minimal, focused APIs
- Hide implementation complexity

### 3. Component Composition
- Create specialized components that handle specific concerns
- Compose complex UIs from simpler building blocks
- Control what props are exposed to consumers

### 4. Context Modules
- Combine Context API with reducers for state management
- Expose actions, not state manipulation functions
- Create provider components that encapsulate context logic

## 🌟 Benefits

- **Maintainability** - Easier to understand, debug, and modify
- **Testability** - Isolated functionality is easier to test
- **Code Organization** - Clear boundaries between different parts of the application
- **Reduced Coupling** - Minimizes dependencies between modules
- **Team Collaboration** - Different teams can work on separate modules
- **Controlled Interfaces** - Changes to implementation don't affect external code

## 🚀 Implementation Guide

### Creating a Module with Custom Hooks

1. **Identify a Cohesive Piece of Functionality**
   - Look for related state, effects, and event handlers
   - Group functionality by domain or feature

2. **Structure the Module**
   ```
   src/
   ├── features/
   │   ├── authentication/             # Authentication module
   │   │   ├── api.ts                  # API calls (private)
   │   │   ├── types.ts                # TypeScript types
   │   │   ├── hooks/                  # Custom hooks
   │   │   │   ├── useAuth.ts          # Public hook API
   │   │   │   └── useAuthInternal.ts  # Internal hook logic
   │   │   ├── components/             # UI components
   │   │   │   ├── LoginForm.tsx
   │   │   │   └── ...
   │   │   └── index.ts                # Public module API
   ```

3. **Define Public and Private APIs**
   - Export only what consumers need from index.ts
   - Keep implementation details in their own files
   - Use clear naming conventions for internal vs. external use

4. **Document the Interface**
   - Add JSDoc comments to explain how to use the module
   - Include examples for common use cases
   - Document any requirements or assumptions

## 💻 Examples

### Custom Hook Module Example

```typescript
// features/counter/hooks/useCounter.ts
import { useState, useCallback } from 'react';

// Private utility function (not exported)
function validateCount(count: number, min: number, max: number) {
  return Math.min(Math.max(count, min), max);
}

// Types that will be exported
export interface CounterOptions {
  initialCount?: number;
  min?: number;
  max?: number;
  step?: number;
}

// The public API of our module
export function useCounter({
  initialCount = 0,
  min = -Infinity,
  max = Infinity,
  step = 1
}: CounterOptions = {}) {
  const [count, setCount] = useState(validateCount(initialCount, min, max));

  const increment = useCallback(() => {
    setCount(c => validateCount(c + step, min, max));
  }, [step, min, max]);

  const decrement = useCallback(() => {
    setCount(c => validateCount(c - step, min, max));
  }, [step, min, max]);

  const reset = useCallback(() => {
    setCount(validateCount(initialCount, min, max));
  }, [initialCount, min, max]);

  return {
    count,
    increment,
    decrement,
    reset
  };
}

// features/counter/index.ts
export { useCounter } from './hooks/useCounter';
export type { CounterOptions } from './hooks/useCounter';
```

### Context Module Example

```typescript
// features/theme/ThemeContext.tsx
import React, { createContext, useContext, useState, useMemo } from 'react';

// Theme types
type Theme = 'light' | 'dark' | 'system';

// Context value type
interface ThemeContextValue {
  theme: Theme;
  setTheme: (theme: Theme) => void;
  toggleTheme: () => void;
  isDark: boolean;
}

// Create the context
const ThemeContext = createContext<ThemeContextValue | undefined>(undefined);

// Provider component that encapsulates theme logic
export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setThemeState] = useState<Theme>('system');
  
  // Derived state - is the theme dark?
  const isDark = useMemo(() => {
    if (theme === 'system') {
      return window.matchMedia('(prefers-color-scheme: dark)').matches;
    }
    return theme === 'dark';
  }, [theme]);
  
  // Public methods
  const setTheme = (newTheme: Theme) => {
    setThemeState(newTheme);
    // Implementation detail: save to localStorage
    localStorage.setItem('theme', newTheme);
  };
  
  const toggleTheme = () => {
    setTheme(isDark ? 'light' : 'dark');
  };
  
  // Create a stable context value
  const value = useMemo(() => ({
    theme,
    setTheme,
    toggleTheme,
    isDark
  }), [theme, isDark]);
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};

// Custom hook for consuming the context
export const useTheme = (): ThemeContextValue => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};

// features/theme/index.ts
export { ThemeProvider, useTheme } from './ThemeContext';
```

## ⚠️ Common Pitfalls

- **Over-modularization** - Creating too many tiny modules with excessive indirection
- **Leaky Abstractions** - Exposing implementation details that should be hidden
- **Poor Documentation** - Failing to clearly document the module's interface
- **Tight Coupling** - Creating modules that depend too heavily on each other
- **Inconsistent Exports** - Mixing named and default exports without a clear pattern

## 📚 Resources

- [JavaScript Module Pattern: In-Depth](https://www.toptal.com/javascript/javascript-module-pattern-commonjs-amd-requirejs)
- [Custom Hooks Documentation](https://reactjs.org/docs/hooks-custom.html)
- [React Context API](https://reactjs.org/docs/context.html)
- [Clean Code in JavaScript](https://github.com/ryanmcdermott/clean-code-javascript)

---

💡 **Pro Tip:** When designing a module, think carefully about its API. What's the minimum interface needed to provide the functionality? How can you make it intuitive and hard to misuse?
