开发环境：VSCode

VSCode 插件： [eslint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) & [prettier-vscode](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) & [stylelint](https://marketplace.visualstudio.com/items?itemName=stylelint.vscode-stylelint)

使用 `create-react-app` 创建项目：

```bash
npx create-react-app my-app
```

## :one: 配置 Prettier

安装依赖：

```bash
# -E 表示 --save-exact
pnpm i -D -E prettier
```

安装 [@trivago/prettier-plugin-sort-imports](https://github.com/trivago/prettier-plugin-sort-imports) 插件，用于import文件排序:

```bash
pnpm i -D @trivago/prettier-plugin-sort-imports
```

设置 `.prettierrc` & `.prettierignore`:

```bash
touch .prettierrc .prettierignore
```

配置 `.prettierrc` ：

```json
{
  "printWidth": 100,
  "tabWidth": 2,
  "semi": false,
  "singleQuote": true,
  "trailingComma": "none",
  "importOrder": ["^@core/(.*)$", "^@server/(.*)$", "^@ui/(.*)$", "^[./]"],
  "importOrderSeparation": true,
  "importOrderSortSpecifiers": true
}
```

配置 `.prettierignore` ：

```tex
build
coverage
node_modules
dist
public/
pnpm-lock.yaml
.vscode/*
```

**设置 `vscode` 以便于保存自动格式化**：

```bash
# 创建 settings.json 文件
touch .vscode/settings.json
```

配置：需要安装 [prettier-vscode](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "eslint.codeAction.showDocumentation": {
    "enable": true
  },
  "eslint.alwaysShowStatus": true
}
```

配置 `package.json` 中的 `scripts` :

```:two:
"scripts": {
  ...
  "prettier:fix": "prettier --write",
  "prettier:check": "prettier --check ."
}
```

## :two: 配置 ESLint & Babel

安装 eslint 相关依赖：

```bash
pnpm i -D eslint eslint-config-airbnb eslint-config-prettier \
	eslint-plugin-import eslint-plugin-jest eslint-plugin-jsx-a11y \
	eslint-plugin-react eslint-plugin-react-hooks
```

安装 babel 相关依赖：

```bash
pnpm i -D @babel/core @babel/eslint-parser @babel/preset-react
```

配置 `.eslintrc` `.eslintignore`:

```bash
touch .eslintrc .eslintignore
```

`.eslintignore`:

```
build
node_modules
dist
public/
pnpm-lock.yaml
.vscode/*
```

`.eslintrc`:

```json
{
  "parser": "@babel/eslint-parser",
  "env": {
    "browser": true,
    "es6": true,
    "node": true,
    "jest/globals": true
  },
  "extends": [
    "airbnb",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "plugin:import/errors",
    "plugin:import/warnings",
    "plugin:jsx-a11y/recommended",
    "prettier",
    "plugin:prettier/recommended"
  ],
  "plugins": ["react", "jest", "prettier"],
  "rules": {
    "prettier/prettier": ["error"],
    "no-console": "off",
    "import/no-extraneous-dependencies": 0,
    "react/jsx-filename-extension": [
      "warn",
      {
        "extensions": [".js", ".jsx"]
      }
    ],
    "react/jsx-uses-react": "off",
    "react/react-in-jsx-scope": "off",
    "import/namespace": 0,
    "import/no-named-as-default": 0,
    "import/default": 0,
    "no-unused-vars": 1,
    "no-plusplus": 0
  },
  "parserOptions": {
    "sourceType": "module",
    "allowImportExportEverywhere": true,
    "ecmaFeatures": {
      "globalReturn": false
    },
    "babelOptions": {
      "configFile": "./.babelrc"
    }
  }
}
```

配置 `.babelrc`:

```bash
touch .babelrc
```

配置：

```json
{
  "presets": ["@babel/preset-react"]
}
```

配置 `package.json` 中的 `scripts` :

```json
"script": {
  ...
  "lint": "eslint --fix \"./src/**/*.{js,jsx,json}\""
},
"eslintConfig": {
  "extends": [
    "react-app",
    "react-app/jest"
  ]
}
```

## :three: 配置 Husky & lint-staged

Husky 版本 `7+`

安装依赖：

```bash
pnpm i -D husky lint-staged
```

创建 `.husky` 配置文件夹：

```bash
npx husky add .husky/pre-commit "pnpm lint-staged"
```

定义 `.lintstagedrc` 文件：

```json
{
  "src/**/*.{js,json,jsx,json}": ["eslint"],
  "src/**/*.{js,jsx,json,css,scss,md}": ["prettier --write ."]
}
```

可以参考：[How to use husky v6 with lint-staged? - @github issue](https://github.com/typicode/husky/issues/949#issuecomment-823807906)

另外还可以配置 `@commitlint/config-conventional`， 这里就没有配置了。





## :four: 配置stylelint

安装依赖：

```bash
pnpm i -D sass stylelint stylelint-scss stylelint-prettier \
	stylelint-config-standard-scss stylelint-config-prettier-scss \
	postcss-scss
```

 配置 `.stylelintrc.js` & `.stylelintignore`:

```bash
touch .stylelintrc.js .stylelintignore
```

`.stylelintrc.js`:

```js
module.exports = {
  plugins: ['stylelint-scss', 'stylelint-prettier'],
  extends: ['stylelint-config-standard-scss', 'stylelint-config-prettier-scss'],
  customSyntax: 'postcss-scss',
  rules: {
    'prettier/prettier': true
  }
}
```

`.stylelintignore`: 

```
# 其他类型文件
*.js
*.ts
*.jpg
*.woff

# 测试和打包目录
/dist/
/node_modules/
```

结合vscode插件自动格式化样式 ，配置可参考 [stylelint](https://marketplace.visualstudio.com/items?itemName=stylelint.vscode-stylelint)

```json
{
  "editor.codeActionOnSave": {
    ...
    "source.fixAll.stylelint": true
  },
  "stylelint.validate": ["css", "postcss", "scss", "sass"],
  "stylelint.snippet": [
    "css",
    "scss",
    "postcss",
    "sass"
  ]
}
```

设置scrits:

```json
"scripts": {
  ...
  "style": "stylelint \"src/**/*.(jsx|scss|css)\" --fix"
}
```

最后注意将项目中的css改为 `scss` 格式，虽然不改也可以，但是为了一致性，最好进行统一



关于stylelint配置相关教程：

1. [万字长文详解从零搭建企业级 vue3 + vite2+ ts4 框架全过程 - 流年丶风尘@掘金](https://juejin.cn/post/7069315908597973023#heading-17)
2. [Stylelint 配置使用 - 阿离王](https://347830076.github.io/myBlog/standard/stylelint.html#%E8%87%AA%E5%AE%9A%E4%B9%89css%E5%B1%9E%E6%80%A7%E9%A1%BA%E5%BA%8F%E8%A7%84%E5%88%99)
3. [Linter上手完全指南 - yanhaixiang](https://github.yanhaixiang.com/linter-guide/practice/stylelint.html#stylelint-x-less)





## :five: 配置EditorConfig

参考 [editorconfig](https://editorconfig.org/) :

```bash
touch .editorconfig
```

配置如下：

```
# top-most EditorConfig file
root = true

# Unix-style newlines with a newline ending every file
[*]
end_of_line = lf
insert_final_newline = true

# Matches multiple files with brace expansion notation
# Set default charset
[*.{js,py}]
charset = utf-8

# 4 space indentation
[*.py]
indent_style = space
indent_size = 2

# Tab indentation (no size specified)
[Makefile]
indent_style = tab

# Indentation override for all JS under lib directory
[lib/**.js]
indent_style = space
indent_size = 2

# Matches the exact files either package.json or .travis.yml
[{package.json,.travis.yml}]
indent_style = space
indent_size = 2
```



## :six: 部分配置文件完整列表

`package.json` 最终配置:

```json
{
  "name": "rush-share",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^5.16.4",
    "@testing-library/react": "^12.1.4",
    "@testing-library/user-event": "^13.5.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "react-scripts": "5.0.0",
    "web-vitals": "^2.1.4"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "devDependencies": {
    "@babel/core": "^7.17.9",
    "@babel/eslint-parser": "^7.17.0",
    "@babel/preset-react": "^7.16.7",
    "customize-cra": "^1.0.0",
    "eslint": "^8.12.0",
    "eslint-config-airbnb": "^19.0.4",
    "eslint-config-prettier": "^8.5.0",
    "eslint-plugin-import": "^2.26.0",
    "eslint-plugin-jest": "^26.1.3",
    "eslint-plugin-jsx-a11y": "^6.5.1",
    "eslint-plugin-prettier": "^4.0.0",
    "eslint-plugin-react": "^7.29.4",
    "eslint-plugin-react-hooks": "^4.4.0",
    "husky": "^7.0.0",
    "lint-staged": "^12.3.7",
    "postcss-scss": "^4.0.3",
    "prettier": "2.6.2",
    "sass": "^1.50.0",
    "stylelint": "^14.6.1",
    "stylelint-config-prettier-scss": "^0.0.1",
    "stylelint-config-standard-scss": "^3.0.0",
    "stylelint-prettier": "^2.0.0",
    "stylelint-scss": "^4.2.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "prepare": "husky install",
    "lint": "eslint --fix \"./src/**/*.{js,jsx,json}\"",
    "prettier:fix": "prettier --write",
    "prettier:check": "prettier --check .",
    "style": "stylelint \"src/**/*.(jsx|scss|css)\" --fix"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  }
}
```



`.vscode/settings.json` 配置：

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.fixAll.stylelint": true
  },
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "eslint.codeAction.showDocumentation": {
    "enable": true
  },
  "stylelint.validate": ["css", "postcss", "scss", "sass"],
  "stylelint.snippet": [
    "css",
    "scss",
    "postcss",
    "sass"
  ],
  "eslint.alwaysShowStatus": true
}
```



项目地址：

- [react18-template](https://github.com/jamessawyer/react18-template)

h5模板参考 `react-h5` 分支：

- [分支 react-h5](https://github.com/jamessawyer/react18-template/tree/react-h5)
- [h5-react-ts - lixiange@github](https://github.com/lixiange/h5-react-ts/blob/master/config-overrides.js) 更多配置可参考这个项目



相关链接：

1. [How to Add ESLint, Prettier, and Husky (Git Hooks) to Your React Application - Artem Diashkin@Medium](https://artem-diashkin.medium.com/react-js-adding-eslint-with-prettier-husky-git-hook-480ad39e65e9)
2. [Deku React](https://dev-yakuza.posstree.com/en/react/prettier/)
3. [How to use husky v6 with lint-staged? - @github issue](https://github.com/typicode/husky/issues/949)
