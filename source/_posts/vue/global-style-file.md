
---
title: vue-cli 全局less,sass文件的注入方法
categories: 工程化
tags: [全局样式, vue-cli]
date: 2021-6-22
---


全局的mixins和variable非常有用，但是在每个文件手动引入就很费劲，需要一个一劳永逸的办法直接在编译时插入文件开头。

[style-resources-loader](https://github.com/yenshih/style-resources-loader)可以解决这个问题。

```
// webpack-chain语法
const types = ['vue-modules', 'vue', 'normal-modules', 'normal']

// 业务代码注入全局变量
types.forEach(type => {
  config.module.rule('less').oneOf(type)
  .use('style-resource')
  .loader('style-resources-loader')
  .options({
      patterns: [
          path.resolve(__dirname, `./src/style/variables.less`),
          path.resolve(__dirname, './src/style/mixins.less')
      ]
  })
})

```

'vue-modules', 'vue', 'normal-modules', 'normal'这几种规则类型都是vue/cli-service定义的：

```
function createCSSRule (lang, test, loader, options) {
    const baseRule = webpackConfig.module.rule(lang).test(test)
    
    // rules for <style lang="module">
    const vueModulesRule = baseRule.oneOf('vue-modules').resourceQuery(/module/)
    applyLoaders(vueModulesRule, true)
    
    // rules for <style>
    const vueNormalRule = baseRule.oneOf('vue').resourceQuery(/\?vue/)
    applyLoaders(vueNormalRule, false)
    
    // rules for *.module.* files
    const extModulesRule = baseRule.oneOf('normal-modules').test(/\.module\.\w+$/)
    applyLoaders(extModulesRule, true)
    
    // rules for normal CSS imports
    const normalRule = baseRule.oneOf('normal')
    applyLoaders(normalRule, modules)
    
    // ...
}
```
vue-cli为`css`,`postcss`,`scss`,`sass`,`less`,`stylus`这几种语言都定义了'vue-modules', 'vue', 'normal-modules', 'normal'这4种子规则。

我们最常用的是`vue`，表示.vue文件中的style标签。

## 参考
[自动化导入](https://cli.vuejs.org/zh/guide/css.html#%E8%87%AA%E5%8A%A8%E5%8C%96%E5%AF%BC%E5%85%A5)