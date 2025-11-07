# Vertical Driven Development (VDD) Guidelines

## Table of Contents

1. [Introduction](#introduction)
2. [Core Concepts](#core-concepts)
3. [Step-by-Step Development Process](#step-by-step-development-process)
4. [Code Organization Patterns](#code-organization-patterns)
5. [When to Extract Shared Code](#when-to-extract-shared-code)
6. [Refactoring Guidelines](#refactoring-guidelines)
7. [Common Pitfalls](#common-pitfalls)
8. [Real-World Examples](#real-world-examples)

## Introduction

Vertical Driven Development (VDD) is an architectural approach that organizes code around **business features and use cases** rather than technical layers. The fundamental principle is: **Build only what SHOULD change, not what COULD change.**

### Why VDD?

Traditional layered architecture (Controllers → Services → Repositories) creates several problems:

- **Over-abstraction**: Creating interfaces and abstractions "just in case"
- **Cross-layer coupling**: Changes ripple across multiple layers
- **Premature optimization**: Building for hypothetical futures
- **Slow feature delivery**: Need to touch multiple layers for one feature

VDD solves these by:
- **Feature-focused**: All code for a feature lives together
- **Simple by default**: Start with Transaction Script, refactor when needed
- **Change-isolated**: Modifications stay within one slice
- **Fast delivery**: Complete features in single iterations

## Core Concepts

### Vertical Slice

A **vertical slice** is a complete, end-to-end feature that includes:

- **Presentation Layer**: UI components, API endpoints, forms
- **Business Logic**: Validation, calculations, business rules
- **Data Access**: Database queries, external API calls, persistence

All of these live together, organized by feature, not by technical concern.

### Coupling Principles

- **Maximize coupling within a slice**: Related code should be close together
- **Minimize coupling between slices**: Slices should be independent
- **Couple along the axis of change**: Features that change together should be together

### Simplicity First

Start with the simplest solution that works:

1. **Transaction Script**: Procedural code in a handler/controller
2. **Extract when needed**: Move to domain model only when complexity emerges
3. **Refactor on code smells**: Not preemptively

## Step-by-Step Development Process

### Step 1: Identify the Vertical Slice

**Question**: What user action or business use case does this represent?

**Example:**
- ✅ "User creates an order" → `create-order` slice
- ✅ "Admin views dashboard" → `admin-dashboard` slice
- ❌ "All repositories" → Not a slice
- ❌ "All services" → Not a slice

### Step 2: Start with the Happy Path

Implement the simplest version that works:

```typescript
// ✅ Start here - simple and direct
async function createOrder(userId: string, items: Item[]) {
  const total = items.reduce((sum, item) => sum + item.price, 0);
  const order = {
    id: generateId(),
    userId,
    items,
    total,
    status: 'pending',
    createdAt: new Date()
  };
  
  await db.orders.insert(order);
  return order;
}
```

**Not this:**
```typescript
// ❌ Don't start with abstractions
interface IOrderRepository {
  save(order: Order): Promise<void>;
}

class OrderService {
  constructor(private repo: IOrderRepository) {}
  // ... complex abstraction
}
```

### Step 3: Build Vertically

Develop one complete slice at a time:

**✅ Vertical Development:**
```
Day 1: Create Order slice (UI + Logic + Data)
Day 2: Cancel Order slice (UI + Logic + Data)
Day 3: View Order slice (UI + Logic + Data)
```

**❌ Horizontal Development:**
```
Day 1: All UI components
Day 2: All business logic
Day 3: All data access
Day 4: Try to wire it all together (often fails)
```

### Step 4: Add Complexity Incrementally

Add features only when the simple solution shows limitations:

1. **Happy path works** → Add validation
2. **Validation works** → Add error handling
3. **Errors handled** → Add edge cases
4. **Code gets complex** → Refactor to domain model

### Step 5: Test Per Slice

Each slice should be independently testable:

```typescript
// Test the entire slice, not individual layers
describe('create-order slice', () => {
  it('creates order with items', async () => {
    const order = await createOrder('user-1', [
      { id: 'item-1', price: 10 },
      { id: 'item-2', price: 20 }
    ]);
    
    expect(order.total).toBe(30);
    expect(order.status).toBe('pending');
    
    // Verify in database
    const saved = await db.orders.findById(order.id);
    expect(saved).toBeDefined();
  });
});
```

## Code Organization Patterns

### Pattern 1: Feature Folder Structure

For most features, organize by feature:

```
features/
  create-order/
    create-order.api.ts          # API endpoint
    create-order.handler.ts       # Business logic
    create-order.repository.ts    # Data access
    create-order.types.ts         # TypeScript types
    create-order.test.ts          # Tests
    index.ts                     # Public API
```

### Pattern 2: Single File for Simple Features

For very simple features, a single file is acceptable:

```
features/
  get-user-profile/
    get-user-profile.ts  # Everything in one file
```

**When to use:**
- Feature is < 100 lines
- No complex logic
- Single responsibility

### Pattern 3: CQRS Pattern (When Appropriate)

For complex features with separate commands and queries:

```
features/
  order-management/
    commands/
      create-order.ts
      cancel-order.ts
      update-order.ts
    queries/
      get-order.ts
      list-orders.ts
      get-order-history.ts
    shared/
      order.types.ts
```

**When to use:**
- Feature has many commands and queries
- Commands and queries have different requirements
- Need to scale reads and writes independently

### Pattern 4: Nested Features

For related features that share some code:

```
features/
  orders/
    create-order/
      create-order.ts
    cancel-order/
      cancel-order.ts
    shared/
      order.types.ts
      order-utils.ts
```

**When to use:**
- Features are closely related
- Some code is genuinely shared
- Features change together

## When to Extract Shared Code

### Rule of Three

Extract code to a shared location only when:
1. **Three or more slices use it**
2. **It's genuinely the same logic** (not similar but different)
3. **It changes for the same reason**

### Examples

**✅ Extract:**
```typescript
// Used by: create-order, update-order, cancel-order
// Same logic: Calculate order total
// Changes together: When pricing rules change

// shared/order-calculations.ts
export function calculateOrderTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}
```

**❌ Don't Extract:**
```typescript
// Used by: create-order (only)
// Keep it in the slice

// features/create-order/create-order.ts
function calculateTotal(items: Item[]) {
  return items.reduce((sum, item) => sum + item.price, 0);
}
```

### Shared Infrastructure vs Shared Business Logic

**✅ Share Infrastructure:**
- Database connections
- HTTP clients
- Configuration
- Logging utilities

**❌ Don't Share Business Logic Prematurely:**
- Validation rules (may differ per feature)
- Calculation logic (may have different requirements)
- Domain models (extract when 3+ slices need them)

## Refactoring Guidelines

### When to Refactor

Refactor when you see **code smells**, not preemptively:

1. **Duplication**: Same logic in 3+ places
2. **Complexity**: Code is hard to understand
3. **Testing Difficulty**: Hard to test in isolation
4. **Large Functions**: Single function does too much

### Refactoring Patterns

#### 1. Extract Function

**When:** Function is too long or does multiple things

```typescript
// Before
async function createOrder(userId: string, items: Item[]) {
  // Validate items
  if (items.length === 0) throw new Error('No items');
  for (const item of items) {
    if (!item.id || !item.price) throw new Error('Invalid item');
  }
  
  // Calculate total
  const total = items.reduce((sum, item) => sum + item.price, 0);
  
  // Create order
  const order = { id: generateId(), userId, items, total };
  await db.orders.insert(order);
  return order;
}

// After
async function createOrder(userId: string, items: Item[]) {
  validateItems(items);
  const total = calculateTotal(items);
  const order = buildOrder(userId, items, total);
  await db.orders.insert(order);
  return order;
}

function validateItems(items: Item[]) {
  if (items.length === 0) throw new Error('No items');
  for (const item of items) {
    if (!item.id || !item.price) throw new Error('Invalid item');
  }
}

function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

function buildOrder(userId: string, items: Item[], total: number): Order {
  return { id: generateId(), userId, items, total };
}
```

#### 2. Extract Domain Model

**When:** Business logic becomes complex

```typescript
// Before - Transaction Script
async function createOrder(userId: string, items: Item[]) {
  // Complex validation logic
  if (items.length === 0) throw new Error('No items');
  if (items.some(item => item.price < 0)) throw new Error('Invalid price');
  if (calculateTotal(items) > 10000) throw new Error('Order too large');
  
  // Complex calculation
  const subtotal = calculateTotal(items);
  const tax = subtotal * 0.1;
  const shipping = subtotal > 100 ? 0 : 10;
  const total = subtotal + tax + shipping;
  
  // ... more complex logic
}

// After - Domain Model
class Order {
  constructor(
    public userId: string,
    public items: Item[]
  ) {
    this.validate();
  }
  
  private validate() {
    if (this.items.length === 0) throw new Error('No items');
    if (this.items.some(item => item.price < 0)) throw new Error('Invalid price');
    if (this.total > 10000) throw new Error('Order too large');
  }
  
  get subtotal(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
  
  get tax(): number {
    return this.subtotal * 0.1;
  }
  
  get shipping(): number {
    return this.subtotal > 100 ? 0 : 10;
  }
  
  get total(): number {
    return this.subtotal + this.tax + this.shipping;
  }
}

async function createOrder(userId: string, items: Item[]) {
  const order = new Order(userId, items);
  await db.orders.insert(order);
  return order;
}
```

#### 3. Extract Shared Code

**When:** Same logic in 3+ slices

```typescript
// Before - Duplicated in 3+ slices
// features/create-order/create-order.ts
function calculateTotal(items: Item[]) {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// features/update-order/update-order.ts
function calculateTotal(items: Item[]) {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// features/estimate-order/estimate-order.ts
function calculateTotal(items: Item[]) {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// After - Extracted to shared
// shared/order-calculations.ts
export function calculateOrderTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// All slices import from shared
import { calculateOrderTotal } from '../../shared/order-calculations';
```

## Common Pitfalls

### Pitfall 1: Building Layers Instead of Slices

**❌ Wrong:**
```
src/
  controllers/
    order-controller.ts
  services/
    order-service.ts
  repositories/
    order-repository.ts
```

**✅ Right:**
```
features/
  create-order/
    create-order.api.ts
    create-order.handler.ts
    create-order.repository.ts
```

### Pitfall 2: Premature Abstraction

**❌ Wrong:**
```typescript
// Creating interface before knowing what we need
interface IOrderRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order>;
}

class OrderRepository implements IOrderRepository {
  // Implementation
}
```

**✅ Right:**
```typescript
// Start concrete, extract interface later if needed
async function saveOrder(order: Order) {
  await db.orders.insert(order);
}

async function findOrderById(id: string) {
  return await db.orders.findById(id);
}
```

### Pitfall 3: Over-Engineering Simple Features

**❌ Wrong:**
```typescript
// Complex domain model for simple CRUD
class Order {
  private _items: Item[];
  private _status: OrderStatus;
  
  constructor(items: Item[]) {
    this._items = items;
    this._status = OrderStatus.Pending;
  }
  
  addItem(item: Item) {
    this.validateItem(item);
    this._items.push(item);
  }
  
  // ... 50 more lines
}

// For a simple "create order" feature
```

**✅ Right:**
```typescript
// Simple handler for simple feature
async function createOrder(userId: string, items: Item[]) {
  const order = {
    id: generateId(),
    userId,
    items,
    total: calculateTotal(items),
    createdAt: new Date()
  };
  await db.orders.insert(order);
  return order;
}
```

### Pitfall 4: Ignoring the Happy Path

**❌ Wrong:**
```typescript
// Handle all edge cases before basic flow works
async function createOrder(userId: string, items: Item[]) {
  try {
    validateUser(userId);
    validateItems(items);
    checkInventory(items);
    calculateTax(items);
    applyDiscounts(items);
    // ... 20 more validations
    // Finally create order
  } catch (error) {
    // Complex error handling
  }
}
```

**✅ Right:**
```typescript
// Make happy path work first
async function createOrder(userId: string, items: Item[]) {
  const order = { id: generateId(), userId, items, total: calculateTotal(items) };
  await db.orders.insert(order);
  return order;
}

// Then add validation incrementally
async function createOrder(userId: string, items: Item[]) {
  if (items.length === 0) throw new Error('No items');
  const order = { id: generateId(), userId, items, total: calculateTotal(items) };
  await db.orders.insert(order);
  return order;
}
```

### Pitfall 5: Breaking Tasks by Layer

**❌ Wrong:**
- Task 1: Build all UI components
- Task 2: Build all business logic
- Task 3: Build all data access

**✅ Right:**
- Task 1: Build complete create-order slice
- Task 2: Build complete cancel-order slice
- Task 3: Build complete view-order slice

## Real-World Examples

### Example 1: E-Commerce Order System

**Traditional Layered Approach:**
```
controllers/
  order-controller.ts        # Handles HTTP
services/
  order-service.ts           # Business logic
repositories/
  order-repository.ts        # Data access
models/
  order.ts                   # Domain model
```

**VDD Approach:**
```
features/
  create-order/
    create-order.api.ts      # HTTP + Logic + Data
    create-order.test.ts
  cancel-order/
    cancel-order.api.ts
    cancel-order.test.ts
  view-order/
    view-order.api.ts
    view-order.test.ts
```

### Example 2: User Registration

**Simple Feature - Single File:**
```
features/
  user-registration/
    user-registration.ts     # Everything in one file
```

**Content:**
```typescript
// user-registration.ts
export async function registerUser(email: string, password: string) {
  // Validation
  if (!isValidEmail(email)) throw new Error('Invalid email');
  if (password.length < 8) throw new Error('Password too short');
  
  // Check duplicate
  const existing = await db.users.findByEmail(email);
  if (existing) throw new Error('Email already exists');
  
  // Create user
  const hashedPassword = await bcrypt.hash(password, 10);
  const user = {
    id: generateId(),
    email,
    password: hashedPassword,
    createdAt: new Date()
  };
  
  await db.users.insert(user);
  return user;
}
```

### Example 3: Complex Feature - Order Processing

**Multiple Files for Complex Logic:**
```
features/
  process-order/
    process-order.api.ts           # Entry point
    process-order.handler.ts       # Main logic
    process-order.validator.ts     # Validation
    process-order.calculator.ts    # Price calculations
    process-order.notifier.ts      # Email notifications
    process-order.repository.ts    # Data access
    process-order.types.ts         # Types
    process-order.test.ts          # Tests
```

## Summary

Vertical Driven Development is about:

1. **Organizing by features**, not layers
2. **Starting simple**, refactoring when needed
3. **Building vertically**, one complete slice at a time
4. **Focusing on business value**, not technical perfection
5. **Isolating changes** to single slices

Remember: **Build only what SHOULD change, not what COULD change.**

## References

- [Jimmy Bogard - Vertical Slice Architecture](https://www.jimmybogard.com/vertical-slice-architecture/)
- [SSENSE - Vertical Software Development](https://medium.com/ssense-tech/vertical-software-development-495b73f7fcdf)
- See `.cursor/rules/vdd-core-principles.mdc` for core rules
- See `.cursor/rules/vdd-development-workflow.mdc` for workflow

