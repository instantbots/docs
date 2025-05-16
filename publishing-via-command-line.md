---
description: Using the Instant.bot command line tools
---

# Publishing via command line

## Installing ibot

Our command line tools are available at [github.com/instantbots/ibot](https://github.com/instantbots/ibot). For the most up to date guide on using the command line tools, please check the repository. This page exists as a quick getting started guide.

## Initialize a new Instant.bot package

To initialize a new Instant.bot package:

```sh
$ npm i ibot -g
$ mkdir new-package
$ cd new-package
$ ibot init
```

You'll be walked through the process. The `ibot` CLI will automatically check for updates to core packages, so make sure you update when available. To play around with your Instant.bot package locally;

```sh
$ ibot serve
```

Will start an HTTP server. To execute a standalone endpoint / tool:

```sh
# run functions/index.js
$ ibot run /

# run functions/some-endpoint.js
$ ibot run some-endpoint

# run functions/index.js with {"name":"hello"} POST parameters
$ ibot run / --name hello
```

## Creating tools aka endpoints

Defining custom tools is easy. You'll find the terms **tool** and **endpoint** used interchangeably as they all refer to the same thing: your bot executing custom code in the cloud.

A **tool** is just an **endpoint** hosted by the Instant.bot Package Registry.

All endpoints for Instant.bot packages live in the `functions/` directory. Each file name maps to the endpoint route e.g. `functions/hello.js` routes to `localhost:8000/hello`. You can export custom `GET`, `POST`, `PUT` and `DELETE` functions from every file. Here's an example "hello world" endpoint:

You can create a new endpoint with:

```sh
$ ibot g:endpoint hello
```

```javascript
// functions/hello.js

/**
 * A basic hello world function
 * @param {string} name Your name
 * @returns {string} message The return message
 */
export async function GET (name = 'world') {
  return `hello ${name}`!
};
```

You can write any code you want and install any NPM packages you'd like to your tool package.

## Installing NPM packages

You can install NPM packages the traditional way, or using your bundler of choice:

```sh
$ npm i stripe --save # or whatever package you want
```

Instant.bot will **automatically install** NPM packages on deployment, we do not use your locally stored packages.

## Deploy an Instant.bot Package

### Public packages

{% hint style="info" %}
You **will not** be billed for other people using your public packages. They are billed directly from their account.
{% endhint %}

By default all packages are created as public projects. Public projects are namespaced to your username, e.g. `@my-username/project`. This can be found in the `"name"` field of `instant.package.json`.

Note that the code for public projects will be shared publicly for anybody to see, and the expectation is that others can use this code in their bots as well. they will be billed from their balance.

To deploy a public project to a `development` environment, you can use:

```
$ ibot up
```

You can also publish to `staging` and `production` using:

```
$ ibot up --env staging
$ ibot up --env production
```

### Private packages

{% hint style="warning" %}
Private packages are in beta and are currently only publishable via our command line tools. We expect the functionality may change.
{% endhint %}

{% hint style="info" %}
You **will be billed** for all usage of your private packages. However, all code and endpoints will not be publicly available; you must share the URL with somebody in order for them to use it.
{% endhint %}

You can publish private project by prepending `private/` on the `"name"` field in `instant.package.json`, e.g.

```
{
  "name": "private/@my-username/private-package"
}
```

You then deploy as normal. These packages will be visible by you in the registry but nobody else.

## Additional utilities

There are a few additional utilities you may find useful with this package;

### Generate endpoints

```sh
# generates functions/my-endpoint/example.js
$ ibot g:endpoint my-endpoint/example
```

### Generate tests

```sh
# Generate blank tests or ones for an endpoint
$ ibot g:test my_test # OR ...
$ ibot g:test --endpoint my-endpoint/example
```

### Run tests

You can write tests for your tools to verify they work. Simply run;

```sh
$ ibot test
```

Your tests in the `test/` directory will be run top-down with shallow folders first, and alphabetically.

### Environment variables

{% hint style="danger" %}
Be careful when using environment variables with public packages.\
By default **we do not expose .env files** in public package code, so your secrets are safe and encrypted. However, all Instant.bot users can run public package code — so if you expose any of your secrets in endpoint results or logs they may leak.
{% endhint %}

You can store environment variables with your packages in the root directory as:

```
.env
.env.staging
.env.production
```

These files **will not** be published for everybody to see, so you can use them to hide secrets within your code. However, be careful when using environment variables with public packages: if you ever return them in an endpoint response, or connect to sensitive data, there's a chance you may expose that information to another user of the platform.
