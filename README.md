## Installation

```bash
$ npm install lighthouse-lambda --save
```

## Lambda function

```js
// index.js

const createLighthouse = require('lighthouse-lambda')

exports.handler = function (event, context, callback) {
  Promise.resolve()
    .then(() => createLighthouse('https://example.com', { logLevel: 'info' }))
    .then(({ chrome, start }) => {
      return start()
        .then((results) => {
          // Do something with `results`
          return chrome.kill().then(() => callback(null))
        })
        .catch((error) => {
          // Handle errors when running Lighthouse
          return chrome.kill().then(() => callback(error))
        })
    })
    // Handle other errors
    .catch(callback)
}
```

## Support

There is none(!), use at your own peril.

## Testing locally

You can use [docker-lambda](https://github.com/lambci/docker-lambda) to test your Lambda function locally.

```bash
$ docker run --rm -v "$PWD":/var/task lambci/lambda:nodejs12.x index.handler
```


## Deployment

You can use [docker-lambda](https://github.com/lambci/docker-lambda) to install dependencies and pack your Lambda function.

```bash
$ docker run --rm -v "$PWD":/var/task lambci/lambda:build-nodejs12.x bash -c "rm -rf node_modules && npm install"

$ docker run --rm -v "$PWD":/var/task lambci/lambda:build-nodejs12.x bash -c "rm -f *.zip && zip lambda.zip -r node_modules index.js package.json"
```

- The file will be big (at least 75MB), so you need to upload it to S3 then deploy to Lambda from S3.
- You should allocate at least 512 MB memory and 15 seconds timeout to the function.

## API

### createLighthouse(url, [options], [config])

Same parameters as [Using Lighthouse programmatically](https://github.com/GoogleChrome/lighthouse/blob/master/docs/readme.md#using-programmatically).

Returns a `Promise` of an Object with the following fields:

- `chrome`: an instance of [`chromeLauncher.launch()`](https://github.com/GoogleChrome/chrome-launcher#launchopts), remember to call `chrome.kill()` in the end.
- `log`: an instance of [lighthouse-logger](https://github.com/GoogleChrome/lighthouse/tree/master/lighthouse-logger) (only if you set `options.logLevel`).
- `start()`: a function to start the scan which returns a `Promise` of Lighthouse results.

## Credits

Based on `lighthouse-lambda`(https://github.com/joytocode/lighthouse-lambda/tree/archived-v2) but updated for node 12.

`lighthouse-lambda` uses the Headless Chrome binary (stable version) from [@serverless-chrome/lambda](https://www.npmjs.com/package/@serverless-chrome/lambda).

## License

[MIT](LICENSE)
