
## What is Deno?

- A runtime environment for Javascript and Typescript based on V8 built with Rust.

--- 

# Why Deno?

1. Security
2. No NPM
3. Run stuff without downloading
4. Distributed dependency system
5. Simple distribution
6. Modern architecture
7. No compiled modules 

----

# Getting started

---

## Installation


```sh
curl -fsSL https://deno.land/x/install/install.sh | sh

# or
brew install deno

# On windows (PowerShell)
iwr https://deno.land/x/install/install.ps1 -useb | iex

```

----

# Hello world

```sh
deno run https://deno.land/std/examples/welcome.ts
```

```js
// hello-server.js

import { serve } from "https://deno.land/std@0.52.0/http/server.ts";
const s = serve({ port: 8000 });
console.log("http://localhost:8000/");
for await (const req of s) {
  req.respond({ body: "Hello World\n" });
}
```

----

## Running Deno

```sh
deno run ./hello-server.js

error: Uncaught PermissionDenied: network access to "0.0.0.0:8000", run again with the --allow-net flag
    at unwrapResponse ($deno$/ops/dispatch_json.ts:43:11)
    at Object.sendSync ($deno$/ops/dispatch_json.ts:72:10)
    at Object.listen ($deno$/ops/net.ts:51:10)
    at listen ($deno$/net.ts:152:22)
    at serve (https://deno.land/std@0.52.0/http/server.ts:261:20)
    at file:///Users/assafgannon/Academy/deno/demo-app/hello-server.js:2:11
```

---

## Deno Permissions

- --allow-net
- --allow-read
- --allow-write
- -A (for dev only)

---

## Running Typescript

```typescript
import { serve, Server, ServerRequest } from 'https://deno.land/std@0.52.0/http/server.ts';
const s: Server = serve({ port: 8000 });
console.log('http://localhost:8000/');
for await (const req: ServerRequest of s) {
    req.respond({ body: 'Hello World\n' });
}

```

---

## Standard Libraries [deno.land/std](https://deno.land/std)

```sh
archive
async
bytes
datetime
encoding
examples
flags
fmt
fs
hash
http
io
log
mime
node
path
permissions
signal
testing
textproto
uuid
ws
```

---

## Simple FS example

```ts
import { readFileStr } from 'https://deno.land/std/fs/mod.ts';

const text = await readFileStr('./read-file.ts');

console.log(text);
```

---

## Testing (using https://deno.land/std/testing)

```ts
import { assertEquals } from 'https://deno.land/std/testing/asserts.ts';

Deno.test('isEqual', function (): void {
    const a = { x: 1 };
    const b = { x: 1 };
    assertEquals(a, b);

    assertEquals('hello', 'hello')
});

```

```deno test *.spec.*```

---

# Workshop

## Creating a Deno-based Microservice

full repo at [https://github.com/assafg/deno-mongo-basic-demo](https://github.com/assafg/deno-mongo-basic-demo)
---

## main.ts

```typescript
import { listenAndServe } from 'https://deno.land/std@0.52.0/http/server.ts';

const port = Number(Deno.env.get('PORT') || `8009`);

console.log(`http://localhost:${port}/`);

listenAndServe({ port }, async (req) => {
    if (req.method === "GET" && req.url === "/") {
        req.respond({
            body: JSON.stringify([])
        })
    }
})

```

`deno run --allow-env --allow-net main.ts`

---

## Oak - Express/Koa for Deno

```ts
import { Application, Router } from 'https://deno.land/x/oak/mod.ts';

const app = new Application();

const router = new Router();
router.get('/', ({ response }: { response: any }) => {
    response.body = 'hello Oak';
});

app.use(router.routes());
app.use(router.allowedMethods());

const port = Number(Deno.env.get('PORT') || `8009`);
console.log(`Listening on port ${port} ...`);
await app.listen(`localhost:${port}`);

```

---

## Adding Mongodb

docker-compose.yaml

```yaml
version: "2.1"

services:
  
  mongo:
    image: mongo
    volumes:
      - ${PWD}/mongo-data:/data/db'
    ports:
      - 27017:27017
```

`docker-compose up -d`

---

## main.ts

```ts
import { Application, Router } from 'https://deno.land/x/oak/mod.ts';
import { MongoClient, ObjectId } from 'https://deno.land/x/mongo@v0.7.0/mod.ts';

const client = new MongoClient();
client.connectWithUri('mongodb://localhost:27017');

const db = client.database('test');
const users = db.collection('users');

const app = new Application();

const router = new Router();
router
    .get('/', ({ response }: { response: any }) => {
        response.body = 'hello Oak';
    })
    .get('/users/:id', async ({ params, response }: { params: { id: string }; response: any }) => {
        const { id } = params;
        const user = await users.findOne({ _id: ObjectId(id) });
        response.body = user;

    }).post('/users', async ({ request, response }: {request: any, response: any}) => {
        const { value } = await request.body();
        console.log( value );
        const insertId = await users.insertOne(value);
        response.status = 201;
        response.body = insertId;
    });

app.use(router.routes());
app.use(router.allowedMethods());

const port = Number(Deno.env.get('PORT') || `8009`);
console.log(`Listening on port ${port} ...`);
await app.listen(`localhost:${port}`);

```

---

## Dockerize the service

Dockerfile

```Dockerfile
FROM hayd/alpine-deno:1.0.0

EXPOSE 8000

WORKDIR /app

USER deno

COPY . . 
RUN deno cache hello-server.ts

CMD ["run", "--allow-env", "--allow-net", "--allow-read", "--allow-write", "--allow-plugin", "--unstable", "hello-server.ts"]

```

* Has build issues: see [https://github.com/denoland/deno/issues/5888](https://github.com/denoland/deno/issues/5888)
