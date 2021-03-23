## Sharing one alias confing for webpack, babel, eslint
1. Setup a `jsconfig.json`
```js
// jsconfig.json
{
  "include": ["./src/**/*"],
  "exclude": ["node_modules", "dist", "devapp-build"],
  "compilerOptions": {
    "baseUrl": "src",
    "module": "commonjs",
    "target": "ES6",
    "sourceMap": true,
    "inlineSources": true,
    "jsx": "react",
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "paths": {
      "Atoms/*": ["./atoms/*"],
      "Components/*": ["./components/*"],
      "Elements/*": ["./elements/*"],
      "Fonts/*": ["./fonts/*"],
      "Form/*": ["./form/*"],
      "Hooks/*": ["./hooks/*"],
      "Layout/*": ["./layout/*"],
      "Misc/*": ["./misc/*"],
      "Services/*": ["./services/*"],
      "Stories/*": ["./stories/*"],
      "Styles/*": ["./styles/*"],
      "Utils/*": ["./utils/*"]
    }
  }
}
```
2. Add this parser file which should import the jsconfig. One file to parse jsconfig.json to a regular dictionairy object
```js 
// aliases.js
const {
  compilerOptions: { paths: unprocessAliases },
} = require(`../jsconfig.json`);

const aliases = Object.keys(unprocessAliases).reduce((obj, alias) => {
  const key = alias.match(/[a-zA-Z0-9]+/g);
  const rawValue = unprocessAliases[alias][0];
  const value = rawValue.match(/[a-zA-Z0-9]+/g)[0];
  return {
    ...obj,
    [key]: value,
  };
}, {});

module.exports = aliases;
```
3. Just import the aliases parser and transform the object into the shape your configuration expects
Webpack Example: 
```js
// webpack config
const aliases = require('./aliases');

module.exports = {
  ...,
  resolve: {
    ...,
    alias: Object.keys(aliases).reduce(
      (obj, alias) => ({
        ...obj,
        [alias]: path.resolve(ROOT_DIR, `src/${aliases[alias]}`),
      }),
      {}
    ),
  },
  ...
};
```
Babel Example
```js
// babel.config.js
const aliases = require('./scripts/aliases');

module.exports = {
  ...,
  plugins: [
    [
      'module-resolver',
      {
        root: ['./src'],
        alias: Object.keys(aliases).reduce(
          (obj, alias) => ({
            ...obj,
            [alias]: `./src/${aliases[alias]}`,
          }),
          {}
        ),
      },
    ],
  ],
};

```
Eslint Example
```js
// eslintrc.js
module.exports = {
  ...,
    settings: {
    'import/resolver': {
      alias: {
        map: Object.keys(aliases).reduce(
          (arr, alias) => [...arr, [alias, `./src/${aliases[alias]}`]],
          []
        ),
        extensions: ['.js', '.jsx'],
      },
    },
  },
  ...,
}
```
