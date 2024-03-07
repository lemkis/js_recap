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
// src/index.js
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
    "start": "node dist/index.js",
    "dev": "nodemon src/index.ts"
  }
}
```
Now you can build `index.js` via `npm run build`. Try `npm run dev` out to see if
`nodemon` works.
10. You can configure `nodemon` via creating `nodemon.json` file or adding object to
    `package.json`. For more details read [nodemon documentation](https://www.npmjs.com/package/nodemon).
