# Provider Pattern 🌐

The Provider Pattern is a component composition pattern in React that allows sharing data across the component tree without manually passing props down through multiple levels. It uses React's Context API to make data accessible to any component that needs it.

## 📋 Table of Contents

- [Core Concepts](#core-concepts)
- [Implementation Approaches](#implementation-approaches)
- [Benefits](#benefits)
- [Implementation Guide](#implementation-guide)
- [Examples](#examples)
- [Common Pitfalls](#common-pitfalls)
- [Resources](#resources)

## 🎯 Core Concepts

The Provider Pattern revolves around the following key concepts:

1. **Central Data Store** - A shared source of truth for multiple components
2. **Provider Component** - A high-level component that supplies data and functions to its descendants
3. **Consumer Components** - Components that access the provided data without direct props
4. **Context API** - React's built-in mechanism for dependency injection

## 🔄 Implementation Approaches

The Provider Pattern can be implemented in several ways:

### 1. Simple Context Provider
- Basic context with single value
- Best for simple, static data
- Example: Theme settings, user preferences

### 2. Context + State Provider
- Combines Context with useState
- Suitable for dynamic data with simple updates
- Example: Simple shopping cart, notification system

### 3. Context + Reducer Provider
- Uses Context with useReducer
- Ideal for complex state with structured updates
- Example: Authentication system, multi-step forms

### 4. Advanced Provider
- Combines multiple contexts
- Handles derived state, side effects
- May include middleware-like functionality
- Example: Data fetching with caching, global application state

## 🌟 Benefits

- **Prevents Prop Drilling** - Eliminates passing props through intermediate components
- **Component Decoupling** - Reduces dependencies between components
- **Simplified Data Access** - Consistent pattern for accessing global state
- **Better Code Organization** - Centralizes related data and logic
- **Testability** - Components can be tested in isolation with mock providers
- **Reusability** - Providers can be used in different parts of the application

## 🚀 Implementation Guide

### Creating a Provider

1. **Define Your Context**
   - Create a context with a meaningful default value or `undefined`
   - Include TypeScript types for context value

2. **Create Provider Component**
   - Implement state management (useState, useReducer)
   - Expose necessary functions for updating state
   - Wrap children with Context.Provider

3. **Create Custom Hook for Consumption**
   - Make a hook that calls useContext
   - Add helpful error handling
   - Provide a clean API for consumers

4. **Structure Your Files**
   ```
   src/
   ├── contexts/
   │   ├── CartContext/
   │   │   ├── CartContext.tsx     # Context definition and provider
   │   │   ├── CartReducer.ts      # State logic (for useReducer)
   │   │   ├── CartActions.ts      # Action creators
   │   │   ├── types.ts            # TypeScript types
   │   │   └── index.ts            # Public API
   ```

## 💻 Examples

### Simple Theme Provider

```tsx
// contexts/ThemeContext.tsx
import React, { createContext, useContext, useState } from 'react';

// Define types
type Theme = 'light' | 'dark';
interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
}

// Create context with a default value
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Create provider component
export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<Theme>('light');
  
  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };
  
  // Value provided to consuming components
  const value = {
    theme,
    toggleTheme
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};

// Custom hook for consuming the context
export const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  
  return context;
};
```

### Advanced Cart Provider with Reducer

```tsx
// contexts/CartContext/types.ts
export interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

export interface CartState {
  items: CartItem[];
  totalItems: number;
  totalPrice: number;
  isLoading: boolean;
  error: string | null;
}

export type CartAction =
  | { type: 'ADD_ITEM'; payload: Omit<CartItem, 'quantity'> }
  | { type: 'REMOVE_ITEM'; payload: string }
  | { type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }
  | { type: 'CLEAR_CART' }
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SET_ERROR'; payload: string | null };

export interface CartContextType {
  state: CartState;
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
}

// contexts/CartContext/CartReducer.ts
import { CartState, CartAction } from './types';

export const initialState: CartState = {
  items: [],
  totalItems: 0,
  totalPrice: 0,
  isLoading: false,
  error: null
};

// Helper function to recalculate totals (kept private)
const recalculateTotals = (items: CartItem[]): { totalItems: number; totalPrice: number } => {
  return items.reduce(
    (acc, item) => ({
      totalItems: acc.totalItems + item.quantity,
      totalPrice: acc.totalPrice + item.price * item.quantity
    }),
    { totalItems: 0, totalPrice: 0 }
  );
};

export const cartReducer = (state: CartState, action: CartAction): CartState => {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingItemIndex = state.items.findIndex(item => item.id === action.payload.id);
      
      if (existingItemIndex >= 0) {
        // Item exists, update quantity
        const newItems = [...state.items];
        newItems[existingItemIndex] = {
          ...newItems[existingItemIndex],
          quantity: newItems[existingItemIndex].quantity + 1
        };
        
        const { totalItems, totalPrice } = recalculateTotals(newItems);
        
        return {
          ...state,
          items: newItems,
          totalItems,
          totalPrice
        };
      } else {
        // Add new item
        const newItem = { ...action.payload, quantity: 1 };
        const newItems = [...state.items, newItem];
        const { totalItems, totalPrice } = recalculateTotals(newItems);
        
        return {
          ...state,
          items: newItems,
          totalItems,
          totalPrice
        };
      }
    }
    
    case 'REMOVE_ITEM': {
      const newItems = state.items.filter(item => item.id !== action.payload);
      const { totalItems, totalPrice } = recalculateTotals(newItems);
      
      return {
        ...state,
        items: newItems,
        totalItems,
        totalPrice
      };
    }
    
    // Other action handlers...

    default:
      return state;
  }
};

// contexts/CartContext/CartContext.tsx
import React, { createContext, useContext, useReducer, useCallback } from 'react';
import { CartContextType, CartState, CartItem } from './types';
import { cartReducer, initialState } from './CartReducer';

const CartContext = createContext<CartContextType | undefined>(undefined);

export const CartProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  
  const addItem = useCallback((item: Omit<CartItem, 'quantity'>) => {
    dispatch({ type: 'ADD_ITEM', payload: item });
  }, []);
  
  const removeItem = useCallback((id: string) => {
    dispatch({ type: 'REMOVE_ITEM', payload: id });
  }, []);
  
  const updateQuantity = useCallback((id: string, quantity: number) => {
    dispatch({ type: 'UPDATE_QUANTITY', payload: { id, quantity } });
  }, []);
  
  const clearCart = useCallback(() => {
    dispatch({ type: 'CLEAR_CART' });
  }, []);
  
  const value = {
    state,
    addItem,
    removeItem,
    updateQuantity,
    clearCart
  };
  
  return (
    <CartContext.Provider value={value}>
      {children}
    </CartContext.Provider>
  );
};

export const useCart = (): CartContextType => {
  const context = useContext(CartContext);
  
  if (context === undefined) {
    throw new Error('useCart must be used within a CartProvider');
  }
  
  return context;
};

// Usage in a component
// import { useCart } from '../contexts/CartContext';
// 
// const ProductCard = ({ product }) => {
//   const { addItem } = useCart();
//   
//   return (
//     <div>
//       <h3>{product.name}</h3>
//       <p>${product.price}</p>
//       <button onClick={() => addItem(product)}>Add to Cart</button>
//     </div>
//   );
// };
```

## ⚠️ Common Pitfalls

- **Context Hell** - Too many nested providers decreasing readability
- **Performance Issues** - Re-renders when context value changes
- **Overusing Context** - Using global state for props that should be local
- **Missing Error Handling** - Not checking if context is undefined
- **Poor TypeScript Integration** - Incorrect or missing type definitions
- **Tight Coupling** - Creating direct dependencies between providers

## 🌟 Best Practices

- **Keep Providers Focused** - Each provider should handle one cohesive piece of functionality
- **Use Memoization** - Memoize context values to prevent unnecessary re-renders
- **Split Contexts** - Divide large contexts into smaller, more focused ones
- **Combine with Other Patterns** - Use alongside hooks, render props, or HOCs when appropriate
- **Custom Hooks** - Create specialized hooks for consuming context

## 📚 Resources

- [React Context API Documentation](https://reactjs.org/docs/context.html)
- [React useContext Hook Documentation](https://reactjs.org/docs/hooks-reference.html#usecontext)
- [React useReducer Hook Documentation](https://reactjs.org/docs/hooks-reference.html#usereducer)
- [Kent C. Dodds: How to use React Context effectively](https://kentcdodds.com/blog/how-to-use-react-context-effectively)

---

💡 **Pro Tip:** Consider creating a "provider composer" component to manage multiple nested providers in your application's root:

```tsx
const AppProviders: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <ThemeProvider>
    <AuthProvider>
      <CartProvider>
        <NotificationProvider>
          {children}
        </NotificationProvider>
      </CartProvider>
    </AuthProvider>
  </ThemeProvider>
);
```
