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


# ORM - prisma

ORM (object relational mapping) is a technique that lets you query and manipulate data
from a database using an object-oriented paradigm. Prisma is a library which implements
ORM. Prisma uses `models` (think about them as about database tables) to represent
entities of your application. Models are mapped to either tables (relational databases
like PostgreSQL) or collections (MongoDB). Below you will see how it works; to
see some more details
visit [docs for models](https://www.prisma.io/docs/orm/prisma-schema/data-model/models).

1. Why prisma? There are many ORM frameworks. In your first project you are
   allowed to use the one you most like. In this tutorial we introduce prisma because
   it is easy to use, well documented, popular, has many examples, can work with many
   databases including `postgresql`, `mysql`, `sqlite`, `sqlserver`, `mongodb` or
   `cockroachdb`. Below just for demonstration we use `sqlite`. Last but not least prisma
   supports typescript which gives you the benefits of type-safety at zero cost by
   auto-generating types from your database schema. Just perfect for our ts-based
   project :)
2. Install `npm install prisma --save-dev`
3. Set up Prisma ORM with the init command of the Prisma CLI, `npx prisma init
   --datasource-provider sqlite`. This creates a new `prisma` directory with your Prisma
   schema file and configures SQLite as your database.
   At the end read the info displayed after the execution of
   this command. In particular, set the `DATABASE_URL` in the `.env` file to point to
   your existing database, which in our case is `DATABASE_URL="file:./database/example.db"` (to see other type of connections
   see [connection urls](https://www.prisma.io/docs/orm/reference/connection-urls))
4. You're now ready to model your data and create your
   database with some tables. Open newly created file `prisma/schema.prisma` (prisma
   framework uses `.prisma`
   files to model data). Add to it

```
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User    @relation(fields: [authorId], references: [id])
  authorId  Int
}
```

5. Generate database with `User` and `Post` tables via
   ```npx prisma migrate dev --name user_and_post```
   Note that the database was created at `./prisma/database/example.db`. In fact this
   command
   did
   three
   things:

- It created a new SQL migration file for this migration in the `prisma/migrations`
  directory.
- It executed the SQL migration file against the database.
- It ran prisma generate under the hood (which installed the @prisma/client package and
  generated a tailored Prisma Client API based on your models).

If in the future you will need to e.g. add a field to `User`
see [how to do it](https://www.prisma.io/docs/orm/prisma-migrate/getting-started)

6. Execute `npm install @prisma/client` (in fact it should be already installed - just
   in case). Add the following code to
   `index.ts`,

```typescript
import {PrismaClient} from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
    // ... you will write your Prisma Client queries here
}

main()
    .then(async () => {
        await prisma.$disconnect()
    })
    .catch(async (e) => {
        console.error(e)
        await prisma.$disconnect()
        process.exit(1)
    })
```

Make sure it runs without errors.

7. Now let's play with [CRUD](https://www.prisma.io/docs/orm/prisma-client/queries/crud).
   Firstly, create a new `user` record by adding

```typescript
  const user = await prisma.user.create({
    data: {
        name: 'Alice',
        email: 'alice@prisma.io',
    },
})
console.log(user)
```

to the body of the `main` function. You should obtain something like

```json
{
  "id": 1,
  "email": "alice@prisma.io",
  "name": "Alice"
}
```

8. You can retrieve records using `findMany` function. Append

```typescript
const users = await prisma.user.findMany()
console.log(users)
```

to the `main` function. Oops, ts does not compile, because you are trying to
insert a new user with existing `id` which should be unique. To fix this, either
change code to create a new user only if it does not exist (you can
use [findUnique](https://www.prisma.io/docs/orm/reference/prisma-client-reference#findunique))
or
comment
creation part of
code.

9. Now see how you can create a new user record with (nested) new post record (replace
   the body of `main`)

```typescript
const user = await prisma.user.create({
    data: {
        name: 'Bob',
        email: 'bob@prisma.io',
        posts: {
            create: {
                title: 'Hello World',
            },
        },
    },
})
console.log(user)
```

Note that by default, Prisma Client only returns scalar fields in the result objects
of a query. To print `posts` append to the `main`

```typescript
const usersWithPosts = await prisma.user.findMany({
    include: {
        posts: true,
    },
})
console.dir(usersWithPosts, {depth: null})
```

10.
Read [an introduction to express api](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Introduction)
to get a feeling how
express works and then analyze
[prisma example](https://github.com/prisma/prisma-examples/blob/latest/typescript/rest-express/src/index.ts)

11. Now you are ready to create a database for the first project :) In case of any doubts
    questions you can always see [prisma docs](https://www.prisma.io/docs).




# Dynamic html pages, views

1. Firstly install handlebars via `npm install express-handlebars`
2. Prepare views directory: create `./views`  and `./views/layouts` directories.
3. Now let see how to use handlebars with express. Add the following lines to you code
   (below `app = express()` stands for the express application):

```typescript
import {engine} from 'express-handlebars';

app.engine('.hbs', engine({extname: '.hbs'}));
app.set('view engine', '.hbs');
app.set('views', './views');
```

Let us analyse what happends in this snippet. Firstly we import `engine` and we create
instance of `engine` in line 2 where configuration `{extname: '.hbs'}` is passed to the
constructor (application now knows that `.hbs` is the extension of our views. In the next
line we let know our application that we use handlebars engine as the rendering engine.
In the last line we tell our application that our views are stored in the `views`
directory. Summing it up, after these lines are executed our application knows where
to look for the files, what is the extension of view file and how to render such a file.
Note that, under the hood, in the constructor of engine, the default parameter
`layoutsDir`
is set
to `express settings.view + layouts/` see
[documentation](https://www.npmjs.com/package/express-handlebars).
Nota bene the configuration of the engine is of the form

```typescript
export interface ConfigOptions {
    handlebars?: HandlebarsImport;
    extname?: string;
    encoding?: BufferEncoding;
    layoutsDir?: string;
    partialsDir?: string | string[] | PartialsDirObject | PartialsDirObject[];
    defaultLayout?: string | false;
    helpers?: UnknownObject;
    compilerOptions?: CompileOptions;
    runtimeOptions?: Handlebars.RuntimeOptions;
}
```

4. Now let us see how to render a home view in express. To this end create file `.
   /views/home.hbs` and paste to it

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Example App - Home</title>
</head>
<body>
{{#if showTitle}}
<h1>Personal webpage</h1>
{{/if}}
<p>Hello my name is {{name}}!</p>
</body>
</html>
```

Some comments are in place. Firstly note that double curly (moustache) brackets are used
for the placeholders `{{#if showTitle}}`, `{{/if}}` and `{{name}}`. The `showTitle`
and `name` are just parameters which our application will pass to this template and
when it happens our renderer (handlebars) will replace these placeholders by the
passed values. E.g.
if our
applications passes
object `{showTitle:true, name="John"}` then the above template will be rendered to

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Example App - Home</title>
</head>
<body>
<h1>Personal webpage</h1>
<p>Hello my name is John!</p>
</body>
</html>
```

Finally, the `if` statement allows to render parts of view conditionally on passed
values. In our case we do or don't display `<h1>Personal webpage</h1>` depending on the
passed
value
of parameter `showTitle`.

Now, replace our old root path handler by

```typescript
app.get('/', (req, res) => {
    res.render('home', {
        layout: false,
        showTitle: true,
        name: "John"
    });
});
```

As mentioned above we pass `showTitle` and `name` to the `home.hbs` view so that our
application can render this view. Note that additionally we passed `layout:false`
which tells the renderer not to use layout (a layout is a concept in handlebar which
we will explain later on).
Now try if your application works as expected!

5. Caching. Handlebars view engine uses a smart template caching strategy. In
   development, templates will always be loaded from disk, i.e., no caching. In
   production, raw files and compiled Handlebars templates are aggressively cached.
   The easiest way to control template/view caching is through Express' view cache
   setting:
   ```
   app.enable('view cache')
   ```
   Express enables this setting by default when in production mode, i.e.:

```
process.env.NODE_ENV === "production"
```

6. Layouts
   A layout is simply a Handlebars template with a `{{{body}}}` placeholder. Usually it
   will be an HTML page wrapper into which views will be rendered. To better
   understand this concept read `renderView(viewPath, options|callback, [callback])`
   documentation [here](https://www.npmjs.com/package/express-handlebars)
   Example:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Example App</title>
</head>
<body>

{{{body}}}

</body>
</html>
```

7. Helpers
   Helper functions, or "helpers" are functions that can be registered with Handlebars and
   can be called within a template. Helpers can be used for transforming output, iterating
   over data, etc. To keep with the spirit of logic-less templates, helpers are the place
   where logic should be defined.
   Handlebars ships with some built-in helpers, such as: `with`, `if`, `each`, etc. Note
   that
   we have already seen a usage of `if` helper. Most
   applications will need to extend this set of helpers to include app-specific logic and
   transformations. Beyond defining global helpers on `Handlebars`, this view engine
   supports
   ExpressHandlebars instance-level helpers via the helpers configuration property, and
   render-level helpers via `options.helpers` when calling the `render()`
   and `renderView()`
   methods. Let us modify our example to include some helpers. Firstly, add `helpers` to  
   engine constructor

```typescript
const view_extension = '.hbs';
const helpers = {
    foo() {
        return 'FOO!';
    },
    bar(argument: string) {
        return 'BAR!' + argument;
    }
};
const hbs_engine = engine({
    extname: view_extension,
    helpers: helpers,
})
app.engine(view_extension, hbs_engine);
```

To see how to use helpers replace the paragraph in `home.hbs` by

```html
<p>Hello my name is {{name}}! Now we call foo helper {{foo}} and bar helper with argument
    name
    {{bar name}}</p>
```

You can overload helpers locally passing helpers object to the `render` method. For
example replace `res.render(...)` by

```typescript
app.get('/', (req, res) => {
    res.render('home', {
        layout: false,
        showTitle: true,
        name: "John",
        helpers: {
            foo(): string {
                return 'FOO! overloaded locally!';
            }
        }
    });
});
```

and see what has changed compared to the previous version.

## Built-in helpers

1. We have seen a usage of `if` helper.

2. There is another one called `with` which allows you
   to change evaluation context (useful e.g. when nested parameters are passed to the
   view).
   For example

```
{{#with person}}
{{firstname}} {{lastname}}
{{/with}}
```

and we pass

```typescript
{
    person: {
        firstname: "Yehuda",
        lastname: "Katz",
    }
,
}
```

In this example `with` allowed to change context to nested object `person` and use its
properties directly via `{{firstname}}` and `{{lastname}}` instead of using `{{person.
firstname}}` and `{{person.lastname}}` respectively.

3. `each` helper allows you to iterate over lists, e.g. you can create unordere html
   list via

```html

<ul class="people_list">
    {{#each people}}
    <li>{{this}}</li>
    {{/each}}
</ul>
```

and pass object containing list of `people`

```typescript
{
    people: [
        "Yehuda Katz",
        "Alan Johnson",
        "Charles Jolley",
    ],
}
```

4. For more see [documentation for built-ins](https://handlebarsjs.com/guide/builtin-helpers.html)

## Partials for code reuse and shared templates

Think about a partial as about small piece of view which can be included in many other 
views.
To do so
you just use the following syntax: `{{>foo/bar}}` which includes 
`views/partials/foo/bar.hbs` partial into a given place. Similarly to `helpers` you 
can pass (additional) `partials` object when rendering. 
Create now `views/partials` directory and empty `views/partials/title.hbs` file. 
Paste to `title.hbs`
```html
{{#if showTitle}}
<h1>Personal webpage with partial</h1>
{{/if}}
```
and modify `body.hbs`
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Example App - Home</title>
</head>
<body>
{{>title}}
<p>Hello my name is {{name}}!</p>
</body>
</html>
```
See if it works.

## Further reading
Now you know the basics of views. To learn more read:
1. [express-handlebars documentation](https://www.npmjs.com/package/express-handlebars)
2. [handlebars expressions](https://handlebarsjs.com/guide/expressions.html)
3. [handlebars intro](https://handlebarsjs.com/guide/#installation)
4. [handlebars partials](https://handlebarsjs.com/guide/partials.html)
5. You can look for even more information in this [guide](https://handlebarsjs.com/guide/)
