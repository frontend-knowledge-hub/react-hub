# Compound Components 🧩

The Compound Components pattern is a flexible and declarative approach to building complex components with a clean, expressive API. It allows components to share state and behavior while giving consumers control over the rendering and composition.

## 📋 Table of Contents

- [Core Concepts](#core-concepts)
- [Implementation Approaches](#implementation-approaches)
- [Benefits](#benefits)
- [Implementation Guide](#implementation-guide)
- [Examples](#examples)
- [Common Pitfalls](#common-pitfalls)
- [Resources](#resources)

## 🎯 Core Concepts

Compound Components are built around these key concepts:

1. **Parent-Child Relationship** - A parent component manages state and behavior shared with child components
2. **Implicit State Sharing** - Child components access parent state without explicit prop passing
3. **Declarative Composition** - Consumers compose components in a readable, HTML-like way
4. **Encapsulated Logic** - Complex behavior is hidden within the implementation
5. **Flexible Rendering** - Consumers control component structure and order

## 🔄 Implementation Approaches

There are several ways to implement Compound Components:

### 1. React.Children with Cloning
- Iterates through children using React.Children utilities
- Clones children with additional props
- Simple but limited to direct children

### 2. React Context API
- Uses Context for state sharing
- Allows for deeply nested children
- More flexible and maintainable

### 3. Composition with Static Properties
- Attaches child components as static properties of the parent
- Provides good IDE autocomplete and discoverability
- Simplifies imports

## 🌟 Benefits

- **Clean API** - Natural, declarative syntax similar to HTML
- **Separation of Concerns** - Parent handles logic, consumer controls rendering
- **Flexible Composition** - Reorder, add, or omit child components
- **Encapsulated State** - State management hidden from consumers
- **Self-documenting** - Component structure makes usage clear
- **Progressive Disclosure** - Can start simple and add complexity as needed

## 🚀 Implementation Guide

### Creating Compound Components with Context

1. **Plan Your Component API**
   - Identify what components will be exposed
   - Determine what state will be shared
   - Define responsibilities of each component

2. **Create Context and Provider**
   - Define a context to share state
   - Create a parent component with provider

3. **Create Child Components**
   - Build specialized components that use the shared context
   - Each child should have a clear, focused purpose

4. **Link Components**
   - Attach child components as static properties of the parent
   - Export the complete component set

5. **Structure Your Files**
   ```
   src/
   ├── components/
   │   ├── Menu/
   │   │   ├── Menu.tsx            # Parent component
   │   │   ├── MenuItem.tsx        # Child component
   │   │   ├── MenuButton.tsx      # Child component
   │   │   ├── MenuContext.tsx     # Shared context
   │   │   ├── types.ts            # TypeScript types
   │   │   └── index.ts            # Public API
   ```

## 💻 Examples

### Tabs Component Example

```tsx
// components/Tabs/TabsContext.tsx
import React, { createContext, useContext, useState } from 'react';

interface TabsContextType {
  activeTab: string;
  setActiveTab: (id: string) => void;
}

const TabsContext = createContext<TabsContextType | undefined>(undefined);

export function useTabsContext() {
  const context = useContext(TabsContext);
  if (context === undefined) {
    throw new Error('useTabsContext must be used within a Tabs component');
  }
  return context;
}

export const TabsProvider = TabsContext.Provider;

// components/Tabs/Tabs.tsx
import React, { useState, useMemo } from 'react';
import { TabsProvider } from './TabsContext';
import { TabList } from './TabList';
import { Tab } from './Tab';
import { TabPanels } from './TabPanels';
import { TabPanel } from './TabPanel';

interface TabsProps {
  defaultTab?: string;
  onChange?: (tabId: string) => void;
  children: React.ReactNode;
}

export function Tabs({ defaultTab, onChange, children }: TabsProps) {
  const [activeTab, setActiveTab] = useState(defaultTab || '');
  
  const handleTabChange = (tabId: string) => {
    setActiveTab(tabId);
    onChange?.(tabId);
  };
  
  const contextValue = useMemo(() => ({
    activeTab,
    setActiveTab: handleTabChange
  }), [activeTab, onChange]);
  
  return (
    <TabsProvider value={contextValue}>
      <div className="tabs-container">
        {children}
      </div>
    </TabsProvider>
  );
}

// Attach static components
Tabs.TabList = TabList;
Tabs.Tab = Tab;
Tabs.TabPanels = TabPanels;
Tabs.TabPanel = TabPanel;

// components/Tabs/TabList.tsx
import React from 'react';

interface TabListProps {
  children: React.ReactNode;
  className?: string;
}

export function TabList({ children, className = '' }: TabListProps) {
  return (
    <div className={`tabs-list ${className}`} role="tablist">
      {children}
    </div>
  );
}

// components/Tabs/Tab.tsx
import React from 'react';
import { useTabsContext } from './TabsContext';

interface TabProps {
  id: string;
  children: React.ReactNode;
  disabled?: boolean;
  className?: string;
}

export function Tab({ id, children, disabled = false, className = '' }: TabProps) {
  const { activeTab, setActiveTab } = useTabsContext();
  const isActive = activeTab === id;
  
  return (
    <button
      id={`tab-${id}`}
      role="tab"
      aria-selected={isActive}
      aria-controls={`panel-${id}`}
      tabIndex={isActive ? 0 : -1}
      disabled={disabled}
      className={`tab ${isActive ? 'active' : ''} ${className}`}
      onClick={() => !disabled && setActiveTab(id)}
    >
      {children}
    </button>
  );
}

// components/Tabs/TabPanels.tsx
import React from 'react';

interface TabPanelsProps {
  children: React.ReactNode;
  className?: string;
}

export function TabPanels({ children, className = '' }: TabPanelsProps) {
  return (
    <div className={`tabs-panels ${className}`}>
      {children}
    </div>
  );
}

// components/Tabs/TabPanel.tsx
import React from 'react';
import { useTabsContext } from './TabsContext';

interface TabPanelProps {
  id: string;
  children: React.ReactNode;
  className?: string;
}

export function TabPanel({ id, children, className = '' }: TabPanelProps) {
  const { activeTab } = useTabsContext();
  const isActive = activeTab === id;
  
  if (!isActive) return null;
  
  return (
    <div
      id={`panel-${id}`}
      role="tabpanel"
      aria-labelledby={`tab-${id}`}
      className={`tab-panel ${className}`}
    >
      {children}
    </div>
  );
}

// components/Tabs/index.ts
export { Tabs } from './Tabs';

// Usage example
// import { Tabs } from './components/Tabs';
//
// function MyComponent() {
//   return (
//     <Tabs defaultTab="profile" onChange={(tabId) => console.log(`Tab changed to ${tabId}`)}>
//       <Tabs.TabList>
//         <Tabs.Tab id="profile">Profile</Tabs.Tab>
//         <Tabs.Tab id="settings">Settings</Tabs.Tab>
//         <Tabs.Tab id="notifications">Notifications</Tabs.Tab>
//       </Tabs.TabList>
//       <Tabs.TabPanels>
//         <Tabs.TabPanel id="profile">
//           <h2>Profile Content</h2>
//           <p>User profile information goes here.</p>
//         </Tabs.TabPanel>
//         <Tabs.TabPanel id="settings">
//           <h2>Settings Content</h2>
//           <p>User settings go here.</p>
//         </Tabs.TabPanel>
//         <Tabs.TabPanel id="notifications">
//           <h2>Notifications Content</h2>
//           <p>User notifications go here.</p>
//         </Tabs.TabPanel>
//       </Tabs.TabPanels>
//     </Tabs>
//   );
// }
```

### Accordion Component Example (simplified)

```tsx
// components/Accordion/AccordionContext.tsx
import React, { createContext, useContext } from 'react';

interface AccordionContextType {
  expandedItems: string[];
  toggleItem: (id: string) => void;
}

const AccordionContext = createContext<AccordionContextType | undefined>(undefined);

export const useAccordionContext = () => {
  const context = useContext(AccordionContext);
  if (context === undefined) {
    throw new Error('useAccordionContext must be used within an Accordion');
  }
  return context;
};

export const AccordionProvider = AccordionContext.Provider;

// components/Accordion/Accordion.tsx
import React, { useState, useMemo } from 'react';
import { AccordionProvider } from './AccordionContext';
import { AccordionItem } from './AccordionItem';
import { AccordionHeader } from './AccordionHeader';
import { AccordionPanel } from './AccordionPanel';

interface AccordionProps {
  children: React.ReactNode;
  allowMultiple?: boolean;
  defaultExpandedItems?: string[];
}

export function Accordion({ 
  children, 
  allowMultiple = false, 
  defaultExpandedItems = [] 
}: AccordionProps) {
  const [expandedItems, setExpandedItems] = useState<string[]>(defaultExpandedItems);
  
  const toggleItem = (id: string) => {
    setExpandedItems(prevItems => {
      if (prevItems.includes(id)) {
        return prevItems.filter(item => item !== id);
      } else {
        return allowMultiple ? [...prevItems, id] : [id];
      }
    });
  };
  
  const value = useMemo(() => ({
    expandedItems,
    toggleItem
  }), [expandedItems]);
  
  return (
    <AccordionProvider value={value}>
      <div className="accordion">
        {children}
      </div>
    </AccordionProvider>
  );
}

// Attach static components
Accordion.Item = AccordionItem;
Accordion.Header = AccordionHeader;
Accordion.Panel = AccordionPanel;

// Additional component implementations (simplified)...
```

## ⚠️ Common Pitfalls

- **Overuse** - Not every component needs to be a compound component
- **API Complexity** - Difficult for other developers to understand without documentation
- **React.Children Limitations** - Cloning children approach doesn't work for deeply nested elements
- **TypeScript Challenges** - Advanced typing can be complex
- **Prop Drilling within Static Properties** - Static subcomponents may still need props passed down
- **Performance Concerns** - Context updates can cause unnecessary re-renders

## 🌟 Best Practices

- **Clear Component Naming** - Use intuitive, consistent names for related components
- **Use Static Properties** - Makes component relationship explicit
- **Document Component Structure** - Provide examples of valid compositions
- **Context Optimization** - Split context if different parts update at different frequencies
- **TypeScript Support** - Add comprehensive type definitions
- **Test as a Unit** - Test compound components together

## 📚 Resources

- [Kent C. Dodds: Compound Components](https://kentcdodds.com/blog/compound-components-with-react-hooks)
- [React Patterns: Compound Components](https://reactpatterns.com/#compound-components)
- [Reach UI](https://reach.tech/) - Examples of well-designed compound components
- [Chakra UI](https://chakra-ui.com/) - UI library using compound components

---

💡 **Pro Tip:** Consider providing sensible defaults for child components while still allowing for customization. For example, a Dropdown component could render a default DropdownMenu if none is provided.
