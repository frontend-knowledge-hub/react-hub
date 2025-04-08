# Feature-Sliced Design (FSD) 🧩

Feature-Sliced Design is a methodology for frontend architecture that helps organize code in large applications by dividing it into layers based on business domain rather than technical functions.

## 📋 Table of Contents

- [Core Principles](#core-principles)
- [Layer Structure](#layer-structure)
- [Benefits](#benefits)
- [Implementation Guide](#implementation-guide)
- [Examples](#examples)
- [Common Pitfalls](#common-pitfalls)
- [Resources](#resources)

## 🎯 Core Principles

Feature-Sliced Design is built on three key principles:

1. **Segregation by Scope** - Components are organized by their business domain scope rather than technical function
2. **Layer Isolation** - Higher layers can import from lower layers, but not vice versa
3. **Shared Kernel** - Common code is extracted to shared layers

## 🔄 Layer Structure

FSD organizes code into the following layers, from highest to lowest:

### 1. App Layer
- Application initialization
- Global providers
- Routing configuration
- Style themes

### 2. Processes Layer (deprecated)
- Complex multi-step business processes
- User flows that span multiple features
- Example: Authentication flow, Checkout process

### 3. Pages Layer
- Compositions of widgets and features
- Route-specific components
- Page-level layouts

### 4. Widgets Layer
- Complex UI blocks composed of entities and features
- Independent components that solve specific interface tasks
- Example: Header, Sidebar, ProductCard

### 5. Features Layer
- User interactions and business logic
- Implements specific user stories
- Example: "Add to cart", "Filter products"

### 6. Entities Layer
- Business entities
- Domain models with business logic
- Example: User, Product, Order

### 7. Shared Layer
- Reusable infrastructure code
- UI kit components
- Utility functions
- API interfaces

## 🌟 Benefits

- **Scalability** - Easily scales with team and project growth
- **Maintainability** - Clear boundaries between domains
- **Team Autonomy** - Teams can work on isolated features
- **Consistent Structure** - Common patterns across the project
- **Reduced Coupling** - Dependencies flow in one direction
- **Improved Developer Experience** - Clear place for new code

## 🚀 Implementation Guide

### Getting Started

1. **Analyze Your Domain**
   - Identify your core business entities
   - Map out key features and user stories

2. **Set Up Project Structure**
   ```
   src/
   ├── app/         # Application initialization
   ├── processes/   # Cross-feature scenarios
   ├── pages/       # Application pages
   ├── widgets/     # Complex UI blocks
   ├── features/    # User interactions
   ├── entities/    # Business entities
   └── shared/      # Shared code
   ```

3. **Define Slice Organization**
   Within each layer, organize code by business domain:
   ```
   features/
   ├── auth/        # Authentication feature
   │   ├── ui/      # Components
   │   ├── model/   # Business logic
   │   ├── api/     # API requests
   │   ├── lib/     # Helper functions
   │   └── index.ts # Public API
   ├── cart/        # Shopping cart feature
   │   └── ...
   └── product/     # Product features
       └── ...
   ```

4. **Enforce Import Rules**
   - Higher layers can import from lower layers
   - Slices can only import from other slices' public APIs
   - Use ESLint rules to enforce architecture

## 💻 Examples

### Entity Layer Example

```typescript
// entities/product/model/types.ts
export interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
  imageUrl: string;
}

// entities/product/index.ts
export { type Product } from './model/types';
export { ProductCard } from './ui/ProductCard';
```

### Feature Layer Example

```typescript
// features/product/filter/ui/ProductFilter.tsx
import { Product } from '@/entities/product';

export const ProductFilter = ({ products, onFilter }) => {
  // Filter implementation
}

// features/product/filter/index.ts
export { ProductFilter } from './ui/ProductFilter';
```

## ⚠️ Common Pitfalls

- **Over-slicing** - Creating too many small slices
- **Under-slicing** - Making slices too large and coupled
- **Wrong layer placement** - Confusing features and entities
- **Circular dependencies** - Breaking the layer hierarchy
- **Public API neglect** - Not controlling what's exported

## Recommendations
I would recommend using ESLint Feature-sliced config for your project codestyle adjustment, so your code will be standardized and it would be very hard to make mistake:

[Eslint Github Repository](https://github.com/feature-sliced/eslint-config) 


## 📚 Resources

- [Feature-Sliced Design Official Website](https://feature-sliced.design/)
- [FSD Methodology Documentation](https://feature-sliced.design/docs/reference/overview)
- [GitHub Repository](https://github.com/feature-sliced/documentation)

---

💡 **Pro Tip:** Start with a less strict implementation and gradually refine your architecture as your understanding of the business domain increases.
