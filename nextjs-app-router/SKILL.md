---
name: nextjs-app-router
description: Use when building with Next.js 13+/14+ App Router — understanding Server Components vs Client Components, data fetching with async/await or fetch(), caching model (static/dynamic/revalidation), Server Actions, route handlers, streaming with Suspense, metadata, and when something renders on the server vs client. Covers the mental model shift from Pages Router.
---

# Next.js App Router

## The Mental Model Shift

App Router's core idea: **components are Server Components by default.** They run on the server, have direct access to databases and APIs, generate zero JavaScript for the browser, and cannot use state, effects, or browser APIs.

You opt individual components into the client with `"use client"`. Everything above a `"use client"` boundary in the tree is a Server Component; everything at and below it is a Client Component.

```
layout.tsx (Server)
  └── page.tsx (Server) — fetches data directly from DB
        ├── Header (Server) — reads session, no JS sent
        └── InteractiveCart (Client) — "use client"; has useState
              └── CartItem (Server? No — Client components can only use Client children)
```

**The key rule:** Server Components can render Client Components. Client Components **cannot** render Server Components (other than as `children` passed from a Server Component).

## Server Components — the Default

```tsx
// app/products/page.tsx — no "use client" needed
async function ProductsPage() {
  const products = await db.product.findMany();  // direct DB access — no API roundtrip
  return (
    <ul>
      {products.map(p => <ProductCard key={p.id} product={p} />)}
    </ul>
  );
}
```

Server Components:
- Async functions with `await` — first-class.
- No bundle cost: zero JS sent to browser.
- No state, no effects, no browser APIs, no event handlers.
- Can import server-only packages (DB clients, SDKs) without shipping them to clients.

## Client Components — Opt In

```tsx
"use client";
import { useState } from "react";

export function AddToCart({ productId }) {
  const [added, setAdded] = useState(false);
  return (
    <button onClick={() => { addToCart(productId); setAdded(true); }}>
      {added ? "Added!" : "Add to Cart"}
    </button>
  );
}
```

Mark `"use client"` at the boundary — the outermost component that needs interactivity. Prefer marking leaf components; don't mark layouts/pages unless they need client features.

## Data Fetching — Caching Model

`fetch()` in Server Components is extended by Next.js with caching:

```tsx
// Static: cached indefinitely (default for static builds)
const data = await fetch("https://api.example.com/items");

// Revalidate on a schedule
const data = await fetch("https://api.example.com/items", {
  next: { revalidate: 3600 }   // re-fetch at most every hour
});

// Always dynamic — never cached
const data = await fetch("https://api.example.com/user", {
  cache: "no-store"
});
```

**When is a route static vs dynamic?**
- Static: no dynamic functions (`cookies()`, `headers()`, `searchParams`), all fetches cached → rendered at build time.
- Dynamic: any dynamic function or `no-store` fetch → rendered at request time.

Prefer static where possible; it's edge-deployable and has zero server latency.

## Server Actions — Mutations from the Server

```tsx
// app/actions.ts
"use server";
export async function createPost(formData: FormData) {
  const title = formData.get("title") as string;
  await db.post.create({ data: { title } });
  revalidatePath("/posts");   // invalidate the cache for /posts
}

// app/posts/new/page.tsx (Server Component — no "use client" needed)
import { createPost } from "@/app/actions";
export default function NewPost() {
  return (
    <form action={createPost}>
      <input name="title" />
      <button type="submit">Create</button>
    </form>
  );
}
```

Server Actions run on the server. The form submits to the server; no client JS needed for this form. Progressive enhancement built in.

For optimistic updates, use `useOptimistic` from a Client Component that calls the action.

## Streaming with Suspense

Wrap slow parts in `<Suspense>` to stream the page progressively — fast parts arrive first:

```tsx
// page.tsx
import { Suspense } from "react";
import { ProductGrid } from "./ProductGrid";       // fast
import { RecommendedForYou } from "./Recommended"; // slow (personalized)

export default function Page() {
  return (
    <>
      <ProductGrid />                              {/* renders immediately */}
      <Suspense fallback={<Skeleton />}>
        <RecommendedForYou />                      {/* streams when ready */}
      </Suspense>
    </>
  );
}
```

`loading.tsx` is a file-based Suspense boundary for the entire route segment.

## Route Handlers vs Server Actions

| | Route Handler (`route.ts`) | Server Action (`"use server"`) |
|---|---|---|
| Called by | External clients, webhooks, mobile apps | Next.js UI components |
| HTTP method | Explicit (GET, POST…) | Always POST under the hood |
| Returns | `Response` object | Any serializable value |
| Use for | Public APIs, webhook endpoints | Form submissions, UI mutations |

Don't use Route Handlers for mutations triggered by your own UI — that's what Server Actions are for.

## Common Mistakes

| Mistake | Fix |
|---|---|
| `"use client"` on layout/page | Mark leaf interactive components instead |
| Fetching in Client Component with `useEffect` | Move to Server Component parent and pass as props |
| Waterfall: sequential awaits | `Promise.all` for parallel fetches |
| Over-invalidating with `revalidatePath("/")` | Invalidate the specific path that changed |
| DB client imported in Client Component | `server-only` package or move to Server Component |
| Server Action for external API | Use a Route Handler for public-facing endpoints |

**REQUIRED COMPANION:** react-internals covers RSC's relationship to the Fiber model. react-patterns covers component composition. react-performance covers when to split server/client for bundle size wins.
