# VDD Integration with Spec-Driven Development (SDD)

## Overview

Vertical Driven Development (VDD) and Spec-Driven Development (SDD) are complementary methodologies that work together to ensure well-planned, business-focused development.

- **SDD** provides structure and planning before coding
- **VDD** ensures implementation focuses on business requirements and vertical slices
- **Together**, they deliver features faster with fewer iterations

This guide shows how to use the [Spec-Kit Command Cursor](https://github.com/madebyaris/spec-kit-command-cursor) toolkit while following VDD principles.

## Table of Contents

1. [Understanding the Integration](#understanding-the-integration)
2. [Command Selection Strategy](#command-selection-strategy)
3. [Creating VDD-Focused Briefs](#creating-vdd-focused-briefs)
4. [Planning Vertical Slices](#planning-vertical-slices)
5. [Task Breakdown for Vertical Development](#task-breakdown-for-vertical-development)
6. [Implementation with VDD Principles](#implementation-with-vdd-principles)
7. [Living Documentation](#living-documentation)
8. [Common Integration Mistakes](#common-integration-mistakes)

## Understanding the Integration

### How VDD and SDD Complement Each Other

**SDD Workflow:**
1. Research → Understand the problem
2. Specify → Define requirements clearly
3. Plan → Design the solution
4. Tasks → Break down into actionable items
5. Implement → Build the feature

**VDD Principles Applied:**
- Specifications describe **complete vertical slices**, not layers
- Plans focus on **slice independence** and minimal coupling
- Tasks follow **vertical development** (UI → Logic → Data)
- Implementation builds **one complete slice at a time**

### The 80/20 Rule

- **80% of features**: Use `/brief` for quick 30-minute planning
- **20% of features**: Use full SDD workflow for complex features

## Command Selection Strategy

### When to Use `/brief`

Use `/brief` for features that are:
- Simple to moderate complexity
- Have clear business requirements
- Represent a single vertical slice
- Don't require extensive research

**Example:**
```
/brief user-registration Allow new users to create accounts with email and password
```

### When to Use Full SDD Workflow

Use full workflow (`/research` → `/specify` → `/plan` → `/tasks` → `/implement`) for:
- Complex features requiring research
- Multiple related vertical slices
- Significant technical decisions
- Integration with external systems
- Features with unclear requirements

**Example:**
```
/research payment-processing
/specify payment-processing
/plan payment-processing
/tasks payment-processing
/implement payment-processing
```

## Creating VDD-Focused Briefs

### Brief Structure for Vertical Slices

When using `/brief`, structure it around the complete vertical slice:

#### 1. Business Requirement

**Focus on:**
- What problem are we solving?
- Who is the user?
- What value does this provide?

**Example:**
```markdown
## Business Requirement
Allow customers to create orders so they can purchase products from our store.
This enables the core e-commerce functionality and drives revenue.
```

#### 2. Vertical Slice Description

**Describe the complete end-to-end flow:**
- UI/API entry point
- Business logic needed
- Data requirements
- Output/response

**Example:**
```markdown
## Vertical Slice
- **Entry Point**: POST /orders API endpoint
- **Input**: User ID, array of items (product ID, quantity)
- **Business Logic**: 
  - Validate items exist and are in stock
  - Calculate total price (item price × quantity)
  - Create order record
- **Data**: Insert order into orders table
- **Output**: Return created order with ID and total
```

#### 3. Slice Boundaries

**Clearly define:**
- What's included in this slice
- What's explicitly excluded
- Dependencies on other slices (minimize these)

**Example:**
```markdown
## Slice Boundaries
- **Included**: 
  - Order creation API
  - Item validation
  - Price calculation
  - Order persistence
- **Excluded**:
  - Payment processing (separate slice)
  - Inventory updates (separate slice)
  - Email notifications (separate slice)
- **Dependencies**:
  - Product database (read-only, infrastructure)
  - User authentication (assumed to exist)
```

#### 4. Success Criteria

**Define measurable outcomes:**
- How do we know it works?
- What does "done" look like?

**Example:**
```markdown
## Success Criteria
- User can submit order via API
- Invalid items are rejected with clear errors
- Valid orders are saved to database
- Order includes correct total calculation
- Order ID is returned for tracking
```

### Complete Brief Example

```markdown
# Feature Brief: Create Order

## Business Requirement
Allow customers to create orders so they can purchase products from our store.
This enables the core e-commerce functionality and drives revenue.

## Vertical Slice
- **Entry Point**: POST /orders API endpoint
- **Input**: User ID, array of items (product ID, quantity)
- **Business Logic**: 
  - Validate items exist and are in stock
  - Calculate total price (item price × quantity)
  - Create order record
- **Data**: Insert order into orders table
- **Output**: Return created order with ID and total

## Slice Boundaries
- **Included**: Order creation, validation, calculation, persistence
- **Excluded**: Payment, inventory updates, notifications
- **Dependencies**: Product database (read-only)

## Success Criteria
- User can submit order via API
- Invalid items are rejected
- Valid orders are saved with correct total
- Order ID is returned
```

## Planning Vertical Slices

### Plan Structure for VDD

When using `/plan`, ensure the plan focuses on vertical slice architecture:

#### 1. Slice Structure

**Define the file organization:**
- How is the feature organized as a vertical slice?
- What files/components are in the slice?

**Example:**
```markdown
## Slice Structure
```
features/
  create-order/
    create-order.api.ts          # API endpoint
    create-order.handler.ts       # Business logic
    create-order.repository.ts    # Data access
    create-order.types.ts         # TypeScript types
    create-order.test.ts          # Tests
```
```

#### 2. Technology Choices

**Justify choices based on slice needs:**
- Keep it simple
- Choose simplest solution that works
- Avoid generic architecture patterns

**Example:**
```markdown
## Technology Choices
- **API Framework**: Express.js (simple, direct)
- **Validation**: Zod (lightweight, TypeScript-first)
- **Database**: Direct SQL queries (no ORM needed for simple CRUD)
- **Testing**: Jest (standard, well-supported)

**Rationale**: 
- No need for complex framework for simple CRUD
- Zod provides type-safe validation without overhead
- Direct SQL is simpler than ORM for straightforward queries
```

#### 3. Slice Independence

**Plan for minimal coupling:**
- How does this slice avoid coupling with other slices?
- What shared code is truly needed?

**Example:**
```markdown
## Slice Independence
- **No dependencies on other feature slices**
- Uses shared database connection (infrastructure, not business logic)
- Order type defined in slice (not shared domain model yet)
- Can be tested independently with mocked database
```

#### 4. Implementation Approach

**Start simple, refactor when needed:**
- Begin with Transaction Script pattern
- Extract to domain model only if complexity emerges
- Refactor on code smells, not preemptively

**Example:**
```markdown
## Implementation Approach
1. **Start Simple**: Handler function with inline logic (Transaction Script)
2. **Add Validation**: Inline validation in handler
3. **Extract if Needed**: Move to domain model only if logic becomes complex
4. **Refactor Triggers**: 
   - Handler > 100 lines → Extract functions
   - Complex business rules → Extract domain model
   - Duplication across slices → Extract shared code (after 3+ occurrences)
```

## Task Breakdown for Vertical Development

### Task Organization Principles

**❌ Wrong (Horizontal):**
```
Task 1: Create all UI components
Task 2: Create all business logic
Task 3: Create all data access
Task 4: Wire everything together
```

**✅ Right (Vertical):**
```
Task 1: Build complete create-order slice (UI + Logic + Data)
Task 2: Add validation and error handling
Task 3: Add edge cases (out of stock, invalid items)
Task 4: Refactor if needed (only if code smells appear)
```

### Task Structure

Each task should:
- Be a **complete vertical slice** or increment
- Be **independently testable**
- Deliver **business value**
- Avoid **partial implementations** across layers

### Example Task List

```markdown
# Tasks: Create Order Feature

## Task 1: Basic Order Creation Slice (2 days)
**Goal**: Complete working order creation from API to database

- [ ] Create API endpoint (POST /orders)
- [ ] Implement order handler with basic logic
- [ ] Add database insertion
- [ ] End-to-end test: user can create order

**Acceptance Criteria**:
- API accepts order request
- Order is saved to database
- Order ID is returned
- Test passes end-to-end

## Task 2: Item Validation (1 day)
**Goal**: Validate items before creating order

- [ ] Add item existence validation
- [ ] Add stock availability check
- [ ] Return appropriate error messages
- [ ] Test validation scenarios

**Acceptance Criteria**:
- Invalid items are rejected
- Out-of-stock items are rejected
- Clear error messages returned
- Tests cover all validation cases

## Task 3: Price Calculation (1 day)
**Goal**: Calculate correct order total

- [ ] Calculate item subtotals (price × quantity)
- [ ] Calculate order total
- [ ] Handle edge cases (zero quantity, negative prices)
- [ ] Test calculation logic

**Acceptance Criteria**:
- Total is calculated correctly
- Edge cases are handled
- Calculation is tested

## Task 4: Refactor if Needed (if code smells appear)
**Goal**: Improve code quality if handler becomes complex

- [ ] Extract validation logic if handler > 100 lines
- [ ] Extract domain model if business rules become complex
- [ ] Extract shared code if duplication appears (3+ slices)

**Acceptance Criteria**:
- Code is maintainable
- Tests still pass
- No unnecessary abstractions
```

## Implementation with VDD Principles

### Using `/implement` with VDD

When using `/implement`, ensure VDD principles are followed:

#### 1. Build Vertically

Build one complete slice at a time, not one layer at a time.

**✅ Correct:**
```typescript
// Build complete slice in one go
// features/create-order/create-order.api.ts
app.post('/orders', async (req, res) => {
  const { userId, items } = req.body;
  
  // Validation
  if (!items || items.length === 0) {
    return res.status(400).json({ error: 'No items provided' });
  }
  
  // Business logic
  const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  
  // Data access
  const order = {
    id: generateId(),
    userId,
    items,
    total,
    createdAt: new Date()
  };
  await db.orders.insert(order);
  
  res.json(order);
});
```

**❌ Wrong:**
```typescript
// Building layers separately
// First: Just the API
app.post('/orders', async (req, res) => {
  // TODO: Add logic
  // TODO: Add data access
});

// Later: Add service layer
// Later: Add repository layer
```

#### 2. Start Simple

Begin with Transaction Script pattern, refactor when needed.

**Start Here:**
```typescript
async function createOrder(userId: string, items: Item[]) {
  const total = items.reduce((sum, item) => sum + item.price, 0);
  const order = { id: generateId(), userId, items, total };
  await db.orders.insert(order);
  return order;
}
```

**Refactor Later (if needed):**
```typescript
class Order {
  constructor(public userId: string, public items: Item[]) {}
  get total() { return this.items.reduce((sum, item) => sum + item.price, 0); }
}

async function createOrder(userId: string, items: Item[]) {
  const order = new Order(userId, items);
  await db.orders.insert(order);
  return order;
}
```

#### 3. Add Incrementally

Add complexity only when simple solution shows limitations.

**Step 1: Happy Path**
```typescript
async function createOrder(userId: string, items: Item[]) {
  const total = items.reduce((sum, item) => sum + item.price, 0);
  const order = { id: generateId(), userId, items, total };
  await db.orders.insert(order);
  return order;
}
```

**Step 2: Add Validation**
```typescript
async function createOrder(userId: string, items: Item[]) {
  if (items.length === 0) throw new Error('No items');
  const total = items.reduce((sum, item) => sum + item.price, 0);
  const order = { id: generateId(), userId, items, total };
  await db.orders.insert(order);
  return order;
}
```

**Step 3: Add Edge Cases**
```typescript
async function createOrder(userId: string, items: Item[]) {
  if (items.length === 0) throw new Error('No items');
  for (const item of items) {
    if (!await db.products.exists(item.productId)) {
      throw new Error(`Product ${item.productId} not found`);
    }
  }
  const total = items.reduce((sum, item) => sum + item.price, 0);
  const order = { id: generateId(), userId, items, total };
  await db.orders.insert(order);
  return order;
}
```

#### 4. Test Per Slice

Write end-to-end tests for each slice.

```typescript
// create-order.test.ts
describe('create-order slice', () => {
  it('creates order with items', async () => {
    const order = await createOrder('user-1', [
      { productId: 'prod-1', price: 10, quantity: 2 },
      { productId: 'prod-2', price: 20, quantity: 1 }
    ]);
    
    expect(order.total).toBe(40);
    expect(order.userId).toBe('user-1');
    
    // Verify in database
    const saved = await db.orders.findById(order.id);
    expect(saved).toBeDefined();
  });
  
  it('rejects empty items', async () => {
    await expect(createOrder('user-1', [])).rejects.toThrow('No items');
  });
});
```

## Living Documentation

### Using `/evolve` for VDD

The `/evolve` command helps keep specs aligned with actual implementation:

**Update specs to reflect:**
- Actual vertical slice structure (what was built)
- Real file organization (not planned, but actual)
- Refactoring decisions and why
- Slice boundaries as they evolved

**Example Evolution:**
```markdown
# Feature: Create Order

## Original Plan
- Separate handler and repository files
- Domain model for Order

## Actual Implementation
- Single file with handler function (simple enough)
- Transaction Script pattern (no domain model needed)
- Validation inline (not complex enough to extract)

## Refactoring Decisions
- Kept simple: Handler is only 50 lines, no need to split
- No domain model: Business logic is straightforward
- Inline validation: Only 3 validation rules, not worth extracting
```

## Common Integration Mistakes

### Mistake 1: Planning Layers Instead of Slices

**❌ Wrong:**
```markdown
## Plan
- Create repository layer
- Create service layer
- Create controller layer
```

**✅ Right:**
```markdown
## Plan
- Create order slice with API, handler, and data access
- Ensure slice is independent
- Plan for simple Transaction Script pattern
```

### Mistake 2: Over-Specifying Architecture

**❌ Wrong:**
```markdown
## Specification
- Use repository pattern
- Implement service layer
- Create DTOs for all data transfer
- Use dependency injection
```

**✅ Right:**
```markdown
## Specification
- User submits order via API
- System validates items
- System calculates total
- System saves order to database
- System returns order with ID
```

### Mistake 3: Breaking Tasks by Layer

**❌ Wrong:**
```markdown
## Tasks
- Task 1: Build all UI components
- Task 2: Build all business logic
- Task 3: Build all data access
```

**✅ Right:**
```markdown
## Tasks
- Task 1: Build complete create-order slice
- Task 2: Add validation
- Task 3: Add edge cases
```

### Mistake 4: Ignoring VDD in Implementation

**❌ Wrong:**
```typescript
// Building layers separately
class OrderController { }
class OrderService { }
class OrderRepository { }
```

**✅ Right:**
```typescript
// Building complete slice
async function createOrder(userId: string, items: Item[]) {
  // Everything in one place
}
```

## Best Practices Summary

1. **Use `/brief` by default** - Most features are simple enough
2. **Full SDD for complex features** - When research or multiple slices needed
3. **Keep specs business-focused** - Not technical architecture
4. **Plan for slice independence** - Minimize cross-slice dependencies
5. **Tasks follow vertical development** - Complete slices, not layers
6. **Evolve specs as you build** - Keep documentation aligned with reality
7. **Start simple, refactor when needed** - Transaction Script first
8. **Test per slice** - End-to-end tests for each slice

## Quick Reference

### Command Selection

| Feature Complexity | Command | VDD Focus |
|-------------------|---------|-----------|
| Simple | `/brief` | Complete vertical slice description |
| Moderate | `/brief` | Complete vertical slice description |
| Complex | Full SDD workflow | Vertical slice planning at each stage |

### Brief Structure

1. Business Requirement
2. Vertical Slice Description (end-to-end)
3. Slice Boundaries (included/excluded)
4. Success Criteria

### Plan Structure

1. Slice Structure (file organization)
2. Technology Choices (simple, justified)
3. Slice Independence (minimal coupling)
4. Implementation Approach (start simple)

### Task Organization

- Task 1: Complete slice (UI + Logic + Data)
- Task 2: Add validation
- Task 3: Add edge cases
- Task 4: Refactor if needed

## References

- [Spec-Kit Command Cursor](https://github.com/madebyaris/spec-kit-command-cursor)
- See `.cursor/rules/vdd-core-principles.mdc` for VDD philosophy
- See `.cursor/rules/vdd-development-workflow.mdc` for development process
- See `.cursor/rules/vdd-sdd-integration.mdc` for integration rules
- See `docs/VDD-GUIDELINES.md` for detailed VDD guidelines

