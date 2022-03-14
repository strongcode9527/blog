这篇文章对 esbuild 在 vite 中的应用做一个总结

# vite 中 esbuild 的应用

esbuild 是 vite 性能快的关键。esbuild 在 vite 中主要被使用在以下场景:

1. 通过入口分析依赖树，收集 node_modules 中的依赖，提前打包处理，并在磁盘持久化缓存处理过后的文件。
2. 对部分业务文件 使用 esbuild 处理转化，比如 ts、jsx、tsx。注意 js 类型文件并不会被处理

## node_modules 处理

需要处理 node_modules 的原因：
1. 第三方库代码不会经常变动，缓存处理，提高响应速度。
2. 打包第三方库，使 lodash 这种文件较多的库，减少网络请求。
3. 代码兼容处理：比如有的依赖代码模块格式并不是 esm 而是 commonjs，利用 esbuild 进行模块格式转换。


### 难点一：vite 如何从 index.html 入口分析依赖的组成？他怎么知道需要处理哪些包？

1. esbuild 天生支持 index.html 作为入口打包

```javascript

require('esbuild').buildSync({
  entryPoints: ['index.html'], // is ok
  bundle: true,
  write: true,
  outdir: 'out',
})


if (resolved.includes('node_modules') || include?.includes(id)) {
    if (OPTIMIZABLE_ENTRY_RE.test(resolved)) {
        // esbuild 插件中直接判断引入的包是否是 node_modules 中的依赖
        depImports[id] = resolved
    }
}


```

### 难点二：esbuild 如何处理 vue 等非 javascript 文件，并无官方 vue-loader

```javascript
const regex = isHtml ? scriptModuleRE : scriptRE

const scriptModuleRE =
    /(<script\b[^>]*type\s*=\s*(?:"module"|'module')[^>]*>)(.*?)<\/script>/gims
export const scriptRE = /(<script\b(?:\s[^>]*>|>))(.*?)<\/script>/gims


```
vite 内部通过编写一个简单的 esbuild 插件，直接利用正则表达式将 script 标签内部内容截取出来，作为 vue 文件的内容。这样 esbuild 就可以将 vue 文件当成 js 文件处理依赖树了。

## 编译业务文件

当用户第一次请求业务文件的时候，浏览器以及 vite 应用内部均没有缓存，这个时候就要倚靠 esbuild 对部分业务文件进行编译。

esbuild 默认会编译 ts、tsx、jsx 文件。

**其中 js 文件默认不会被编译。所以当选择一些新语法开发时，要慎重**

### 语法兼容问题

#### 浏览器兼容

就像刚才说的问题，普通的 js 文件以及 vue 文件中的 js script 内容，是不会被 esbuild 处理编译的，所以不稳定的新语法是不能在 vite 环境下使用的。

#### esbuild 兼容

esbuild 作为新起的打包工具，他对部分语法是不支持的，比如 js 文件中使用装饰器。这个场景 esbuild 就是不兼容的。

所以语法兼容问题要考虑两个问题：

1. 你开发时用的浏览器最好是比较新的 chrome 或者火狐浏览器，因为 vite 并不会默认编译 js 文件，如果有兼容问题，处理起来比较复杂。
2. 代码兼容也要考虑 esbuild 的语法支持程度。如果有 esbuild 不支持的语法，在 vite 提前打包 node_modules 文件时会直接报错。（vite 会在启动服务的时候，默认从入口查找所有需要打包的 node_modules 文件，如果你的业务代码有兼容问题，会导致 esbuild 分析文件直接报错。）


