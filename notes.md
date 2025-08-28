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

# - API routes in Next.js (how to create & handle)

# - Hydration in Next.js

# - Protecting routes in Next.js with authentication

# - SSR vs SSG vs ISR (Incremental Static Regeneration)

# - Performance optimization in Next.js (image optimization, script strategies, dynamic imports)

# - Middleware in Next.js (for auth, logging, redirects)

# - Next.js App Router vs Pages Router (differences and use cases)

# - Deployment considerations (Vercel, custom server)
