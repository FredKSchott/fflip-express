# fflip-express

**fflip-express** is an [Express.js](http://expressjs.com/) integration for the [fflip](https://github.com/FredKSchott/fflip) feature-flagging library.

```
npm install fflip-express --save
```


## Configuration

```javascript
var FFlipExpressIntegration = require('fflip-express');

// The first `fflip` argument is the fflip module used in your application
// The second `options` argument is optional, and lets you configure your integration
var fflipExpress = new FFlipExpressIntegration(fflip, options);
```

Available configuration options include:

- `cookieName`: The name of the cookie where a users manual feature flags will be saved. (Defaults to 'fflip')
- `cookieOptions`: Set additional options for the fflip cookie, such as `expires` & `maxAge`. (No default)
- `manualRoutePath`: The URL path for the manual feature flipping endpoint. Must include both `:name` and `:action` route params. (Defaults to '/fflip/:name/:action')


## Usage

This integration provides two different ways to connect fflip into your express application; both of them require that you have `req.cookies` available from cookieParser:

```javascript
app.use(cookieParser());
```

### `fflipExpress.middleware`

```javascript
app.use(fflipExpress.middleware);
```

**req.fflip:** A special fflip request object is attached to the request object, and includes the following functionality:

```
req.fflip = {
  setForUser(user): Given a user, attaches the features object to the request (at req.fflip.features). Make sure you do this before calling has()!
  has(featureName): Given a feature name, returns the feature boolean, undefined if feature doesn't exist. Throws an error if setForUser() hasn't been called
}
```

**Use fflip in your templates:** Once `setForUser()` has been called, fflip will include a `Features` template variable that contains your user's enabled features. Here is an example of how to use it with Handlebars: `{{#if Features.closedBeta}} Welcome to the Beta! {{/if}}`

**Use fflip on the client:** Once `setForUser()` has been called, fflip will also include a `FeaturesJSON` template variable that is the JSON string of your user's enabled features. To deliver this down to the client, just make sure your template something like this: `<script>var Features = {{ FeaturesJSON }}; </script>`.


### `fflipExpress.manualRoute`

```javascript
// Feel free to use any route you'd like, as long as `name` & `action` exist as route parameters.
app.get('/custom_path/:name/:action', fflipExpress.manualRoute);
```

**A route for manually flipping on/off features:** If you have cookies enabled, you can visit this route to manually override a feature to always return true/false for your own session. Just replace ':name' with the Feature name and ':action' with 1 to enable, 0 to disable, or -1 to reset (remove the cookie override). This override is stored in the user's cookie under the name `fflip`, which is then read by `fflip.expressMiddleware()` and `req.fflip` automatically.

### `fflipExpress.connectAll(app)`

Sets up the express middleware and route automatically. Equivalent to running:

```javascript
app.use(fflipExpress.middleware);
app.get(fflipExpress.options.manualRoutePath, fflipExpress.manualRoute);
```
