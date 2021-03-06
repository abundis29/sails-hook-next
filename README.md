# sails-hook-next

Next.js Hook for Sails

The idea is to completely integrate the Next.js framework with a Sails API
so that we can have the power of a server-rendered React app and an awesome REST API.

## Installation

Initialize a Sails project with no frontend:

```
sails new my-project --no-frontend && cd my-project
```

Then install the hook using npm. You also need to include its dependencies.

```
npm install --save sails-hook-next react react-dom next
```

## Compatibility

The v2.0 of `sails-hook-next` is only compatible with latest major releases of Next.js and Sails.

| Framework | Version |
| --- | --- |
| Next.js | >= 6.0.0 |
| Sails | >= 1.0.0 |

For compatibility with Next.js <= 4.x.x and Sails 0.12.x [checkout the v1.1.0](https://github.com/RasCarlito/sails-hook-next/tree/1.1.0) of the hook.

## Usage

Create a `pages` folder at the root of your project to store your Next.js pages. Example:

```js
// pages/index.js
export default () => (
  <div>
    <h1>Hello Next.js</h1>
  </div>
)
```

A custom `.babelrc` configuration is necessary for Next.js components.

```js
{
  "presets": ["next/babel"]
}
```

*For more info see the [Next.js documentation](https://github.com/zeit/next.js) or the awesome [learnnextjs.com](https://learnnextjs.com/) tutorial*

The only necessary Sails configuration is that all your API routes are prefixed with `/api`.
This can be achieved for blueprints by setting the `prefix` key to `/api` in your `config/blueprints.js` file.

Then just lift your Sails application to run in development mode:

```
sails lift
```

### Custom routes

To handle custom routes we need to configure a Sails controller to render the correct Next.js page.

Define a route pointing to the controller:

```js
// config/routes.js
module.exports = {
  // Listen to GET requests to the /blog/:articleId pattern
  // and route requests to the `BlogController.artcile()` method
  'get /blog/:articleId': 'BlogController.article'
}
```

Define the controller which calls `sails.config.next.app.render()` method passing the route parameters along:

```js
// api/controllers/BlogController.js
module.exports = {
  article (req, res) {
    // Extract `params` which contains the `articleId` parsed from the URL path
    // and `query` parameters parsed as an object
    const { params, query } = req

    // Render the Next.js `pages/blog.js` component passing the `query` and `params`
    // merged together so that the `query` can still get accessed by the `getInitialProps()` method.
    sails.config.next.app.render(req, res, '/blog', { ...params, ...query })
  }
}
```

Define the Next.js blog page which receives the parameters through the `query` parameter in the `getInitialProps()` method:

```js
// pages/blog.js

// React stateless component receiving the `articleId` through the `props`
const BlogPage = ({ articleId }) => (
  <div>
    <h1>My {articleId} blog post</h1>
  </div>
)

// Next.js `getInitialProps` receiving the `articleId` through the `query` parameters
// and returning it for the component to get it through the `props`
BlogPage.getInitialProps = ({ query: { articleId } }) => {
  return { articleId }
}

export default BlogPage
```

Linking to the blog page using the Next.js `Link` component:

```js
// pages/index.js
import Link from 'next/link'

export default () => {
  return (
    <div>
      <h1>Home</h1>
      <ul>
        <li>
          {/*
            Link to a first blog post passing the `articleId` as query parameter
            and aliasing to the url slug `/blog/first`
          */}
          <Link href='/blog?articleId=first' as='/blog/first'>
            <a>My first blog</a>
          </Link>
        </li>
        <li>
          {/* Link to another blog post */}
          <Link href='/blog?articleId=second' as='/blog/second'>
            <a>My second blog</a>
          </Link>
        </li>
      </ul>
    </div>
  )
}
```

You can find more information in the [Next.js custom routes](https://github.com/zeit/next.js/#custom-server-and-routing) documentation and the [Sails url-slugs routes](https://sailsjs.com/documentation/concepts/routes/url-slugs) documentation.

### Hook configuration

You can override hook configuration by creating a `config/next.js` file in your Sails application.

Default configuration:

```js
module.exports.next = {
  // Sails integration options
  api: {
    // Prefix for all Sails API routes
    prefix: '/api'
  },

  // Next.js instance options. Passed to `next()`.
  server: {
    // Next.js root directory
    dir: '.',
    // Dev mode. Is overridden by `process.env.NODE_ENV !== 'production'`
    dev: false,
    // Hide error messages
    quiet: false,
    // Equivalent to a `next.config.js` file
    conf: {}
  }
}
```

### Production

To run in production mode you need to build your Next.js application

```
next build
```

_If `next` is not installed globally you can run it using `npx next build`._

Then run Sails in production mode using

```
NODE_ENV=production node app.js
```

For more information see the [Next.js](https://github.com/zeit/next.js/#production-deployment) and [Sails](https://sailsjs.com/documentation/concepts/deployment) deployment documentation.

### Interactions between Next.js and Sails

* The Next.js app instance can be accessed anywhere with `sails.config.next.app`.
* The Next.js request handler for SSR is attached to `sails.config.next.handle`.

## sails-hook-next in the wild

Don't have a real example using the latest version of `sails-hook-next` yet.

If you have an example don't hesitate to add it here by submitting a pull request.
