# [The Twelve-Factor App](https://12factor.net/)

## Introduction

- Methodology?
- Declarative formats for setup automation
  - Що це значе, як саме?
- Have a clean contract with the underlying operating system
  - ?
- Adopted for cloud platforms
  - Do we still need DevOps?
- Minimize divergence between development and production, enabling continuous deployment
  - which differences? hard/soft, data, configuration?
  - чи дійсно це дає максимальну гнучкисть?
- can scale up without significant changes to tooling, architecture, or development practices
  - silver bullet?

### Background

- It is a triangulation on **ideal** practices
  - мені вже страшно
- organic growth
- the dynamics of collaboration between developers
- and avoiding the cost of software erosion

TODO: Table of content

## 1. Codebase

### One codebase tracked in revision control(repo), many deploys

- There is always a one-to-one correlation between the codebase and the app
  - multiple codebases - distributed system consisted from apps
  - If multiple apps sharing the same code, it should be factored into libraries

## 2. Dependencies

### Explicitly declare and isolate dependencies

- dependencies declaration, completely and exactly, via a dependency declaration manifest
- dependency isolation during execution to ensure that no implicit dependencies “leak in” from the surrounding system
- it simplifies setup for developers new to the app
- requiring only the language runtime and dependency manager installed as prerequisites

## 3. Config

### Store config in the environment

- An app’s config is everything that is likely to vary between deploys
  - Resource handles to the database, Memcached, and other backing services
  - Credentials to external services
  - Per-deploy values such as the canonical hostname for the deploy
- requires strict separation of config from code
  - Config varies substantially across deploys, code does not
- store config in environment variables
- env vars are granular controls, each fully orthogonal to other env vars.
  - They are never grouped together as “environments”, but instead are independently managed for each deploy.

## 4. Backing services

### Treat backing services as attached resources

- A backing service is any service the app consumes over the network as part of its normal operation.
- The code for app makes no distinction between local and third party services.
  - To the app, both are attached resources, accessed via a URL or other locator/credentials stored in the config.
  - only the resource handle in the config needs to change.
- Resources can be attached to and detached from deploys at will.

## 5. Build, release, run

### Strictly separate build and run stages

- A codebase is transformed into a deploy through three stages:
  - The *build stage* is a transform which converts a code repo into an executable bundle known as a build.
  Using a version of the code at a commit specified by the deployment process,
  the build stage fetches vendors dependencies and compiles binaries and assets.
  - The *release stage* takes the build produced by the build stage and combines it with the deploy’s current config.
  The resulting release contains both the build and the config and is ready for immediate execution in the execution environment.
  - The *run stage* (also known as “runtime”) runs the app in the execution environment,
  by launching some set of the app’s processes against a selected release.
- The app uses strict separation between the build, release, and run stages.
- Builds are initiated. Whereas runtime execution, by contrast, can happen automatically.

## 6. Processes

### Execute the app as one or more stateless processes

- App processes are stateless and share-nothing.
- Any data that needs to persist must be stored in a stateful backing service, typically a database.
  - What is asset compiling?

## 7. Port binding

### Export services via port binding

- The app is completely self-contained and does not rely on runtime injection of a webserver into the execution environment to create a web-facing service.
- The web app exports HTTP as a service by binding to a port, and listening to requests coming in on that port.

## 8. Concurrency

### Scale out via the process model

- The developer can architect their app to handle diverse workloads by assigning each type of work to a process type. For example, HTTP requests may be handled by a web process, and long-running background tasks handled by a worker process.
  - неконкретно. что такое unix модель процессов?
  - что такое демони?
- This does not exclude individual processes from handling their own internal multiplexing, via threads inside the runtime VM, or the async/evented model found in tools. But an individual VM can only grow so large (vertical scale), so the application must also be able to span multiple processes running on multiple physical machines.
- The app processes should never daemonize or write PID files. Instead, rely on the operating system’s process manager to manage output streams, respond to crashed processes, and handle user-initiated restarts and shutdowns.

## 9. Disposability

### Maximize robustness with fast startup and graceful shutdown

- The app’s processes are disposable, meaning they can be started or stopped at a moment’s notice. 
  - This facilitates fast elastic scaling, rapid deployment of code or config changes, and robustness of production deploys.
- The app is architected to handle unexpected, non-graceful terminations. Crash-only design takes this concept to its logical conclusion.
  - What is crash-only design?

## 10. Dev/prod parity

### Keep development, staging, and production as similar as possible

- The app is designed for continuous deployment by keeping the gap between development and production small.
  - Make the time gap small: a developer may write code and have it deployed hours or even just minutes later.
  - Make the personnel gap small: developers who wrote code are closely involved in deploying it and watching its behavior in production.
  - Make the tools gap small: keep development and production as similar as possible.
- All deploys of the app (developer environments, staging, production) should be using the same type and version of each of the backing services.

## 11. Logs

### Treat logs as event streams

- The app never concerns itself with routing or storage of its output stream. It should not attempt to write to or manage logfiles. 
- Instead, each running process writes its event stream, unbuffered, to stdout.
- In staging or production deploys, each process’ stream will be captured by the execution environment,
 collated together with all other streams from the app, and routed to one or more final destinations for viewing and long-term archival.
- The event stream for an app can be routed to a file, or watched via real-time tail in a terminal.
- Most significantly, the stream can be sent to a log indexing and analysis system

## 12. Admin processes

### Run admin/management tasks as one-off processes

- One-off admin processes should be run in an identical environment as the regular long-running processes of the app.
- They run against a release, using the same codebase and config as any process run against that release.
- Admin code must ship with application code to avoid synchronization issues.
- The same dependency isolation techniques should be used on all process types.
