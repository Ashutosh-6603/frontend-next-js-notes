# - What is Next.js and why use it over CRA (Create React App)?

- Next.js is a React framework that adds first-class support for server rendering, static generation, routing, data fetching, API routes, built-in performance optimizations (image optimization, code-splitting, automatic routing) and opinionated DX for building production apps.

- Why pick Next.js over CRA?

  - Server-side rendering (SSR) and Static Site Generation (SSG) out of the box → better SEO and faster first contentful paint for many apps.

  - Hybrid rendering: Per page you can choose SSR/SSG/ISR or client-side-only. CRA is purely client-side.

  - File-based routing (automatic route creation).

  - API routes bundled in the same project (no separate server needed for many use-cases).

  - Built-in optimizations: next/image (automatic image optimization), code-splitting, prefetching, and smart caching.

  - Great deployment story (Vercel particularly) and first-class support for edge functions.

  - App Router (modern architecture) with React Server Components, nested layouts and streaming.

  - CRA is still fine for purely client-rendered SPAs; use Next.js when you need faster first paint, SEO, or server-side logic.

# - Static asset serving (public folder)

- Static asset serving (public folder)

  - Put static assets (images, robots.txt, favicon) in public/.

  - They are served from the root of the site: public/logo.png -> /logo.png.

  - Use <img src="/logo.png" /> or next/image <Image src="/logo.png" ... />.

# - getServerSideProps (SSR)

- getServerSideProps (SSR) — Pages Router only

  - Runs on every request on the server.

  - Useful when you need per-request (user-specific) data (session, A/B tests, auth).

  - Exported from a page file in pages/.

  ```
  // pages/profile.js
  export async function getServerSideProps(context) {
    const { req, res, query } = context;

    // example: read cookie to check session

    const token = req.cookies.token; // Node req (with cookie parser or Next's default)

    if (!token) {
        return {
            redirect: {
                destination: '/login',
                permanent: false,
            },
        };
    }

    const user = await fetchUserFromToken(token);

    return { props: { user } };
  }

  export default function Profile({ user }) {
    return <div>Hello {user.name}</div>;
  }
  ```

# - getStaticProps (SSG)

- getStaticProps (SSG) — Pages Router only

  - Runs at build time and produces props for a page.

  - Fast: served as static HTML.

  - Can be combined with revalidate to enable ISR.

  ```
  // pages/posts.js

  export async function getStaticProps() {

    const posts = await fetch('https://api.example.com/posts').then(r => r.json());

    return {
        props: { posts },
        revalidate: 60, // optional: ISR (rebuild after 60 seconds)
    };
  }

    export default function Posts({ posts }) {
        return <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>;
    }
  ```

# - getStaticPaths (for dynamic static pages)

- getStaticPaths (for dynamic static pages) — Pages Router only

  - For dynamic routes that use SSG you must tell Next.js which paths to pre-render at build time.

  - fallback can be 'false' | 'true' | 'blocking'.

```
// pages/posts/[slug].js
export async function getStaticPaths() {
  const posts = await fetch('https://api.example.com/posts').then(r => r.json());
  const paths = posts.map(p => ({ params: { slug: p.slug } }));
  return { paths, fallback: 'blocking' }; // fallback 'blocking' commonly used
}

export async function getStaticProps({ params }) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`).then(r => r.json());
  if (!post) return { notFound: true };
  return { props: { post }, revalidate: 60 };
}
```

fallback behavior quick:

    - false: only paths returned in paths exist; others 404.

    - true: Next will initially render a placeholder and fetch on client (older behavior).

    - blocking: user waits for server to generate page on first request, then cached.

# - Hydration in Next.js

- Hydration = browser takes server-rendered HTML and attaches React event listeners + rehydrates components into an interactive app.

- Server renders static HTML → client downloads JS → React hydrates.

- Hydration mismatches happen when the server and client render different markup (e.g., using Math.random() on server vs client, reading window on server).

- Fixes / strategies:-

  - Move client-only code into client components ("use client" in App Router) or use useEffect for browser-only behavior.

  - Use suppressHydrationWarning or dynamic(import(), { ssr: false }) for purely client components.

  - Keep deterministic render output on server.

# - Protecting routes in Next.js with authentication

- Principles

  - Server-side protection (recommended): check session on the server and redirect or return 401 before rendering.

  - Middleware: run checks at the edge to redirect unauthenticated requests early (good for static+SSR hybrids).

  - Client-side protection: only for UX (not secure on its own).

```
// pages/dashboard.js
export async function getServerSideProps({ req }) {
  const token = req.cookies.token;
  if (!token) {
    return { redirect: { destination: '/login', permanent: false } };
  }
  // validate token...
  const user = await getUser(token);
  return { props: { user } };
}
```

Example — middleware (works for both routers)

```
// middleware.ts (root)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  // protect /dashboard and its subroutes
  if (pathname.startsWith('/dashboard')) {
    const token = request.cookies.get('token')?.value;
    if (!token) {
      // redirect to login
      const url = request.nextUrl.clone();
      url.pathname = '/login';
      return NextResponse.redirect(url);
    }
  }
  return NextResponse.next();
}

// optional: limit middleware to specific paths in next.config.js via matcher
export const config = {
  matcher: ['/dashboard/:path*', '/account/:path*'],
};
```

# - SSR vs SSG vs ISR (Incremental Static Regeneration)

- SSR (Server-Side Rendering)

  - Page HTML is generated on each request (e.g., getServerSideProps).

  - Good for per-request personalization, dynamic content that can’t be cached.

- SSG (Static Site Generation)

  - Pages generated at build time (getStaticProps / static rendering in app dir).

  - Very fast; great for content that changes infrequently.

- ISR (Incremental Static Regeneration)

  - Hybrid: pre-render static pages, and rebuild them in the background when stale.

  - Pages can revalidate on a schedule (revalidate) and thus be mostly static while allowing content updates.

    - Pages Router: revalidate returned from getStaticProps or as property in Next Config.

    - App Router: export const revalidate = 60 or fetch(..., { next: { revalidate: 60 } }).

- Trade-offs

  - SSR: more server cost, slower per-request latency (but always fresh).

  - SSG: cheap and fast, but stale until redeploy (unless ISR used).

  - ISR: good balance — low-latency cached content with eventual consistency.
