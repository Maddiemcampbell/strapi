# Error catching

In this guide we will see how you can catch errors and send them to the Application Monitoring / Error Tracking Software you want to utilize.

::: tip
In this example we will use [Sentry](https://sentry.io).
:::

## Create a middleware

A [middleware](../concepts/middlewares.md) will be used in order to catch the errors which will then be sent to Sentry.

- Create a `./middlewares/sentry/index.js` file.

**Path —** `./middlewares/sentry/index.js`

```js
module.exports = strapi => {
  return {
    initialize() {
      strapi.app.use(async (ctx, next) => {
        await next();
      });
    },
  };
};
```

## Handle errors

Here is the [Node.js client documentation](https://docs.sentry.io/platforms/node/).

Install it with `yarn add @sentry/node` or `npm install @sentry/node --save`.

- Now add the logic that will catch errors.

**Path —** `./middlewares/sentry/index.js`

```js
const Sentry = require('@sentry/node');
Sentry.init({
  dsn: 'https://<key>@sentry.io/<project>',
  environment: strapi.config.environment,
});

module.exports = strapi => {
  return {
    initialize() {
      strapi.app.use(async (ctx, next) => {
        try {
          await next();
        } catch (error) {
          Sentry.captureException(error);
          throw error;
        }
      });
    },
  };
};
```

::: warning
It's important to call `throw(error);` to avoid stopping the middleware stack. If you don't re-throw the error, it won't be handled by the Strapi's error formatter and the api will never respond to the client.
:::

## Configure the middleware

Make sure your middleware is added at the end of the middleware stack.

**Path —** `./config/middleware.json`

```json
{
  ...
    "after": [
      "parser",
      "router",
      "sentry"
    ]
  }
}
```

And finally you have to enable the middleware.

**Path —** `./config/environments/**/middleware.json`

```json
{
  "sentry": {
    "enabled": true
  }
}
```
