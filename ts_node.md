# nodejs with ts and express

1. Firstly make sure that `npm` and `node.js` is installed on you computer.
2. Initialize configuration file via`npm init -y` (in the root dir)
3. Install `npm i express dotenv` (`dotenv` manages environment variables in file `.env`).
Now create file `.env` in the root dir and add to it line `PORT=3000` e.g. via `echo PORT=3000 >> .env`
4. Install typescript, express and typings for ts (see [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped))
via `npm i -D typescript @types/express @types/node` (see what was added to `package.
   json` file)
5. Create configuration file for ts via `npx tsc --init`. Change the output dir to `./build`
6. Create `index.ts` file in the root dir. Add to it
```typescript
import express, { Express, Request, Response } from "express";
import dotenv from "dotenv";

dotenv.config();

const app: Express = express();
const port = process.env.PORT || 3000;

app.get("/", (req: Request, res: Response) => {
  res.send("Express + TypeScript Server");
});

app.listen(port, () => {
  console.log(`[server]: Server is running at http://localhost:${port}`);
});
```
7. Run this server via `npx ts-node index.ts`. Add script `./scripts/run.sh` which
   will run this application.
8. Install `nodemon` via `npm i -D nodemon ts-node`. It restarts an application when
   changes in a file are detected.
9. Add the following to `package.json`,
```json
{
  "scripts": {
    "build": "npx tsc",
    "start": "node build/index.js",
    "dev": "nodemon index.ts"
  }
}
```
Now you can build `index.js` via `npm run build`. Try `npm run dev` out to see if
`nodemon` works. Note that starting from node 19 there is more modern approach via `"dev": "node --watch -r ts-node/register index.ts"`, in particular, there is no need for nodemon. You can configure `nodemon` via creating `nodemon.json` file or adding object to
    `package.json`. For more details read [nodemon documentation](https://www.npmjs.com/package/nodemon).
# At the end read docs

   - [console](https://nodejs.org/api/console.html)
   - [errors](https://nodejs.org/api/errors.html)
   - [events](https://nodejs.org/api/events.html)
   - [file system](https://nodejs.org/api/fs.html)
   - [globals](https://nodejs.org/api/globals.html)
   - [os](https://nodejs.org/api/os.html)
   - [path](https://nodejs.org/api/path.html)
   - [process](https://nodejs.org/api/process.html)
   - [url](https://nodejs.org/api/url.html)
   - [test runner](https://nodejs.org/api/test.html)
   - [assert](https://nodejs.org/docs/latest/api/assert.html)
   - learn also about new features  e.g. [fetch api](https://dev.to/andrewbaisden/the-nodejs-18-fetch-api-72m)
Some examples:
```typescript
import {test, mock} from 'node:test';
import assert from 'node:assert';
import fs from 'node:fs';
import url from 'node:url'
import path from 'node:path'
import { cwd } from 'node:process';
import { URL } from 'node:url';
import {readFile} from 'node:fs/promises';


try {
    test('asynchronous passing test', async t => {
        // This test passes because it does not throw an exception.
        assert.strictEqual(await readFile('a.txt', {encoding: 'utf8'}), 'Hello World');
    });
} catch (err:unknown) {
    if (err instanceof  Error) {
        console.error(err.message);
    } else {
        console.error("non instance of Error was thrown")
    }
}


const getAPI = async () => {
    const res = await fetch('https://nodejs.org/api/documentation.json');
    if (res.ok) {
        const data = await res.json();
        console.log(data);
    }
};

getAPI();
console.log(`Current directory: ${cwd()}, file name ${path.basename(cwd())}`);


```

