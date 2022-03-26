这篇文章总结一下 vite 开发模式，内部文件处理的流程。

vite 在开发模式会启动一个 http 的 server。利用 server 的中间件对于文件进行相关处理。

optimizedDepsMedadata {
  hash: '04d91b62',
  browserHash: 'f63205ee',
  optimized: {
    vue: {
      file: '/Users/xxx/code/xxxxx/node_modules/.vite/vue.js',
      src: '/Users/xxx/code/xxxxx/node_modules/vue/dist/vue.runtime.esm-bundler.js',
      needsInterop: false
    },
    xxxx
  }


其中 hash 为 package-lock.json, yarn.lock, pnpm-lock.yaml 以及 vitel.config.js 的其中部分配置的 hash 值
browserHash 是代码的依赖项(比如引用了 vue、axios 等)，加上 hash 中内容的 hash 值


# preAliasPlugin

第一个插件，其中有 resolveId 钩子

- 当遇到 node_modules 引用时,如果有缓存，直接将链接返回

```
vue 

/Users/xxx/code/xxx/node_modules/.vite/vue.js?v=f63205ee


```

# importAnalysis

分析引用，在这里修改引用地址

## 业务应用修改

@/const/api => /src/const/api

## 第三方包

this.resolve

这里 this.resolve 这里是如何修改 node_modules 继续深入
vue => node_modules/.vite/vue



