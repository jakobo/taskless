# @taskless/express

## createQueue

```ts
function createQueue<T>(
  queueName: string,
  path: string,
  handler: JobHandler<T>,
  queueOptions?: QueueOptions
): TasklessExpressRouter<T>;

interface JobHandler<T> {
  (payload: T, meta: JobMetadata): Promised<unknown>;
}

interface TasklessExpressRouter<T> {
  enqueue: CreateQueueMethods<T>["enqueue"];
  update: CreateQueueMethods<T>["update"];
  delete: CreateQueueMethods<T>["delete"];
  get: CreateQueueMethods<T>["get"];
  router: (mount?: string) => express.Router;
}
```

`createQueue` is the primary way to create an Express-ready handler for Taskless. It takes in the standard arguments for creating a queue, and returns a Taskless compliant API with additional methods specific to express.

### Core Methods

These core methods are available on any Taskless integration.

`enqueue(name: string, payload: T, jobOptions?: JobOptions): Promise<Job<T>>`
Add a job to the queue named `name` with `payload`. Returns the `Job` instance created. If a job already exists with the specified `name`, it will be updated and return the updated `Job` values

`update(name: string, payload: T, jobOptions?: JobOptions): Promise<Job<T>>`
Similar to `enqueue` but will throw an error if a `Job` with the specified `name` already exists

`delete(name: string): Promise<Job<T> | null>`
Delete a job by the specified `name`, and return the `Job` or `null` if no deletion occured

`get(name: string): Promise<Job<T> | null>`
Retrieve a `Job` from Taskless named `name`, or `null` if no matches found

### Express Methods

`router(mount?: string): express.Router`
Create an [Express Router](https://expressjs.com/en/4x/api.html#router), optionally mounted to the subroute path of `mount`. In larger Express apps, it is common to nest routers as a form of code organization. When Taskless queues are not being mounted to the application root via `app.use()`, it is necessary to tell Taskless where its queue was placed. The `mount` argument lets you specify a full mount location for the Taskless Router. Without this information, Taskless cannot construct a proper URL during `enqueue()` that will end up back at your Job handler.

## Exported from @taskless/client

In addition to the above, the following items are rexported from `@taskless/client` as a convienence.

- [JobError](./client/job-error.md) An error object capable of handling advanced [return codes](./client/return-codes.md)
- [Queue](./client/queue.md) The Taskless Queue object for advanced integrations
- [JobOptions](./client/job-options.md) the set of Job options that can be passed into the Taskless client via `queueOptions.defaultJobOptions` on creation as a default for all Jobs in the queue. Can also be passed to `enqueue` and `update` to change the runtime configuration of a Job.
- [core Taskless types](https://github.com/taskless/taskless/tree/main/packages/types) for Typescript users