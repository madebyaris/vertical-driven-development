# Vertical Driven Development (VDD) for Cursor IDE

> **Build only what SHOULD change, not what COULD change.**

A comprehensive ruleset and documentation system for Cursor IDE that guides AI-assisted development to focus exclusively on business requirements and necessary changes, avoiding over-engineering and premature abstractions.

## 🎯 What is Vertical Driven Development?

Vertical Driven Development (VDD) is an architectural approach that organizes code around **business features and use cases** rather than technical layers. Instead of building horizontal layers (Controllers → Services → Repositories), VDD builds **vertical slices** where each feature contains everything it needs from UI to database.

### Core Principles

1. **Business Requirement First** - Always start with the business problem, not technical architecture
2. **Vertical Slice Organization** - Group by feature/use case, not by technical layer
3. **Simplicity First** - Start with Transaction Script, refactor when code smells emerge
4. **Change-Focused Development** - Only modify what needs to change, couple along the axis of change

### Key Benefits

- ✅ **Faster Delivery** - Complete features in single iterations
- ✅ **Less Over-Engineering** - Build only what's needed, when it's needed
- ✅ **Better Maintainability** - Changes isolated to single slices
- ✅ **Clearer Code** - Related code lives together
- ✅ **Easier Testing** - Test complete slices end-to-end

## 🚀 Quick Start

### Installation

1. **Clone or copy this repository** to your project
2. **The rules are automatically active** - Cursor will use the `.cursor/rules/*.mdc` files
3. **Read the documentation** to understand VDD principles

### Using with Cursor IDE

The VDD rules are automatically applied when you use Cursor's AI features. The rules guide the AI to:

- Focus on business requirements first
- Build vertical slices instead of layers
- Start simple and refactor when needed
- Minimize coupling between features

### Integration with SDD Toolkit

This VDD ruleset integrates seamlessly with the [Spec-Kit Command Cursor](https://github.com/madebyaris/spec-kit-command-cursor) toolkit:

- Use `/brief` for most features (80% of cases)
- Full SDD workflow for complex features (20% of cases)
- All specs and plans focus on vertical slices

See [SDD Integration Guide](docs/SDD-INTEGRATION.md) for details.

## 📚 Documentation

### Core Documentation

- **[VDD Guidelines](docs/VDD-GUIDELINES.md)** - Comprehensive guide with examples, patterns, and best practices
- **[SDD Integration](docs/SDD-INTEGRATION.md)** - How to use VDD with Spec-Driven Development toolkit

### Cursor Rules

The following rules are automatically applied in Cursor:

- **[vdd-core-principles.mdc](.cursor/rules/vdd-core-principles.mdc)** - Core VDD philosophy and principles
- **[vdd-development-workflow.mdc](.cursor/rules/vdd-development-workflow.mdc)** - Step-by-step development process
- **[vdd-sdd-integration.mdc](.cursor/rules/vdd-sdd-integration.mdc)** - SDD toolkit integration guidelines

## 🏗️ Architecture Comparison

### Traditional Layered Architecture ❌

```
src/
  controllers/
    order-controller.ts
  services/
    order-service.ts
  repositories/
    order-repository.ts
  models/
    order.ts
```

**Problems:**
- Changes ripple across multiple layers
- Over-abstraction for simple features
- Hard to see complete feature flow
- Slow feature delivery

### Vertical Slice Architecture ✅

```
features/
  create-order/
    create-order.api.ts
    create-order.handler.ts
    create-order.repository.ts
    create-order.test.ts
  cancel-order/
    cancel-order.api.ts
    cancel-order.handler.ts
    cancel-order.repository.ts
    cancel-order.test.ts
```

**Benefits:**
- Changes isolated to single slices
- Simple by default, complex when needed
- Complete feature flow visible
- Fast feature delivery

## 💡 Key Concepts

### Vertical Slice

A **vertical slice** is a complete, end-to-end feature that includes:

- **Presentation**: UI components, API endpoints, forms
- **Business Logic**: Validation, calculations, business rules
- **Data Access**: Database queries, external API calls, persistence

All of these live together, organized by feature.

### Coupling Principles

- **Maximize coupling within a slice**: Related code should be close together
- **Minimize coupling between slices**: Slices should be independent
- **Couple along the axis of change**: Features that change together should be together

### Development Workflow

1. **Start with Happy Path** - Simplest implementation that works
2. **Build Vertically** - One complete slice at a time
3. **Add Complexity Incrementally** - Only when simple solution shows limitations
4. **Refactor When Needed** - Based on code smells, not preemptively

## 📖 Examples

### Example 1: Simple Feature (Single File)

```typescript
// features/user-registration/user-registration.ts
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

### Example 2: Feature Folder Structure

```
features/
  create-order/
    create-order.api.ts          # API endpoint
    create-order.handler.ts       # Business logic
    create-order.repository.ts    # Data access
    create-order.types.ts         # Types
    create-order.test.ts         # Tests
```

## 🎓 Learning Resources

### Core References

- [Jimmy Bogard - Vertical Slice Architecture](https://www.jimmybogard.com/vertical-slice-architecture/)
- [SSENSE - Vertical Software Development](https://medium.com/ssense-tech/vertical-software-development-495b73f7fcdf)
- [Model-Driven Software Development Research](https://www.researchgate.net/publication/228388816_Model-Driven_Software_Development)

### Related Tools

- [Spec-Kit Command Cursor](https://github.com/madebyaris/spec-kit-command-cursor) - SDD toolkit for Cursor IDE

## 🤝 Contributing

Contributions are welcome! Areas for improvement:

- Additional examples and patterns
- Language/framework-specific guidance
- More detailed refactoring guidelines
- Integration with other tools

## 📝 License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Inspired by Jimmy Bogard's Vertical Slice Architecture
- Built for the Cursor IDE community
- Integrates with Spec-Kit Command Cursor by [@madebyaris](https://github.com/madebyaris)

## 🔗 Quick Links

- [VDD Guidelines](docs/VDD-GUIDELINES.md) - Detailed guidelines and examples
- [SDD Integration](docs/SDD-INTEGRATION.md) - Using VDD with SDD toolkit
- [Core Principles](.cursor/rules/vdd-core-principles.mdc) - VDD philosophy
- [Development Workflow](.cursor/rules/vdd-development-workflow.mdc) - Step-by-step process
- [SDD Integration Rules](.cursor/rules/vdd-sdd-integration.mdc) - Integration guidelines

---

**Remember: Build only what SHOULD change, not what COULD change.**

