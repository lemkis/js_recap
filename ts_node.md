- [nodejs with ts and express](#nodejs-with-ts-and-express)
- [At the end read docs](#at-the-end-read-docs)
- [ORM - prisma](#orm---prisma)
- [Dynamic html pages, views](#dynamic-html-pages-views)
  - [Built-in helpers](#built-in-helpers)
  - [Partials for code reuse and shared templates](#partials-for-code-reuse-and-shared-templates)
  - [Exercise](#exercise)
  - [Further reading](#further-reading)
- [EXPRESS](#express)
  - [Creating route handlers](#creating-route-handlers)
  - [Routers](#routers)
  - [Middleware](#middleware)
- [Routes and controllers](#routes-and-controllers)
- [Exercise](#exercise-1)
- [Passport middleware](#passport-middleware)
- [Security](#security)
  - [Acknowledgements](#acknowledgements)
  - [Cross-Site Scripting (XSS)](#cross-site-scripting-xss)
    - [A reflected XSS vulnerability](#a-reflected-xss-vulnerability)
    - [A persistent XSS vulnerability](#a-persistent-xss-vulnerability)
    - [DOM XSS](#dom-xss)
  - [SQL injection](#sql-injection)
  - [Cross-Site Request Forgery (CSRF)](#cross-site-request-forgery-csrf)
  - [Clickjacking (UI redressing)](#clickjacking-ui-redressing)
  - [Denial of Service (DoS)](#denial-of-service-dos)
  - [Directory Traversal](#directory-traversal)
  - [File Inclusion](#file-inclusion)
  - [Command injection](#command-injection)
- [Security services provided by browsers](#security-services-provided-by-browsers)
  - [Same-origin policy and CORS](#same-origin-policy-and-cors)
- [Example](#example)

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

6. A layout is simply a Handlebars template with a `{{{body}}}` placeholder. Usually it
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

7.  Helper functions, or "helpers" are functions that can be registered with Handlebars and
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
`views/partials/foo/bar.hbs` partial into a given place.
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
## Exercise
Create in `layouts` directory file named `main.hbs` (this default layout used for all views) and modify you code so that `home.hbs` uses `main.hbs` layout.
## Further reading
Now you know the basics of views. To learn more read:
1. [express-handlebars documentation](https://www.npmjs.com/package/express-handlebars)
2. [handlebars expressions](https://handlebarsjs.com/guide/expressions.html)
3. [handlebars intro](https://handlebarsjs.com/guide/#installation)
4. [handlebars partials](https://handlebarsjs.com/guide/partials.html)
5. You can look for even more information in this [guide](https://handlebarsjs.com/guide/)


# EXPRESS

## Creating route handlers
You can respond to get requests via
```typescript
app.get("/", function (req, res) {
  res.send("Hello World!");
});
```
or in response to any HTTP method
```typescript
app.all("/secret", function (req, res, next) {
  console.log("Accessing the secret section…");
  next(); // pass control to the next handler
});

```
## Routers
Often it is useful to group route handlers for a particular part of a site together and access them using a common route-prefix. This is achieved by using the `express.Router` object, e.g.

```typescript
// wiki.js - Wiki route module
router.get("/", function (req, res) {
  res.send("Wiki home page");
});
router.get("/about", function (req, res) {
  res.send("About this wiki");
});
```
and then `app.use("/wiki", wiki);`

## Middleware

Middleware is used extensively in Express apps, for tasks from serving static files to error handling, to compressing HTTP responses. Whereas route functions end the HTTP request-response cycle by returning some response to the HTTP client, middleware functions typically perform some operation on the request or response and then call the next function in the "stack", which might be more middleware or a route handler. The order in which middleware is called is up to the app developer.

You can add a middleware function to the processing chain for `all` responses with `app.use()`, or for a specific HTTP verb using the associated method: `app.get()`, `app.post()`, etc. General structure of middleware is the following one:
```typescript
function (req, res, next) {
  // Perform some operations
  next(); // Call next() so Express will call the next middleware function in the chain.
};
```
or if you want to handle errors
```typescript
function (err, req, res, next) {
  console.error(err.stack);
  res.status(500).send("Something broke!");
};
```
Note that Express comes with a built-in error handler, which takes care of any remaining errors that might be encountered in the app. This default error-handling middleware function is added at the end of the middleware function stack. If you pass an error to `next()` and you do not handle it in an error handler, it will be handled by the built-in error handler; the error will be written to the client with the stack trace.
You can serve static files via
```typescript
app.use("/virtual_prefix", express.static("public"));
```
Now you may load files accessing e.g. `http://localhost:3000/virtual_prefix/images/dog.jpg`.

A full list of middlewares is available [here](https://expressjs.com/en/resources/middleware.html)

# Routes and controllers

- "Routes" to forward the supported requests (and any information encoded in request URLs) to the appropriate controller functions.

- Controller functions to get the requested data from the models, create an HTML page displaying the data, and return it to the user to view in the browser.

- Views (templates) used by the controllers to render the data.

-  Router functions are Express middleware, which means that they must either complete (respond to) the request or call the next function in the chain. 

- Route paths can also be string patterns. String patterns use a form of regular expression syntax to define patterns of endpoints that will be matched see [syntax here](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/routes#route_paths)

- Route parameters are named URL segments used to capture values at specific positions in the URL. The named segments are prefixed with a colon and then the name (E.g., /:your_parameter_name/). The captured values are stored in the req.params object using the parameter names as keys (E.g., `req.params.your_parameter_name`). The names of route parameters must be made up of "word characters" (A-Z, a-z, 0-9, and _).

- The route functions shown earlier all have arguments `req` and `res`, which represent the request and response, respectively. Route functions are also called with a third argument `next`, which can be used to pass errors to the Express middleware chain.
```typescript
router.get("/about", (req, res, next) => {
    
    if (err) {
      return next(err);
    }
    res.render("about_view", { title: "About", list: queryResults });
  });
});
```
If you want to handle exceptions you must firstly catch it and then pass it as a regular error. To simplify things use in such a case the express-async-handler module which defines a wrapper function that hides the `try, catch` block and the code to forward the error so that you need to write code only for the case where we assume success.
```typescript
// const asyncHandler = require("express-async-handler");
  asyncHandler(async (req, res, next) => {
    const successfulResult = await some_function();
    res.render("about_view", { title: "About", list: successfulResult });
  }
);
```

# Exercise

Create LocalLibrary following [tutorial](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/routes#routes_needed_for_the_locallibrary)

For a solution you can look [here](https://github.com/mdn/express-locallibrary-tutorial)


# Passport middleware

See [signin with username and password github](https://github.com/passport/todos-express-password) and the corresponding [tutorial](https://www.passportjs.org/tutorials/password/). You can look at [this](https://www.passportjs.org/howtos/password/) as well.

You can protect routes by using 
```typescript
function ensureAuthenticated(req, res, next) {
  if (req.isAuthenticated())
    return next();
  else
    // Return error content: res.jsonp(...) or redirect: res.redirect('/login')
}

app.get('/account', ensureAuthenticated, function(req, res) {
  // Do something with user via req.user
});
```
 or as in the tutorial use `var ensureLogIn = require('connect-ensure-login').ensureLoggedIn`.
See other added request attributes in passport's, `lib/http/request.js` file.

For more information visit [official passport-local](https://github.com/jaredhanson/passport-local)


# Security
## Acknowledgements

This note is based on [mozilla documentation](https://developer.mozilla.org/en-US/docs/Learn/Server-side/First_steps/Website_security) and https://portswigger.net/ materials e.g.
[os command injection](https://portswigger.net/web-security/os-command-injection)
and [dom based attacts](https://portswigger.net/web-security/dom-based)

## Cross-Site Scripting (XSS)


XSS is a term used to describe a class of attacks that allow an attacker to inject client-side scripts through the website into the browsers of other users. Because the injected code comes to the browser from the site, the code is trusted and can do things like send the user's site authorization cookie to the attacker. When the attacker has the cookie, they can log into a site as though they were the user and do anything the user can, such as access their credit card details, see contact details, or change passwords.

Consider having a page where the latest user opinion is being loaded. Imagine a malicious user provide a following review:
```html
<script type=”text/javascript”>
var test=’../example.php?cookie_data=’+escape(document.cookie);
</script>
```
What happens in such a situation? If someone asks for the last review  the cookies are escaped and then sent to example.php script’s variable ‘cookie_data’.

A malicious user can execute in this way different kinds of scripts, for example
```html
<body onload=alert("you got hacked")>;
  ```
  or 
  ```html
  <script>destroy_website();</script>
  ```


The best defense against XSS vulnerabilities is to remove or disable any markup that can potentially contain instructions to run the code. For HTML this includes elements, such as `<script>`, `<object>`, `<embed>`, and `<link>`.

The process of modifying user data so that it can't be used to run scripts or otherwise affect the execution of server code is known as input `sanitization`. Many web frameworks automatically sanitize user input from HTML forms by default.



### A reflected XSS vulnerability 

Occurs when user content that is passed to the server is returned immediately and unmodified for display in the browser. Any scripts in the original user content will be run when the new page is loaded. For example, consider a site search function where the search terms are encoded as URL parameters, and these terms are displayed along with the results. An attacker can construct a search link that contains a malicious script as a parameter 
``` 
http://developer.mozilla.org?q=beer<script%20src="http://example.com/tricky.js"></script>)
``` 
and email it to another user. If the target user clicks this "interesting link", the script will be executed when the search results are displayed. As discussed earlier, this gives the attacker all the information they need to enter the site as the target user, potentially making purchases as the user or sharing their contact information.


A question. Imagine a logging system that in case of error displays the provided username along with information that it is wrong. Is this a safe solution?


### A persistent XSS vulnerability 

Occurs when the malicious script is stored on the website and then later redisplayed unmodified for other users to execute unwittingly. For example, a discussion board that accepts comments that contain unmodified HTML could store a malicious script from an attacker. When the comments are displayed, the script is executed and can send to the attacker the information required to access the user's account. This sort of attack is extremely popular and powerful, because the attacker might not even have any direct engagement with the victims.


### DOM XSS

This type of attack occurs when the DOM environment is being changed, but the client-side code does not change. DOM-based XSS vulnerabilities usually arise when JavaScript takes data from an attacker-controllable source (a source is a JavaScript property that accepts data that is potentially attacker-controlled) such as the URL, and passes it to a sink (a sink is a potentially dangerous JavaScript function or DOM object that can cause undesirable effects if attacker-controlled data is passed to it) that supports dynamic code execution, such as `eval()` or `innerHTML`. This enables attackers to execute malicious JavaScript, which typically allows them to hijack other users' accounts.

Consider, there is a webpage with URL `http://testing.com/book.html?default=1`. As we know, “default” is a parameter and “1” is its value. Therefore, in order to perform an XSS DOM attack, we would send a script as the parameter.

For Example:
```
http://testing.com/book.html?default=<script>alert(document.cookie)</script>
```
In this Example, the request is sent for the page `book.html?default=<script>alert(document.cookie)</script>` to testing.com. Therefore for that page, a DOM object is being created by the browser, where the document location object will contain the appropriate string.
```
http://testing.com/book.html?default=<script>alert(document.cookie)</script>
```
This way the DOM environment is being affected. Of course, instead of this simple script, something more harmful may also be entered.
## SQL injection

SQL injection vulnerabilities enable malicious users to execute arbitrary SQL code on a database, allowing data to be accessed, modified, or deleted irrespective of the user's permissions. A successful injection attack might spoof identities, create new identities with administration rights, access all data on the server, or destroy/modify the data to make it unusable.

SQL injection types include Error-based SQL injection, SQL injection based on boolean errors, and Time-based SQL injection.

This vulnerability is present if user input that is passed to an underlying SQL statement can change the meaning of the statement. For example, the following code is intended to list all users with a particular name (userName) that has been supplied from an HTML form:

```
statement = "SELECT * FROM users WHERE name = '" + userName + "';"
```
If the user specifies a real name, the statement will work as intended. However, a malicious user could completely change the behavior of this SQL statement to the new statement in the following example, by specifying 
```
a';DROP TABLE users; SELECT * FROM userinfo WHERE 't' = 't 
```
for the userName.

The modified statement
```
SELECT * FROM users WHERE name = 'a';DROP TABLE users; SELECT * FROM userinfo WHERE 't' = 't';
```
 creates a valid SQL statement that deletes the users table and selects all data from the userinfo table (which reveals the information of every user). This works because the first part of the injected text (a';) completes the original statement.

To avoid this sort of attack, you must ensure that any user data that is passed to an SQL query cannot change the nature of the query. One way to do this is to escape all the characters in the user input that have a special meaning in SQL.

Note: The SQL statement treats the ' character as the beginning and end of a string literal. By putting a backslash in front of this character (\'), we escape the symbol, and tell SQL to instead treat it as a character (just a part of the string).

## Cross-Site Request Forgery (CSRF)

CSRF attacks allow a malicious user to execute actions using the credentials of another user without that user's knowledge or consent.

This type of attack is best explained by example. Josh is a malicious user who knows that a particular site allows logged-in users to send money to a specified account using an HTTP POST request that includes the account name and an amount of money. Josh constructs a form that includes his bank details and an amount of money as hidden fields, and emails it to other site users (with the Submit button disguised as a link to a "get rich quick" site).

If a user clicks the submit button, an HTTP POST request will be sent to the server containing the transaction details and any client-side cookies that the browser associated with the site (adding associated site cookies to requests is normal browser behavior). The server will check the cookies, and use them to determine whether or not the user is logged in and has permission to make the transaction.

The result is that any user who clicks the Submit button while they are logged in to the trading site will make the transaction. Josh gets rich.

One way to prevent this type of attack is for the server to require that POST requests include a user-specific site-generated secret. The secret would be supplied by the server when sending the web form used to make transfers. This approach prevents Josh from creating his own form, because he would have to know the secret that the server is providing for the user. Even if he found out the secret and created a form for a particular user, he would no longer be able to use that same form to attack every user.

## Clickjacking (UI redressing)

In this attack, a malicious user hijacks clicks meant for a visible top-level site and routes them to a hidden page beneath. This technique might be used, for example, to display a legitimate bank site but capture the login credentials into an invisible `<iframe>` controlled by the attacker. Clickjacking could also be used to get the user to click a button on a visible site, but in doing so actually unwittingly click a completely different button. As a defense, your site can prevent itself from being embedded in an iframe in another site by setting the appropriate HTTP headers.

Clickjacking attacks use CSS to create and manipulate layers. The attacker incorporates the target website as an iframe layer overlaid on the decoy website. An example using the style tag and parameters is as follows:


```html
<head>
	<style>
		#target_website {
			position:relative;
			width:128px;
			height:128px;
			opacity:0.00001;
			z-index:2;
			}
		#decoy_website {
			position:absolute;
			width:300px;
			height:400px;
			z-index:1;
			}
	</style>
</head>
...
<body>
	<div id="decoy_website">
	...decoy web content here...
	</div>
	<iframe id="target_website" src="https://vulnerable-website.com">
	</iframe>
</body>
```

The target website iframe is positioned within the browser so that there is a precise overlap of the target action with the decoy website using appropriate width and height position values. Absolute and relative position values are used to ensure that the target website accurately overlaps the decoy regardless of screen size, browser type and platform. The z-index determines the stacking order of the iframe and website layers. The opacity value is defined as 0.0 (or close to 0.0) so that the iframe content is transparent to the user. Browser clickjacking protection might apply threshold-based iframe transparency detection (for example, Chrome version 76 includes this behavior but Firefox does not). The attacker selects opacity values so that the desired effect is achieved without triggering protection behaviors.


## Denial of Service (DoS)

DoS is usually achieved by flooding a target site with fake requests so that access to a site is disrupted for legitimate users. The requests may be numerous, or they may individually consume large amounts of resource (e.g., slow reads or uploading of large files). DoS defenses usually work by identifying and blocking "bad" traffic while allowing legitimate messages through. These defenses are typically located before or in the web server (they are not part of the web application itself)


## Directory Traversal

 In this attack, a malicious user attempts to access parts of the web server file system that they should not be able to access such as
- Application code and data.
- Credentials for back-end systems.
- Sensitive operating system files.

In the extreme case an attacker might be able to write to arbitrary files on the server, allowing them to modify application data or behavior, and ultimately take full control of the server.

 This vulnerability occurs when the user is able to pass filenames that include file system navigation characters (for example, `../../`). The solution is to sanitize input before using it.


Imagine a shopping application that displays images of items for sale. This might load an image using the following HTML:

```html
<img src="/loadImage?filename=218.png">
```
The loadImage URL takes a `filename` parameter and returns the contents of the specified file. The image files are stored on disk in the location `/var/www/images/`. To return an image, the application appends the requested `filename` to this base directory and uses a filesystem API to read the contents of the file. In other words, the application reads from the following file path:
```
/var/www/images/218.png
```
This application implements no defenses against path traversal attacks. As a result, an attacker can request the following URL to retrieve the `/etc/passwd` file from the server's filesystem:
```
https://insecure-website.com/loadImage?filename=../../../etc/passwd
```
This causes the application to read from the following file path:
```
/var/www/images/../../../etc/passwd
```
The sequence ../ is valid within a file path, and means to step up one level in the directory structure. The three consecutive ../ sequences step up from /var/www/images/ to the filesystem root, and so the file that is actually read is:

/etc/passwd
On Unix-based operating systems, this is a standard file containing details of the users that are registered on the server, but an attacker could retrieve other arbitrary files using the same technique.

On Windows, both ../ and ..\ are valid directory traversal sequences. The following is an example of an equivalent attack against a Windows-based server:

 ## File Inclusion

  In this attack, a user is able to specify an "unintended" file for display or execution in data passed to the server. When loaded, this file might be executed on the web server or the client-side (leading to an XSS attack). The solution is to sanitize input before using it.

Consider 
```php
<?php

$file = $_GET[‘file’];

if(isset($file))

{

include(“pages/$file”);

}

else

{

include(“index.php”);

}

?>
```

Imagine what happens if we manipulate the file location parameter, e.g.
```
/script.php?page=../../../../../../../../etc/passwd
```
## Command injection 

These attacks allow a malicious user to execute arbitrary system commands on the host operating system. The solution is to sanitize user input before it might be used in system calls.

In this example, a shopping application lets the user view whether an item is in stock in a particular store. This information is accessed via a URL:
```
https://insecure-website.com/stockStatus?productID=381&storeID=29
```
To provide the stock information, the application must query various legacy systems. For historical reasons, the functionality is implemented by calling out to a shell command with the product and store IDs as arguments:
```
stockreport.pl 381 29
```
This command outputs the stock status for the specified item, which is returned to the user.

The application implements no defenses against OS command injection, so an attacker can submit the following input to execute an arbitrary command:
```
& echo aiwefwlguh &
```
If this input is submitted in the productID parameter, the command executed by the application is:
```
stockreport.pl & echo aiwefwlguh & 29
```
The echo command causes the supplied string to be echoed in the output. This is a useful way to test for some types of OS command injection. The & character is a shell command separator. In this example, it causes three separate commands to execute, one after another. The output returned to the user is:
```
Error - productID was not provided
aiwefwlguh
29: command not found
```
The three lines of output demonstrate that:

- The original stockreport.pl command was executed without its expected arguments, and so returned an error message.
- The injected echo command was executed, and the supplied string was echoed in the output.
- The original argument 29 was executed as a command, which caused an error.

Placing the additional command separator & after the injected command is useful because it separates the injected command from whatever follows the injection point. This reduces the chance that what follows will prevent the injected command from executing.


# Security services provided by browsers

## Same-origin policy and CORS

Web content's  `origin` is defined by the scheme (protocol), hostname (domain), and port of the URL used to access it. Two objects have the same origin only when the scheme, hostname, and port all match.

Same-origin policy is a fundamental security mechanism of the web that restricts how a document or script loaded from one origin can interact with a resource from another origin. It helps isolate potentially malicious documents, reducing possible attack vectors.

    In general, documents from one origin cannot make requests to other origins. This makes sense because you don't want sites to be able to interfere with one another and access things they shouldn't. It does make sense to relax this restriction in some circumstances; for example, you might have multiple sites that interact with each other, and you may want these sites to request resources from one another, such as using fetch().

This can be permitted using Cross-Origin Resource Sharing (CORS), an HTTP-header-based mechanism that allows a server to indicate any origins (domain, scheme, or port) other than its own from which a browser should permit loading resources.


# Example 
[vulnerable nodejs](https://github.com/xuezzou/Vulnerable-nodejs)
