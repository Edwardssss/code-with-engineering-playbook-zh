# JavaScript / TypeScript

## 风格指南

开发者应当使用 [prettier](https://prettier.io/) 对 JavaScript 进行代码格式化。

使用 Prettier 这类自动化代码格式化工具，可以强制执行一套被广泛接受的风格指南，它由包括 Microsoft、Facebook 和 AirBnB 在内的众多公司共同构建。

对于 prettier 未覆盖的更高层风格指导，我们遵循 [AirBnB 风格指南](https://github.com/airbnb/javascript)。

## 代码分析 / Lint

### eslint

根据 [Palantir 2019 年的 TSLint 路线图](https://medium.com/palantir/tslint-in-2019-1a144c2317a9)中的指导，TypeScript 代码应当用 [ESLint](https://github.com/eslint/eslint) 来 lint。关于用 ESLint 检查 TypeScript 代码的更多信息，见 [typescript-eslint](https://typescript-eslint.io/) 文档。

要[安装并配置 ESLint 的 lint 能力](https://typescript-eslint.io/)，请把以下包作为 dev-dependency 安装：

```bash
npm install -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

在项目根目录添加 `.eslintrc.js`：

```javascript
module.exports = {
  root: true,
  parser: '@typescript-eslint/parser',
  plugins: [
    '@typescript-eslint',
  ],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/eslint-recommended',
    'plugin:@typescript-eslint/recommended',
  ],
};
```

在 `package.json` 的 `scripts` 中加入：

```json
"scripts": {
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx --ignore-path .gitignore"
}
```

这会 lint 项目中所有 `.js`、`.jsx`、`.ts`、`.tsx` 文件，并忽略 `.gitignore` 中指定的文件和目录。

你可以这样运行 lint：

```bash
npm run lint
```

## 配置 Prettier

[Prettier](https://prettier.io/docs/en/) 是一个有主见的代码格式化工具。

[入门指南](https://prettier.io/docs/en/integrating-with-linters.html)。

用 `npm` 作为 dev-dependency 安装：

```bash
npm install -D prettier eslint-config-prettier eslint-plugin-prettier
```

在 `.eslintrc.js` 中加入 `prettier`：

```javascript
module.exports = {
  root: true,
  parser: '@typescript-eslint/parser',
  plugins: [
    '@typescript-eslint',
  ],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/eslint-recommended',
    'plugin:@typescript-eslint/recommended',
    'prettier/@typescript-eslint',
    'plugin:prettier/recommended',
  ],
};
```

这样在用 ESLint 做 lint 时就会应用 `prettier` 规则集。

## 用 VS Code 自动格式化

VS Code 可以配置为在保存时自动执行 `eslint --fix`。

在项目根目录创建 `.vscode` 文件夹，并在 `.vscode/settings.json` 中加入：

```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
}
```

默认情况下，我们会在 VS Code 配置中加入以下覆盖项，以统一使用单引号、四空格缩进，并启用 ESLint：

```json
{
    "prettier.singleQuote": true,
    "prettier.eslintIntegration": true,
    "prettier.tabWidth": 4
}
```

## 配置测试

强烈建议在项目中配置 Playwright。它是微软创建的一个开源测试套件。

安装命令：

```bash
npm install playwright
```

由于 Playwright 会在浏览器中展示测试，你需要选择用哪个浏览器运行 —— 除非使用 Chrome，它是默认值。你可以通过……

> 译注：原文档在此处句子未完（原文亦如此）。

## 构建校验

要在 Azure DevOps 中自动化这一过程，可以把下面的片段加入你的流水线定义 yaml 文件。它会 lint `./scripts/` 文件夹下的所有脚本。

```yaml
- task: Npm@1
  displayName: 'Lint'
  inputs:
    command: 'custom'
    customCommand: 'run lint'
    workingDir: './scripts/'
```

## 提交前钩子

所有开发者都应当在提交前钩子中运行 `eslint`，以确保格式统一。我们强烈建议使用 [vscode-eslint](https://github.com/Microsoft/vscode-eslint) 这类编辑器集成，以获得实时反馈。

1. 在 `.git/hooks` 下把 `pre-commit.sample` 重命名为 `pre-commit`
2. 删除该文件中现有的示例代码
3. 关于这类脚本有很多现成的示例（gist），例如 [pre-commit-eslint](https://gist.github.com/linhmtran168/2286aeafe747e78f53bf)
4. 相应修改，把 TypeScript 文件也包含进去（加入 ts 扩展名，并确保配置好 typescript-eslint）
5. 让文件可执行：`chmod +x .git/hooks/pre-commit`

作为替代方案，可以考虑用 [husky](https://github.com/typicode/husky) 来简化提交前钩子。

## 代码评审检查清单

除了通用的代码评审检查清单，你还应当检查以下 JavaScript 与 TypeScript 特有的条目。

### JavaScript / TypeScript 检查清单

* [ ] 代码是否遵守我们的格式化与编码标准？对代码运行 prettier 和 ESLint 是否分别不产生警告和错误？
* [ ] 本次改动是否重新实现了某些代码，而引入生态中某个知名模块会是更好的做法？
* [ ] 是否使用 `"use strict";` 来减少未声明变量带来的错误？
* [ ] 在可能的地方（包括 API）是否都使用了单元测试？
* [ ] 测试是否按照 **Arrange/Act/Assert** 模式正确组织，并据此写了文档？
* [ ] 是否遵循了错误处理的最佳实践，以及 `try catch finally` 语句的使用？
* [ ] 异步调用是否恰当使用了 `doWork().then(doSomething).then(checkSomething)`，包括 `expect`、`done`？
* [ ] 主类中是否使用常量而不是裸字符串？或者，如果这些字符串跨文件/类使用，是否有专门的静态类存放常量？
* [ ] 魔法数字是否有解释？代码中不应出现没有任何注释说明其来由的数字。如果该数字反复出现，是否有对应的常量/枚举或等价物？
* [ ] 如果是异步方法，方法名是否以 `Async` 后缀结尾？
* [ ] 是否设置了最低限度的日志？使用的日志级别是否合理？
* [ ] 对 DOM 片段的操作是否仅限于需要操作多个子元素时？
* [ ] TypeScript 代码是否能编译通过、不产生 lint 错误？
* [ ] 各类类和方法中是否有恰当的 `/* */` 注释？
* [ ] 重量级操作是否实现在后端，让控制器尽可能保持轻量？
* [ ] HTML 上的事件处理是否高效？

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/recipes/javascript-and-typescript.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。原文检查清单中有两条逐字重复的条目，本译文保留各一条，未丢失信息。
{% endhint %}
