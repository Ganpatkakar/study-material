# Frontend System Design: The RADIO Framework — Expanded Study Guide
## 55 Concepts Deep-Dive for Interview Preparation

**Source Video:** https://www.youtube.com/watch?v=MsKyeVbHRZs (LearnersBucket)  
**Base Structure:** RADIO Framework — Requirements & Rendering, Architecture, Data, Interfaces, Optimization  
**This Version:** Expanded with research, trade-offs, production examples, code patterns, and interview answers

---

# PART 1: R — REQUIREMENTS & RENDERING
> *How does the page reach the user?*

## 1.1 Server-Side Rendering (SSR)

**What:** Server generates full HTML for each request. Next.js: `getServerSideProps` (Pages Router) or dynamic Server Components (App Router) or `fetch` with `no-store`, `cookies()`, `headers()` forcing dynamic.

**How it works:**
```
Request → Server (auth + fetch data + render to HTML) → HTML + RSC Payload → Browser hydration
```
Next.js determines rendering per-route by inspecting data fetching code. Using `cookies()`, `headers()`, searchParams forces SSR.

**Pros:**
- Fresh data per request, great personalization
- Good SEO (crawler sees full HTML)
- Faster FCP than pure CSR for first visit

**Cons:**
- Server cost (compute per request), higher TTFB if APIs slow
- Harder caching, origin load

**When to use:** Dashboards, checkout, account, personalized SEO pages.

**Interview answer:** "I'd use SSR for personalized dashboards because data must be fresh and request-specific. I'd add server cache and CDN with `s-maxage=0, must-revalidate` for safety, and monitor TTFB."

**Production tip:** In Next.js App Router, SSR is not a flag but a consequence. If route reads cookies or uses `no-store`, it becomes dynamic automatically.

## 1.2 Client-Side Rendering (CSR)

**What:** Server sends minimal HTML shell + JS bundle. Browser builds UI.

**Pros:** Rich interactivity, fast navigations after load (SPA), all logic in browser.

**Cons:** Large JS hurts LCP, SEO needs extra work (prerender), blank loading state.

**When to use:** Figma, Notion, Linear, internal tools, apps where user stays long time after load.

**Metric impact:** CSR often has worse LCP (>2.5s) if bundle >300KB. Use code splitting.

## 1.3 Static Site Generation (SSG)

**What:** HTML generated at build time. Served from CDN forever until next deploy.

```
Build: Data → HTML → CDN
User: CDN → Instant HTML
```

Next.js: `generateStaticParams`, `export const dynamic = 'force-static'`

**Pros:** Fastest TTFB, cheap, infinitely cacheable, best SEO for static content.

**Cons:** Stale until redeploy.

**When to use:** Docs, blogs, marketing, landing pages, help centers.

## 1.4 Incremental Static Regeneration (ISR)

**What:** SSG with background revalidation. `revalidate = 60` means: serve cached HTML immediately, and after 60s trigger background regeneration.

```
Request → Cached? Yes fresh → serve
               → stale → serve stale + regenerate in background
```

**Pros:** CDN speed + freshness control, page-level regeneration.

**Cons:** Slight staleness, need on-demand revalidation strategy.

**When to use:** E-commerce product pages, news articles, catalog with 10k+ pages where full rebuild expensive.

**Next.js nuance:** `revalidateTag()` / `revalidatePath()` invalidates Next.js cache but CDN needs `s-maxage` + `stale-while-revalidate` headers. Otherwise CDN serves old copy until TTL.

## 1.5 Hybrid Rendering

Real apps mix strategies per route:
- Homepage → ISR (60s)
- Product pages → ISR (300s)
- Checkout → SSR
- Admin → CSR
- Docs → SSG

Modern frameworks support component-level mixing. This is the most mature answer: "Not SSR vs CSR, but which strategy for which part?"

## 1.6 React Server Components (RSC)

**Core mechanics:**
- Server Components (default in App Router): run only on server, can access DB/secrets directly, zero JS shipped.
- Client Components (`'use client'`): hydrate in browser for interactivity.

```tsx
// Server Component - can fetch directly
export default async function Page() {
  const posts = await db.posts.findMany() // no API needed
  return <PostList posts={posts} />
}
```

**Anti-pattern:** Adding `"use client"` at top of large tree just because one leaf needs onClick. Instead:
```
Server Layout
 ├─ Server Content
 └─ Client Button (only this is client)
```
Use children prop to keep server code out of client bundle: passing `<Cart />` (server) as children to `<Modal />` (client) works because children is serializable output, not code.

**Interview point:** RSC !== SSR. SSR renders client components to HTML on server (JS still ships). RSC never ships its JS.

## 1.7 Streaming SSR

**Problem:** Traditional SSR blocks entire response until slowest API finishes.
```
API A (100ms), API B (2s) → entire page waits 2s
```

**Solution:** React 18 `renderToPipeableStream` + Suspense. Server flushes shell immediately, streams sections as they resolve.

```tsx
<Suspense fallback={<ProductSkeleton />}>
  <ProductDetails /> {/* fast */}
</Suspense>
<Suspense fallback={<ReviewsSkeleton />}>
  <Reviews /> {/* slow - streams later via <script> injection */}
</Suspense>
```

Benefits: Better TTFB, better LCP (browser paints shell early), selective hydration prioritizes interacted sections.

**Pitfalls:** Too many boundaries = popcorn effect (content popping erratically). Place boundaries around independently loadable sections, not every component. Set abort timeout (e.g., 10s).

**CDN note:** Can you cache streamed response? Most CDNs cache final assembled HTML; for dynamic streaming, use `Transfer-Encoding: chunked` and ensure edge doesn't buffer.

## 1.8 Core Web Vitals

Google measures at 75th percentile of real users (field data, not lab).

| Metric | Meaning | Good | Needs Improvement | Poor |
|---|---|---|---|---|
| **LCP** | Largest content paint time | ≤2.5s | 2.5-4s | >4s |
| **INP** | Interaction responsiveness (replaced FID Mar 2024) | ≤200ms | 200-500ms | >500ms |
| **CLS** | Visual stability (layout shift score) | ≤0.1 | 0.1-0.25 | >0.25 |

**LCP** is usually hero image, product title, large block. Optimize: preload hero image, priority hints, compress, CDN.
**CLS:** Caused by images without dimensions, dynamic ads, fonts swapping. Fix: width/height attrs, `aspect-ratio`, font-display: optional, reserve space.
**INP:** Caused by long JS tasks blocking main thread after click. Fix: break tasks with `startTransition`, reduce JS, use web workers.

Interview: "I'd track LCP/INP/CLS in RUM, set performance budgets, fail CI if LCP >2.5s."

## 1.9 CDN and Edge Delivery

CDN caches static assets at edge PoPs close to user: JS, CSS, images, fonts, static HTML, ISR pages.

Edge functions (Cloudflare Workers, Vercel Edge, Lambda@Edge, Fastly Compute@Edge) run code at edge:
- redirects, auth checks, A/B testing (assign variant via cookie without origin hit)
- personalization within constraints
- cache key manipulation, tagging

**Strategies for dynamic content:**
- Short TTL (1-30s) for high-traffic pages tolerating staleness
- Edge Side Includes (ESI): `<esi:include src="/header"/>` cache shell long, fragment short
- Stale-while-revalidate: `Cache-Control: s-maxage=60, stale-while-revalidate=300`

**Edge A/B testing advantage:**
Traditional: User → CDN → Origin determines variant → uncacheable
Edge: User → Edge Worker assigns variant → cached variants served from edge

## 1.10 Service Workers and Caching

Service worker intercepts fetch events.

**Strategies:**
- **Cache First:** Check cache → if hit return, else network. Fast but stale. Good for versioned assets `/app.a1b2c3.js`
- **Network First:** Network → if success return, else cache. Fresher. Good for API.
- **Stale While Revalidate:** Return cache immediately + fetch update in background + update cache. Best for content where speed > absolute freshness.

```js
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(cached => {
      const fetchPromise = fetch(event.request).then(net => {
        caches.open('v1').then(c => c.put(event.request, net.clone()))
        return net
      })
      return cached || fetchPromise
    })
  )
})
```

## 1.11 Progressive Web Apps (PWA)

Requirements: HTTPS + Web Manifest (`manifest.json` with name, icons, display) + Service Worker.

Enables: installable, offline, background sync, push notifications (via Push API).

Use when you want app-like without native app store overhead. Trade-off: iOS PWA limitations (no push until recent).

---

# PART 2: A — ARCHITECTURE

## 2.1 Component-Based Architecture

Rule: **If component has >1 reason to change, split. If two components always change together, merge.**

Avoid 2000-line mega component, and also avoid component-per-div.

**Boundary heuristics:**
- Single responsibility (data vs presentation)
- Reusability
- Testability
- Team ownership

## 2.2 One-Way Data Flow

```
Parent (state) → props → Child
Child → callback/onEvent → Parent
```

Makes data flow predictable. Controlled components example:

```jsx
<input value={name} onChange={e => setName(e.target.value)} />
```

Uncontrolled for perf: `ref` + `defaultValue` avoids re-render per keystroke — used by React Hook Form.

## 2.3 Micro-frontends (MFE)

Inspired by microservices: independent apps owned by different teams, integrated at runtime or build time.

**Implementation:**
- Webpack Module Federation 5 (most popular): `exposes` and `remotes` dynamically load remoteEntry.js at runtime
- Single-spa, import maps, Web Components

**Pros:** Team autonomy, independent deploys, tech diversity, incremental migration, fault isolation, faster time-to-market for large orgs.

**Cons:** Complexity (routing, shared deps duplication, performance overhead — multiple bundles), inconsistent UX without strong design system, coordination overhead, versioning.

**When NOT to use:** <4 teams, small product. Use monorepo instead. Monolith is fine.

**Production:** Share dependencies via `shared: {react: {singleton: true}}` to avoid duplicate React.

## 2.4 Design Systems

**Structure:**
```
Design Tokens (primitive): colors, spacing, typography, radii, shadows, breakpoints
 ↓
Semantic tokens: bg-primary, text-muted, border-default
 ↓
Component library: Button, Input, Card (consumes tokens)
 ↓
Products
```

Tools: Figma Variables → Style Dictionary/Tokens Studio → CSS variables → Storybook as single source of truth in code world.

**Governance:** Treat as API. Use semver, avoid breaking changes, document migrations, run visual regression (Chromatic), a11y addon (axe-core).

**Token example:**
```css
:root {
  --color-primary-500: #6366f1;
  --space-4: 16px;
  --radius-md: 8px;
}
```

## 2.5 Styling Strategies (2025-26 landscape)

| Approach | Runtime Cost | SSR Friendly | Bundle Size | Best For |
|---|---|---|---|---|
| CSS Modules | None | Yes | Small, scoped | Predictable, no runtime |
| SCSS | None | Yes | Small | Legacy, variables/mixins |
| Tailwind (utility) | None (JIT) | Yes | 5-20KB | Velocity, consistency |
| CSS-in-JS (styled-components, Emotion) | Medium-High | Tricky | +12-30KB JS | Dynamic theming |
| Zero-runtime (vanilla-extract, Linaria) | None | Yes | Small CSS | Performance + type safety |

**Trend:** Ecosystem moving away from runtime CSS-in-JS toward Tailwind or build-time solutions. Reason: Core Web Vitals + RSC incompatibility — runtime CSS-in-JS uses `useContext` which RSC forbids → build errors/FOUC. styled-components in maintenance mode since Jan 2024.

**Recommendation:** Default to Tailwind v4 (CSS-first config) or CSS Modules + CSS variables for theming. Use vanilla-extract if extreme TS needs.

## 2.6 Monorepos

Structure:
```
/apps/web, /apps/mobile, /apps/admin
/packages/design-system, /packages/utils, /packages/api-client
```

**Tools:**
- **Turborepo:** Simple, zero-config caching, Rust-based fast task runner, best for <20 packages, frontend-heavy, Vercel-native.
- **Nx:** Advanced: generators, affected commands, module boundary rules, distributed caching. Better for 20+ packages, polyglot.

**Benefits:** Code sharing (components, types), atomic changes (one PR spans frontend+backend+shared), faster CI via affected detection, consistent tooling.

**Cost:** Larger repo, need for build caching strategy.

## 2.7 CI/CD for Frontend

Pipeline:
```
PR → Lint (Biome/ESLint) → Type check → Unit tests (Vitest) → Build → Visual regression (Chromatic) → Preview deploy (Vercel/Netlify) → E2E (Playwright) → Prod deploy
```

**Preview deployments:** Every PR gets URL like `pr-123.app.vercel.app` for manual QA, product review, stakeholder approval.

**Feature flags:** Decouple deploy from release.
```ts
if (flags.newCheckout) return NewCheckout else OldCheckout
```
Rollout: 1% → 10% → 100%, kill switch. Tools: LaunchDarkly, Unleash, Vercel flags. Flags that live forever become tech debt — clean up.

**Performance budgets in CI:** Lighthouse CI, bundle analyzer — fail if JS >300KB, LCP >2.5s, image >200KB.

## 2.8 Error Boundaries

Prevent one widget crash from taking down entire app.

```tsx
<ErrorBoundary fallback={<WidgetError />}>
  <Recommendations /> {/* third-party, may crash */}
</ErrorBoundary>
```

Granular placement: per widget, not just top-level. Log to Sentry/Datadog.

React 19: `ErrorBoundary` + `useErrorBoundary`. In Next.js App Router, `error.tsx` per segment.

## 2.9 Accessibility (a11y)

Should be architectural, not afterthought.

**Principles:**
- Semantic HTML: `<button>` not `<div onClick>`
- Keyboard: All interactive via Tab, Enter/Space, focus visible, focus management in modals (trap focus, return focus on close)
- ARIA only when semantic insufficient: `aria-label`, `aria-live="polite"` for dynamic updates
- Color contrast ≥4.5:1 for text
- Labels associated: `<label htmlFor>`

**Testing:** axe-core, Lighthouse a11y, Storybook a11y addon, manual keyboard test.

## 2.10 Frontend Security

**XSS:** Attacker injects script executed as JS. React escapes by default, but `dangerouslySetInnerHTML`, `v-html`, `innerHTML`, `eval` bypass. Mitigation: context-aware encoding, sanitize with DOMPurify, strict CSP.

**CSRF:** Tricks auth browser into unwanted request (e.g., `<img src="https://bank.com/transfer">`). Mitigation: SameSite=Lax/Strict cookies, CSRF token (synchronizer), custom header validation (`X-Requested-With`).

**CSP:** Header restricting script sources.
```
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-xyz'; object-src 'none'
```
Start with `Content-Security-Policy-Report-Only`, collect violations, then enforce. Add `Trusted Types` to prevent DOM XSS sinks.

**Token storage:**
- `localStorage/sessionStorage` readable by JS → XSS can steal → avoid for sensitive tokens
- `httpOnly, Secure, SameSite` cookie → JS cannot read → safer. Use Backend-for-Frontend (BFF) pattern: browser gets httpOnly cookie, server holds JWT.
- PKCE for OAuth: Authorization Code + PKCE (code_verifier) for SPAs/mobile, state param for CSRF, validate ID token via JWKS, iss, aud, nonce.

**OWASP Top 10 frontend lens:** Broken access control (frontend not boundary, server enforces), Injection/XSS, Security misconfig (CSP, CORS allow-list not `*` with credentials), Vulnerable deps.

---

# PART 3: COMPONENT RENDERING PATTERNS

## 3.1 Container & Presentational (now: Custom Hooks)

Old pattern: Container fetches, presentational renders. Modern: custom hook.

```tsx
function usePosts() { /* fetch logic */ }
function PostList({posts}) { /* pure UI */ }
```
Benefits: Testable, data source swappable (REST/GraphQL/mock).

## 3.2 Higher-Order Components (HOC)

```js
const Enhanced = withAuth(Component)
```
For cross-cutting: auth, logging. Less common now (hooks preferred). Problem: nesting hell, props collision, wrapper hell.

## 3.3 Provider Pattern

Context for subtree:
```tsx
<ThemeProvider value={theme}>
  <App />
</ThemeProvider>
```
Good for stable values: theme, locale, user. Bad for high-frequency changing values (every consumer re-renders). Split contexts, or use Zustand for frequent.

## 3.4 Compound Components

Cooperating components with shared state via context, but flexible composition.

```tsx
<Tabs defaultValue="profile">
  <Tabs.List><Tabs.Trigger value="profile">Profile</Tabs.Trigger></Tabs.List>
  <Tabs.Content value="profile">...</Tabs.Content>
</Tabs>
```
Use for tabs, accordion, menu, dialog. Avoid giant config object.

## 3.5 Polymorphic Components

Render as different element via `as` prop.

```tsx
<Button as="a" href="/pricing">Pricing</Button>
<Button as={NextLink} href="/pricing">Pricing</Button>
```
Design systems use generics to infer correct props for `as`.

---

# PART 4: D — DATA MODEL & STATE

Classify first: Local, Global client, Server, URL.

## 4.1 Local State

One component or small subtree: dropdown open, tab, input, modal.

`useState` for simple, `useReducer` when multiple values change together or transitions complex.

Don't lift unnecessarily.

## 4.2 Global State

Unrelated parts need same client data: theme, notifications, cart UI, preferences.

**Comparison:**

| Library | Bundle | Boilerplate | Best For |
|---|---|---|---|
| Zustand | ~1KB | Minimal | Greenfield, small-medium team, client state focus |
| Redux Toolkit | ~10KB+ | More | Existing Redux, strict traceability, RTK Query |
| Jotai/Recoil | ~2-3KB | Minimal | Atomic state |
| Context | 0 | Minimal | Stable, low-frequency |

Rule: "TanStack Query for server state, Zustand for client state". Don't put everything in global — increases blast radius.

## 4.3 Server State

Lives on server, can become stale, needs caching, deduping, background sync.

**TanStack Query** provides: caching, garbage collection, retries, refetch, optimistic, invalidation.

```
Component A,B,C → same queryKey → single network request, shared cache
```

Why not Redux for API data? Reinventing cache poorly: need to handle loading, error, retry, dedup, stale, GC manually.

**Decision matrix:**
- Server state (any fetched data) → TanStack Query / RTK Query / SWR
- Client state (UI prefs, auth flags, unsaved draft) → Zustand

## 4.4 Data Normalization

Without normalization:
```json
{ posts: [{author: {id:42, name:"Alice"}}, {author: {id:42, name:"Alice"}}] } // duplicate
```

Normalized:
```
users: {42: {id:42, name:"Alice"}}
posts: [{id:1, authorId:42}]
```
Update Alice once. Essential for large interconnected datasets (social feed, e-commerce). Tools: Normalizr, manual entity map.

## 4.5 Cookies vs localStorage vs IndexedDB

| Storage | Size | Accessible | Persists | Use For |
|---|---|---|---|---|
| Cookies | ~4KB each | JS (unless httpOnly) + auto sent with requests | Depends | Auth (httpOnly), session |
| localStorage | ~5-10MB | JS sync | Until cleared | Non-sensitive prefs, simple cache |
| sessionStorage | ~5MB | JS sync | Tab lifetime | Temporary tab state |
| IndexedDB | Large (50MB+) | JS async | Until cleared | Large datasets, offline docs, drafts, cache |

Think of IndexedDB as browser database. Use wrappers: idb, Dexie.

## 4.6 Optimistic Updates

UX: Update UI immediately, request in background, rollback on failure.

```
Previous state → Save snapshot → Apply optimistic → Send request
  → Success: keep
  → Failure: restore snapshot + toast
```

**TanStack pattern:**
```tsx
useMutation({
  onMutate: async (vars) => {
    await queryClient.cancelQueries({queryKey})
    const prev = queryClient.getQueryData(queryKey)
    queryClient.setQueryData(queryKey, optimistic)
    return {prev}
  },
  onError: (err, vars, ctx) => queryClient.setQueryData(queryKey, ctx.prev),
  onSettled: () => queryClient.invalidateQueries({queryKey})
})
```

Use for likes, follows, toggles, lightweight edits where failure rate low.

## 4.7 URL State

State that should be shareable/bookmarkable: search, filters, sort, pagination, category.

```
/products?category=shoes&sort=price_asc&page=2
```

Benefits: shareable, refresh persistence, reproducible bug reports, back/forward works.

Use library: `nuqs` (Next.js) or custom hook syncing `useSearchParams`.

## 4.8 Form State

Large controlled forms re-render per keystroke → slow.

**Solution:** React Hook Form uses uncontrolled inputs + refs, reduces re-renders. Zod as single source of truth.

```tsx
const schema = z.object({email: z.string().email(), age: z.number().min(18)})
type Form = z.infer<typeof schema>

const {register, handleSubmit} = useForm<Form>({resolver: zodResolver(schema)})
```

Client validation for UX, server validation for security (must validate again on backend).

Multi-step wizard: persist partial data in URL or IndexedDB, validate per step.

## 4.9 Real-Time Data

| Method | Direction | Protocol | Latency | Complexity | When |
|---|---|---|---|---|---|
| Short Polling | Client→Server periodic | HTTP | Near real-time (gap) | Simple | Status badges, small data, few seconds ok |
| Long Polling | Client→Server, server holds | HTTP | Real-time | Medium | Immediate sync without WS |
| SSE (Server-Sent Events) | Server→Client only | HTTP + EventSource | Real-time | Simple | Live feeds, scores, AI streaming, dashboards |
| WebSocket | Bidirectional, persistent | ws/wss (HTTP Upgrade) | Lowest | High | Chat, multiplayer, trading, collaborative |
| Webhooks | Server→Your URL POST | HTTP | Real-time | Medium | Cross-system events |

**Decision:** "If you need two-way low-latency → WebSocket. Server push only → SSE (simpler, auto-reconnect, text-only). Simple/low-volume → polling. Cross-system → webhooks."

**SSE advantage:** Native browser `EventSource`, auto-reconnect, works over HTTP/2, 90% of real-time cases.

**WebSocket challenges:** Reconnection, auth, message ordering, backpressure, connection lifecycle, scaling (need sticky or pub/sub like Redis).

---

# PART 5: I — INTERFACES & APIS

## 5.1 REST vs GraphQL vs tRPC

| Aspect | REST | GraphQL | tRPC |
|---|---|---|---|
| Use case | Public APIs, partner | Flexible data shape, multiple clients, BFF aggregation | Full-stack TS team, same monorepo |
| Over/under-fetching | Yes | Client chooses shape → solves | Client chooses via function args |
| Caching | HTTP caching easy | Harder (needs normalized cache) | Query cache via TanStack |
| Type safety | OpenAPI | SDL + codegen | End-to-end TS, zero codegen |
| Best for | Public, simple | Complex UI aggregating many services | Internal apps where you control both ends |

**Rule:** REST + TanStack Query is good default. Use GraphQL as BFF when aggregating Product+Inventory+Pricing+Reviews. Use tRPC for internal full-stack TS monorepo to eliminate API schema duplication.

**BFF pattern:**
```
Browser → BFF (one UI-shaped response) → Product service, Inventory, Pricing, Reviews, Recs
```
Benefits: fewer browser round trips, hide internal topology, frontend-specific shape.

## 5.2 Query Patterns

Distinguish:
- Queries (reads, cacheable) → `useQuery`
- Mutations (writes) → `useMutation`
- Infinite queries (pagination) → `useInfiniteQuery`

**Query keys hierarchical:**
```
['posts']
['posts', 'user', 42]
['posts', 'user', 42, 'page', 2]
```
Enables precise invalidation: `invalidateQueries(['posts', 'user', 42])` not entire cache.

## 5.3 Offset vs Cursor Pagination

| Criterion | Offset (`?page=3`) | Cursor (`?after=cursor123`) |
|---|---|---|
| Impl | Simple skip/take | Anchor to row (WHERE id > cursor) |
| Performance | Degrades at deep offset (OFFSET 10000 scans) | Constant O(limit) |
| Stability | Shifts when inserts/deletes between pages (duplicates/gaps) | Stable |
| Jump to page 50 | Easy | Hard (need walk) |
| Use | Admin tables, static datasets <10k, need totals | Feeds, chat, large changing datasets, infinite scroll |

**Interview:** "Stable dataset/admin table → offset. Live feed/chat → cursor. Transactional tables (orders) → cursor. Small ref tables → offset."

Cursor format: base64 JSON `{id, createdAt}` opaque to client.

## 5.4 Custom Hooks as API Abstraction

Don't let components know fetch details.

```tsx
// Bad: component knows URL
fetch("/api/posts")

// Good: abstraction
const {data, isLoading} = usePosts()
```
Component doesn't care if underlying is REST/GraphQL/WS/cache/BFF. Easy to replace, test.

## 5.5 Backend for Frontend (BFF) — deep dive

Without BFF: browser makes 5 calls, waterfall, exposes internal service topology.
With BFF: one call aggregated.

BFF can be Next.js Route Handler, or separate Node/Go service. It can also handle auth (httpOnly cookie → internal JWT), rate limiting, transformation.

## 5.6 Authentication

**Flows:**
- **Session:** Server holds session, browser gets httpOnly cookie. Simple, revocation easy.
- **JWT:** Stateless, signed claims verified via JWKS. Short-lived access (15m) + long-lived refresh in httpOnly cookie (7d).
- **OAuth 2.0 / OIDC:** Delegate to provider (Google). Flow: Authorization Code + PKCE for SPAs/mobile. PKCE prevents stolen code exchange, state prevents CSRF. Validate ID token signature, iss, aud, nonce.

**Best practice for new apps:** Use managed provider: Auth0, Clerk, Supergirl. Don't implement auth from scratch unless necessary.

**Token storage security:** Never localStorage for sensitive tokens (XSS steal). Use httpOnly cookie + BFF. For SPAs that must store in JS, consider short-lived and refresh rotation.

## 5.7 tRPC

End-to-end type safety for TS monorepo.

```
Server function → shared TS types → typed client call (no OpenAPI/codegen)
```
Great DX, eliminates entire bug class. Trade-off: only when you control both ends, not for public API consumed by third parties.

## 5.8 WebSocket Architecture

Don't create new WS per component mount → many connections.

**Bad:**
```
Component A mounts → WS
Component B mounts → WS
```

**Good:** Shared app-level connection + global event layer:
```
App → WS → Event bus → A,B,C subscribe to events they need
```
Handle: connection, auth, reconnection (exponential backoff), cleanup, message routing.

## 5.9 API Error Handling

Layered:
- **Network:** timeout, offline, DNS → offline indicator, retry with backoff, queue
- **HTTP:** 401 → login redirect, 403 → access denied, 404 → not found, 422 → validation (map to field), 500 → temporary, retry
- **Validation:** Map directly to field, not generic "Something went wrong".

```tsx
if (error.status === 422) setError(error.field, error.message)
```

Use `QueryErrorResetBoundary` + `ErrorBoundary` for query errors.

---

# PART 6: O — OPTIMIZATIONS & OBSERVABILITY

## 6.1 Code Splitting

Don't send entire app on first load.

- **Route-based:** Each route ~50-100KB, others lazy.
- **Component-based:** Heavy modals, editors, charts split.
- **Library splitting:** Vendor chunk stable.

```tsx
const Dashboard = lazy(() => import('./Dashboard'))
<Suspense fallback={<Spinner />}><Dashboard /></Suspense>
```

Next.js does automatic route splitting. Use `next/dynamic` with `ssr: false` for heavy client-only.

## 6.2 Tree Shaking

Remove unused exports. Requires ES modules, `sideEffects: false` in package.json.

```js
// Good - enables shaking
import { debounce } from 'lodash-es'
// Bad - imports entire lib
import _ from 'lodash'
```

Use `webpack-bundle-analyzer`, `rollup-plugin-visualizer`, Bundlephobia before installing package. Replace heavy: moment → date-fns/dayjs.

## 6.3 Lazy Loading

Load only when needed.

**Images:** `loading="lazy"` for below-fold, IntersectionObserver for custom.

**Components:**
```
Page loads → Chart not visible → don't download chart lib → user scrolls → download + render
```

## 6.4 Image Optimization

Images often largest weight. Modern formats: WebP 25-35% better than JPEG, AVIF 50% better.

Responsive:
```html
<img srcSet="small.webp 400w, medium.webp 800w, large.webp 1200w"
     sizes="(max-width: 768px) 100vw, 50vw"
     width="800" height="600" loading="lazy" decoding="async" />
```
Or `<picture>` with AVIF → WebP → JPEG fallback.

Always reserve dimensions to prevent CLS. Use Next.js `<Image>` which auto does optimization, formats `["image/avif","image/webp"]`, responsive sizes.

## 6.5 Memoization

`React.memo`, `useMemo`, `useCallback`.

**When to use:**
- >5ms calculation or prevents heavy child re-render
- Stable reference needed for memoized child or useEffect dep

**When NOT:** Everywhere. Premature optimization increases complexity. React Compiler (2024+) auto-memoizes; manual memo only where compiler can't.

```tsx
// Provider value - new object every render breaks memoization
const value = useMemo(() => ({user, logout}), [user, logout])
```

## 6.6 Virtualization

Render 10k rows → only 20-50 DOM nodes visible.

```
10,000 logical rows → 30 DOM rows recycled on scroll
```

Libraries: `react-window` (light), `react-virtualized` (heavier), `@tanstack/react-virtual`.

Essential for large tables, lists, chat history, file explorers. Without it, DOM nodes cause jank.

**Pitfall:** Unstable rowProps (new object each render) defeats memoization → every row re-renders on keystroke.

## 6.7 Vendor and Component Splitting

Third-party libs change less frequently than app code → separate chunk → cached longer between deploys.

Defer heavy: charting, rich text editors, code editors, maps. User who never uses them shouldn't download.

## 6.8 Debouncing

Wait until events stop before executing.

```
User types: H He Hel Hell Hello → wait 300ms → API request
```

Use for search, autocomplete. Keep function stable (not recreated per render) via `useCallback` or `useMemo`.

## 6.9 Throttling

Limit execution frequency: at most once per interval.

```
Scroll events many → execute at most every 100ms
```

Use for scroll, resize, mousemove.

## 6.10 React Transitions (Concurrent Features)

Mark updates as non-urgent.

```tsx
const [isPending, startTransition] = useTransition()
const deferredQuery = useDeferredValue(query)

// Urgent: input
// Non-urgent: filtering 10k results
startTransition(() => setFilteredResults(filter(query)))
```

Allows urgent interactions remain responsive while expensive UI updates in background. Good for search filtering, tab switching.

## 6.11 Observability

Three pillars:

**1. Error Tracking (Sentry):**
- Captures exceptions, stack traces, breadcrumbs (events before error), release, environment, user.
- Upload source maps in CI (never expose to client) → minified `line 1 col 47829` → maps to `checkout-flow.tsx:42`
- Group errors by boundary, alert.

```ts
Sentry.init({dsn, integrations: [browserTracingIntegration(), replayIntegration()], tracesSampleRate: 0.1, replaysOnErrorSampleRate: 1.0})
```

**2. Session Replay:**
Shows what user did before error: opened checkout → changed address → clicked payment → freeze → error. Dramatically reduces debugging time. Privacy: maskAllText, maskAllInputs, blockAllMedia, disable network bodies for sensitive pages.

**3. RUM (Real User Monitoring):**
Lab Lighthouse might show LCP 1.4s but real users on slow devices/networks experience 4.2s. RUM collects field data from actual users. Essential for understanding real-world performance. Tools: Sentry performance, Cloudflare Web Analytics, Vercel Analytics.

## 6.12 Performance Budgets

Enforce continuously in CI:

```
Max JS bundle: 300KB (initial <150KB)
Max LCP: 2.5s
Max image: 200KB
Max third-party: 100KB
```

CI fails if PR exceeds. Prevents gradual degradation:
```
Dependency added +30KB, another +50KB, year later app noticeably slower → budgets make visible immediately
```

Tools: Lighthouse CI, bundle analyzer, `size-limit`.

---

# PART 7: RADIO INTERVIEW CHECKLIST (Memorize)

**Q1: How does page reach user?** Discuss SSR/CSR/SSG/ISR/hybrid/streaming/CDN/Web Vitals. Ask: SEO? Freshness? Personalized?

**Q2: How are components structured?** Boundaries, container/presentational, provider, compound, polymorphic, HOCs, design systems.

**Q3: Where does state live?** Local (useState/useReducer), Global client (Zustand/Redux), Server (TanStack Query), URL (nuqs). Classify first.

**Q4: How does client communicate with backend?** REST/GraphQL/tRPC/BFF/WS/SSE/auth/pagination/error handling. Ask: what data does UI need, how often changes, how should API expose?

**Q5: What actually ships to browser?** Code splitting, tree shaking, lazy, images, virtualization, memo, debounce/throttle, transitions.

**Q6: How do we know when it breaks?** Error tracking, source maps, session replay, RUM, Core Web Vitals, perf budgets, CI.

Walk through in order — prevents random tech jumping. Every decision must follow from requirement + trade-off.

---

# PART 8: WORKED EXAMPLE — E-Commerce Product Page (Expanded)

**Requirement:** SEO, fast initial, product info, inventory, pricing, reviews, personalized recs, cart.

**R:** Product page → ISR 300s (SEO + cache), metadata server-rendered, reviews streamed via Suspense, cart client-side, recs lazy-loaded below fold.

**A:**
```
ProductPage
 ├─ ProductInfo (Server)
 ├─ ProductGallery (Client for zoom)
 ├─ ProductOptions (Local state: selected size)
 ├─ AddToCart (Global cart + optimistic)
 ├─ Reviews (Server + streamed, cursor pagination)
 └─ Recommendations (Lazy + virtualized)
```
Design system: Button, Rating, Price. Compound for selector.

**D:**
- Selected size → local
- Cart → global (Zustand + persist to localStorage)
- Product, reviews → server (TanStack Query)
- Sort/filter reviews → URL `/product/123?sort=latest`

**I:**
```
GET /products/:id
GET /products/:id/reviews?after=cursor
POST /cart
GET /recommendations
```
BFF aggregates Product + Inventory + Pricing + Reviews + Recs into one UI-shaped response.

**O:**
Critical path: HTML → hero image (preload, WebP/AVIF, width/height) → core interaction (Add to Cart).
Lazy: recs, reviews below fold, chart lib.
Optimizations: responsive images, CDN (s-maxage=300, stale-while-revalidate=600), code split, virtualization for 1000+ reviews, debounce search.
Monitor: LCP, CLS, INP, bundle, API latency, Sentry errors, RUM.

**Trade-offs table:**

| Decision | Alternative | Why chosen |
|---|---|---|
| ISR over SSR | SSR fresher | Product changes not per-second, ISR cheaper/faster |
| Cursor over offset for reviews | Offset jump to page | Reviews live, inserts cause shift |
| Zustand over Redux | Redux more structure | Cart simple, need minimal bundle |
| BFF over direct | Direct simpler | Need aggregate 5 services |

---

# PART 9: COMMON INTERVIEW MISTAKES

1. **Tech before requirements:** "I'll use Redux, GraphQL, WS, MFE" → Better: "Let's determine data type, scale, team, interaction."
2. **Everything global:** Dropdown open is not global.
3. **Server state as client state:** API data is not global app state → use TanStack Query.
4. **WebSockets everywhere:** If 30s delay ok, polling enough. WS adds operational complexity.
5. **MFE because large:** Scale alone insufficient. MFE attractive when independent teams/deploys justify complexity.
6. **Premature optimization:** Don't useMemo everywhere. Profile then optimize bottlenecks.
7. **Ignoring a11y/security:** Architectural requirements, not checklist at end.
8. **Single Suspense boundary:** Wrapping entire page defeats streaming — each independently loadable section own boundary.
9. **Unstable callbacks breaking virtualization:** New object per render → all virtual rows re-render.

---

# PART 10: MENTAL MODEL

```
                  FRONTEND SYSTEM DESIGN
                           |
                    ┌──────┴──────┐
                    │    RADIO    │
                    └──────┬──────┘
                           |
       ┌───────────┬───────┼────────┬────────────┐
       ↓           ↓       ↓        ↓            ↓
 Requirements  Architecture Data  Interfaces Optimization
       |           |       |        |            |
   Rendering    Components State    APIs       Performance
   SSR/CSR      Patterns   Local    REST       Bundles
   SSG/ISR      Design     Global   GraphQL    Images
   Streaming    Systems    Server   BFF        Lists
   CDN          Security   URL      WebSocket  Memoization
   Web Vitals   a11y       Forms    Auth       Observability
```

**Sequence under pressure:**
1. How does page reach user? (Rendering, CDN, Web Vitals)
2. How is frontend structured? (Components, design systems)
3. Where does each piece of state live? (Local/Global/Server/URL)
4. How does frontend talk to backend? (REST/GraphQL/BFF/WS)
5. What ships to browser? (Bundles, images, virtualization)
6. How measure failures? (Sentry, Replay, RUM, budgets)

**Goal is not mentioning max technologies, but showing every decision follows requirement + understood trade-off.**

---

## Sources & Further Reading

This expanded guide synthesizes research from official docs and engineering blogs:

- Next.js Docs: Server/Client Boundary, Rendering, Caching, Streaming
- React Docs: Server Components, Suspense, Transitions
- Google Web Vitals: LCP ≤2.5s, INP ≤200ms, CLS ≤0.1 at 75th percentile
- CDN/Edge: Cloudflare Workers, Vercel Edge, Lambda@Edge, ESI, short TTL, stale-while-revalidate
- Micro-frontends: Module Federation, team autonomy, fault isolation, operational overhead
- Design Systems: Tokens → semantic → components, Storybook as source of truth, Chromatic
- Styling: Tailwind vs CSS Modules vs runtime CSS-in-JS trade-offs, vanilla-extract, RSC incompatibility
- Monorepo: Turborepo vs Nx, atomic changes, affected detection
- CI/CD: Preview deployments, feature flags lifecycle, performance budgets
- Security: OWASP XSS/CSRF/CSP, httpOnly cookies, PKCE, BFF auth pattern
- State: TanStack Query for server state, Zustand for client state, normalization, optimistic triad (onMutate/onError/onSettled)
- Real-time: Polling vs SSE vs WebSocket vs Webhooks decision framework
- API: REST vs GraphQL vs tRPC vs BFF trade-offs, cursor vs offset pagination
- Performance: Code splitting (route/component/lib), tree shaking sideEffects:false, lazy loading, WebP/AVIF responsive images, virtualization react-window, debounce/throttle stable refs, useTransition/useDeferredValue
- Observability: Sentry error tracking, source maps upload in CI, session replay masking, RUM field data

*Next steps: Want this turned into flashcards, diagrams, or a 30-day interview prep plan with mock questions per module?*
