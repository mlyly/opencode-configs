---
name: angular
description: Modern Angular engineering guidelines using standalone components, Signals, inject() DI, built-in control flow (@if/@for), and OnPush change detection. Invoke when generating or modifying Angular components, services, routes, HTTP interceptors, or tests.
metadata:
  version: "1.0.0"
  domain: frontend
  triggers: Angular, Signals, Standalone Components, RxJS, TypeScript, Reactive Forms
---

# Angular Engineering Standards & Agent Guidelines

## 1. Environment & Project Discovery

Before writing or modifying code, inspect the project root to align with existing conventions:

- **Package Manager**: Check lockfiles (`pnpm-lock.yaml`, `package-lock.json`, `yarn.lock`) and use the matching package
  manager (`pnpm`, `npm`, or `yarn`). Prefer local CLI execution (`npx ng` or `pnpm ng`).
- **Angular & TypeScript Versions**: Inspect `package.json` and `angular.json` to confirm the Angular version (
  v18/v19/v20+), styling preprocessor (`scss` vs `css`), UI library (Angular Material, PrimeNG, Spartan/Tailwind), and
  zoneless status (`provideExperimentalZonelessChangeDetection` / `provideZonelessChangeDetection`).
- **Linting & Formatting**: Check `eslint.config.js` (or `.eslintrc.json`) and `.prettierrc` to align with Angular
  ESLint rules (such as selector prefixes like `app-`).

## 2. Architecture & Directory Structure

Follow a scalable, feature-driven architecture:

- **Core (`src/app/core`)**: Singleton services (`providedIn: 'root'`), functional HTTP interceptors, functional route
  guards, and global error handlers.
- **Shared (`src/app/shared`)**: Reusable dumb/presentational components, directives, and pipes with zero
  feature-specific business logic.
- **Features (`src/app/features/<feature-name>`)**: Self-contained domain modules containing feature routes (
  `<feature>.routes.ts`), smart container components, presentational components, feature services, and TypeScript
  models.
- **Routing**: Use lazy-loaded standalone routes (`loadComponent` and `loadChildren`) with `withComponentInputBinding()`
  when configured.

## 3. Modern Angular & Signals Conventions

- **Standalone First**: Always generate and write **standalone** components, directives, and pipes. Do not create new
  `NgModule` files unless maintaining legacy boundaries. (`standalone: true` is the default in Angular 19+).
- **Dependency Injection**: Use the `inject()` function instead of constructor parameter injection for cleaner
  inheritance and functional guards/interceptors.
- **Reactivity with Signals**:
    - Prefer Angular Signals (`signal()`, `computed()`, `effect()`) for local component state and synchronous derived
      state.
    - Use signal-based component APIs: `input()`, `input.required()`, `output()`, `model()`, `viewChild()`, and
      `contentChild()` instead of legacy `@Input()` / `@Output()` decorators.
    - Use `linkedSignal()` for state that resets when a source signal changes, and `resource()` / `rxResource()` when
      adopting signal-driven async data loading.
- **Change Detection**: Always set `changeDetection: ChangeDetectionStrategy.OnPush` on every component. Avoid mutating
  objects or arrays in place; update signals immutably via `.update()` or `.set()`.
- **Built-in Control Flow**: Always use the modern built-in template syntax (`@if`, `@else`, `@for` with mandatory
  `track`, `@switch`, `@defer`) instead of legacy structural directives (`*ngIf`, `*ngFor`, `*ngSwitch`).

## 4. RxJS, HTTP & Forms

- **RxJS Interop**: Use RxJS for complex async event streams, debouncing, and HTTP pipelines. Bridge observables to
  templates cleanly using `toSignal()` (with `initialValue` or `requireSync`) or the `async` pipe.
- **Subscription Management**: Never leave open subscriptions. Use `takeUntilDestroyed()` (from
  `@angular/core/rxjs-interop`) inside injection contexts or pass a `DestroyRef`.
- **HTTP Client**: Use `provideHttpClient(withInterceptors([...]))` and functional `HttpInterceptorFn` functions rather
  than class-based `HTTP_INTERCEPTORS`.
- **Reactive Forms**: Prefer strictly typed **Reactive Forms** (`FormGroup`, `FormControl`, `NonNullableFormBuilder`)
  over template-driven forms for non-trivial forms and validation.

## 5. TypeScript & Styling Quality

- **Strict TypeScript**: Respect `strict: true` and `strictTemplates: true` in `tsconfig.json`. Never use `any`; define
  explicit interfaces or types for all DTOs, API responses, and component inputs.
- **Component Encapsulation**: Keep component templates and styles focused and small. Extract complex template
  expressions into `computed()` signals rather than calling methods directly in HTML templates.
- **Accessibility**: Follow ARIA best practices, use semantic HTML elements, and ensure keyboard navigation works across
  custom interactive elements.

## 6. Testing Standards

Every new feature, service, or bug fix must include automated tests aligned with the project's test runner (
Karma/Jasmine, Jest, or Vitest):

- **Component Tests**: Use `TestBed` with `ComponentFixture`. Test rendered DOM output, signal inputs (
  `fixture.componentRef.setInput(...)`), and user interactions rather than private component internals.
- **HTTP & Service Tests**: Use `provideHttpClient()` paired with `provideHttpClientTesting()` and
  `HttpTestingController` to verify API requests, headers, and error handling.

## 7. Verification Workflow

Before completing any task, verify compilation, linting, and tests:

- **Typecheck & Build**: Run `npm run build` (or `npx ng build --configuration development`) to catch template type
  errors and compilation issues.
- **Lint**: Run `npm run lint` (or `npx ng lint`) to verify Angular ESLint rules.
- **Tests**: Run `npm test` (or `npx ng test --watch=false`) to ensure all unit tests pass.
