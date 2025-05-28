---
description: A guide to packages on Instant.bot
---

# Package specification

## Overview

All packages on Instant.bot are available as both **REST API servers** and **MCP servers**. They are hosted on `{package}.instant.bot` and expose HTTP endpoints that act as tools. They are built using the [Instant API framework](https://gitub.com/instant-dev/api), a JavaScript framework and gateway that turns JavaScript functions into type-validated HTTP endpoints with built-in bindings for the MCP Streamable HTTP specification.

**REST** means **Re**presentational **S**tate **T**ransfer and is a fancy way of saying these servers expose endpoints that support multiple HTTP verbs: `GET`, `POST`, `PUT`, `DELETE`.

**MCP** stands for **M**odel **C**ontext **P**rotocol and is an emerging standard for connecting LLMs to remote procedure calls. It is currently supported by OpenAI, Anthropic and Google Gemini APIs.

## Quick examples

For an example of what hosted tools look like in production, visit:

* [Current weather package](https://instant.bot/packages/@keith/weather)
* [Stripe customer package](https://instant.bot/packages/@keith/stripe) (includes **required** keys)
* [GPT image generator package](https://instant.bot/packages/@keith/openai-gpt-image) (outputs image)

## Instant API

[Instant API](https://github.com/instant-dev/api) is a combined framework and HTTP gateway for turning JavaScript functions into typed HTTP endpoints. It uses JSDoc comments above function exports to automatically generate OpenAPI and JSON schemas for each endpoint, as well as tool definitions. **It is vendor-agnostic**, meaning Instant API projects can be hosted anywhere like Vercel, AWS, GCP, etc.

### Project history

Instant API is part of a [broader JavaScript framework + ORM called instant.dev](https://github.com/instant-dev/instant.dev). That project itself is a derivative of [Nodal](https://github.com/keithwhor/nodal), an now-defunct API framework we first built in 2015. Instant.dev is our "Ruby on Rails for JavaScript", with over 10 years of development time behind it. We have used it to build dozens of products and the most successful have handled over 1B API requests per week. In an era of LLM extensibility, we are really excited about its potential to generate tools for LLMs quickly and easily.

## Package hosting

The Instant.bot package registry automatically hosts and scales your Instant API projects for you as packages. However, Instant API projects can be hosted on any infrastructure provider that supports Node.js 20.x or above: like Vercel, Railway and AWS. This means projects that you build on Instant.bot are _transportable_. When you build tools for Instant.bot you are not locked in to using us as a hosting provider, though we do manage secret storage, package verification and more.

### Our registry domain

All packages for Instant.bot are hosted on our gateway at `{package}.instant.host`

### Authentication and making requests

You authenticate into packages using **API keychains**, a primitive we have created for securely storing, managing and delegating access to third-party secrets and auth.

These keychains (1) authenticate you as an Instant.bot user and (2) can scaffold one or more API secrets that can be shared with packages. For security considerations, please read the [api-keychain-specification.md](api-keychain-specification.md "mention").

```sh
curl https://{package}.instant.host/endpoint-name \
  -X POST \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {API_KEYCHAIN_SECRET}' \
  --data '{"some":"json"}'
```

## Package directory structure

Every package contains a directory structure with the following format:

```sh
functions/
  index.js            # root endpoint, path = /
  my-function.js      # endpoint, path = /my-function
  404.js              # catchall endpoint, path = /my-function/*
  subdir/
    index.js          # subdir root endpoint, path = /subdir
    do-thing.js       # endpoint, path = /subdir/do-thing
    404.js            # catchall endpoint, path = /subdir/*
instant.package.json  # instant.bot package info
package.json          # traditional node.js package.json
```

As you can see, Instant.bot packages use **file-based routing**. Everything exported from the `functions/` folder is available as an API endpoint.

## instant.package.json

This is your Instant.bot package configuration. It typically has the following format;

```json
{
  "name": "@user/package",
  "timeout": 10
}
```

Where `"name"` is the package name, usually of the format `@user/package` for public packages. Agent code packages, which are private, are of the format `agent/@user/package`.

### Changing timeout

The default timeout for Instant.bot packages is 10 seconds. This means tools that are part of this package will run for 10 seconds before automatically halting execution. You can manually change this value to any number between 1 (1 second) and 300 (5 minutes).

### Setting required keychain keys

Public packages can request secret keys from API keychains. For example, the [Stripe customers package](https://instant.bot/packages/@keith/stripe) requires a **STRIPE\_SECRET\_KEY** to use it successfully. To set these per package, add them to your `instant.package.json` like so:

```json
{
  "name": "@user/package",
  "timeout": 10,
  "keychain": {
    "required": [
      {
        "name": "STRIPE_SECRET_KEY",
        "description": "Your Stripe secret key, available at dashboard.stripe.com/apikeys
      }
    ]
  }
}
```

## Package endpoints

Every file in the `functions/` directory is used to create an HTTP API endpoint that can respond to `GET`, `POST`, `PUT` and `DELETE` requests. Each of these endpoints is available to your agents for use as a tool.

Here's an example of a weather endpoint that is accessible via HTTP GET.

```javascript
/**
 * Retrieve the weather for a specific location
 * @param {?string{1..64}} location Search by location
 * @param {?object} coords Provide specific latitude and longitude
 * @param {number{-90,90}} coords.lat Latitude
 * @param {number{-180,180}} coords.lng Longitude
 * @param {string[]} tags Nearby locations to include
 * @returns {object} weather Your weather result
 * @returns {number} weather.temperature Current temperature of the location
 * @returns {string} weather.unit Fahrenheit or Celsius
 */
export async function GET (location = null, coords = null, tags = []) {

  if (!location && !coords) {
    // Prefixing an error message with a "###:" between 400 and 404
    // automatically creates the correct client error:
    // BadRequestError, UnauthorizedError, PaymentRequiredError,
    // ForbiddenError, NotFoundError
    // Otherwise, will throw a RuntimeError with code 420
    throw new Error(`400: Must provide either location or coords`);
  } else if (location && coords) {
    throw new Error(`400: Can not provide both location and coords`);
  }
  
  // Fetch your own API data
  await getSomeWeatherDataFor(location, coords, tags);
  
  // mock a response
  return { temperature: 89.2 units: `°F` };
  
}
```

### Using different HTTP methods

A single file can output up to **four** different endpoints corresponding to an HTTP method, and each can be used as an individual tool. `GET`, `POST`, `PUT` and `DELETE` are all supported.

For simple tools, exporting a `default` function will always work as an endpoint that responds to **all HTTP methods**.

```javascript
/**
 * A simple hello world endpoint, responds to all HTTP methods
 */
export default async function () {
  return `hello world!`;
}
```

You can `GET`, `POST`, `PUT` or `DELETE` to this endpoint. If you'd like to know which method was called, you can use the magic `context` object to check the HTTP method:

```javascript
/**
 * A simple hello world endpoint, responds to all HTTP methods
 */
export default async function (context) {
  return `hello world, method is ${context.http.method}`;
}
```

If you want different functionality depending on the HTTP method, it's best to export named functions for `GET`, `POST`, `PUT` and `DELETE`.

```javascript
/**
 * A simple hello world endpoint, responds to GET
 */
export async function GET () {
  return `Hello HTTP GET!`;
}

/**
 * A simple hello world endpoint, responds to POST
 */
export async function POST () {
  return `Hello HTTP POST!`;
}

/**
 * A simple hello world endpoint, responds to PUT
 */
export async function PUT () {
  return `Hello HTTP PUT!`;
}

/**
 * A simple hello world endpoint, responds to DELETE
 */
export async function DELETE () {
  return `Hello HTTP DELETE!`;
}
```

If you don't export a specific HTTP method, requests to that method will fail with a `501: Not implemented` error.

### Endpoint arguments

To create arguments for your endpoint, you simply add arguments to your function signature and comment them using JSDoc.

```javascript
/**
 * Generates a hello world message
 * @param {string} name
 * @param {number} age
 */
export async function POST (name, age) {
  return `Hello ${name}, you are ${age}!`;
}
```

When you make an HTTP POST request to this endpoint, you can provide an `application/json` payload with `{"name":"test","age":20}` and will receive the result `"Hello test, you are 20!"`.

#### Required arguments

To set arguments as **required**, simply **do not** provide a default value for the argument in the function signature.

```javascript
/**
 * Generates a hello world message
 * @param {string} name
 */
export async function POST (name) {
  return `Hello ${name}!`;
}
```

If you try to send an HTTP POST with an empty payload, you will receive the following response:

```json
{
  "type": "ParameterError",
  "message": "Invalid parameter \"name\": required",
  "details": {
    "name": {
      "message": "required",
      "required": true
    }
  }
}
```

#### Optional arguments

To mark an argument as optional, simply give it a default value. A default value **must** be either `null` or match the type specified by JSDoc.

```javascript
/**
 * Generates a hello world message
 * @param {string} name
 */
export async function POST (name = 'world') {
  return `Hello ${name}!`;
}
```

Sending an empty payload will now result in a `"Hello world!"` response.

#### The context argument

Every function signature can include an optional `context` argument. The context argument must **always be the last argument** when provided and **must not be documented** by JSDoc.

```javascript
/**
 * Generates a hello world message
 * @param {string} name
 */
export async function POST (name = 'world', context) {
  return context;
}
```

The `context` object contains contextual runtime data, like the HTTP method, raw HTTP body, parameters and more.

* `context.name` the function name
* `context.path` an array of path segments used to run the endpoint
* `context.params` a SON representation of arguments passed in to the endpoint
* `context.remoteAddress` the IPv4 or IPv6 address that requested the endpoint
* `context.uuid` a unique execution id
* `context.http.method` the HTTP method used to invoke the endpoint
* `context.http.headers` the HTTP headers used to invoke the endpoint
* `context.http.body` a raw utf-8 string of the HTTP POST body, if applicable
* `context.keychain.key("MY_KEY")` a method to access API keychain keys provided to the endpoint at runtime

### Endpoint returns: JSON, custom HTTP responses and files

Similar to arguments, you can specify return types for endpoints. By default, the return type is `any`. We will allow any return type, though there are a few special cases.

```javascript
/**
 * Generates a hello world message
 * @param {string} name
 */
export async function POST (name) {

  // basic return types
  return `Hello ${name}!`; // returns JSON string: "Hello world"
  return 23; // returns JSON number: 23
  return true; // returns JSON boolean: true
  return false; // returns JSON boolean: false
  return null; // returns JSON: null
  return void 0; // undefined, coerced to return JSON: null
  return undefined; // undefined, coerced to return JSON: null
  return ['some', 'array']; // returns JSON: ["some","array"]
  return {some: 'object'}; // returns JSON: {"some":"object"}
  
  // custom http responses
  // this will return the exact http response described
  return {
    statusCode: 200,
    headers: {'Content-Type': 'text/plain'},
    body: Buffer.from('What')
  };
  
  // file responses
  const file = Buffer.from('...');
  file.contentType = 'image/png'; // optional: sets Content-Type header
  return file; // returns raw buffer as an HTTP response
  
  // nested files in JSON objects
  const file = Buffer.from('...');
  return { file }; // returns JSON: {"file":{"_base64":"b64_value"}}
  
}
```

You can specify endpoint `returns` types with the `@returns` directive:

```javascript
/**
 * Generates a hello world message
 * @param {string} name
 * @returns {string}
 */
export async function POST (name = 'world') {
  returns `Hello ${name}!`;
}
```

By specifying the return type, you can force our gateway to do type validation on responses as well: if something goes wrong, we can throw an error. For example, in this code the return type is specified as **number**, but it's returning a string.

```javascript
/**
 * Generates a hello world message
 * @param {string} name
 * @returns {number}
 */
export async function POST (name = 'world') {
  returns `Hello ${name}!`;
}
```

We'll get the following error.

```json
{
  "type": "ValueError",
  "message": "The value returned by the function did not match the specified type",
  "details": {
    "returns": {
      "message": "invalid return value: \"Hello world!\" (string), expected (number)",
      "invalid": true,
      "expected": {
        "type": "number"
      },
      "actual": {
        "value": "Hello world!",
        "type": "string"
      }
    }
  }
}
```

## Returning attachments in the Instant.bot UI

To have a package return attachments in the Instant.bot UI, simply make sure you set the return type to `buffer` like so:

```javascript
import fs from 'fs';

/**
 * Returns an image that gets embedded in chat
 * @returns {buffer}
 */
export async function POST () {
  const file = fs.readFileSync('my-image.png');
  file.contentType = 'image/png';
  return file;
}
```

By setting the `@returns` directive you tell the OpenAPI spec and tool schema that we expect to return a file. The `image/png` content type header gets read from the `.contentType` field of the buffer. This content type will direct the attachment within the UI to render an image.

## Custom errors

To throw custom errors from an endpoint you have two options.

### 1. Throw a 400 to 404 error

```javascript
export default async function () {
  // Prefixing an error message with a "###:" between 400 and 404
  // automatically creates the correct client error:
  // BadRequestError, UnauthorizedError, PaymentRequiredError,
  // ForbiddenError, NotFoundError
  // Otherwise, will throw a RuntimeError with code 420
  throw new Error(`400: Some error`); // BadRequestError, code 400
  throw new Error(`401: Unauthed!`); // UnauthorizedError, code 401
  throw new Error(`402: Pay me`); // PaymentRequiredError, code 402
  throw new Error(`403: Nope`); // ForbiddenError, code 403
  throw new Error(`404: Where'd it go?`); // NotFoundError, code 404
  throw new Error(`Oh no!`); // RuntimeError, code 420
}
```

### 2. Return a custom HTTP response

```javascript
export default async function () {
  return {
    statusCode: 500,
    headers: {},
    body: Buffer.from(`My custom 500 error`)
  };
}
```

## Type validation

You get tool type validation for free when building packages with Instant.bot as part of the [Instant API](https://github.com/instant-dev/api) framework. You can define types for both `@param` and `@returns` arguments in the function signature. Instant API supports the following types.

### Supported types

| Type        | Definition                                                                                     | Example Parameter Input Values (JSON)                                                                                                         |
| ----------- | ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| boolean     | True or False                                                                                  | `true` or `false`                                                                                                                             |
| string      | Basic text or character strings                                                                | `"hello"`, `"GOODBYE!"`                                                                                                                       |
| number      | Any double-precision [Floating Point](https://en.wikipedia.org/wiki/IEEE_floating_point) value | `2e+100`, `1.02`, `-5`                                                                                                                        |
| float       | Alias for `number`                                                                             | `2e+100`, `1.02`, `-5`                                                                                                                        |
| integer     | Subset of `number`, integers between `-2^53 + 1` and `+2^53 - 1` (inclusive)                   | `0`, `-5`, `2000`                                                                                                                             |
| object      | Any JSON-serializable Object                                                                   | `{}`, `{"a":true}`, `{"hello":["world"]}`                                                                                                     |
| object.http | An object representing an HTTP Response. Accepts `headers`, `body` and `statusCode` keys       | `{"body": "Hello World"}`, `{"statusCode": 404, "body": "not found"}`, `{"headers": {"Content-Type": "image/png"}, "body": Buffer.from(...)}` |
| array       | Any JSON-serializable Array                                                                    | `[]`, `[1, 2, 3]`, `[{"a":true}, null, 5]`                                                                                                    |
| buffer      | Raw binary octet (byte) data representing a file.                                              | `{"_bytes": [8, 255]}` or `{"_base64": "d2h5IGRpZCB5b3UgcGFyc2UgdGhpcz8/"}`                                                                   |
| any         | Any value mentioned above                                                                      | `5`, `"hello"`, `[]`                                                                                                                          |

### Type coercion when making HTTP requests

The `buffer` type will automatically be converted to a `Buffer` from any `object` with a **single key-value pair matching the footprints** `{"_bytes": []}` or `{"_base64": ""}`.

Otherwise, parameters provided to a function are expected to match their defined types. Requests made over HTTP GET via query parameters or POST data with type `application/x-www-form-urlencoded` will be automatically converted from strings to their respective expected types, when possible.

Once converted, all types will undergo a final type validation. For example, passing a JSON `array` like `["one", "two"]` as a string to a parameter that expects an `object` will convert from string to JSON successfully but fail the `object` type check, as it is an array.

| Type        | Conversion Rule                                                                                        |
| ----------- | ------------------------------------------------------------------------------------------------------ |
| boolean     | `"t"` and `"true"` become `true`, `"f"` and `"false"` become `false`, otherwise will be kept as string |
| string      | No conversion: already a string                                                                        |
| number      | Determine float value, if NaN keep as string, otherwise convert                                        |
| float       | Determine float value, if NaN keep as string, otherwise convert                                        |
| integer     | Determine float value, if NaN keep as string, otherwise convert: may fail integer check                |
| object      | Parse as JSON, if invalid keep as string, otherwise convert: may fail object check                     |
| object.http | Parse as JSON, if invalid keep as string, otherwise convert: may fail object.http check                |
| array       | Parse as JSON, if invalid keep as string, otherwise convert: may fail array check                      |
| buffer      | Parse as JSON, if invalid keep as string, otherwise convert: may fail buffer check                     |
| any         | No conversion: keep as string                                                                          |

### Combining types

You can combine types using the pipe `|` operator. For example;

```javascript
/**
 * @param {string|integer} myparam String or an integer
 */
export async function GET (myparam) {
  // do something
} 
```

Will accept a `string` or an `integer`. Types defined this way will validate against the provided types in order of appearance. In this case, since it is a GET request and all parameters are passed in as strings via query parameters, `myparam` will **always** be received a string because it will successfully pass the string type coercion and validation first.

However, if you use a POST request:

```javascript
/**
 * @param {string|integer} myparam String or an integer
 */
export async function POST (myparam) {
  // do something
} 
```

Then you can pass in `{"myparam": "1"}` or `{"myparam": 1}` via the body which would both pass type validation.

You can combine as many types as you'd like:

```jsdoc
@param {string|buffer|array|integer}
```

Including `any` in your list will, as expected, override any other type specifications.

### Enums and restricting to specific values

Similar to combining types, you can also include specific JSON values in your type definitions:

```javascript
/**
 * @param {"one"|"two"|"three"|4} myparam String or an integer
 */
export async function GET (myparam) {
  // do something
} 
```

This allows you to restrict possible inputs to a list of allowed values. In the case above, sending `?myparam=4` via HTTP GET **will** successfully parse to `4` (`Number`), because it will fail validation against the three string options.

You can combine specific values and types in your definitions freely:

```jsdoc
@param {"one"|"two"|integer}
```

Just note that certain combinations will invalidate other list items. Like `{1|2|integer}` will accept any valid integer.

### Sizes (lengths)

The types `string`, `array` and `buffer` support sizes (lengths) via the `{a..b}` modifier on the type. For example;

```jsdoc
@param {string{..9}}  alpha
@param {string{2..6}} beta
@param {string{5..}}  gamma
```

Would expect `alpha` to have a maximum length of `9`, `beta` to have a minimum length of `2` but a maximum length of `6`, and `gamma` to have a minimum length of `5`.

### Ranges

The types `number`, `float` and `integer` support ranges via the `{a,b}` modifier on the type. For example;

```jsdoc
@param {number{,1.2e9}} alpha
@param {number{-10,10}} beta
@param {number{0.870,}} gamma
```

Would expect `alpha` to have a maximum value of `1 200 000 000`, `beta` to have a minimum value of `-10` but a maximum value of `10`, and `gamma` to have a minimum value of `0.87`.

### Arrays

Arrays are supported via the `array` type. You can optionally specify a schema for the array which applies to **every element in the array**. There are two formats for specifying array schemas, you can pick which works best for you:

```jsdoc
@param {string[]}      arrayOfStrings1
@param {array<string>} arrayOfStrings2
```

For multi-dimensional arrays, you can use nesting:

```jsdoc
@param {integer[][]}           array2d
@param {array<array<integer>>} array2d_too
```

**Please note**: Combining types are not currently available in array schemas. Open up an issue and let us know if you'd like them and what your use case is! In the meantime;

```jsdoc
@param {integer[]|string[]}
```

Would successfully define an array of integers or an array of strings.

### Object schemas

To define object schemas, use the subsequent lines of the schema after your initial object definition to define individual properties. For example, the object `{"a": 1, "b": "two", "c": {"d": true, "e": []}` Could be defined like so:

```jsdoc
@param {object}  myObject
@param {integer} myObject.a
@param {string}  myObject.b
@param {object}  myObject.c
@param {boolean} myObject.c.d
@param {array}   myObject.c.e
```

To define object schemas that are members of arrays, you must identify the array component in the property name with `[]`. For example:

```jsdoc
@param {object[]} topLevelArray
@param {integer}  topLevelArray[].value
@param {object}   myObject
@param {object[]} myObject.subArray
@param {string}   myObject.subArray[].name
```

### Parameter validation

The process for parameter validation takes the following steps:

1. Read parameters from the HTTP query string as type `application/x-www-form-urlencoded`
2. If applicable, read parameters from the HTTP body based on the request `Content-Type`
   * Supported content types:
     * `application/json`
     * `application/x-www-formurlencoded`
     * `multipart/form-data`
     * `application/xml`, `application/atom+xml`, `text/xml`
3. Query parameters **can not** conflict with body parameters, throw an error if they do
4. Perform type coercion on `application/x-www-form-urlencoded` inputs (query and body, if applicable)
5. Validate parameters against their expected types, throw an error if they do not match

During this process, you can encounter a `ParameterParseError` or a `ParameterError` both with status code `400`. `ParameterParseError` means your parameters could not be parsed based on the expected or provided content type, and `ParameterError` is a validation error against the schema for your endpoint.

### Query and Body parsing with `application/x-www-form-urlencoded`

Many different standards have been implemented and adopted over the years for HTTP query parameters and how they can be used to specify objects and arrays. To make things easy, Instant API supports all common query parameter parsing formats.

Here are some query parameter examples of parsing form-urlencoded data:

* Arrays
  * Duplicates: `?arr=1&arr=2` becomes `[1, 2]`
  * Array syntax: `?arr[]=1&arr[]=2` becomes `[1, 2]`
  * Index syntax: `?arr[0]=1&arr[2]=3` becomes `[1, null, 3]`
  * JSON syntax: `?arr=[1,2]` becomes `[1, 2]`
* Objects
  * Bracket syntax: `?obj[a]=1&obj[b]=2` becomes `{"a": 1, "b": 2}`
  * Dot syntax: `?obj.a=1&obj.b=2` becomes `{"a": 1, "b": 2}`
    * Nesting: `?obj.a.b.c.d=t` becomes `{"a": {"b": {"c": {"d": true}}}}`
  * JSON syntax: `?obj={"a":1,"b":2}` becomes `{"a": 1, "b": 2}`

### Query vs. Body parameters

With Instant API, **query and body parameters can be used interchangeably**. If you send both query parameters and body parameters to an endpoint, they will be combined into a single parameter object. GET and DELETE requests only support query parameters.

## That's it!

That covers the basics of building packages on Instant.bot. If you have any questions, please do not hesitate to jump into our community Discord server at [discord.gg/instant](https://discord.gg/instant).

