**💛 You can help the author become a full-time open-source maintainer by [sponsoring him on GitHub](https://github.com/sponsors/eliushhimel).**
Originally created by [EGOIST](https://github.com/egoist/vite-plugin-mix)
---

# vite-plugin-mix

[![npm version](https://badgen.net/npm/v/@eliushhimel/vite-plugin-mix)](https://www.npmjs.com/package/@eliushhimel/vite-plugin-mix)
[![npm downloads](https://badgen.net/npm/dt/@eliushhimel/vite-plugin-mix)](https://www.npmjs.com/package/@eliushhimel/vite-plugin-mix)

## Motivation

Writing front-end and back-end API in a single project allows faster development (imo), this plugin essentially brings Next.js' API routes to your Vite app.

## Update from `vite-plugin-mix` to `@eliushhimel/vite-plugin-mix`
This project has been renamed to `@eliushhimel/vite-plugin-mix` to avoid confusion with another project named `vite-plugin-mix` which is the origin of this package. I needed to use it and found that the original author @egoist no longer maintain the project, and the last update was for `vite: ^3`, so I decided to update it for the latest version. If you are using the old package, and after updating vite it stopped working then please uninstall it and install the new package:

```bash
npm uninstall vite-plugin-mix
npm install @eliushhimel/vite-plugin-mix -D
```

Supported Vite versions: `^7`

## Usage

`vite.config.ts`:

```ts
import { defineConfig } from 'vite'
import mix from '@eliushhimel/vite-plugin-mix'

export default defineConfig({
  plugins: [
    mix({
      handler: './handler.ts',
    }),
  ],
})
```

`handler.ts`:

```ts
import type { Handler } from '@eliushhimel/vite-plugin-mix'

export const handler: Handler = (req, res, next) => {
  if (req.path === '/hello') {
    return res.end('hello')
  }
  next()
}
```

The `handler` runs before serving static files, so you should make sure to call `next()` as a fallback. You can also use express-compatible middlewares in the handler.

To start developing, run the command `vite` as usual.

To create a production build, run the command `vite build` as usual.

Now `vite build` will create a server build to `./build` folder alongside your regular client build which is the `./dist` folder by default. To run the production build as a Node.js server, run `node build/server.mjs` or if you have `"type": "module"` in your `package.json`, run `node build/server.js` instead.

### Request flow

<img src="https://user-images.githubusercontent.com/8784712/116026214-d424af80-a684-11eb-9126-b188d7976be2.png" width="300" alt="request flow">

## Adapters

### Node.js

By default the server is built for Node.js target, you can run `node build/server.mjs` or `node build/server.js` after `vite build` to start the production server.

By default the server runs at port `3000`, you can switch to a custom port by using the `PORT` environment variable.

### Vercel

> **Warning**
>
> This may not work with some packages in a monorepo when using pnpm.

To build for [Vercel](https://vercel.com), use the `vercelAdapter` in `vite.config.ts`:

```ts
import { defineConfig } from 'vite'
import mix, { vercelAdapter } from '@eliushhimel/vite-plugin-mix'

export default defineConfig({
  plugins: [
    mix({
      handler: './handler.ts',
      adapter: vercelAdapter(),
    }),
  ],
})
```

Then you can run `vite build` to build for Vercel.

## Guide

### Using Express

```ts
import express from 'express'

const app = express()

export const handler = app
```

### Using Polka

```ts
import polka from 'polka'

export const handler = (req, res, next) => {
  const app = polka({
    onNoMatch: () => next(),
  })

  return app.handler(req, res)
}
```

### Using Apollo GraphQL

```ts
import { ApolloServer } from 'apollo-server-micro'
import { typeDefs } from './schemas'
import { resolvers } from './resolvers'

const apolloServer = new ApolloServer({ typeDefs, resolvers })

const GRAPHQL_ENDPOINT = '/api/graphql'

const apolloHandler = apolloServer.createHandler({ path: GRAPHQL_ENDPOINT })

export const handler = (req, res, next) => {
  if (req.path === '/api/graphql') {
    return apolloHandler
  }
  next()
}
```

You can also use `express` + `apollo-server-express` if you want.

## Common Issues
### `Error: mix is not a function`
I'll fix it later, but for now, this is caused by the way how the package is exported. The package is exported in a way that is compatible with both CommonJS and ES Modules, but sometimes TypeScript or your bundler might not interpret it correctly.

Workaround this issue by importing like this:
`TypeScript`
```ts
import { defineConfig } from 'vite'
import mixPlugin, { Adapter } from '@eliushhimel/vite-plugin-mix'

interface MixConfig {
  handler: string
  adapter?: Adapter | undefined
}
type MixPlugin = (config: MixConfig) => Plugin

interface Mix {
  default: MixPlugin
}
const mix = (mixPlugin as unknown as Mix).default

export default defineConfig({
  plugins: [
    mix({
      handler: './handler.ts',
    }),
  ],
})
```

`JavaScript`
```js
import mixPlugin from 'vite-plugin-mix'

const mix = mixPlugin.default
```


## License

MIT &copy;
