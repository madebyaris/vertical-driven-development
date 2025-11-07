---
description: Build a complete app/feature using VDD principles - Creates vertical slices with full SDD workflow in one command
alwaysApply: false
---

# /vdd-build Command

Build a complete application or feature using Vertical Driven Development (VDD) principles. This command combines the SDD full-plan workflow with VDD's vertical slice architecture to create a production-ready feature in one go.

## Usage

```
/vdd-build [feature-name] [description] [--implement]
```

**Parameters:**
- `feature-name`: Name of the feature/app to build (e.g., `e-commerce-platform`, `user-authentication`)
- `description`: Brief description of what the feature should do
- `--implement`: (Optional) Automatically implement all slices after planning

**Examples:**
```
/vdd-build e-commerce-platform "Full e-commerce platform with cart, checkout, and order management"
/vdd-build user-authentication "User registration, login, and password reset" --implement
/vdd-build blog-cms "Content management system for blog posts with categories and tags"
```

## What This Command Does

This command follows VDD principles to:

1. **Analyze Business Requirements** - Identifies the core business problem
2. **Identify Vertical Slices** - Breaks down into independent, complete features
3. **Prioritize by Business Value** - Orders slices by importance and dependencies
4. **Create Full SDD Workflow** - For each slice: research → specify → plan → tasks
5. **Organize Vertically** - Each slice contains UI → Logic → Data (not layers)
6. **Optionally Implement** - Can build all slices automatically

## VDD Principles Applied

### ✅ Vertical Slice Organization
- Each feature is a complete vertical slice
- Slices are independent and can be developed separately
- No shared layers unless genuinely needed

### ✅ Business Requirement First
- Starts with business problem, not technical architecture
- Focuses on user value, not technical perfection
- Only builds what's needed for current requirements

### ✅ Simplicity First
- Plans start with Transaction Script pattern
- Avoids premature abstractions
- Refactoring only when code smells appear

### ✅ Change-Focused Development
- Slices organized along axis of change
- Minimal coupling between slices
- Maximum coupling within slices

## Workflow

### Step 1: Business Analysis
Analyzes the description to identify:
- Core business problem
- User personas and needs
- Success criteria
- Business value

### Step 2: Vertical Slice Identification
Breaks down into vertical slices:
- Each slice is a complete, end-to-end feature
- Slices are prioritized by business value
- Dependencies between slices are identified

### Step 3: SDD Workflow Per Slice
For each vertical slice, creates:
- **Research** (if needed): Best practices and patterns
- **Specification**: Complete vertical slice description (UI → Logic → Data)
- **Plan**: Technical design focused on slice independence
- **Tasks**: Vertical development tasks (complete slice, not layers)

### Step 4: Implementation Roadmap
Creates a master roadmap showing:
- All vertical slices
- Dependencies between slices
- Implementation order
- Estimated effort per slice

### Step 5: Optional Implementation
If `--implement` flag is used:
- Implements slices in priority order
- Builds complete slices (not layers)
- Tests each slice end-to-end
- Documents as it builds

## Output Structure

```
specs/
  active/
    [feature-name]/
      vdd-master-plan.md          # Overall roadmap
      vertical-slices.md          # Slice breakdown
      implementation-order.md     # Priority and dependencies
      [slice-1]/
        research.md              # (if needed)
        spec.md                  # Complete vertical slice spec
        plan.md                  # Technical plan
        tasks.md                 # Vertical development tasks
      [slice-2]/
        ...
```

## Example: E-Commerce Platform

**Command:**
```
/vdd-build e-commerce-platform "Full e-commerce platform with product catalog, shopping cart, checkout, and order management"
```

**Identified Vertical Slices:**
1. **Product Catalog** - Browse and search products
2. **Shopping Cart** - Add/remove items, update quantities
3. **Checkout** - Payment processing and order creation
4. **Order Management** - View orders, track status
5. **User Authentication** - Registration and login (if needed)

**Each Slice Contains:**
- UI/API endpoints
- Business logic
- Data access
- Tests
- All in one vertical slice

## Integration with SDD Commands

This command internally uses SDD commands but organizes them by VDD principles:

- Uses `/research` only when technical decisions are needed
- Uses `/specify` to describe complete vertical slices
- Uses `/plan` to design for slice independence
- Uses `/tasks` to break down vertically (not horizontally)
- Uses `/implement` to build complete slices

## Best Practices

### ✅ Do:
- Start with the highest business value slice
- Build complete slices before moving to next
- Keep slices independent
- Test each slice end-to-end
- Document as you build

### ❌ Don't:
- Build all UI first, then all logic
- Create shared layers prematurely
- Over-engineer for hypothetical futures
- Skip testing individual slices
- Couple slices unnecessarily

## Comparison with Traditional Approach

**Traditional (Horizontal):**
```
Task 1: Build all UI components
Task 2: Build all business logic
Task 3: Build all data access
Task 4: Wire everything together
```

**VDD (Vertical):**
```
Slice 1: Complete product catalog (UI + Logic + Data)
Slice 2: Complete shopping cart (UI + Logic + Data)
Slice 3: Complete checkout (UI + Logic + Data)
```

## Advanced Options

### Custom Slice Priority
You can manually adjust slice priority in `implementation-order.md` after generation.

### Incremental Implementation
Build slices one at a time:
1. Review the master plan
2. Choose a slice to implement
3. Use `/implement [slice-name]` for that specific slice

### Slice Dependencies
The command automatically identifies dependencies:
- Slice A depends on Slice B → B is built first
- Independent slices can be built in parallel

## Troubleshooting

**"Too many slices identified"**
- Review `vertical-slices.md` and merge related slices
- Remember: Start simple, split when needed

**"Slices are too coupled"**
- Review slice boundaries in `vertical-slices.md`
- Ensure each slice can work independently

**"Implementation order unclear"**
- Check `implementation-order.md` for dependencies
- Start with slices that have no dependencies

## References

- See `.cursor/rules/vdd-core-principles.mdc` for VDD philosophy
- See `.cursor/rules/vdd-development-workflow.mdc` for development process
- See `.cursor/rules/vdd-sdd-integration.mdc` for SDD integration
- See [Spec-Kit Command Cursor](https://github.com/madebyaris/spec-kit-command-cursor) for SDD commands
