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

Importantly, Next.js lets you choose which pre-rendering form you'd like to use for each page. You can create a "hybrid" Next.js app by using Static Generation for most pages and using Server-side Rendering for others.

We recommend using Static Generation over Server-side Rendering for performance reasons. Statically generated pages can be cached by CDN with no extra configuration to boost performance. However, in some cases, Server-side Rendering might be the only option.

You can also use Client-side Rendering along with Static Generation or Server-side Rendering. That means some parts of a page can be rendered entirely by client side JavaScript. To learn more, take a look at the Data Fetching documentation.

### Static Generation Without Data

By default, Next.js pre-renders pages using Static Generation without fetching data. Here's an example:

```js
function About() {
  return <div>About</div>
}

export default About
```

Note that this page does not need to fetch any external data to be pre-rendered. In cases like this, Next.js generates a single HTML file per page during build time.

### Static Generation With Data

Some pages require fetching external data for pre-rendering. There are two scenarios, and one or both might apply. In each case, you can use a special function Next.js provides:

1. Your page content depends on external data: Use `getStaticProps`.
2. Your page paths depend on external data: Use `getStaticPaths` (usually in addition to getStaticProps)

**Your Page Content depends on external data**

Example: Your blog page might need to fetch the list of blog posts from a CMS (content management system).

```js
// TODO: Need to fetch `posts` (by calling some API endpoint)
//       before this page can be pre-rendered.
function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li>{post.title}</li>
      ))}
    </ul>
  )
}

export default Blog
```

To fetch this data on pre-render, Next.js allows you to export an `async` function called `getStaticProps` from the same file. This function gets called at build time and lets you pass fetched data to the page's `props` on pre-render.

```js
function Blog({ posts }) {
  // Render posts...
}

// This function gets called at build time
export async function getStaticProps() {
  // Call an external API endpoint to get posts
  const res = await fetch('https://.../posts')
  const posts = await res.json()

  // By returning { props: { posts } }, the Blog component
  // will receive `posts` as a prop at build time
  return {
    props: {
      posts,
    },
  }
}

export default Blog
```
