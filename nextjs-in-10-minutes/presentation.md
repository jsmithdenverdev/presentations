---
theme: gaia
\_class: lead
paginate: true
backgroundColor: #fff
---

![bg left:40% 80%](./logo.svg)

# NextJS in 10(ish) minutes

Jake Smith

_Solutions Architect_
_CarLotz Web Modernization Project_

---

# What we'll cover

- Quick React Refresher
- What is NextJS
- Pages vs Components
- Filesystem Based Routing
- API Routes
- Rendering
- Server Side Rendering
- Static Pre Generation

---

# Quick refresher on React

- React is a UI library.
- In React, UI's are made up of components.
- Components are written in a js/html hybrid called JSX (or TSX for Typescript).
- Components encapsulate logic for rendering a UI and reacting to events and state changes.
- Small atomic components are composed into larger pieces of functionality.
- The highest level of composition is typically a page.

---

# What is NextJS

- Framework for React JS
- Batteries included with lots of available configuration.
- Emphasis on performance and maintainability.
- Slightly more opionated than Create React App.
- Filesystem based routing.
  - Pages are a first class citizen.
  - API routes.

---

# Pages vs Components

- Recall a page is typically composed of small, atomic components.
- A page is what your user will see when they visit a URL with your app.
- NextJS gives us a clean deliniation between a page and a component.
- Pages are stored in their own area in a NextJS project.
- Pages control things like routing, server side rendering, static pre-generation, among others.

---

# Filesystem Based Routing

- NextJS scaffolds a project with a top level `pages` directory.
- Components in that directory are served up when the user hits a URL.

**Example.**

```
└── pages
    ├── _app.tsx    # _app wraps each page in our Application
    ├── index.tsx   # Rendered when a user visits oursite.com
    ├── about.tsx   # Rendered when a user visits oursite.com/about
    └── contact.tsx # Rendered when a user visits oursite.com/contact
```

---

# Filesystem Based Routing Continued

- NextJS also supports dynamic routing.
- Files or directories can wrap their name in square brackets (e.g. `[sku].tsx`).
- The value in the brackets will be treated as a path parameter and available to the page.

---

# Dynamic Routing Example

```
.
└── pages
    └── products
        └── [sku].tsx # Rendered when a user visits outsite.com/products/123.
```

`[sku].tsx`

```tsx
const Product = () => {
  const router = useRouter(); // useRouter hook provided by NextJS.
  const { sku } = router.query;

  return <p>Product: {sku}</p>; // will render <p>Product: 123</p>
};
```

---

# API Routes

- NextJS also gives us the ability to define API routes.
- These special routes live under the `pages/api` directory.
- We define express like request handlers in these files and can fetch them from the UI.
- Dynamic routing is supported for API routes as well.

```
.
└── pages
    └── api
        └── greeting.ts # Handler called a user fetches oursite.com/api/greeting.
```

---

# API Route Example

`/pages/api/greeting.ts`

```tsx
export default function greetingHandler(req, res) {
  res.status(200).json({ hello: "world" });
}
```

`/pages/greeting.tsx`

```tsx
// within a useEffect
const response = await fetch("/api/greeting");
const result = await response.json(); // { hello: "world" }

setGreeting(result);
```

---

# Rendering

- NextJS pages are rendered on the server.
- Next supports multiple forms of server side rendering.
- Pages can be built per request, or built when the site builds (e.g. in a pipeline).
  - Server Side Rendering (SSR) - Built per request.
  - Static Pre Generation (SRG) - Built at build time.
- A page can still fetch data after its been rendered.

---

# Server Side Rendering

- Pages in NextJS can export a function called `getServerSideProps`.
- This function will be run when the page is requested and can be used to do things like fetch data.
- The page will be built on the server and returned to the client as the raw HTML and Javascript needed to finish hydrating the page.
- One of the benifits of this paradigm is that it takes the responsibility of fetching data away from a page.
- A page only needs to consume props returned from `getServerSideProps`.

---

# Static Pre Generation

- Instead of rendering a page for every request a page can be rendered when the site is built.
- To opt into this functionality a page can export a function called `getStaticProps`.
- Can be combined with dynamic paths by exporting a function named `getStaticPaths`.
- This function will be run when the site is built and can be used to do things like fetch data.
- Pages can also leverage Incremental Static Regeneration (ISR) to occasionally rebuild themselves.

---

# Questions / Additional Resources

## Questions?

## Resources

- https://nextjs.org/docs/getting-started
