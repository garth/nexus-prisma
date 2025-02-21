# Architecture

Understanding the following information should not be required knowledge for using Nexus Prisma. It is primarily here for internal maintenance and collaboration. But we're happy for whatever benefit you get out of it too.

## System

![nexus-prisma-architecture](https://user-images.githubusercontent.com/284476/118728589-70fce780-b802-11eb-8c8b-4328ef5d6fb5.png)

1. You or a script (CI, programmatic, etc.) run `$ prisma generate`.
2. Prisma generator system reads your Prisma schema file
3. Prisma generator system runs the Nexus Prisma generator passing it the "DMMF", a structured representation of your Prisma schema.
4. Nexus Prisma generator reads your Nexus Prisma generator configuration if present.
5. Nexus Prisma generator writes generated source code. By default into a special place within the `nexus-prisma` package in your `node_modules`. However, you can configure this location.
6. Later when you run your code it imports `nexus-prisma` which hits the generated entrypoint.
7. The generated runtime is actually thin, making use of a larger static runtime.

## Package Exports

When users choose a custom Nexus Prisma output directory the generated runtime needs to import the static runtime. There are a few things that need to happen in order for this to work:

1. The output's imports needs to specify the package to import from
2. The output's imports needs to specify the subpath to import from

For (1) we import from `'nexus-prisma'`. We just rely on the assumption that the user is outputting into a directory where eventually `node_modules/nexus-prisma` will should up in the CWD path as Node module resolution algorithm executes.

For (2) we look if user is outputting ESM or CJS (gentime config) and based on that access either `/dist-cjs` or `/dist-esm`. In order to support this we need to break the encapsulation provided by package.json `exports` field. We need to configure in it:

```json
"./*": {
	"default": "./*.js"
}
```

We cannot do something slightly cleaner either like this:

```json
"./_/*": {
	"require": "./dist-cjs/*.js",
	"import": "./dist-esm/*.js"
}
```

Because `ncc`, at least, does not support resolving `.../_/...` imports to what is specified in the package exports.
