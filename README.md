# Routar ![npm](https://img.shields.io/npm/v/routar.svg) ![Travis (.com)](https://img.shields.io/travis/com/Cretezy/routar.svg) ![Codecov](https://img.shields.io/codecov/c/github/Cretezy/routar.svg)

Modern Node router.

Features:

- Easy to use, good performance
- Modern API, native Promise support, fully typed (TypeScript)
- Close compatibility with Express/Koa, fast migration
- Few dependencies (currently: 1), small API surface, easy to fully understand
- 100% code coverage, well tested

## Usage

Install:

```bash
yarn add routar
# or
npm add routar
```

Setup your app:

```js
const { Routar, TextBody } = require("routar");

const port = process.env.PORT || 3000;
const app = new Routar();

app.get("/", ctx => {
  ctx.body = new TextBody("Hello world!");
});

app.listen(port).then(() => console.log(`Listening on ${port}`));
```

[Full example](example)

### Routes

Routing in Routar is slightly different from other routes, but is targeted towards making it much simpler.

To start, you can use the `.get`, `.post`, `.delete`, `.patch`, `.put`, and `.any` (all aliasing to `.route`) to attach single routes to a router. These methods are chainable, and can be in any order (uses exact match):

```js
const { TextBody, JsonBody } = require("routar");

app
  .get("/", ctx => {
    ctx.body = new TextBody("GET /");
  })
  .post("/submit", ctx => {
    ctx.body = new TextBody("POST /submit");
    ctx.statusCode = 400;
  })
  .get("/json", ctx => {
    ctx.body = new JsonBody({ name: "john" });
  })
  .get(
    "/catch",
    ctx => {
      ctx.body = new TextBody("GET /catch/*");
    },
    { exact: false }
  );

// Long form
app.route("POST", "/", ctx => {
  ctx.body = new TextBody("GET /");
});

app.any("/", ctx => {
  // Will catch all other methods on /
  ctx.body = new TextBody("DELETE/PUTCH/PUT /");
});
```

### Child Routers

Child routers are useful to functionally split your application in smaller units:

```js
app.child("/child").get("/", ctx => {
  ctx.body = new TextBody("GET /child");
});

// Or

const { Router } = require("routar");

const parentsRouter = new Router();

parentsRouter.get("/", ctx => {
  ctx.body = new TextBody("GET /parents");
});

app.child("/parents", parentsRouter);
```

### Handler

A handler can be:

- A function (`(ctx) => ...`)
- A Promise (`async (ctx) => ...`)
- A router (`new Router()`)
- A list of handlers (`[middleware, (ctx) => ..., router]`)

### Middlewares

Middlewares are triggered at the start and end of handing a router. A middleware is a handler that can return a function/Promise (to be called at the end of the request):

```js
app
  .middleware(ctx => {
    // Attaches data to the request in the root router
    ctx.data.name = "john";
  })
  .get("/", ctx => {
    ctx.body = new JsonBody({ name: ctx.data.name });
  });

app
  .child("/child")
  .middleware(ctx => {
    return () => {
      // Will append ' ... smith!' to all requests in this router
      ctx.res.write(" ... smith!");
    };
  })
  .get("/", ctx => {
    ctx.res.write(`My name is ${ctx.data.name}`);
  });
```

#### Express Middlewares

Routar has built-in support for Express/Connect/callback style middlewares.

```js
const { useExpress, JsonBody } = require("routar");
const bodyParser = require("body-parser");

app.middleware(useExpress(bodyParser.json()));

app.post("/", ctx => {
  ctx.body = new JsonBody({ data: ctx.req.body });
});
```

This enables the `(req, res) => ...` or `(req, res, next) => ...` syntax.

### Listening

Using `app.listen` is a simple way to start your Routar server:

```js
// Can parse port from string
app.listen(process.env.PORT || 3000);

// Will randomly assign port
app.listen();

// Returns a Promise with port and server
app.listen().then(async ({ port, server, close }) => {
  console.log(`Listening on :${port}`);
  console.log(`Max connections: ${server.maxConnections}`);

  // Wait 1s
  await new Promise(resolve => setTimeout(resolve, 1000));

  // Close server,
  await close();
});
```

To use in testing or in `http.createServer`, you can use `app.handler`:

```js
const server = http.createServer(app.handler);

// Using supertest

request(app.handler);
```

## Support

We support all currently active and maintained [Node LTS versions](https://github.com/nodejs/Release), include current Node versions.

Please file feature requests and bugs at the [issue tracker](https://github.com/Cretezy/routar/issues).
