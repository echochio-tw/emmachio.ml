## Version

* vue: 2.5.2
* vue-loader: 13.3.0

## Extensions

* [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur)
* [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
* [EditorConfig for VS Code](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig)

## Initial Project

Use [vue-cli](https://github.com/vuejs/vue-cli) to initial project.

## npm install

* Prettier([reference](https://alligator.io/vuejs/vue-eslint-prettier/))

```bash
# prettier
npm install --save-dev prettier eslint-plugin-prettier eslint-config-prettier

# pug
npm install --save-dev pug
```

## eslintrc.js

Change `extends` in `eslintrc.js`:

```js
{
  extends: ['plugin:vue/recommended', 'plugin:prettier/recommended', 'airbnb-base'],
}
```

## .prettierrc

Set single quote and trailing comma to prevent prettier conflicting with eslint for airbnb javascript style.([reference](https://stackoverflow.com/questions/46201647/prettier-airbnbs-eslint-config?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa))

```js
{
  "singleQuote": true,
  "trailingComma": 'all',
}
```

## settings.json

Add `vue` to `eslint.validate` in VS Code config:

```js
{
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    {
      "language": "vue",
      "autoFix": true
    },
  ]
}

```