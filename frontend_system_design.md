# Frontend System Design: The RADIO Framework
## A Practical Guide to Requirements, Architecture, Data, Interfaces, and Optimization

**Source Video:** https://www.youtube.com/watch?v=MsKyeVbHRZs (LearnersBucket - 55 Frontend System Design Concepts)  
**Source Content:** ChatGPT transcript provided by you  
**Purpose:** Standalone tutorial / interview guide, not just a summary

---

> Frontend system design interviews are not primarily about knowing every React API or being able to name the latest frontend library.
>
> They are about demonstrating that you can take an ambiguous product requirement and systematically turn it into a scalable, maintainable, performant, and accessible frontend architecture.

A useful way to structure almost any frontend system design problem is the **RADIO framework**:

- **R — Requirements & Rendering**
- **A — Architecture**
- **D — Data Model & State**
- **I — Interfaces & APIs**
- **O — Optimizations & Observability**

Think of RADIO as a checklist. When an interviewer asks you to design a news feed, dashboard, e-commerce site, collaborative editor, chat application, or social network, walk through these five areas rather than jumping immediately into components and APIs.

---

## 1. R — Requirements & Rendering

The first question is:

> **How does this page reach the user?**

This decision affects:

- SEO
- initial page-load performance
- server cost
- caching
- personalization
- data freshness
- infrastructure complexity

There is no universally correct rendering strategy. The correct answer depends on the requirements.

### 1.1 Server-Side Rendering — SSR

With SSR, the server generates HTML for each request.

```
Browser
   |
   | HTTP request
   v
Server
   |
   | Fetch data
   | Render HTML
   v
Browser
```

For example:

```
GET /dashboard

        ↓

Server
  ├── authenticate user
  ├── fetch user-specific data
  ├── render HTML
  └── return response
```

#### Advantages

- Good SEO
- Fresh data on every request
- Easy to personalize
- Faster initial content visibility than a pure CSR application

#### Disadvantages

- Server computation for requests
- Higher infrastructure cost
- More complicated caching
- Potentially slower if the server or downstream APIs are slow

#### Good use cases

Use SSR for pages such as:

- personalized dashboards
- checkout
- account pages
- user-specific pages
- SEO-sensitive pages requiring fresh data

#### Interview answer

> "I'd use SSR for this page because the content is personalized and needs to be indexed or generated with request-specific data. I'd then use CDN or server-side caching where possible to reduce repeated computation."

---

### 1.2 Client-Side Rendering — CSR

With CSR, the server initially sends a relatively small HTML shell and JavaScript. The browser downloads the JavaScript and builds the UI.

```
Browser
   |
   | Request
   v
Server
   |
   | HTML shell + JS
   v
Browser
   |
   | Execute JS
   v
Application UI
```

This is the traditional architecture for many single-page applications.

#### Advantages

- Rich interactivity
- Excellent navigation after the application loads
- Most application logic can run in the browser
- Good fit for long-lived sessions

#### Disadvantages

- Large JavaScript bundles can hurt initial load
- SEO can be more difficult
- Users may initially see a loading state
- More work happens on the client

#### Good use cases

CSR is particularly suitable for highly interactive applications such as:

- Figma-like applications
- Notion-like applications
- Linear-like applications
- internal dashboards
- applications where SEO is not important

> Does the user spend a long time inside an interactive application? If yes, CSR may make sense.

---

### 1.3 Static Site Generation — SSG

With SSG, pages are generated ahead of time during the build.

```
Build time

Data
 ↓
Build process
 ↓
HTML
 ↓
CDN

User
 ↓
CDN
 ↓
HTML
```

There is no server rendering work for every request.

#### Advantages

- Extremely fast
- Excellent CDN caching
- Low server cost
- Great SEO
- Highly reliable

#### Disadvantage

The major trade-off is freshness. If your product information changes after the site is built, the generated page remains unchanged until the next deployment.

#### Good use cases

- documentation
- blogs
- marketing pages
- product documentation
- static landing pages

---

### 1.4 Incremental Static Regeneration — ISR

ISR combines the benefits of static generation with controlled revalidation.

Instead of rebuilding the entire site whenever content changes, individual pages can be regenerated.

Conceptually:

```
Request
   |
   v
Cached HTML
   |
   +---- fresh → return immediately
   |
   +---- stale → return cached page
                 +
                 trigger regeneration
```

This gives you:

- CDN performance
- relatively fresh content
- lower server computation
- page-level regeneration

#### Good use cases

- e-commerce product pages
- news articles
- large content sites
- catalogs

For example, a product page doesn't necessarily need to be regenerated every time somebody visits it.

> "I'd use ISR because the product data changes periodically, but I don't need to generate the page on every request."

---

### 1.5 Hybrid Rendering

Real applications rarely use one rendering strategy everywhere.

A single application might use:

```
Homepage       → SSR / SSG
Product pages  → ISR
Checkout       → SSR
Admin panel    → CSR
Live cart      → CSR
Documentation  → SSG
```

This is often the most realistic answer in a system design interview.

Instead of asking:

> "Should the application use SSR or CSR?"

Ask:

> "Which rendering strategy is appropriate for each part of the application?"

Modern frameworks can also mix server and client rendering at the component level.

---

### 1.6 React Server Components

React Server Components allow components to execute on the server and avoid sending their JavaScript to the browser.

Conceptually:

```
Server Component
       |
       | Server execution
       v
Serialized component tree
       |
       v
Browser
```

Interactive components can be explicitly marked as client components. In Next.js, this is commonly represented with:

```js
"use client";
```

A common mistake is putting `"use client"` at the top of a large component tree merely because one small child needs interactivity.

Prefer:

```
Server Component
   |
   ├── Server Component
   |
   └── Client Component
```

Rather than:

```
Client Component
   |
   ├── Everything becomes client-side
   ├── Everything becomes client-side
   └── Everything becomes client-side
```

Keep interactivity as close to the interactive component as possible.

---

### 1.7 Streaming SSR

Traditional SSR can suffer from a bottleneck:

```
API A ────────┐
API B ────────┤
API C ────────────────────┐
API D ────────┘           |
                          ↓
                    Render page
                          ↓
                    Send response
```

If one API is slow, the entire response can be delayed.

Streaming allows the server to send parts of the page as they become available.

```
HTML shell
   ↓
Header
   ↓
Main content
   ↓
Fast section
   ↓
Slow section
```

React's `Suspense` is commonly used to define these boundaries.

This improves perceived performance because users don't have to wait for the slowest piece before seeing anything.

---

### 1.8 Core Web Vitals

A strong frontend system design answer should include performance metrics.

Three important metrics are:

#### LCP — Largest Contentful Paint

Measures how quickly the main content becomes visible.

Target: `< 2.5 seconds`

#### CLS — Cumulative Layout Shift

Measures unexpected movement of page content.

For example:

```
Initial page:

[Article]
[Image loading...]

After image loads:

[Article moves down]
```

Reserve space for images and dynamic content to reduce layout shifts.

#### INP — Interaction to Next Paint

Measures how responsive the application feels after user interactions.

A slow click handler, expensive rendering operation, or long JavaScript task can make INP worse.

> "I'd monitor LCP for loading performance, CLS for visual stability, and INP for interaction responsiveness."

---

### 1.9 CDN and Edge Delivery

A CDN places cached resources closer to users geographically.

Without a CDN:

```
User in India
     |
     |-------------------->
     |
Origin server in US
```

With a CDN:

```
User in India
     |
     v
Indian edge node
     |
     v
Cached content
```

CDNs can cache:

- JavaScript
- CSS
- images
- fonts
- static HTML
- other static assets

Edge functions can also execute lightweight logic closer to users.

Examples include:

- redirects
- authentication checks
- A/B testing
- personalization decisions

However, edge runtimes often have more constraints than a traditional server runtime.

---

### 1.10 Service Workers and Caching

A service worker runs in the background and can intercept network requests.

Common caching strategies include:

#### Cache First

```
Cache
 ↓
Available?
 ├── yes → return cache
 └── no  → network
```

Fast, but potentially stale.

#### Network First

```
Network
 ↓
Success?
 ├── yes → return network
 └── no  → cache
```

Fresher, but dependent on network availability.

#### Stale While Revalidate

```
Return cached response
        +
fetch updated response
        +
update cache
```

This is often an excellent strategy for content where immediate display matters but freshness is still important.

---

### 1.11 Progressive Web Apps

A PWA generally combines:

- HTTPS
- a web manifest
- service workers

This can enable capabilities such as:

- installation
- offline behavior
- caching
- push notifications

PWAs are especially useful when you want app-like behavior without building a fully separate native application.

---

## 2. A — Architecture

Once you know how the page reaches the user, ask:

> **How should the application be structured so it remains maintainable as the codebase and team grow?**

### 2.1 Component-Based Architecture

Modern frontend applications are composed from reusable components.

The challenge is deciding the right component boundaries.

A useful rule is:

> **If a component has more than one reason to change, consider splitting it.**

Conversely:

> **If two components always change together, consider keeping them together.**

Avoid both extremes:

```
Bad:
One giant 2,000-line component

Also bad:
Every <div> becomes a component
```

The goal is meaningful boundaries.

---

### 2.2 One-Way Data Flow

React follows a predominantly one-way data flow model.

```
Parent
  |
  | props
  v
Child
  |
  | callback/event
  v
Parent
```

This makes it easier to understand:

- where state lives
- who owns state
- who can modify it
- why a component rendered

Controlled components are a common example:

```jsx
<input
  value={name}
  onChange={event => setName(event.target.value)}
/>
```

---

### 2.3 Micro-frontends

Micro-frontends apply ideas from microservices to frontend applications.

Instead of one giant frontend being owned by one team:

```
Application
 ├── Search team
 ├── Checkout team
 ├── Profile team
 └── Recommendations team
```

Each team can potentially develop and deploy its portion independently.

Technologies such as Module Federation can help with runtime composition.

#### When to use micro-frontends

Consider them when:

- many teams own different business domains
- independent deployments are important
- organizational boundaries justify the complexity

#### When not to use them

For a small team, micro-frontends can create unnecessary:

- infrastructure complexity
- duplicated dependencies
- deployment complexity
- communication overhead

In an interview, don't recommend micro-frontends merely because the application is "large." Explain the organizational problem they solve.

---

### 2.4 Design Systems

A design system provides a shared visual language.

Typical design tokens include:

```
Colors
Spacing
Typography
Border radius
Shadows
Breakpoints
```

Then shared components consume those tokens:

```
Design tokens
      ↓
Component library
      ↓
Applications
```

A design system prevents different teams from independently creating slightly different versions of the same button, modal, input, etc.

Tools such as Storybook are commonly used to document and develop components.

Treat a shared component library like an API:

- avoid unnecessary breaking changes
- version carefully
- document migrations
- maintain backward compatibility when practical

---

### 2.5 Styling Strategies

Common styling approaches include:

#### CSS Modules

Automatically scoped class names. Good default when you want:

- predictable CSS
- no runtime styling cost
- local component styles

#### SCSS

Adds features such as:

- variables
- nesting
- mixins

Be careful with deeply nested selectors.

#### CSS-in-JS

Styles can be colocated with component logic and dynamically generated. The trade-off can include runtime overhead depending on the library and implementation.

#### Utility-first CSS

Frameworks such as Tailwind use utility classes directly in markup. The trade-off is often more verbose markup in exchange for consistent, colocated styling.

The important interview point is not choosing a universally "best" solution. Explain the trade-offs.

---

### 2.6 Monorepos

A monorepo keeps multiple related projects in one repository.

For example:

```
/apps
  /web
  /mobile

/packages
  /design-system
  /utils
  /api-client
  /config
```

Tools such as Turborepo or Nx can manage builds and caching.

One major advantage is atomic changes.

For example:

```
Update shared Button API
        ↓
Update Button
        ↓
Update web application
        ↓
Update mobile application
        ↓
One commit
```

This avoids version drift between independently maintained repositories.

---

### 2.7 CI/CD

A production frontend pipeline might look like:

```
Pull Request
     ↓
Lint
     ↓
Type check
     ↓
Unit tests
     ↓
Build
     ↓
Visual regression
     ↓
Preview deployment
     ↓
Smoke tests
     ↓
Production deployment
```

Preview deployments are particularly useful because reviewers can inspect a real deployed version of the application.

Feature flags allow code to be deployed without immediately exposing it to users.

---

### 2.8 Error Boundaries

A frontend application should isolate failures.

Imagine a third-party recommendations widget crashes.

You don't want:

```
Recommendations crash
        ↓
Entire application crashes
        ↓
Checkout unavailable
```

Instead:

```
Application
 ├── Header
 ├── Product
 ├── Checkout
 └── Recommendations
          ↓
       Error Boundary
          ↓
       Fallback UI
```

Error boundaries should be appropriately granular. Also send errors to an error-monitoring system so they aren't silently swallowed.

---

### 2.9 Accessibility

Accessibility should be part of architecture rather than an afterthought.

Prefer:

```html
<button>Save</button>
```

Over:

```html
<div onclick="save()">Save</div>
```

Use semantic HTML whenever possible. For dynamic content, ARIA live regions can notify assistive technologies about changes.

Also consider:

- keyboard navigation
- focus management
- labels
- color contrast
- screen reader behavior
- accessible error messages

Automated tools can help catch issues, but automated testing should be combined with manual keyboard testing.

---

### 2.10 Frontend Security

Three concepts should be ready for a system design interview:

#### XSS — Cross-Site Scripting

Cross-Site Scripting occurs when attacker-controlled content is executed as JavaScript.

React escapes strings by default, but APIs such as `dangerouslySetInnerHTML` can bypass normal escaping.

Don't insert untrusted HTML without appropriate sanitization.

#### CSRF — Cross-Site Request Forgery

Tricks an authenticated browser into making an unintended request.

Common mitigations include:

- SameSite cookies
- CSRF tokens
- appropriate request validation

#### CSP — Content Security Policy

Restricts which scripts and resources the browser is allowed to load. It provides an additional defense layer if an XSS vulnerability exists.

#### Authentication Token Storage

Sensitive authentication tokens should generally not be placed in `localStorage`.

If JavaScript can read a token, an XSS vulnerability can potentially steal it.

HTTP-only cookies prevent JavaScript from directly reading the cookie.

A common architecture is therefore:

```
Browser
   |
HTTP-only secure cookie
   |
Backend
```

---

## 3. Component Rendering Patterns

These patterns demonstrate that you understand component architecture beyond simply writing JSX.

### 3.1 Container and Presentational Components

Separate data logic from presentation.

```
Container
 ├── Fetch data
 ├── Manage state
 └── Handle side effects
          |
          | props
          v
Presentation
 └── Render UI
```

The presentation component can then be tested independently.

For example:

```jsx
<UserList users={users} />
```

Doesn't need to know whether `users` came from:

- REST
- GraphQL
- local storage
- a mock
- WebSocket data

Modern React applications often use custom hooks rather than literally creating container components, but the architectural principle remains valuable:

> Keep data logic separate from visual logic.

---

### 3.2 Higher-Order Components

A Higher-Order Component is a function that takes a component and returns an enhanced component.

Conceptually:

```js
const EnhancedComponent = withAuth(Component);
```

It can be used for cross-cutting concerns such as:

- authentication
- permissions
- error handling
- logging

HOCs are less common in modern React than hooks and composition, but they remain important to understand because they exist in many mature codebases.

Avoid excessive HOC nesting because it can make component trees difficult to understand.

---

### 3.3 Provider Pattern

React Context can make information available to an entire subtree.

```
ThemeProvider
      |
      ├── Header
      ├── Sidebar
      └── Content
```

This avoids passing the theme through every component.

Good candidates include:

- theme
- authenticated user
- localization
- relatively stable configuration

Be careful with frequently changing context values because consumers can rerender when the provider value changes.

Split contexts when appropriate. For high-frequency global state, a dedicated state-management solution may be more appropriate.

---

### 3.4 Compound Components

Compound components allow related components to cooperate while giving the consumer control over composition.

For example:

```jsx
<Tabs>
  <Tabs.List>
    <Tabs.Tab>Profile</Tabs.Tab>
    <Tabs.Tab>Settings</Tabs.Tab>
  </Tabs.List>

  <Tabs.Panel>...</Tabs.Panel>
</Tabs>
```

The parent can manage shared state while the child components access that state through context.

This is useful for:

- tabs
- accordions
- menus
- dialogs
- dropdowns

The benefit is flexibility without requiring a giant configuration object.

---

### 3.5 Polymorphic Components

A polymorphic component can render different underlying elements while preserving a common API.

For example, `<Button />` could render as `<button>` or `<a>` or a framework-specific link component.

Conceptually:

```jsx
<Button as="a" href="/pricing">
  Pricing
</Button>
```

TypeScript generics can be used to ensure that the appropriate props are accepted for the selected element.

This is especially useful for design systems.

---

## 4. D — Data Model & State Management

One of the most important questions in frontend architecture is:

> **Where should this piece of state live?**

Don't start by asking:

> "Should I use Redux or Zustand?"

First classify the data.

A useful model is:

```
Local state
     ↓
Global client state
     ↓
Server state
     ↓
URL state
```

### 4.1 Local State

Use local state for information that belongs to one component or small component subtree.

Examples:

- dropdown open/closed
- active tab
- input value
- modal visibility
- temporary UI state

Use `useState()` for simple state. Use `useReducer()` when several state values change together or state transitions are more complex.

Don't automatically lift state to the top of the application. Lift state only when multiple components genuinely need it.

---

### 4.2 Global State

Global state is appropriate when unrelated parts of the application need access to the same client-side data.

Examples:

- theme
- notifications
- shopping cart UI state
- application preferences

Libraries include:

- Zustand
- Redux Toolkit
- MobX

Zustand provides a lightweight store-based approach. Redux Toolkit is useful when strong traceability, explicit actions, predictable state transitions, and mature developer tooling are important.

> Don't put everything in global state. Global state increases the number of components affected by changes and makes ownership harder to reason about.

---

### 4.3 Server State

Server state is fundamentally different from client state.

It:

- lives on a server
- can become stale
- must be fetched
- may be shared by multiple components
- needs caching
- may need background synchronization

This is where TanStack Query is useful.

Instead of manually managing `loading`, `error`, `data`, `retry`, `cache`, `refetch`, `deduplication`, `stale data`, TanStack Query provides abstractions.

For example:

```
Component A ─┐
Component B ─┼──→ Query Cache ───→ API
Component C ─┘
```

If several components request the same query, they can share cached data and avoid unnecessary duplicate requests.

> "I would treat API data as server state and use TanStack Query rather than putting it into a global client-state store."

---

### 4.4 Data Normalization

Suppose several posts contain the same user:

```json
{
  "post": {
    "author": {
      "id": 42,
      "name": "Alice"
    }
  }
}
```

If this user appears in 100 cached responses, you may have 100 copies of Alice's data.

Normalization instead stores:

```
users
  42 → Alice

posts
  1 → authorId: 42
  2 → authorId: 42
```

Now changing Alice's name only requires changing one entity. This becomes particularly valuable in applications with large, interconnected datasets.

---

### 4.5 Cookies vs localStorage vs IndexedDB

#### Cookies

Useful for:

- authentication
- session identifiers
- HTTP-only sensitive data

Cookies automatically accompany HTTP requests under appropriate conditions.

#### localStorage

Useful for:

- non-sensitive preferences
- simple persistent client settings

JavaScript can access it, so don't store sensitive authentication tokens there.

#### IndexedDB

Useful for:

- large browser-side datasets
- offline applications
- cached documents
- drafts
- structured data

Think of IndexedDB as a browser database rather than a simple key-value preference store.

---

### 4.6 Optimistic Updates

Suppose a user clicks a Like button.

Without optimistic updates:

```
Click
 ↓
API request
 ↓
Server response
 ↓
Update UI
```

The user may perceive a delay.

With optimistic updates:

```
Click
 ↓
Update UI immediately
 ↓
API request
 ↓
Success → keep change
Failure → rollback
```

The important part is rollback.

Conceptually:

```
Previous state
      ↓
Save snapshot
      ↓
Apply optimistic change
      ↓
Send request
      ↓
 ┌───────────────┐
 │               │
Success         Failure
 │               │
Keep change     Restore snapshot
```

This pattern is common for:

- likes
- follows
- toggles
- lightweight edits

---

### 4.7 URL State

Some state should not live in React state at all.

Examples:

- search query
- filters
- sorting
- pagination
- selected category

Instead:

```
/products?category=shoes&sort=price&page=2
```

URL state provides:

- shareability
- bookmarkability
- refresh persistence
- reproducibility

It can also make bug reports easier because the URL captures the application's state.

---

### 4.8 Form State

Large forms can cause unnecessary rerenders if every field is fully controlled.

Libraries such as React Hook Form use techniques such as uncontrolled inputs and refs to reduce unnecessary rendering.

Schema validation libraries such as Zod can provide a consistent validation model.

A useful architecture is:

```
Form
  ↓
React Hook Form
  ↓
Zod schema
  ↓
Frontend validation
  +
Backend validation
```

The backend must still validate input because frontend validation cannot be trusted for security.

---

### 4.9 Real-Time Data

Choose the communication model based on how real-time the requirement actually is.

#### WebSockets

Bidirectional, persistent connection:

```
Client ←────────→ Server
```

Good for:

- chat
- multiplayer games
- collaborative editors
- real-time presence

The application must handle:

- reconnection
- connection failures
- message ordering
- authentication
- backpressure where relevant

#### Server-Sent Events — SSE

One-directional:

```
Server ─────────→ Client
```

Good when the server needs to continuously send updates. A common example is AI response/token streaming.

SSE is simpler than WebSockets when client-to-server communication doesn't need to remain continuously bidirectional.

#### Polling

The client periodically asks:

```
GET /status
wait 5 seconds
GET /status
wait 5 seconds
GET /status
```

It's simple and often perfectly adequate when a few seconds of delay is acceptable.

Don't introduce WebSockets when polling already solves the problem.

---

## 5. I — Interfaces & APIs

Now ask:

> **How does the frontend communicate with the backend?**

This decision affects:

- network usage
- type safety
- caching
- developer experience
- backend/frontend coupling
- performance

### 5.1 REST vs GraphQL

#### REST

REST exposes resources through endpoints.

```
GET /users/42
GET /users/42/posts
GET /posts/123
```

Advantages:

- simple
- widely understood
- easy to cache
- works well with HTTP semantics
- straightforward infrastructure

Potential problems:

- **Over-fetching:** The client receives more data than needed.
- **Under-fetching:** The client needs multiple requests to build one screen.

#### GraphQL

GraphQL allows the client to request a specific data shape.

```graphql
query {
  user {
    id
    name
    avatar
  }
}
```

This can reduce over-fetching and under-fetching.

However, GraphQL introduces additional complexity around:

- caching
- schema management
- resolver performance
- query complexity
- authorization

A reasonable default for many applications is `REST + TanStack Query`. Choose GraphQL when the application genuinely benefits from flexible data shapes across many consumers.

---

### 5.2 Query Patterns

A data-fetching layer should distinguish different operations.

- **Queries:** For reads and cached data.
- **Mutations:** For writes.
- **Infinite queries:** For paginated/infinite scrolling data.

Good query keys are hierarchical.

For example:

```
posts
posts:user:42
posts:user:42:page:2
posts:user:42:sort:latest
```

This allows precise invalidation instead of unnecessarily clearing unrelated cached data.

---

### 5.3 Offset vs Cursor Pagination

Offset pagination `?page=3` is simple and allows users to jump to a specific page. But it can become inconsistent when data changes.

Imagine:

```
Page 1:
A B C

Page 2:
D E F
```

If three new records are inserted before page 2 is fetched, the boundary shifts.

Cursor pagination uses a position derived from an item: `?after=cursor123`

This is generally better for:

- social feeds
- chat history
- frequently changing datasets

The trade-off is that arbitrary page jumping is harder.

> Stable dataset / admin table → offset
> Live feed / chat → cursor

---

### 5.4 Custom Hooks as API Abstractions

Instead of allowing components to know how data is fetched:

```js
fetch("/api/posts")
```

Directly inside every component, expose an abstraction:

```js
const {
  data,
  isLoading,
  error
} = usePosts();
```

Now the component doesn't care whether the underlying implementation uses:

- REST
- GraphQL
- WebSockets
- a cache
- a BFF

This makes the data source easier to replace and the component easier to test.

---

### 5.5 Backend for Frontend — BFF

A BFF is a backend layer designed specifically for a frontend.

Suppose a product page requires:

```
Product service
Inventory service
Pricing service
Reviews service
Recommendations service
```

Without a BFF:

```
Browser
 ├── Product API
 ├── Inventory API
 ├── Pricing API
 ├── Reviews API
 └── Recommendations API
```

With a BFF:

```
Browser
   |
   v
BFF
 ├── Product service
 ├── Inventory service
 ├── Pricing service
 ├── Reviews service
 └── Recommendations service
```

The BFF can aggregate and transform these responses into one UI-oriented response.

Benefits include:

- fewer browser round trips
- hiding internal service topology
- frontend-specific response shapes
- centralized orchestration

---

### 5.6 Authentication

Common approaches include:

- **Sessions:** The server maintains session state.
- **JWTs:** The token contains signed claims that can be verified by servers.
- **OAuth / OIDC:** Authentication is delegated to an identity provider.

For many new applications, using a managed identity provider is safer than implementing authentication from scratch. Examples: Auth0, Clerk, enterprise identity providers.

If you implement authentication yourself, pay particular attention to:

- token storage
- expiration
- refresh
- rotation
- revocation
- CSRF
- XSS

Authentication is an area where small mistakes can have serious security consequences.

---

### 5.7 tRPC

tRPC provides end-to-end type safety when the frontend and backend are controlled by the same team.

Conceptually:

```
Server function
      ↓
Shared type information
      ↓
Typed client call
```

This can provide excellent developer experience without manually maintaining separate API schemas.

The trade-off is that it is best suited to systems where you control both ends of the contract. It is not necessarily the right choice for a public API consumed by unrelated clients.

---

### 5.8 WebSocket Architecture

Don't create a new WebSocket connection every time a component mounts.

A problematic design:

```
Component A mounts → WebSocket
Component B mounts → WebSocket
Component C mounts → WebSocket
```

You can end up with multiple unnecessary connections.

Instead, use a shared application-level connection:

```
Application
     |
 WebSocket
     |
 Global event layer
   /    |    \
 A     B     C
```

Components subscribe only to the events they need.

The shared layer handles:

- connection
- authentication
- reconnection
- cleanup
- message routing

---

### 5.9 API Error Handling

Don't expose raw API failures directly to users. Handle failures at multiple levels.

#### Network errors

Examples: timeout, offline device, connection failure

Show: offline indicators, retry controls, exponential backoff where appropriate

#### HTTP errors

Translate technical responses into useful UX.

```
401 → authenticate / redirect to login
403 → access denied
404 → resource not found
422 → validation error
500 → temporary server error
```

#### Validation errors

A validation response should ideally map directly to the affected field.

Better:

```
Email
[invalid email address]
```

Than:

```
Something went wrong.
```

---

## 6. O — Optimizations & Observability

A system that works correctly but performs poorly is still a poor system.

The final stage is:

> **What actually ships to the browser, how efficiently does it run, and how do we know when it breaks?**

### 6.1 Code Splitting

Don't send the entire application JavaScript bundle on the first page load.

Instead:

```
Initial bundle
      |
      ├── Homepage
      ├── Product
      └── Common code

Later:
      ├── Admin
      ├── Analytics
      └── Editor
```

Route-based splitting is often the first optimization to consider. In React, lazy loading and `Suspense` can support this pattern.

---

### 6.2 Tree Shaking

Tree shaking removes unused code from the final bundle.

Prefer targeted imports when appropriate. Instead of pulling in an entire library for one utility, import only what you need.

Be particularly careful with namespace imports because they can prevent effective tree shaking depending on the library and bundler configuration.

---

### 6.3 Lazy Loading

Don't load resources until they're needed.

For images below the fold:

```html
<img
  src="image.webp"
  loading="lazy"
/>
```

For heavy components:

```
Page loads
    ↓
Chart isn't visible
    ↓
Don't download chart library
    ↓
User scrolls
    ↓
Download + render chart
```

`IntersectionObserver` can be used to detect when content approaches the viewport.

---

### 6.4 Image Optimization

Images are often one of the largest contributors to page weight.

Modern formats such as WebP, AVIF can significantly reduce file size compared with older formats.

Use responsive images so a mobile device doesn't download a huge desktop image.

Conceptually:

```
Mobile → small image
Tablet → medium image
Desktop → large image
```

Always reserve the correct dimensions when possible:

```html
<img
  width="800"
  height="600"
  ...
/>
```

This helps prevent layout shifts.

---

### 6.5 Memoization

React provides `React.memo`, `useMemo`, `useCallback`

But these should not be used everywhere.

The correct process is:

```
Measure
  ↓
Find expensive rendering
  ↓
Optimize
  ↓
Measure again
```

`useCallback` is particularly useful when a function is passed to a memoized child and function identity is causing unnecessary renders.

Don't add memoization simply because it exists. Premature optimization increases complexity.

---

### 6.6 Virtualization

Suppose you need to render 10,000 rows. Rendering 10,000 DOM nodes is expensive.

Virtualization instead renders only the visible portion:

```
10,000 logical rows

       ↓

~20–50 DOM rows
```

As the user scrolls, visible nodes are recycled.

This is extremely useful for:

- large tables
- long lists
- chat history
- file explorers

For very large lists, virtualization can dramatically reduce DOM and rendering costs.

---

### 6.7 Vendor and Component Splitting

Third-party libraries often change less frequently than application code.

You can separate `Vendor code + Application code` so that stable vendor assets remain cached between deployments.

Also defer heavy functionality such as:

- charting libraries
- rich text editors
- code editors
- advanced data visualization

Users who never use those features shouldn't have to download them.

---

### 6.8 Debouncing

Debouncing waits until events stop occurring before executing.

Example:

```
User types:

H
He
Hel
Hell
Hello

       ↓

wait 300 ms

       ↓

API request
```

Ideal for:

- search boxes
- autocomplete
- expensive input processing

---

### 6.9 Throttling

Throttling limits execution frequency.

For example:

```
Scroll event
Scroll event
Scroll event
Scroll event
Scroll event

        ↓

Execute at most once every 100ms
```

Useful for:

- scroll handlers
- resize handlers
- mouse movement
- high-frequency events

A common mistake is recreating debounce/throttle functions on every render. Keep the function stable using an appropriate hook or utility.

---

### 6.10 React Transitions

React's concurrent features allow certain updates to be treated as lower priority.

For example:

```
User types
   ↓
Input update → high priority

Filtering 10,000 results
   ↓
Transition → lower priority
```

This allows urgent interactions to remain responsive while expensive UI updates happen in the background.

`startTransition` and `useDeferredValue` are useful tools for this type of scenario.

---

### 6.11 Observability

Production systems need visibility into failures and performance. There are three important areas.

#### Error Tracking

Tools such as Sentry can capture:

- exceptions
- stack traces
- affected users
- environment information
- release information

Upload source maps so production errors map back to your original source code.

#### Session Replay

Session replay can help reconstruct what the user did before an error occurred.

For example:

```
User opened checkout
       ↓
Changed address
       ↓
Clicked payment
       ↓
UI froze
       ↓
Error occurred
```

This can dramatically reduce debugging time. Privacy must be considered when recording user sessions.

#### Real User Monitoring — RUM

Lab testing might show `LCP = 1.4s` but real users may experience `LCP = 4.2s` because they use:

- slower devices
- slower networks
- older browsers
- different geographic regions

RUM collects performance information from actual users. This is essential for understanding real-world performance.

---

### 6.12 Performance Budgets

Performance should be enforced continuously.

For example:

```
Maximum JavaScript bundle: 300 KB
Maximum LCP: 2.5s
Maximum image size: 200 KB
```

A CI pipeline can fail if a pull request exceeds defined limits.

This prevents gradual performance degradation.

Without budgets:

```
Dependency added
   ↓
Bundle +30 KB

Another dependency
   ↓
Bundle +50 KB

Another feature
   ↓
Bundle +80 KB

One year later
   ↓
Application is noticeably slower
```

Performance budgets make regressions visible immediately.

---

## 7. The RADIO Interview Checklist

When you're under interview pressure, don't try to remember every individual concept. Remember six questions.

### Question 1 — How does the page reach the user?

Discuss:

- SSR, CSR, SSG, ISR, hybrid rendering, streaming, CDN, Core Web Vitals

Ask:

> Does this page need SEO? How fresh is the data? Is it personalized?

### Question 2 — How are the components structured?

Discuss:

- component boundaries, container/presentation, provider, compound components, polymorphic components, HOCs, design systems

Ask:

> Where should responsibilities and ownership boundaries exist?

### Question 3 — Where does the state live?

Classify state as: Local, Global, Server, URL

Then choose appropriate tools:

```
Local UI state     → useState/useReducer
Global client      → Zustand/Redux
Server state       → TanStack Query
Shareable state    → URL
```

### Question 4 — How does the client communicate with the backend?

Discuss:

- REST, GraphQL, tRPC, BFF, WebSockets, SSE, authentication, pagination, error handling

Ask:

> What data does the UI need, how often does it change, and how should the API expose it?

### Question 5 — What actually ships to the browser?

Discuss:

- code splitting, tree shaking, lazy loading, image optimization, virtualization, memoization, debouncing, throttling, transitions

Ask:

> What is the critical path, and what can be delayed?

### Question 6 — How do we know when it breaks?

Discuss:

- error tracking, source maps, session replay, RUM, Core Web Vitals, performance budgets, CI checks

Ask:

> How will we detect and diagnose problems after deployment?

---

## 8. Worked Example — Designing an E-Commerce Product Page

Let's apply RADIO to a concrete problem.

> "Design the frontend architecture for an e-commerce product page."

Don't immediately start talking about React components. Start with requirements.

### R — Requirements & Rendering

The product page needs:

- SEO
- fast initial load
- product information
- inventory
- pricing
- reviews
- personalized recommendations
- cart interaction

A reasonable rendering architecture might be:

```
Product page → ISR
Product metadata → server-rendered
Reviews → streamed
Cart → client-side
Recommendations → lazy loaded
```

Why? Product pages are highly SEO-sensitive and can tolerate some caching.

### A — Architecture

Component tree:

```
ProductPage
├── ProductInformation
├── ProductGallery
├── ProductOptions
├── AddToCart
├── Reviews
└── Recommendations
```

The design system supplies: Button, Input, Select, Modal, Rating, Price

The page can use compound components for complex widgets such as product selectors.

### D — Data

Classify the data:

```
Selected size         → local state
Cart                  → global/client state
Product information   → server state
Reviews               → server state
Sort/filter reviews   → URL state
```

TanStack Query can manage server data.

### I — Interfaces

The frontend could call:

```
GET /products/:id
GET /products/:id/reviews
POST /cart
GET /recommendations
```

If these services are separate internally, introduce a BFF:

```
Browser
   ↓
Product BFF
   ├── Product service
   ├── Inventory service
   ├── Pricing service
   ├── Reviews service
   └── Recommendations service
```

Use cursor pagination for reviews if the dataset is large and changing.

### O — Optimization

Optimize the critical path:

```
Product HTML
   ↓
Hero image
   ↓
Core interaction
```

Lazy-load: recommendations, reviews below the fold, expensive widgets

Use: responsive images, WebP/AVIF, CDN, caching, code splitting, virtualization if review lists are very large

Monitor: LCP, CLS, INP, JavaScript bundle size, API latency, frontend errors

---

## 9. Common Interview Mistakes

### Mistake 1 — Choosing technologies before understanding requirements

Bad: "I'll use Redux, GraphQL, WebSockets and micro-frontends."

Better: "Let's first determine the type of data, scale, team structure, and interaction requirements."

Architecture should follow requirements.

### Mistake 2 — Putting everything in global state

Not everything needs Redux or Zustand. A dropdown being open is probably not global state.

Ask: Who needs this data? If the answer is "only this component," keep it local.

### Mistake 3 — Treating server state as client state

API data is not automatically global application state. A server cache such as TanStack Query is often a better abstraction.

### Mistake 4 — Using WebSockets everywhere

WebSockets are powerful but introduce operational complexity. If updates every 30 seconds are acceptable, polling may be enough.

### Mistake 5 — Using micro-frontends because the application is large

Scale alone isn't sufficient. Micro-frontends primarily become attractive when independent teams and deployments justify their complexity.

### Mistake 6 — Premature optimization

Don't say: "I'll use useMemo everywhere."

Say: "I'll profile the application and optimize the actual bottlenecks."

### Mistake 7 — Ignoring accessibility and security

Accessibility and security aren't final checklist items. They are architectural requirements.

---

## 10. The Mental Model to Remember

The entire framework can be compressed into this:

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

When faced with a new system design problem, walk through it in order:

```
1. How does the page reach the user?
             ↓
2. How is the frontend structured?
             ↓
3. Where does each piece of state live?
             ↓
4. How does the frontend communicate with the backend?
             ↓
5. What should actually ship to the browser?
             ↓
6. How do we measure failures and performance?
```

That sequence prevents you from jumping randomly between technologies.

The goal of a frontend system design interview is not to mention the largest number of technologies.

The goal is to demonstrate that **every architectural decision follows from a requirement and has an understood trade-off**.

That is the core lesson of the RADIO framework.

---
