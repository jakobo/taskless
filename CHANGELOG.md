Packages in this repository are synchronized on release, with a single changelog across all pacakges. This changelog is specifically limited to items in the `packages/` folder.

# 3.1.2 - released September 20, 2022

3.1.2 is a maintenance release, and pins the next.js version for the Taskless Dev Server to work around a startup issue caused by the new edge function support.

#### 🔧 Fixes

- **@taskless/dev** Pins the version of next.js in use

# 3.1.1 - released September 16, 2022

3.1.1 is a maintenance release, focused on reducing the total weight of the package while continuing to offer strong type safety. Because Taskless is often used in serverless environments, we want to continue making every effort to keep the client libraries lean and reserve the lambda space for your userland code. As a baseline, `@taskless/client` went from 100.5kb (luxon) to 133.6kb (luxon + zod). With the removal of luxon, the library is expected to drop to ~61kb as reported by bundlephobia.

#### 🔧 Fixes

- **@taskless/types** Updates `zod` typings to better support defaults during the parse/transform step. Now correctly uses `z.infer` for output types and `z.input` for input types.
- **@taskless/types** Improves `ENV` secret detection when resolving defaults inside of a zod `parse` operation

#### 🎒 Misc

- **@taskless/client** Removes `luxon` dependency, pushing time zone management to the calling userland code
- **@taskless/types** Removes `luxon` dependency, pushing time zone management to the calling userland code
- **@taskless/types** Adds `tinyduration` for a mimimal yet sane way to verify `Duration` values without the larger luxon dependency (from 71.5kb to 1.4kb)

# 3.1.0 - released September 14, 2022

3.1 is a feature release, introducing `zod` to replace `generic-type-guard`. The type safety from gtg was not sufficient for JS users, warranting a more robust solution that still provides proper types out of the box for TS.

#### 🎒 Misc

- **@taskless/types** Changed to `zod` for better type validation for non-typescript users. The type changes, while backwards compatible, expose additional helper guards and parsers in the package

# 3.0.1 - released August 30, 2022

3.0.1 is a patch release that ensures `TASKLESS_ENDPOINT` is being checked in both development and production mode.

#### 🔧 Fixes

- **@taskless/client** Respects `process.env.TASKLESS_ENDPOINT` when running in development mode

# 3.0.0 - released August 23, 2022

3.0 is a breaking change release that adds support for project level secrets. Legacy application secrets will be accepted until 4.0. Additionally, the deprecated CRUD interfaces are removed in favor of the simpler enqueue/cancel methods.

#### 💥 BREAKING CHANGES

Version 3 of the Taskless Client makes use of the new project level secrets. Queue based secrets will continue to work with the v2 client, but going forward, you should migrate to the per-project secrets. In preparation for this change, docs are updated to reflect the new values.

- **@taskless/client** the operations tied to legacy CRUD APIs are no longer available with 3.x. If you were using these, you can often replace `queue.update` with `queue.enqueue` and `queue.delete` with `queue.removeJob`. The `get` method did not offer a rich API experience, and it makes more sense for those calls to be made against the `for.taskless.io` API directly which is a full GraphQL service.
- **@taskless/express** no longer exposes `queue.update`, `queue.delete`, and `queue.get`. See the change for `@taskless/client` for alternatives.
- **@taskless/next** no longer exposes `queue.update`, `queue.delete`, and `queue.get`. See the change for `@taskless/client` for alternatives.

#### 🎉Features

- **@taskless/client** uses simpler endpoints for enqueing and removing jobs from Taskless.io. The legacy CRUD endpoints managed both the job queue and the primary job database, while the new `enqueueJob` and `removeJob` operate solely on the primary job database.
- **@taskless/dev** Uses [docmq](https://github.com/jakobo/docmq) under the hood with a local Mongo Driver for job scheduling. DocMQ supports multiple drivers, but it made sense to use the built-in mongo driver given the success of [mongo-memory-server](https://www.npmjs.com/package/mongodb-memory-server)

#### 🔧 Fixes

- **@taskless/dev** Removes the need for cron job dispatching in development
- **@taskless/ui** Fixes `Record` typing issue with `DataTable`

#### 🎒 Misc

- **@taskless/client** Pulls GraphQL schema directly from `for.taskless.io` instead of requiring a local copy
- **@taskless/dev** Pulls GraphQL schema directly from `for.taskless.io` instead of requiring a local copy
- **docs/** Updated to reflect breaking changes

# 2.1.3 - released June 2, 2022

#### 🔧 Fixes

- **@taskless/dev** Moves mongod and cron initialization to code that runs outside of the next.js handler. Fixes `serverRuntimeConfig.mongod is not a function` errors
- **@taskless/client** Improved detection of the development environment, checking `TASKLESS_ENV`, followed by `NODE_ENV`. This allows us to run production code from next.js in the Dev Server while also running @taskless/client in development mode for improved debugging and error messages

# 2.1.2 - released June 2, 2022

#### 🔧 Fixes

- **@taskless/dev** Fixes issue where the commonjs `server` directory was not included in the new distribution package

# 2.1.1 - released June 2, 2022

#### 🔧 Fixes

- **@taskless/dev** Fixes issue where the custom `server.js` was not included in the new distribution package

# 2.1.0 - released June 2, 2022

#### 🎉Features

- **@taskless/client** Now formally supports unverified signatures in production scenarios. There are some instances where you do not want to check signatures (for example via a webhook), or you might be checking the authenticity of the payload in some other manner. In these cases, you can explicitly override the signature checking behavior of the Taskless client on a per-queue basis. To enable this, add `{ __dangerouslyAllowUnverifiedSignatures: { allowed: true } }` to your `QueueOptions`. After looking at a variety of APIs, we felt the `__dangerously` is both easy to project search for and requires opting in via a manner that does not have ambiguity. In development, the behavior remains unchanged.
- **@taskless/client** Added support for serialized error messages. Previously, only the error's `message` property was transmitted. We now serialize the whole error (as best we can) to improve the logging and debugging experience.

#### 🔧 Fixes

- **@taskless/dev** Previously, the mongod implementation required a hack to create a globally shared in-memory instance. There were corner cases that could arise where next.js could create multiple mongos or multiple crons. To simplify the code, we're now using [runtime configuration](https://nextjs.org/docs/api-reference/next.config.js/runtime-configuration) to create singleton instances of the mongod and cron tools. We now also use a [custom server.js](https://nextjs.org/docs/advanced-features/custom-server) to start our workers, removing the original code that required an incoming job to start the job infrastructure in development.

#### 🎒 Misc

- **@taskless/client** Reduced dependencies needed by manually merging queue and job options
- **@taskless/dev** Removed unused dependencies
- **@taskless/express** Removed unused dependencies
- **@taskless/root** Fixed missing optional dependency for cosmicconfig until [this fix](https://github.com/EndemolShineGroup/cosmiconfig-typescript-loader/issues/147) lands in an upstream dependency

# 2.0.2 - released May 27, 2022

#### 🎒 Misc

- **@taskless/dev** Updated @headless/react to latest version, changed tailwind colors to use `-primary-` instead of `-brand-`. Now checks the UI module for additional rendering styles at build time
- **@taskless/dev** Fixes mongoose typings to be more correct
- **@taskless/root** Now using [syncpack](https://github.com/JamieMason/syncpack) to keep dependencies consistent between modules and additional formatting in package.json that prettier cannot handle such as ordering the keys
- **@taskless/ui** Created a common module for UI components, ensuring a consistent experience between taskless.io and the Taskless Dev server (@taskless/dev) for some of the most common blocks such as `DataTable`, `Modal`, and form controls

# 2.0.1 - released May 24, 2022

#### 🔧 Fixes

- **@taskless/client** In some cases, the default header of `content-type = application/json` was not being set. This could result in situations where a JSON middleware such as `express.json()` was not correctly parsing the body

# 2.0.0 - released May 20, 2022

#### 💥 BREAKING CHANGES

- **@taskless/client** Integrations were split out to avoid conflicting namespace issues. Next and Express users can now reference `@taskless/next` and `@taskless/express` respectively. The `@taskless/client` contains only the raw Taskless client.

#### 🎉Features

- **@taskless/client**, **@taskless/next**, **@taskless/express** Added the ability to specify an array as a job identifier instead of just a string key, making namespacing identifiers require less cognitive overhead
- **@taskless/dev** Added the ability to create jobs via the Taskless dev dashboard
- **@taskless/express** Added a `mount` method for working with Taskless when it's attached to a sub-router.

#### 🔧 Fixes

- **@taskless/client** Fixed default export of `Queue` in CJS environments
- **@taskless/client** Fixed issue in development where a mismatched signature would throw instead of logging an error

#### 🎒 Misc

- **@taskless/client** Updated node specific modules to import from the `node:` namespace
- **@taskless/client** Moved to home in `/packages` matching its package name to reduce confusion
- **@taskless/dev** Switched PouchDB for [mongo-memory-server](https://www.npmjs.com/package/mongodb-memory-server). While it adds a bit more overhead to start up a Mongo server in development, it makes it much easier to use Mango queries for querying task and job information.
- **@taskless/root** Added a `dev` script that gets every integration up and running in dev mode for fast debugging

# 1.1.0 - released May 2, 2022

#### 🎉Features

- **@taskless/dev** New devlopment server pages at `/` and `/logs` to mirror Taskless.io, replacing the old completed/scheduled structure
- **@taskless/dev** Common UI components. Imported Taskless' DataTable, Logo, and Slash components

# 1.0.0 - released April 25, 2022

Initial 1.0 release of @taskless/client and @taskless/dev
