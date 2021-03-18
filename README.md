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

### Statically Generates both HTML and CSS

When a page with `getStaticProps` is pre-rendered at build time, in addition to the page HTML file, `Next.js` generates a JSON file holding the result of running getStaticProps.

This JSON file will be used in client-side routing through next/link (documentation) or next/router (documentation). When you navigate to a page that’s pre-rendered using `getStaticProps`, Next.js fetches this JSON file (pre-computed at build time) and uses it as the props for the page component. This means that client-side page transitions will not call getStaticProps as only the exported JSON is used.

### Only allowed in a page

`getStaticProps` can only be exported from a page. You can’t export it from non-page files.

One of the reasons for this restriction is that React needs to have all the required data before the page is rendered.

Also, you must use `export async function getStaticProps() {}` — it will not work if you add getStaticProps as a property of the page component.

### Runs on every request in development

In development (next dev), getStaticProps will be called on every request.

## getStaticPaths

If you export an `async function called getStaticPaths` from a page that uses dynamic routes, Next.js will statically pre-render all the paths specified by getStaticPaths.

```js
export async function getStaticPaths() {
  return {
    paths: [
      { params: { ... } } // See the "paths" section below
    ],
    fallback: true or false // See the "fallback" section below
  };
}
```

### The Paths key required

The paths key determines which paths will be pre-rendered. For example, suppose that you have a page that uses dynamic routes named pages/posts/[id].js. If you export getStaticPaths from this page and return the following for paths:

```js
return {
  paths: [
    { params: { id: '1' } },
    { params: { id: '2' } }
  ],
  fallback: ...
}
```

Then `Next.js` will statically generate `posts/1` and `posts/2` at build time using the page component in `pages/posts/[id].js`.

Note that the value for each params must match the parameters used in the page name:

### The fallback is required

The object returned by getStaticPaths must contain a boolean fallback key.

If fallback is false, then any paths not returned by getStaticPaths will result in a 404 page. You can do this if you have a small number of paths to pre-render - so they are all statically generated during build time. It’s also useful when the new pages are not added often. If you add more items to the data source and need to render the new pages, you’d need to run the build again.

Here’s an example which pre-renders one blog post per page called pages/posts/[id].js. The list of blog posts will be fetched from a CMS and returned by getStaticPaths . Then, for each page, it fetches the post data from a CMS using getStaticProps. This example is also in the Pages documentation.

```js
// pages/posts/[id].js

function Post({ post }) {
  // Render post...
}

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

## When Should I use getStaticPaths?

You should use `getStaticPaths` if you're statically pre-rendering pages that use dynamic routes.

## Typescript use GetStaticPaths

or TypeScript, you can use the GetStaticPaths type from next:

```js
export const getStaticPaths: GetStaticPaths = async () => {...}
```

### Technical Details

**Use together with `getStaticProps`**

When you use `getStaticProps` on a page with dynamic route parameters, you must use `getStaticPaths`

You `cannot` use `getStaticPaths` with `getServerSideProps`.

`getStaticPaths` only runs `at build time` on `server-side`.

`getStaticPaths` can only be exported from a `page`. You can’t export it from non-page files.

## getServerSideProps

if you export an async function called `getServerSideProps` from a page, Next.js will pre-render this page on each request using the data returned by `getServerSideProps`.

```js
export async function getServerSideProps(context) {
  return {
    props: {...} // will be passed to the page component as props
  }
}
```

## Built in CSS Support

### Adding a Global Stylesheet

To add a stylesheet to your application, import the CSS file within `pages/_app.js`.

For example, consider the following stylesheet named `styles.css`:

```js
body {
  font-family: 'SF Pro Text', 'SF Pro Icons', 'Helvetica Neue', 'Helvetica',
    'Arial', sans-serif;
  padding: 20px 20px 60px;
  max-width: 680px;
  margin: 0 auto;
}
```

Create a `pages/_app.js` file if not already present. Then, import the styles.css file.

```js
import '../styles.css'

// This default export is required in a new `pages/_app.js` file.
export default function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}
```

These `styles (styles.css)` will apply to all pages and components in your application. Due to the global nature of stylesheets, and to avoid conflicts, you may only import them inside `pages/_app.js`.

In development, expressing stylesheets this way allows your styles to be hot reloaded as you edit them—meaning you can keep application state.

In production, all CSS files will be automatically concatenated into a single minified `.css` file.

### import styles from `node-modules`

Since Next.js 9.5.4, importing a CSS file from node_modules is permitted anywhere in your application.

For global stylesheets, like `bootstrap` or `nprogress`, you should import the file inside `pages/_app.js`. For example:

```js
// pages/_app.js
import 'bootstrap/dist/css/bootstrap.css'

export default function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}
```

For importing `CSS` required by a third party component, you can do so in your component. For example:

```js
// components/ExampleDialog.js
import { useState } from 'react'
import { Dialog } from '@reach/dialog'
import '@reach/dialog/styles.css'

function ExampleDialog(props) {
  const [showDialog, setShowDialog] = useState(false)
  const open = () => setShowDialog(true)
  const close = () => setShowDialog(false)

  return (
    <div>
      <button onClick={open}>Open Dialog</button>
      <Dialog isOpen={showDialog} onDismiss={close}>
        <button className="close-button" onClick={close}>
          <VisuallyHidden>Close</VisuallyHidden>
          <span aria-hidden>×</span>
        </button>
        <p>Hello there. I am a dialog</p>
      </Dialog>
    </div>
  )
}
```

### Adding a Component Level CSS

Next.js supports CSS Modules using the `[name].module.css` file naming convention.

CSS Modules locally scope CSS by automatically creating a unique class name. This allows you to use the same CSS class name in different files without worrying about collisions.

This behavior makes CSS Modules the ideal way to include component-level CSS. CSS Module files can be imported anywhere in your application.

For example, consider a reusable `Button` component in the components/ folder:

First, create `components/Button.module.css` with the following content:

```js
/*
You do not need to worry about .error {} colliding with any other `.css` or
`.module.css` files!
*/
.error {
  color: white;
  background-color: red;
}
```

Then, create `components/Button.js`, importing and using the above CSS file:

```js
import styles from './Button.module.css'

export function Button() {
  return (
    <button
      type="button"
      // Note how the "error" class is accessed as a property on the imported
      // `styles` object.
      className={styles.error}
    >
      Destroy
    </button>
  )
}
```

CSS Modules are an optional feature and are only enabled for files with the `.module.css` extension. Regular `<link>` stylesheets and global CSS files are still supported.

In production, all CSS Module files will be automatically concatenated into many minified and code-split `.css` files. These `.css` files represent hot execution paths in your application, ensuring the minimal amount of CSS is loaded for your application to paint.

### Sass Support

Next.js allows you to import `Sass` using both the `.scss` and `.sass` extensions. You can use component-level Sass via CSS Modules and the `.module.scss` or `.module.sass`
extension.

Before you can use `Next.js'` built-in `Sass` support, be sure to install sass:

`$. npm install sass`

### Less and Stylus Support

To support importing `.less` or `.styl` files you can use the following plugins:

- `@zeit/next-less`
- `@zeit/next-stylus`

If using the less plugin, don't forget to add a dependency on less as well, otherwise you'll see an error like:

`Error: Cannot find module 'less'`


### CSS in JS

It's possible to use any existing `CSS-in-JS` solution. The simplest one is inline styles:

```js
function HiThere() {
  return <p style={{ color: 'red' }}>hi there</p>
}

export default HiThere
```

See the above examples for other popular CSS-in-JS solutions (like Styled Components).

A component using `styled-jsx` looks like this:

```js
function HelloWorld() {
  return (
    <div>
      Hello world
      <p>scoped!</p>
      <style jsx>{`
        p {
          color: blue;
        }
        div {
          background: red;
        }
        @media (max-width: 600px) {
          div {
            background: blue;
          }
        }
      `}</style>
      <style global jsx>{`
        body {
          background: black;
        }
      `}</style>
    </div>
  )
}

export default HelloWorld
```


## Image Component and Image Optimization

Since version 10.0.0, Next.js has a built-in Image Component and Automatic Image Optimization.

The Next.js Image Component, `next/image`, is an extension of the HTML <img> element, evolved for the modern web.

The Automatic Image Optimization allows for `resizing`, `optimizing`, and `serving` images in modern formats like `WebP` when the browser supports it. This avoids shipping large images to devices with a smaller viewport. It also allows Next.js to automatically adopt future image formats and serve them to browsers that support those formats.

Automatic Image Optimization works with any image source. Even if the image is hosted by an external data source, like a CMS, it can still be optimized.

Instead of optimizing images at build time, Next.js optimizes images on-demand, as users request them. Unlike static site generators and static-only solutions, your build times aren't increased, whether shipping 10 images or 10 million images.

Images are lazy loaded by default. That means your page speed isn't penalized for images outside the viewport. Images load as they are scrolled into viewport.

### Image Component

To add an image to your application, import the `next/image` component:

```js
import Image from 'next/image'

function Home() {
  return (
    <>
      <h1>My Homepage</h1>
      <Image
        src="/me.png"
        alt="Picture of the author"
        width={500}
        height={500}
      />
      <p>Welcome to my homepage!</p>
    </>
  )
}

export default Home
```

## Static File Serving

Next.js can serve static files, like `images`, under a folder called `public` in the root directory. **Files inside public can then be referenced by your code starting from the base URL (/)**.

For example, if you add an image to `public/me.png`, the following code will access the image:

```js
import Image from 'next/image'

function Avatar() {
  return <Image src="/me.png" alt="me" width="64" height="64" />
}

export default Avatar
```

**Note: next/image requires Next.js `10 or later`.**

This folder is also useful for `robots.txt`, `favicon.ico`, Google Site Verification, and any other static files (including .html)!

**Note: Don't name the `public` directory anything else. The name cannot be changed and is the only directory used to serve static assets.**

**Note: Be sure to not have a static file with the same name as a file in the `pages/` directory, as this will result in an error.**

**Note: Only assets that are in the public directory at build time will be served by Next.js. Files added at runtime won't be available. We recommend using a third party service like AWS S3 for persistent file storage.**


## Fast Refresh

Fast Refresh is a Next.js feature that gives you instantaneous feedback on edits made to your React components. Fast Refresh is enabled by default in all Next.js applications on `9.4 or newer`. With Next.js Fast Refresh enabled, most edits should be visible within a second, without losing component state.

### How It Works?

- If you edit a file that only exports React component(s), Fast Refresh will update the code only for that file, and re-render your component. You can edit anything in that file, including styles, rendering logic, event handlers, or effects.
- If you edit a file with exports that aren't React components, Fast Refresh will re-run both that file, and the other files importing it. So if both Button.js and Modal.js import theme.js, editing theme.js will update both components.
- Finally, if you edit a file that's imported by files outside of the React tree, Fast Refresh will fall back to doing a full reload. You might have a file which renders a React component but also exports a value that is imported by a non-React component. For example, maybe your component also exports a constant, and a non-React utility file imports it. In that case, consider migrating the constant to a separate file and importing it into both files. This will re-enable Fast Refresh to work. Other cases can usually be solved in a similar way.


## TypeScript

Next.js provides an integrated TypeScript experience out of the box, similar to an IDE.

To get started, create an `empty tsconfig.json` file in the root of your project:

`$. touch tsconfig.json`

Next.js will automatically configure this file with default values. Providing your own tsconfig.json with custom compiler options is also supported.

Next.js uses `Babel` to handle `TypeScript`, which has some caveats, and some compiler options are handled differently.

Then, run next (normally `npm run dev` or `yarn dev`) and Next.js will guide you through the installation of the required packages to finish the setup:

```js
npm run dev

# You'll see instructions like these:
#
# Please install typescript, @types/react, and @types/node by running:
#
#         yarn add --dev typescript @types/react @types/node
#
# ...
```

You're now ready to start converting files from `.js` to `.tsx` and leveraging the benefits of TypeScript!.

**A file named `ext-env.d.ts`will be created in the root of your project. This file ensures Next.js types are picked up by the TypeScript compiler. `ou cannot remove it` however, you can edit it `but you don't need to)`**

**TypeScript `strict mode` is `turned off` by default. When you feel comfortable with TypeScript, `it's recommended to turn it on in your tsconfig.json.`**

By default, Next.js will do type checking as part of next build. We recommend using code editor type checking during development.

### Static Generation and Server-side Rendering

For `getStaticProps`, `getStaticPaths`, and `getServerSideProps`, you can use the `GetStaticProps`, `GetStaticPaths`, and `GetServerSideProps` types respectively:

```js
import { GetStaticProps, GetStaticPaths, GetServerSideProps } from 'next'

export const getStaticProps: GetStaticProps = async (context) => {
  // ...
}

export const getStaticPaths: GetStaticPaths = async () => {
  // ...
}

export const getServerSideProps: GetServerSideProps = async (context) => {
  // ...
}
```


## Routing

Next.js has a file-system based router built on the concept of pages.

When a file is added to the pages directory it's automatically available as a route.


### Index Routes

The router will automatically route files named `index` to the `root` of the directory.

-`pages/index.js → /`
-`pages/blog/index.js → /blog`
