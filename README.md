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

**Your Page Paths depends on external data**

Next.js allows you to create pages with dynamic routes. For example, you can create a file called `pages/posts/[id].js` to show a single blog post based on id. This will allow you to show a blog post with id: 1 when you access posts/1.

However, which `id` you want to pre-render at build time might depend on external data.

Example: suppose that you've only added one blog post `(with id: 1)` to the database. In this case, you'd only want to pre-render `posts/1` at build time.

Later, you might add the second post with `id: 2`. Then you'd want to pre-render `posts/2` as well.

So your page paths that are pre-rendered depend on external data. To handle this, Next.js lets you export an `asyn`c function called `getStaticPaths` from a dynamic page (pages/posts/[id].js in this case). This function gets called at build time and lets you specify which paths you want to pre-render.

```js
// This function gets called at build time
export async function getStaticPaths() {
  // Call an external API endpoint to get posts
  const res = await fetch('https://.../posts')
  const posts = await res.json()

  // Get the paths we want to pre-render based on posts
  const paths = posts.map((post) => ({
    params: { id: post.id },
  }))

  // We'll pre-render only these paths at build time.
  // { fallback: false } means other routes should 404.
  return { paths, fallback: false }
}
```

Also in `pages/posts/[id].js`, you need to export `getStaticProps` so that you can fetch the data about the post with this `id` and use it to pre-render the page:

```js
function Post({ post }) {
  // Render post...
}

export async function getStaticPaths() {
  // ...
}

// This also gets called at build time
export async function getStaticProps({ params }) {
  // params contains the post `id`.
  // If the route is like /posts/1, then params.id is 1
  const res = await fetch(`https://.../posts/${params.id}`)
  const post = await res.json()

  // Pass post data to the page via props
  return { props: { post } }
}

export default Post
```

## When should I use Static Generation?

We recommend using Static Generation (with and without data) whenever possible because your page can be built once and served by CDN, which makes it much faster than having a server render the page on every request.

You can use Static Generation for many types of pages, including:

1.Marketing pages
2.Blog posts
3.E-commerce product listings
4.Help and documentation

You should ask yourself: "Can I pre-render this page ahead of a user's request?" If the answer is yes, then you should choose Static Generation.

On the other hand, Static Generation is not a good idea if you cannot pre-render a page ahead of a user's request. Maybe your page shows `frequently updated data`, and the `page content changes on every request`.

In cases like this, you can do one of the following:

1.Use Static Generation with Client-side Rendering: You can skip pre-rendering some parts of a page and then use client-side JavaScript to populate them. To learn more about this approach, check out the Data Fetching documentation.

2.Use Server-Side Rendering: Next.js pre-renders a page on each request. It will be slower because the page cannot be cached by a CDN, but the pre-rendered page will always be up-to-date. We'll talk about this approach below.

## Server Side Rendering

`Also referred to as "SSR" or "Dynamic Rendering".`

If a page uses **Server-side Rendering**, the page HTML is generated on **each request**.

To use Server-side Rendering for a page, you need to export an `async` function called `getServerSideProps`. This function will be called by the server on every request.

For example, suppose that your page needs to `pre-render frequently updated data (fetched from an external API)`. You can write `getServerSideProps` which fetches this data and passes it to Page like below:

```js
export default function Page({ data }) {
  return (
    // Render Data
  )
}

// This gets called on every request
export async function getServerSideProps() {
  // Fetch Data from external API
  const res = await fetch('...API')
  const data = await res.json()
  
  return {
    props: {
      data
    }
  }
}
```

As you can see, `getServerSideProps` is similar to `getStaticProps`, but the difference is that `getServerSideProps` **is run on every request instead of on build time**.


### Summary

We've discussed two forms of pre-rendering for Next.js.

`Static Generation (Recommended):` The HTML is generated at build time and will be reused on each request. To make a page use Static Generation, either export the page component, or export getStaticProps (and getStaticPaths if necessary). It's great for pages that can be pre-rendered ahead of a user's request. You can also use it with Client-side Rendering to bring in additional data.

`Server-side Rendering:` The HTML is generated on each request. To make a page use Server-side Rendering, export getServerSideProps. Because Server-side Rendering results in slower performance than Static Generation.


## Data Fetching

We’ll talk about the three unique Next.js functions you can use to fetch data for pre-rendering:

1.`getStaticProps (Static Generation)`: Fetch data at build time.
2.`getStaticPaths (Static Generation)`: Specify dynamic routes to pre-render pages based on data.
3.`getServerSideProps (Server-side Rendering)`: Fetch data on each request.

In addition, we’ll talk briefly about how to fetch data on the client side.

### getStaticProps (Static Generation)

If you export an `async` function called `getStaticProps` from a page, Next.js will pre-render this page at build time using the props returned by `getStaticProps`.

```js
export async function getStaticProps(context) {
  return {
    props: {} // will be passed to the page component as props
  }
}
```

**The `context` parameter is an object containing the following keys:**

1.`params` contains the `route parameters` for pages using dynamic routes. For example, if the page name is [id].js , then params will look like `{ id: ... }`. To learn more, take a look at the Dynamic Routing documentation. You should use this together with getStaticPaths, which we’ll explain later.

2.`preview` is true if the page is in the preview mode and undefined otherwise.
3.`previewData` contains the preview data set by setPreviewData. See the Preview Mode documentation.
4.`locale` contains the active locale (if enabled).
5.`locales` contains all supported locales (if enabled).
6.`defaultLocale` contains the configured default locale (if enabled).
7.`getStaticProps` should return an object with:

`props` - A required object with the props that will be received by the page component. It should be a serializable object

`revalidate` - An optional amount in seconds after which a page re-generation can occur. More on Incremental Static Regeneration

`notFound` - An optional boolean value to allow the page to return a 404 status and page. Below is an example of how it works:

```js
export async function getStaticProps(context) {
  const res = await fetch(API)
  const data = await res.json()
  
  if (!data) {
    return {
      notFound: true
    }
  }
  
  return {
    props: { data } // will be passed to the component as props
  }
}
````

Note: `notFound` is not needed for `fallback: false` mode as only paths returned from `getStaticPaths` will be pre-rendered.

`redirect` - An optional redirect value to allow redirecting to internal and external resources. It should match the shape of { destination: string, permanent: boolean }. In some rare cases, you might need to assign a custom status code for older HTTP Clients to properly redirect. In these cases, you can use the statusCode property instead of the permanent property, but not both. Below is an example of how it works:

```js
export async function getStaticProps(context) {
  const res = await fetch(`https://...`)
  const data = await res.json()

  if (!data) {
    return {
      redirect: {
        destination: '/',
        permanent: false,
      },
    }
  }

  return {
    props: { data }, // will be passed to the page component as props
  }
}
```

**Note: Redirecting at build-time is currently not allowed and if the redirects are known at build-time they should be added in next.config.js.**

### Simple Example

Here's an example which uses `getStaticProps` to fetch a list of blog posts from a CMS (content management system)

```js
export default function Blog({ posts }) {
  return (
    <div>
      {posts.map(post => <p>{post.title}</p>)}
    </div>
  )
}

// This function gets called at build time on server-side.
// It won't be called on client-side, so you can even do
// direct database queries. See the "Technical details" section.
export async function getStaticProps() {
  // Call an external API endpoint to get posts.
  // You can use any data fetching library
  const res = await fetch(API)
  const posts = await res.json()
  
  // By returning { props: { posts } }, the Blog component
  // will receive `posts` as a prop at build time
  return {
    props: {
      posts
    }
  }
}
```

### When Should I use `getStaticProps`?

You should use `getStaticProps` if:

The data required to render the page is `available at build time` ahead of a user’s request.
The data comes from a `headless CMS`.
The data can be publicly cached (not user-specific).
The page must be pre-rendered (for SEO) and be very fast — `getStaticProps` generates HTML and JSON files, both of which can be cached by a CDN for performance.

### Typescript use `GetStaticProps`

for typescript you can use the `GetStaticProps` type from `next`.

```js
import { GetStaticProps } from 'next'

export const getStaticProps: GetStaticProps = async (context) => {
  ...
}
```

With getStaticProps you don't have to stop relying on dynamic content, as static content can also be dynamic. Incremental Static Regeneration allows you to update existing pages by re-rendering them in the background as traffic comes in.

Inspired by stale-while-revalidate, background regeneration ensures traffic is served uninterruptedly, always from static storage, and the newly built page is pushed only after it's done generating.

Consider our previous `getStaticProps` example, but now with regeneration enabled:

```js
function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li>{post.title}</li>
      ))}
    </ul>
  )
}

// This function gets called at build time on server-side.
// It may be called again, on a serverless function, if
// revalidation is enabled and a new request comes in
export async function getStaticProps() {
  const res = await fetch('https://.../posts')
  const posts = await res.json()

  return {
    props: {
      posts,
    },
    // Next.js will attempt to re-generate the page:
    // - When a request comes in
    // - At most once every second
    revalidate: 1, // In seconds
  }
}

export default Blog
```

Now the list of blog posts will be revalidated once per second; if you add a new blog post it will be available almost immediately, without having to re-build your app or make a new deployment.

This works perfectly with `fallback: true`. Because now you can have a list of posts that's always up to date with the latest posts, and have a blog post page that generates blog posts on-demand, no matter how many posts you add or update.

### Static Content at scale

Unlike traditional SSR, Incremental Static Regeneration ensures you retain the benefits of static:

1.No spikes in latency. Pages are served consistently fast
2.Pages never go offline. If the background page re-generation fails, the old page remains unaltered
3.Low database and backend load. Pages are re-computed at most once concurrently

### Reading Files: use `process.cwd()`

file can be read directly from the file system in `getStaticProps`

in order to do so, you have to have to get the full path to the file.

Since, Next.js compiles your code into a separate directory, you can't use `__dirname` as the path it will return will be different from the pages directory.

Instead you can use `process.cwd()` which gives you the directory where Next.js is being executed.

```js
import { promises as fs } from 'fs'
import path from 'path'

// posts will be populated at build time by getStaticProps()
function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li>
          <h3>{post.filename}</h3>
          <p>{post.content}</p>
        </li>
      ))}
    </ul>
  )
}

// This function gets called at build time on server-side.
// It won't be called on client-side, so you can even do
// direct database queries. See the "Technical details" section.
export async function getStaticProps() {
  const postsDirectory = path.join(process.cwd(), 'posts')
  const filenames = await fs.readdir(postsDirectory)

  const posts = filenames.map(async (filename) => {
    const filePath = path.join(postsDirectory, filename)
    const fileContents = await fs.readFile(filePath, 'utf8')

    // Generally you would parse/transform the contents
    // For example you can transform markdown to HTML here

    return {
      filename,
      content: fileContents,
    }
  })
  // By returning { props: { posts } }, the Blog component
  // will receive `posts` as a prop at build time
  return {
    props: {
      posts: await Promise.all(posts),
    },
  }
}

export default Blog
```

## Technical Details

### Only runs at build time

Note that `getStaticProps` runs only on the `server-side`. It will never be run on the `client-side`. It won’t even be included in the JS bundle for the browser. That means you can write code such as direct database queries without them being sent to browsers. You should not fetch an API route from getStaticProps — instead, you can write the server-side code directly in getStaticProps.
