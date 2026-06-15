---
name: nextjs-patterns
description: Use when architecting a Next.js App Router application — organizing routes and layouts, handling auth/middleware, structuring Server vs Client boundaries, setting up metadata and SEO, managing environment variables, monorepo patterns, and error/loading/not-found conventions. Covers structural decisions and file-system conventions.
---

# Next.js — Architecture Patterns

## File-System Conventions (App Router)

```
app/
  layout.tsx          ← root layout (html, body, providers)
  page.tsx            ← route: /
  loading.tsx         ← Suspense boundary for this segment
  error.tsx           ← error boundary ("use client" required)
  not-found.tsx       ← 404 for this segment
  (auth)/             ← route group: doesn't appear in URL
    login/page.tsx    ← route: /login
    register/page.tsx ← route: /register
  dashboard/
    layout.tsx        ← nested layout (sidebar, nav)
    page.tsx          ← route: /dashboard
    [id]/
      page.tsx        ← route: /dashboard/123
      @modal/         ← parallel route: rendered alongside main
        page.tsx
  api/
    webhook/route.ts  ← route handler: POST /api/webhook
```

**Route groups `(name)`**: group routes under a layout without affecting the URL. Use for auth vs no-auth sections, different layouts for marketing vs app.

**Parallel routes `@slot`**: render two independent routes in the same layout simultaneously (e.g., a modal alongside the main content, both deeply linkable).

## Auth Pattern — Middleware + Server Component

```ts
// middleware.ts (runs on the edge before every request)
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const token = request.cookies.get("token");
  const isProtected = request.nextUrl.pathname.startsWith("/dashboard");
  if (isProtected && !token) {
    return NextResponse.redirect(new URL("/login", request.url));
  }
  return NextResponse.next();
}
export const config = { matcher: ["/dashboard/:path*"] };
```

Middleware handles redirects. In Server Components, read the session for fine-grained authorization:

```tsx
// app/dashboard/page.tsx
import { getSession } from "@/lib/auth";
import { redirect } from "next/navigation";

export default async function Dashboard() {
  const session = await getSession();
  if (!session) redirect("/login");   // server-side redirect
  return <DashboardContent user={session.user} />;
}
```

Never trust session data from Client Components for authorization — always verify on the server.

## Provider Pattern — Minimal Client Wrapping

Client providers (React Query, auth context, theme) must live in a Client Component wrapper:

```tsx
// app/providers.tsx
"use client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
const queryClient = new QueryClient();

export function Providers({ children }) {
  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;
}

// app/layout.tsx (Server Component)
import { Providers } from "./providers";
export default function RootLayout({ children }) {
  return (
    <html><body>
      <Providers>{children}</Providers>
    </body></html>
  );
}
```

The children passed into Providers are Server Components — this is the correct pattern for keeping the tree mostly server-side.

## Environment Variables

```ts
// .env.local
NEXT_PUBLIC_SITE_URL=https://example.com   ← exposed to browser (prefix NEXT_PUBLIC_)
DATABASE_URL=postgres://...                ← server only, never in NEXT_PUBLIC_

// Validation at startup with t3-env / zod
import { createEnv } from "@t3-oss/env-nextjs";
export const env = createEnv({
  server: { DATABASE_URL: z.string().url() },
  client: { NEXT_PUBLIC_SITE_URL: z.string().url() },
  runtimeEnv: { DATABASE_URL: process.env.DATABASE_URL, NEXT_PUBLIC_SITE_URL: process.env.NEXT_PUBLIC_SITE_URL },
});
```

Validate envs at startup so the app crashes loudly with a useful message rather than silently at runtime.

## Metadata and SEO

```tsx
// Static metadata
export const metadata = {
  title: "My App",
  description: "...",
  openGraph: { images: ["/og.png"] },
};

// Dynamic metadata (per page)
export async function generateMetadata({ params }) {
  const product = await getProduct(params.id);
  return { title: product.name, description: product.description };
}
```

Metadata cascades from root layout down; child exports override parent fields. `generateMetadata` runs on the server — same data access as the page.

## Error Handling Architecture

```tsx
// app/error.tsx — catches errors in this route segment's children
"use client";   // required — must be a Client Component
export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Something went wrong</h2>
      <button onClick={reset}>Retry</button>
    </div>
  );
}

// app/global-error.tsx — catches errors in the root layout
"use client";
export default function GlobalError({ error, reset }) { ... }
```

`error.tsx` doesn't catch errors in the same layout — only in its children. For errors in the layout, use `global-error.tsx`.

## Project Structure (recommended)

```
app/               ← routing (thin: layouts, pages, route handlers)
src/
  components/      ← shared UI components
  lib/             ← utilities, configs, db clients
  hooks/           ← custom hooks (client-only)
  server/          ← server-only: db queries, auth, actions
  types/           ← shared TypeScript types
public/            ← static assets
```

Keep `app/` as thin as possible — it's routing glue. Business logic, data access, and components go in `src/`.

## Common Mistakes

| Mistake | Fix |
|---|---|
| DB client in Client Component | Move to `src/server/`, use `server-only` package |
| Missing `error.tsx` → unhandled error in prod | Add at every meaningful route boundary |
| One giant root layout | Nested layouts per section; route groups |
| Env vars without validation | `t3-env` or manual zod schema at startup |
| `useSearchParams` in Server Component | `searchParams` prop in `page.tsx` params |
| Redirect in Client Component | `redirect()` in Server Component or `router.push` in client |
