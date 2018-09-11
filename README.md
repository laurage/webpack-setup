Creating a React app from scratch.

>>> Read this to understand: https://www.valentinog.com/blog/webpack-4-tutorial/

https://developers.google.com/web/fundamentals/performance/webpack/decrease-frontend-size

https://webpack.js.org/concepts/#entry

add jest
compare with packages used in elementary

# Table of Contents

- [Adding .editorconfig file to the root folder](#adding-editorconfig)

# Initialise our app

Create a new directory for our app:

```
mkdir react-scratch
cd react-scratch
```

# Create the package.json file

```npm init -f```

# Styling configs

## Adding .editorconfig file to the root folder

Add .editorconfig file to maintain consistent coding styles between different editors and IDEs.

http://editorconfig.org/

```
# EditorConfig is awesome: http://EditorConfig.org

root = true

[*]
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
charset = utf-8
indent_style = space
indent_size = 2

[*.{md,markdown}]
indent_size = 4
trim_trailing_whitespace = false

[{package.json}]
indent_style = space
indent_size = 2
```

Make sure that your editor reads the .editorconfig file instead of its own settings.
For instance, VisualStudioCode requires a plugin installation: https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig

## Adding Eslint

Add Eslint to provide a pluggable linting utility for JavaScript.

```npm i -D eslint eslint-plugin-react```

Optional:

```npm i -D eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-jsx-a11y```

Then create .eslint and write:

```
module.exports = {
  root: true,
  env: {
    'browser': true,
    'jest': true,
  },
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    sourceType: 'module'
  },
  rules: {
    'semi': [2, 'never'],
    'max-len': 0,
    'react/jsx-filename-extension': [1, { 'extensions': ['.js', '.jsx'] }],
    'linebreak-style': 0,
    "import/no-extraneous-dependencies": [
      "error", {
        "devDependencies": true,
        "optionalDependencies": false,
        "peerDependencies": false
      }
    ]
  }
}
```

If we added eslint-config-airbnb:   
```
extends: 'airbnb',
  'settings': {
    'import/resolver': {
      'webpack': {
        'config': 'configs/webpack.config.js'
      }
    }
  },
```

And add lint command to package.json scripts section:

```
"scripts": {
  "test": "eslint ."
},
```

To run it use npm run command:

```npm test```

### eslintignore

No need to check webpack files and js files in build by eslint, so creation of an .eslintignore file with:

```
webpack.config.js
build/*.js
``` 

## Adding StyleLint

Add StyleLint to lint CSS/SCSS/Less/Sass.

Follow the steps here (for visual studio code): https://github.com/shinnn/vscode-stylelint

## Browserlist

Several tools (such as Autoprefixer or StyleLint and babel-preset-env) require a list of supported browsers. Rather than writing this list for every different tools, use Browserlist to create a unique list of supported browser, to which each of the tools that need it will refer to. (see https://css-tricks.com/browserlist-good-idea/ for detailed explanation)

Create a .browserslistrc

```
# Browsers that we support

> 1%
Last 2 versions
IE 10 # sorry
```

Use http://browserl.ist/ to test which browsers are supported for each string.

# Transcribe development code into browser readable code

## Babel

Babel is compiler for writing next generation JavaScript. It transforms react jsx and es6 code into old es5 code, readable by old browsers.

We must install babel plugins and add it to the .babelrc file.

To install Babel for nodeJS, we need:
- babel-core: babel compiler
- babel-preset-env: compiles ES2015+ down to ES5 by automatically determining the Babel plugins and polyfills needed based on my targeted browser or runtime environments.
- babel-preset-react: transform jsx into JS

- babel-plugin-transform-class-properties: install to use the new static object properties feature
- babel-plugin-transform-object-rest-spread: isntall to use the rest spread
- [babel-polyfill](#https://babeljs.io/docs/usage/polyfill/): polyfills Promises, Object.assign, etc

- babel-plugin-lodash: cherry-picks Lodash modules

This is only needed in dev environment:

Core:
```
npm i babel-loader @babel/core @babel/preset-env @babel/preset-react babel-plugin-transform-class-properties babel-plugin-transform-object-rest-spread  --save-dev
npm i -S babel-polyfill
```

Other Addition (not added here):
```
npm i babel-plugin-lodash --save-dev
```

Create a configuration file .babelrc
```
{
    "presets": [
        "@babel/preset-env",
        "@babel/preset-react"
    ],
    "plugins": [
      "transform-class-properties",
      "transform-object-rest-spread",
    ]
}
```

# Prepare and serve the code for development and production

## Webpack

Bundles resources and assets, and outputs out js code in specified location.
The webpack-dev-server lib will helps running a development server which will give benefits like hmr & live reload .

We will use:
- webpack
- webpack-cli
- [html-webpack-plugin](#) — simplifies creation of HTML files to serve your webpack bundles.
- html-loader
- [style-loader](#https://github.com/webpack-contrib/style-loader) — adds CSS to the DOM by injecting a <style> tag.
- [css-loader](#) — interprets @import and url() like import/require() and will resolve them.
- [sass-loader with node-sass](#) — sass loader for webpack. Compiles Sass to CSS.
- [DevServer](#https://webpack.js.org/guides/development/) — development server that provides live reloading.

In addition(not used here:)
- [mini-css-extract-plugin](#https://github.com/webpack-contrib/mini-css-extract-plugin) (replaces extract-text-webpack-plugin)
- [copy-webpack-plugin](#https://www.npmjs.com/package/copy-webpack-plugin) — copies individual files or entire directories to the build directory.
- [postcss-loader](#https://github.com/postcss/postcss-loader) — PostCSS loader for webpack.
- [file-loader](#) — instructs webpack to emit the required object as file and to return its public URL.
- [clean-webpack-plugin](#) — a webpack plugin to remove your build folder(s) before building.
- [webpack-bundle-analyzer](#) — webpack plugin and CLI utility that represents bundle content as convenient
- [interactive zoomable treemap](#)
- [lodash-webpack-plugin](#) — create smaller Lodash builds by replacing feature sets of modules with noop, identity, or simpler alternatives.

Needed:
```
npm i webpack webpack-cli html-webpack-plugin html-loader style-loader css-loader sass-loader node-sass webpack-dev-server --save-dev
```

Additional: (not addded here)
```
npm i mini-css-extract-plugin copy-webpack-plugin postcss-loader file-loader clean-webpack-plugin webpack-bundle-analyzer lodash-webpack-plugin --save-dev
```

## edit package.json

### --mode
webpack --mode development : creates a bundle file that isn't minified
webpack --mode production : creates a bundle file that is minified

webpack 4 doesn't (for now at least) allow the access to the --mode production or --mode development later on
so we still have to setup the NODE--ENV variables ourselves to be able to access this in loaders
In webpack.config.js, define: const devMode = process.env.NODE_ENV !== 'production'

https://github.com/webpack/webpack/issues/6496 

### other flags
entry point: ./src/index.jsx
output: --output ./build/main.js

--open: opens the app in the browser

--hot: enables react-hot-reloader. (see other sections for full implementation)

```
"scripts": {
    "dev": "NODE_ENV=development webpack --mode development ./src/index.jsx --output ./build/main.js --open",
    "build": "NODE_ENV=production webpack --mode production ./src/index.jsx --output ./build/main.js",
    "test": "eslint ."
  },
```

# React

Install the following packages:

- React
- React-DOM — This package serves as the entry point of the DOM-related rendering paths.

```
npm i -S react react-dom
```

Optional, but useful for most projects:
- [Proptypes](#https://reactjs.org/docs/typechecking-with-proptypes.html)
- Redux — a predictable state container for JavaScript apps.
- react-redux — official React bindings for Redux.
- react-router — declarative routing for react.
- react-router-redux — ruthlessly simple bindings to keep react-router and redux in sync.
- [react-hot-loader](#https://github.com/gaearon/react-hot-loader) - React Hot Loader is a plugin that allows React components to be live reloaded without the loss of state. See https://github.com/gaearon/react-hot-loader for steps

```
npm i -S redux react-redux react-router react-router-redux prop-types
npm i react-hot-loader --save-dev
```

In a component:
```js
import PropTypes from 'prop-types'; 
```

# Source folder

create a ```src``` folder in root.

## src/index.html

## src/index.jsx

Entry point.

# Testing frameworks
## Jest

## Enzyme

```
npm install --save-dev jest enzyme enzyme-adapter-react-16
```
 
# Additional interesting packages:

Dependencies:
- Dompurify: HTML, MathML and SVG sanitizer
- hammerjs: touch gestures for web apps

// from diverse sources:
- moment — parse, validate, manipulate, and display dates and times in JavaScript.
- RxJS — a library for reactive programming using Observables, to make it easier to compose asynchronous or callback-based code.
- redux-observable — RxJS middleware for action side effects in Redux using “Epics”.
- react-virtualized — React components for efficiently rendering large lists and tabular data.
- localForage — offline storage, improved. Wraps IndexedDB, WebSQL, or localStorage using a simple but powerful API.
- react-loadable — a higher order component for loading components with promises.

DevDependencies:
- sass-vars-loader
- es6-promise
- file-loader
- expose-loader
- raw-loader
- storybook
- storybook-readme
- friendly-errors-webpack-plugin
- husky
- json-loader
- lodash — a JavaScript utility library delivering consistency, modularity, performance, & extras.
- postcss
- rimraf
- react-test-renderer
- redux-mock-store
- stylefmt
- stylefmt-loader
- whatwg-fetch


# PostCSS (not added here)

PostCSS is a tool for transforming styles with JS plugins. These plugins can lint your CSS, support variables and mixins, transpile future CSS syntax, inline images, and more.

We want to install:
- postcss
- postcss-cssnext: transforms new CSS specs into compatible CSS that aren't yet supported by all browsers. (it uses autoprefixer)
- cssnano: minifies CSS.

```
npm i -D postcss postcss-cssnext cssnano
```

Config file .postcss.config.js:

```js
module.exports = {
  plugins: {
    'postcss-cssnext': {
      warnForDuplicates: false,
    },
    cssnano: {},
  },
};
```
