# Basic Features

## Pages

**Simple Route**

`/pages/about.tsx` => route: `/about`

**Dynamic Route**

`/pages/posts/[id].tsx` => route: `/posts/1` or `/posts/2`  and ...


## Pre-Rendering

By default, Next.js pre-renders every page. This means that Next.js generates HTML for each page in advance, instead of having it all done by client-side JavaScript. Pre-rendering can result in better performance and SEO.

Each generated HTML is associated with minimal JavaScript code necessary for that page. When a page is loaded by the browser, its JavaScript code runs and makes the page fully interactive. (This process is called hydration.)

### Two Form of Pre-rendering

Next.js has two forms of pre-rendering: `Static Generation` and `Server-side Rendering`. The difference is in when it generates the HTML for a page.

**Static Generation (Recommended):** The HTML is generated at build time and will be reused on each request.

**Server Side Rendering:** The HTML is generated on each request.
