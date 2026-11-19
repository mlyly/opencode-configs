---
name: nextjs-shadcn
description: Frontend engineering conventions for Next.js App Router, React Server Components, shadcn/ui, Tailwind CSS, and TypeScript. Invoke when creating pages, layouts, Server Actions, forms, or UI components.
metadata:
  version: "1.0.0"
  domain: frontend
  triggers: Next.js, App Router, shadcn, Tailwind CSS, React Server Components, TypeScript, Zod
---

# Next.js & shadcn/ui Engineering Standards & Agent Guidelines

## 1. Environment & Project Discovery

Before writing or modifying code, inspect the project root to align with existing conventions:

- **Package Manager**: Check lockfiles (`pnpm-lock.yaml`, `package-lock.json`, `yarn.lock`, or `bun.lockb`) and always
  use the matching package manager (`pnpm`, `npm`, `yarn`, or `bun`).
- **Next.js & React Versions**: Check `package.json` for Next.js (15/16+), React (19+), and Tailwind CSS version (v3 vs.
  v4 CSS-first config) to ensure API compatibility (e.g., async `params`/`searchParams`/`cookies()` in Next.js 15+).
- **shadcn/ui Configuration**: Inspect `components.json` to verify path aliases (`@/components`, `@/lib/utils`), icon
  library (`lucide-react`), style preset, and Tailwind CSS setup.

## 2. Architecture & Directory Structure

Follow standard Next.js App Router conventions with clear separation between server and client boundaries:

- **App Router (`src/app` or `app`)**: Keep `page.tsx` and `layout.tsx` thin. Colocate route-specific components,
  loading skeletons (`loading.tsx`), and error boundaries (`error.tsx`) inside route segments.
- **UI Primitives (`components/ui`)**: Reserved exclusively for `shadcn/ui` components. Do not put domain-specific
  business logic inside `components/ui/*`.
- **Feature & Shared Components (`components/`)**: Organize reusable domain components by feature (e.g.,
  `components/dashboard/`, `components/auth/`).
- **Server Actions & Data Access (`lib/` or `server/`)**: Keep database queries, external API clients, and Next.js
  Server Actions isolated from UI components.

## 3. React Server Components (RSC) & Client Boundaries

- **Server-First Default**: Default to React Server Components (`page.tsx`, `layout.tsx`, and data-display components).
  Only add `'use client'` at the lowest possible leaf component when interactivity, browser APIs, or React hooks (
  `useState`, `useEffect`, `useOptimistic`) are required.
- **Async Request APIs**: Always `await` dynamic Next.js request APIs (`params`, `searchParams`, `cookies()`,
  `headers()`).
- **Data Fetching & Caching**: Fetch data directly in Server Components when possible. Never expose secret environment
  variables (`process.env.*` without `NEXT_PUBLIC_`) to Client Components.

## 4. shadcn/ui, Tailwind CSS & Forms

- **Adding shadcn Components**: Prefer installing official primitives via the CLI (`npx shadcn@latest add <component>`)
  rather than hand-coding Radix UI wrappers from scratch.
- **Class Merging**: Always use the `cn()` helper (`@/lib/utils`) when merging conditional Tailwind classes or accepting
  `className` props.
- **Styling & Theming**: Use semantic design tokens (`bg-background`, `text-foreground`, `text-muted-foreground`,
  `border-border`, `bg-primary`) instead of hardcoded colors (`bg-white`, `text-gray-900`) so dark mode and theme
  overrides work seamlessly.
- **Forms & Validation**: Build forms with **React Hook Form** + **Zod** schemas paired with `shadcn/ui` `<Form>` (or
  `<Field>`) components. Share the same Zod schema between client form validation and Server Action / Route Handler
  validation.
- **Accessibility**: Preserve Radix UI keyboard navigation, focus rings, and ARIA attributes. Always pair icon-only
  buttons with `aria-label` or `<span className="sr-only">`.

## 5. TypeScript & Code Quality

- **Strict Typing**: Avoid `any`. Infer types from Zod schemas (`z.infer<typeof schema>`), ORM models, or API responses.
- **Component Props**: Type component props explicitly using `type` or `interface`, extending
  `React.ComponentProps<"element">` when wrapping native HTML or shadcn primitives.
- **Images & Links**: Use `next/image` with explicit dimensions or `fill` + `sizes` for local/remote images, and
  `next/link` for all internal navigation.

## 6. Verification Workflow

Before completing any task, run the project's typecheck, lint, and build scripts:

- **Typecheck**: `pnpm tsc --noEmit` (or `npm run typecheck` / `npx tsc --noEmit`) to catch TypeScript errors across
  Server and Client boundaries.
- **Lint**: `pnpm lint` (or `npm run lint`) to verify ESLint and Next.js rules.
- **Build Check**: Run `pnpm build` (or `npm run build`) when modifying routing, Server Actions, or dynamic rendering
  configs to ensure production build success.
