# Build Complete App/Feature with VDD Principles

## Objective

When the user invokes `/vdd-build [feature-name] [description] [--implement]`, analyze the business requirements, identify vertical slices, and create a complete SDD workflow for each slice following Vertical Driven Development (VDD) principles.

## Steps

### 1. Parse User Input
- Extract `feature-name` from the command
- Extract `description` from the command
- Check if `--implement` flag is present

### 2. Analyze Business Requirements
- Identify the core business problem from the description
- Determine user personas and their needs
- Define success criteria
- Assess business value

### 3. Identify Vertical Slices
- Break down the feature/app into independent, complete vertical slices
- Each slice must be a complete end-to-end feature (UI → Logic → Data)
- Prioritize slices by business value
- Identify dependencies between slices
- Ensure slices are organized along the axis of change (features that change together)

### 4. Create Master Plan Structure
Create the directory structure:
```
specs/active/[feature-name]/
  vdd-master-plan.md
  vertical-slices.md
  implementation-order.md
```

### 5. Generate Master Plan Documents

**vdd-master-plan.md:**
- Overall roadmap for the entire feature/app
- Business requirements summary
- List of all vertical slices
- High-level architecture overview

**vertical-slices.md:**
- Detailed breakdown of each vertical slice
- Description of what each slice contains (UI, logic, data)
- Slice boundaries (what's included/excluded)
- Dependencies between slices

**implementation-order.md:**
- Prioritized list of slices by business value
- Dependency graph showing which slices depend on others
- Recommended implementation order
- Estimated effort per slice

### 6. Process Each Vertical Slice

For each identified slice, create a subdirectory and run SDD workflow:

**For each slice:**

1. **Create slice directory:** `specs/active/[feature-name]/[slice-name]/`

2. **Research (if needed):**
   - Only if technical decisions are required
   - Research best practices for this specific slice
   - Document findings in `research.md`

3. **Specify:**
   - Create `spec.md` describing the complete vertical slice
   - Include: UI/API endpoints, business logic, data access
   - Focus on business flow, not technical architecture
   - Describe the complete user flow end-to-end

4. **Plan:**
   - Create `plan.md` with technical design
   - Focus on slice independence (minimal coupling with other slices)
   - Start with Transaction Script pattern (simplest approach)
   - Avoid premature abstractions
   - Plan for testing the complete slice

5. **Tasks:**
   - Create `tasks.md` with vertical development tasks
   - Tasks should be: "Build complete [slice-name] slice (UI + Logic + Data)"
   - NOT: "Build UI", "Build logic", "Build data access" separately
   - Include end-to-end testing task for the slice

### 7. Implementation (if --implement flag present)

If `--implement` flag is provided:

1. Process slices in dependency order (slices with no dependencies first)
2. For each slice:
   - Build the complete vertical slice (UI → Logic → Data)
   - Start with happy path implementation
   - Use Transaction Script pattern (simple procedural code)
   - Add edge cases incrementally
   - Write end-to-end tests
   - Document as you build
3. Ensure each slice is independently testable and deployable

## VDD Principles to Follow

- **Business Requirement First**: Always start with the business problem, not technical architecture
- **Vertical Slice Organization**: Each slice contains everything needed (UI, logic, data) - NOT organized by layers
- **Simplicity First**: Start with Transaction Script pattern, refactor only when code smells appear
- **Change-Focused**: Couple code along the axis of change (vertical slices), minimize coupling between slices
- **Build Only What's Needed**: Don't create abstractions "just in case", don't build for hypothetical futures

## Expected Output

After execution, the following structure should exist:

```
specs/active/[feature-name]/
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

If `--implement` flag was used, actual code should also exist in the project following vertical slice organization.

## Notes

- Never organize by technical layers (controllers, services, repositories)
- Each slice should be independently testable
- Keep slices simple - avoid premature abstractions
- Focus on business value, not technical perfection
- Build complete slices, not partial implementations across layers
