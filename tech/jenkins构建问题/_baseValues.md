# jenkins 构建错误 _baseValues

## 错误信息

```bash
Error: Cannot find module './_baseValues'
Require stack:
- /data/jenkins/workspace/smart-bi-miniprogram-mp/node_modules/lodash/values.js
- /data/jenkins/workspace/smart-bi-miniprogram-mp/node_modules/webpack-merge/lib/index.js
- /data/jenkins/workspace/smart-bi-miniprogram-mp/node_modules/@vue/cli-service/lib/Service.js
- /data/jenkins/workspace/smart-bi-miniprogram-mp/node_modules/@vue/cli-service/bin/vue-cli-service.js
    at Function.Module._resolveFilename (node:internal/modules/cjs/loader:956:15)
    at Function.Module._load (node:internal/modules/cjs/loader:804:27)
    at Module.require (node:internal/modules/cjs/loader:1028:19)
    at require (node:internal/modules/cjs/helpers:102:18)
    at Object.<anonymous> (/data/jenkins/workspace/smart-bi-miniprogram-mp/node_modules/lodash/values.js:1:18)
    at Module._compile (node:internal/modules/cjs/loader:1126:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1180:10)
    at Module.load (node:internal/modules/cjs/loader:1004:32)
    at Function.Module._load (node:internal/modules/cjs/loader:839:12)
    at Module.require (node:internal/modules/cjs/loader:1028:19) {
  code: 'MODULE_NOT_FOUND',
  requireStack: [
    '/data/jenkins/workspace/smart-bi-miniprogram-mp/node_modules/lodash/values.js',
    '/data/jenkins/workspace/smart-bi-miniprogram-mp/node_modules/webpack-merge/lib/index.js',
    '/data/jenkins/workspace/smart-bi-miniprogram-mp/node_modules/@vue/cli-service/lib/Service.js',
    '/data/jenkins/workspace/smart-bi-miniprogram-mp/node_modules/@vue/cli-service/bin/vue-cli-service.js'
  ]
}
```

## 解决方案

### 1. 问题分析

该错误 `Error: Cannot find module './_baseValues'` 发生于 `lodash/values.js` 引用内部文件时，这通常意味着 `node_modules` 中的 `lodash` 包文件不完整或损坏。

**主要原因**：
*   **缓存污染**：Jenkins 构建环境的 Yarn 缓存中可能存在损坏的 `lodash` 压缩包。
*   **错误的构建命令**：脚本中出现的 `/data/nodejs/node-v16.17.0/bin/yarn add lodash._basevalues` 是无效命令（`lodash._basevalues` 并非标准的公共 npm 包），这可能是为了解决报错而错误添加的，反而可能导致依赖树混乱。

### 2. 修复步骤

在 Jenkins 构建配置中，采用“**清理缓存 + 重新安装**”的策略。

建议将构建脚本修改为以下完整流程：

```bash
# 1. 确保使用私有源
/data/nodejs/node-v16.17.0/bin/yarn config set registry http://npm.pt.hydee.cn/

# 2. 清理环境（关键步骤）
# 删除 node_modules 和 yarn.lock 以强制重新解析依赖
rm -rf node_modules yarn.lock
# 清理 yarn 缓存，防止使用损坏的本地缓存包
/data/nodejs/node-v16.17.0/bin/yarn cache clean

# 3. 安装依赖
# 注意：直接运行 yarn 或 yarn install 即可，移除多余的 add 命令
/data/nodejs/node-v16.17.0/bin/yarn install

# 4. 执行构建
/data/nodejs/node-v16.17.0/bin/yarn build:mp-weixin-${env}-up
```
