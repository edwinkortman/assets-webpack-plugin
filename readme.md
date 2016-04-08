# assets-webpack-plugin

[ ![Codeship Status for sporto/assets-webpack-plugin](https://codeship.com/projects/c9171f30-f64d-0132-8e3e-02d99c35d383/status?branch=master)](https://codeship.com/projects/85994)

Webpack plugin that emits a json file with assets paths.

## Table of Contents

- [Why Is This Useful?](#why-is-this-useful)

- [Install](#install)

- [Configuration](#configuration)

- [Test](#test)

- [Change Log](./CHANGELOG.md)

- [Contributing Guidelines](./CONTRIBUTING.md)

## Why Is This Useful?

When working with Webpack you might want to generate your bundles with a generated hash in them (for cache busting).

This plug-in outputs a json file with the paths of the generated assets so you can find them from somewhere else.

### Example output:

The output is a JSON object in the form:

```json
{
    "chunks": {
      "bundle_name": {
          "asset_kind": "/public/path/to/asset"
      }
    },
    "images": [{
        "name": "hello.svg",
        "path": "/public/path/to/asset/hello.svg"
      }
    ]
}
```

Where:

  * `chunks` is an object that contains all entry points listed in your webpack config
    * `"bundle_name"` is the name of the bundle (the key of the entry object in your webpack config, or "main" if your entry is an array).
    * `"asset_kind"` is the camel-cased file extension of the asset
  * `images` is an array containing all images processed by webpack
    * `name` is the name of the image (path used in `require` or `import`)
    * `path` is the webpack generated public path to the image

For example, given the following webpack config:

```js
{
    entry: {
        one: ['src/one.js'],
        two: ['src/two.js']
    },
    output: {
        path: path.join(__dirname, "public", "js"),
        publicPath: "/js/",
        filename: '[name]_[hash].bundle.js'
    }
}
```

and import of an image somewhere in application code using

```js
import user from '../../assets/images/user.svg';
```

The plugin will output the following json file:

```json
{
    "one": {
        "js": "/js/one_2bb80372ebe8047a68d4.bundle.js"
    },
    "two": {
        "js": "/js/two_2bb80372ebe8047a68d4.bundle.js"
    },
    "images":[{
      "name":"assets/images/user.svg",
      "path":"/js/assets/images/user.svg"
    }]
}
```

## Install

```sh
npm install assets-webpack-plugin --save-dev
```

## Configuration

In your webpack config include the plug-in. And add it to your config:

```js
var path = require('path')
var AssetsPlugin = require('assets-webpack-plugin')
var assetsPluginInstance = new AssetsPlugin()

module.exports = {
    // ...
    output: {
        path: path.join(__dirname, "public", "js"),
        filename: "[name]-bundle-[hash].js",
        publicPath: "/js/"
    },
    // ....
    plugins: [assetsPluginInstance]
}
```

### Options

You can pass the following options:

__filename__: Name for the created json file. Defaults to `webpack-assets.json`

```js
new AssetsPlugin({filename: 'assets.json'})
```

__fullPath__: True by default. If false the output will not include the full path of the generated file.

```js
new AssetsPlugin({fullPath: false})
```

e.g.

`/public/path/bundle.js` vs `bundle.js`

__path__: Path where to save the created json file. Defaults to the current directory.

```js
new AssetsPlugin({path: path.join(__dirname, 'app', 'views')})
```

__prettyPrint__: Whether to format the json output for readability. Defaults to false.

```js
new AssetsPlugin({prettyPrint: true})
```

__processOutput__: Formats the assets output. Defaults is JSON stringify function.

```js
new AssetsPlugin({
    processOutput: function (assets) {
        return 'window.staticMap = ' + JSON.stringify(assets)
    }
})
```

__update__: When set to true, the output json file will be updated instead of overwritten. Defaults to false.

```js
new AssetsPlugin({update: true})
```

__metadata__: Inject metadata into the output file. All values will be injected into the key "metadata".

```js
new AssetsPlugin({metadata: {version: 123}})

// Manifest will now contain:
// {
//   metadata: {version: 123}
// }
```


### Using in multi-compiler mode

If you use webpack multi-compiler mode and want your assets written to a single file,
you __must__ use the same instance of the plugin in the different configurations.

For example:

```js
var webpack = require('webpack')
var AssetsPlugin = require('assets-webpack-plugin')
var assetsPluginInstance = new AssetsPlugin()

webpack([
    {
        entry: {one: 'src/one.js'},
        output: {path: 'build', filename: 'one-bundle.js'},
        plugins: [assetsPluginInstance]
    },
    {
        entry: {two:'src/two.js'},
        output: {path: 'build', filename: 'two-bundle.js'},
        plugins: [assetsPluginInstance]
    }
])
```


### Using this with Rails

You can use this with Rails to find the bundled Webpack assets via sprockets. In `ApplicationController` you might have:

```ruby
def script_for(bundle)
  path = Rails.root.join('app', 'views', 'webpack-assets.json') # This is the file generated by the plug-in
  file = File.read(path)
  json = JSON.parse(file)
  json['chunks'][bundle]['js']
end
```

Then in the actions:

```ruby
def show
  @script = script_for('clients') # this will retrieve the bundle named 'clients'
end
```

And finally in the views:

```erb
<div id="app">
  <script src="<%= @script %>"></script>
</div>
```

## Test

```sh
npm test
```
