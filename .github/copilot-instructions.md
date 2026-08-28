# Angular Components - AI Agent Instructions

This is the official Angular Material, CDK, and related component libraries monorepo. This guide helps AI agents understand the codebase structure, conventions, and workflows.

## Project Architecture

### Repository Structure
- `src/material/` - Angular Material components (Material Design implementation)
- `src/cdk/` - Component Dev Kit (framework-agnostic interaction patterns)
- `src/material-experimental/` - Experimental Material components
- `src/google-maps/` - Google Maps Angular components
- `src/youtube-player/` - YouTube Player Angular component
- `src/dev-app/` - Development application for testing components
- `src/components-examples/` - Example code shown in documentation
- `docs/` - Documentation site (material.angular.dev)
- `tools/` - Build tooling and Bazel rule definitions
- `integration/` - Integration test suites

### Key Packages
Each component follows the pattern: `src/{package}/{component-name}/`. Standard files include:
- `BUILD.bazel` - Bazel build configuration with `ng_project`, `sass_binary`, `sass_library`
- Component TypeScript files, templates, and styles
- `{component}-module.ts` - NgModule definition
- `public-api.ts` - Public API exports
- `testing/` - Component test harnesses
- Theme SCSS files: `_m3-{component}.scss` (Material 3), `_m2-{component}.scss` (Material 2), `_{component}-theme.scss`

## Build System

### Bazel
This project uses **Bazel** for builds, not webpack or Angular CLI directly. Use `ibazel` for watch mode.

**Common commands:**
```bash
pnpm build                    # Build release packages
pnpm dev-app                  # Run dev server (ibazel run //src/dev-app:devserver)
pnpm docs-app                 # Run docs server
pnpm test <target>            # Test specific component (e.g., pnpm test button)
pnpm test <path>              # Test by path (e.g., pnpm test src/cdk/stepper)
pnpm test --local             # Run tests locally
pnpm e2e                      # Run e2e tests
```

### Package Manager
**Always use `pnpm`** (v10.20.0). NPM and Yarn are explicitly disallowed.

### Bazel Build Files
Components use custom wrappers from `tools/defaults.bzl`:
- `ng_project` - Compiles Angular TypeScript projects
- `ng_web_test_suite` - Runs Karma/Jasmine tests
- `sass_binary` - Compiles Sass with module mappings for `@angular/cdk`, `@angular/material`
- `sass_library` - Defines reusable Sass partials

## Coding Standards

### TypeScript Conventions

**Naming:**
- Classes with `mat-` selector prefix → `Mat` prefix (e.g., `MatButton`)
- CDK classes with `cdk` selector → `Cdk` prefix
- Avoid "Service" suffix; name by responsibility (e.g., `UniqueSelectionDispatcher` not `RadioService`)
- Methods describe actions performed, not when called (e.g., `activateRipple()` not `handleClick()`)
- Boolean properties/methods: use `is`/`has` prefix (except `@Input` properties)
- Observables: **no `$` suffix**

**Access Modifiers:**
- Omit `public` (default)
- `private` with underscore prefix: `private _internalState`
- `protected` without prefix
- Library-internal (Angular-visible but non-user-facing) with underscore, no `private` keyword

**Documentation:**
- All public APIs require JsDoc comments (extracted to material.angular.dev)
- Boolean properties: "Whether..." not "True if..."
- Use `@docs-private` to hide symbols from public docs
- Private/internal APIs need JsDoc when non-obvious

**Input Transforms:**
```typescript
import {Input, booleanAttribute} from '@angular/core';

@Input({transform: booleanAttribute}) disabled: boolean = false;
```

**Host Bindings:**
Prefer `host` object over `@HostBinding`/`@HostListener` decorators (prevents type preservation issues in SSR).

**Avoid:**
- `any` types (use generics)
- Inheritance for reusable behaviors (use TypeScript mixins)
- `try-catch` without explanatory comments
- Getters with no setter (use `readonly` property instead)

### Angular Patterns

**Selectors:**
- Components: lowercase, hyphen-delimited (e.g., `mat-button`)
- Directives: camelCase (e.g., `matTooltip`)

**Component Structure:**
- Expose native inputs via `<ng-content>` rather than wrapping
- Prefer granular components over complex configurable ones
- Avoid `display: flex` on projected content containers

### SCSS/CSS Standards

**Material Theming:**
- Use `@mixin` for theme-aware styles
- Organize: `_m3-{component}.scss`, `_m2-{component}.scss`, `_{component}-theme.scss`
- Sass module mappings: `@angular/cdk`, `@angular/material`, `@angular/material-experimental`

**Best Practices:**
- Lowest specificity possible
- Avoid nesting for code organization
- Prefer CSS classes over tag names/attributes
- Support Windows high-contrast mode with `@include cdk-high-contrast()`
- Add comments explaining non-obvious class purposes
- 100 column limit

## Testing

### Component Test Harnesses
Located in `{component}/testing/`. Extend `ComponentHarness` or `ContentContainerComponentHarness`:

```typescript
export class MatButtonHarness extends ComponentHarness {
  static hostSelector = 'button[mat-button]';
  // Methods for interacting with component in tests
}
```

### Test Commands
```bash
pnpm test button              # Test button component
pnpm test-local               # Run tests locally
pnpm test-firefox             # Test in Firefox
pnpm test-tsec                # Run TypeScript security checks
```

## Public API Management

**Approving API Changes:**
```bash
pnpm approve-api <target>     # Update golden files in goldens/<package>/
```

Review changes in `goldens/<package>/<entry-point>.api.md` before committing.

## Commit Message Format

```
<type>(<package>/<scope>): <subject>

<body>

<footer>
```

**Types:** `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `build`, `ci`, `release`  
**Package:** `material`, `cdk`, `google-maps`, `youtube-player`  
**Scope:** Specific component (e.g., `material/button`, `cdk/overlay`)

**Example:**
```
fix(material/button): unable to disable button through binding

Fixes a bug where buttons cannot be disabled through a binding.
The disabled input did not set the .mat-button-disabled class.

Fixes #1234
```

## Development Workflows

### Adding a New Component
1. Create directory: `src/{package}/{component-name}/`
2. Add `BUILD.bazel` with `ng_project`, `sass_binary`, `sass_library` targets
3. Create component files, module, `public-api.ts`
4. Add theme mixins: `_m3-*.scss`, `_m2-*.scss`, `_*-theme.scss`
5. Create test harness in `testing/`
6. Update `goldens/` with `pnpm approve-api`
7. Add examples to `src/components-examples/`
8. Add to dev-app for manual testing

### Windows Development
Use Windows Subsystem for Linux (WSL). Native Windows development is not supported.

### Environment Variables
For dev-app:
- `GOOGLE_MAPS_KEY` - Google Maps API key (optional)

### Circular Dependencies
```bash
pnpm ts-circular-deps:check   # Check for circular dependencies
pnpm ts-circular-deps:approve # Approve known circular deps
```

### Code Quality
```bash
pnpm lint                     # Run all linters (tslint, stylelint, ownerslint, format check)
pnpm format                   # Format changed files
pnpm check-tooling-setup      # Validate TypeScript tooling configuration
```

## Key Principles

1. **"Less is more"** - Avoid adding features that don't offer high value
2. **Single API** - Only one way to accomplish a task
3. **Smallest files** - Aim for 200-300 lines; refactor at 400+
4. **High-quality components** - Accessible, well-tested, performant, well-documented
5. **100-column limit** - All code, docs, scripts, markdown

## Resources

- [Full Contributing Guide](../CONTRIBUTING.md)
- [Coding Standards](../CODING_STANDARDS.md)
- [Dev Environment Setup](../DEV_ENVIRONMENT.md)
- [Material Design Spec](https://material.io)
- [Public Documentation](https://material.angular.dev)
