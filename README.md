[![Build Status](https://img.shields.io/travis/imagerjs/imager/generators.svg?style=flat)](https://travis-ci.org/imagerjs/imager)
[![Gittip](https://img.shields.io/gratipay/madhums.svg?style=flat)](https://www.gratipay.com/madhums/)
[![Dependencies](https://img.shields.io/david/imagerjs/imager.svg?style=flat)](https://david-dm.org/imagerjs/imager)

**wip**: This is work in progress. Converting to use generators and [graphicsmagick-stream](https://github.com/e-conomic/graphicsmagick-stream) library. The uploading is handled by [pkgcloud](https://github.com/pkgcloud/pkgcloud).

## Imager

A node module to resize, crop and upload images (with different variants and presets) to the cloud.

## Dependencies

1. node >= 0.11.12 with `--harmony` flag
2. You need to install libgraphicsmagicks.

  Using osx

  ```sh
  $ brew install graphicsmagick --build-from-source
  ```

  Using ubuntu

  ```sh
  $ sudo apt-get install libgraphicsmagick1-dev
  ```

## Installation

```sh
$ npm install imager
```

## Config

### variants

```js
exports.variants = {
  item: {             // variant
    thumb: {          // preset
      options: {      // preset options
        pool: 5,
        scale: { width: 200, height: 150, type: 'contain' },
        crop: { width: 200, height: 150, x: 0, y: 0 },
        format: 'png',
        rotate: 'auto',
      }
    },
    large: {
      original: true  // upload original image without image processing
    }
  },
  gallery: {
    // ...
  }
};
```

In the above config, item and gallery are variants. thumb and large are presets. Each preset has an options object which is a [graphicsmagick-stream](https://github.com/e-conomic/graphicsmagick-stream#usage) config object.

#### preset options

- `options` - An object that is passed to graphicsmagick. See what options are available [here](https://github.com/e-conomic/graphicsmagick-stream#usage)
- `rename` - A function that accepts an object `file` as an argument. It has the following properties: `name`, `size`, `type` and `path`. It is called before uploading each file.

  Example:
  ```js
  variants.item.thumb.rename = function (file) {
    return 'users/1/thumb/' + file.name;
  };
  var imager = new Imager(variants.item, ...);
  ```
- `original` - A true value. If this option is set, the original image will be uploaded without any image processing.

### storages

```js
exports.storages = {
  local: {
    provider: 'local',
    path: '/tmp',
    mode: 0777
  },
  rackspace: {
    provider: 'rackspace',
    username: process.env.IMAGER_RACKSPACE_USERNAME,
    apiKey: process.env.IMAGER_RACKSPACE_KEY,
    authUrl: 'https://lon.auth.api.rackspacecloud.com',
    region: 'IAD', // https://github.com/pkgcloud/pkgcloud/issues/276
    container: process.env.IMAGER_RACKSPACE_CONTAINER
  },
  amazon: {
    provider: 'amazon',
    key: process.env.IMAGER_S3_KEY,
    keyId: process.env.IMAGER_S3_KEYID,
    container: process.env.IMAGER_S3_BUCKET
  }
}
```

## Usage

```js
var imager = new Imager(variants.item, storages.amazon);
// You can also pass only the storage without a variant which will upload the
// original image
// new Imager(storages.amazon)
```

## API

### .upload(files, callback)

`files` is an array of files or a single file. A file can be a file object, absolute file path pointing a local file or base64 encoded image data. `callback` accepts `err` and an object containing the array of uploaded images.

```js
var config = require('./imager-config.js');
var imager = new Imager(config.variants.item, config.storage);
imager.upload(files, function (err, avatar) {
  // avatar =>
  // {
  //   thumb: [ 'https://fudge.s3.amazonaws.com/user/1/thumb/image-1.png', ],
  //   large: [ 'https://fudge.s3.amazonaws.com/user/1/large/image-1.png', ]
  // }
});
```

### .remove()
### .regenerate()

## Tests

```sh
$ npm test
```

## License

MIT
