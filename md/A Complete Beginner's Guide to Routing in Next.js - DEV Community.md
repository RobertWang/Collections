> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [dev.to](https://dev.to/ibrahima92/a-complete-beginner-s-guide-to-routing-in-next-js-3e67)

> Next.js is a React framework that ships with all the features you need for production. Next.js enable......

Next.js is a React framework that ships with all the features you need for production. Next.js enables routing in your app by using the file-system-based routing. In this guide, I will show you how routing works in Next.js.

> This article is part of a series about Next.js. Feel free to [subscribe to my newsletter](https://ibrahima-ndaw.us5.list-manage.com/subscribe?u=8dedf5d07c7326802dd81a866&id=5d7bcd5b75) to receive the next one in your inbox.

> If you're interested in learning Next.js in a comprehensive way, I highly recommend this [bestseller course](https://click.linksynergy.com/deeplink?id=o1JCNdqL0gw&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Freact-the-complete-guide-incl-redux%2F).

_Let's dive in._

*   [How Routing works in Next.js?](#how-routing-works-in-nextjs)
*   [Linking between pages](#linking-between-pages)
*   [Passing route parameters](#passing-route-parameters)
*   [Dynamic routes](#dynamic-routes)
*   [Dynamic nested routes](#dynamic-nested-routes)

How Routing works in Next.js?
-----------------------------

Next.js uses the file system to enable routing in the app. Next treats automatically every file with the extensions `.js`, `.jsx`, `.ts`, or `.tsx` under the `pages` directory as a route. A page in Next.js is a React component that has a route based on its file name.

Consider this folder structure as an example:  

```
├── pages
|  ├── index.js
|  ├── contact.js
|  └── my-folder
|     ├── about.js
|     └── index.js
```

Here, we have four pages:

*   `index.js` is the home page accessible on _[http://localhost:3000](http://localhost:3000/)_
*   `contact.js` is the contact page accessible on _[http://localhost:3000/contact](http://localhost:3000/contact)_
*   `my-folder/index.js` is the page located on the sub-folder _my-folder_ accessible on _[http://localhost:3000/my-folder](http://localhost:3000/my-folder)_
*   `my-folder/about.js` is the about page located on the sub-folder _my-folder_ accessible on _[http://localhost:3000/my-folder/about](http://localhost:3000/my-folder/about)_

Linking between pages
---------------------

By default, Next.js pre-renders every page to makes your app fast and user-friendly. It uses the `Link` component provided by _next/link_ to enable transitions between routes.  

```
import Link from "next/link"

export default function IndexPage() {
  return (
    <div>
      <Link href="/contact">
        <a>My second page</a>
      </Link>
      <Link href="/my-folder/about">
        <a>My third page</a>
      </Link>
    </div>
  )
}
```

Here, we have two routes:

*   The first link leads to the page `http://localhost:3000/contact`
*   The second link leads to the page `http://localhost:3000/my-folder/about`

The `Link` component can receive several properties, but only the `href` attribute is required. Here, we use a `<a></a>` tag as a child component to link pages. But, you can use alternatively any element that supports the `onClick` event on the `Link` component.

Passing route parameters
------------------------

Next.js allows you to pass route parameters and then get back the data using the `useRouter` hook or `getInitialProps`. It gives you access to the router object that contains the params.

*   index.js

```
import Link from "next/link"

export default function IndexPage() {
  return (
    <Link
      href={{
        pathname: "/about",
        query: { id: "test" },
      }}
    >
      <a>About page</a>
    </Link>
  )
}
```

As you can see here, instead of providing a string to the `href` attribute, we pass in an object that contains a `pathname` property which is the route, and a query element that holds the data.

*   about.js

```
import { useRouter } from "next/router"

export default function AboutPage() {
  const router = useRouter()
  const {
    query: { id },
  } = router
  return <div>About us: {id}</div>
}
```

Here, we import the `useRouter` hook to get the data passed in. Next, we pull it from the `query` object using destructuring.

If you are using server-side rendering, you have to use `getInitialProps` to get the data.  

```
export default function AboutPage({ id }) {
  return <div>About us: {id}</div>
}

AboutPage.getInitialProps = ({ query: { id } }) => {
  return { id }
}
```

Dynamic routes
--------------

Next.js enables you to define dynamic routes in your app using the brackets (`[param]`). Instead of setting a static name on your pages, you can use a dynamic one.

Let's use this folder structure as an example:  

```
├── pages
|  ├── index.js
|  ├── [slug].js
|  └── my-folder
|     ├── [id].js
|     └── index.js
```

Next.js will get the route parameters passed in and then use it as a name for the route.

*   index.js

```
export default function IndexPage() {
  return (
    <ul>
      <li>
        <Link href="/">
          <a>Home</a>
        </Link>
      </li>
      <li>
        <Link href="/[slug]" as="/my-slug">
          <a>First Route</a>
        </Link>
      </li>
      <li>
        <Link href="/my-folder/[id]" as="/my-folder/my-id">
          <a>Second Route</a>
        </Link>
      </li>
    </ul>
  )
}
```

Here, we have to define the value on the `as` attribute because the path is dynamic. The name of the route will be whatever you set on the `as` prop.

*   [slug].js

```
import { useRouter } from "next/router"

export default function DynamicPage() {
  const router = useRouter()
  const {
    query: { id },
  } = router
  return <div>The dynamic route is {id}</div>
}
```

You can get the route parameters as well with the `useRouter` hook on the client or `getInitialProps` on the server.

*   my-folder/[id].js

```
export default function MyDynamicPage({ example }) {
  return <div>My example is {example}</div>
}

MyDynamicPage.getInitialProps = ({ query: { example } }) => {
  return { example }
}
```

Here, we use `getInitialProps` to get the dynamic route passed in.

Dynamic nested routes
---------------------

With Next.js, you can also nest dynamic routes with the brackets (`[param]`).

Let's consider this file structure:  

```
├── pages
|  ├── index.js
|  └── [dynamic]
|     └── [id].js
```

*   index.js

```
export default function IndexPage() {
  return (
    <ul>
      <li>
        <Link href="/">
          <a>Home</a>
        </Link>
      </li>
      <li>
        <Link href="/[dynamic]/[id]" as="/my-folder/my-id">
          <a>Dynamic nested Route</a>
        </Link>
      </li>
    </ul>
  )
}
```

As you can see here, we set the dynamic values on the `as` attribute as we did in the previous example. But this time, we have to define the name of the folder and its file.  

```
import { useRouter } from "next/router"

export default function DynamicPage() {
  const router = useRouter()
  const {
    query: { dynamic, id },
  } = router
  return (
    <div>
      Data: {dynamic} - {id}
    </div>
  )
}
```

Here, we pull out the route parameters from the query object with the `useRouter` hook.

That's it! Thanks for reading

You can find other great content like this on [my blog](https://www.ibrahima-ndaw.com/) or follow me [on Twitter](https://twitter.com/ibrahima92_) to get notified.

Photo by [Javier Allegue Barros](https://unsplash.com/@soymeraki?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/route?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)