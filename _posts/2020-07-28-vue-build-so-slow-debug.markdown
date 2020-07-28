---
author: tangliangcheng
comments: true
date: 2020-07-28 17:08:00+00:00
layout: post
slug: 记一次解决Vue项目打包时间太慢问题
title: 记一次解决Vue项目打包时间太慢问题
excerpt: 最近项目打包变得很慢，在对比提交历史，排除开发环境等因素后
categories:
- 技术分享
---

# 记一次解决Vue项目打包时间太慢问题

最近项目打包变得很慢，在对比提交历史，排除开发环境等因素后

执行打包通过smp 日志可以看到，速度慢主要集中在耗时23mins左右的 `css-loader`, `mini-css-extract-plugin`, `postcss-loader`, `sass-loader` 这几个loader中。

通过对比 23mins, 17mins, 10mins 等，取公共部分，进一步缩小问题范围，可以大概确定打包速度慢主要是 `postcss-loader` 的问题。

```shell
 SMP  ⏱
General output time took 30 mins, 59.55 secs

 SMP  ⏱  Plugins
TerserPlugin took 2 mins, 41.67 secs
DllReferencePlugin took 2.2 secs
ProvidePlugin took 0.331 secs

 SMP  ⏱  Loaders
mini-css-extract-plugin, and
css-loader, and
postcss-loader, and
sass-loader took 23 mins, 36.78 secs
  module count = 1
css-loader, and
postcss-loader, and
sass-loader took 23 mins, 36.52 secs
  module count = 1
mini-css-extract-plugin, and
css-loader, and
postcss-loader took 17 mins, 31.02 secs
  module count = 14
css-loader, and
postcss-loader took 17 mins, 30.93 secs
  module count = 14
cache-loader, and
vue-loader took 10 mins, 27.67 secs
  module count = 2149
vue-loader, and
cache-loader, and
thread-loader, and
babel-loader, and
cache-loader, and
vue-loader took 9 mins, 14.56 secs
  module count = 1432
css-loader, and
vue-loader, and
postcss-loader took 5 mins, 29.27 secs
  module count = 4
cache-loader, and
thread-loader, and
babel-loader took 3 mins, 18.14 secs
```

检查项目 `postcss.config.js` 文件，发现主要是引入了 `tailwindcss`，因此做先做测试定位问题，注释掉引入`tailwindcss`的代码
```js
module.exports = {
	plugins: [
		require("tailwindcss"),
		require("autoprefixer")
	]
};
```

执行build，输出日志如下
```
SMP  ⏱
General output time took 47.83 secs

 SMP  ⏱  Plugins
TerserPlugin took 1.11 secs
DllReferencePlugin took 0.681 secs
ProvidePlugin took 0.064 secs

 SMP  ⏱  Loaders
mini-css-extract-plugin, and
css-loader, and
postcss-loader, and
sass-loader took 33.17 secs
  module count = 1
css-loader, and
postcss-loader, and
sass-loader took 33.034 secs
  module count = 1
```

可以看到，build时间骤降值**1分钟内**，所以现在就是 **如何解决tailwindcss打包问题?**

查阅资料后，最终定位到 `purgecss` 问题，官方文档也有这样一段描述:

`
When building for production, you should always use Tailwind's purge option to tree-shake unused styles and optimize your final build size. When removing unused styles with Tailwind, it's very hard to end up with more than 10kb of compressed CSS
`

所以最后需要引入 `purgecss` 库减少编译体积，减少执行时间

```
npm i -D @fullhuman/postcss-purgecss
```

update postcss.config.js
```js
const autoprefixer = require("autoprefixer");
const tailwindcss = require("tailwindcss");
const postcssPurgecss = require(`@fullhuman/postcss-purgecss`);

const purgecss = postcssPurgecss({
	// Specify the paths to all of the template files in your project.
	content: [`./public/**/*.html`, `./src/**/*.vue`],
	// Include any special characters you're using in this regular expression.
	// See: https://tailwindcss.com/docs/controlling-file-size/#understanding-the-regex
	defaultExtractor: content => content.match(/[\w-/:]+(?<!:)/g) || [],
	whitelistPatterns: [
		/-(leave|enter|appear)(|-(to|from|active))$/,
		/^(?!cursor-move).+-move$/,
		/^router-link(|-exact)-active$/
	]
});

module.exports = {
	plugins: [
		autoprefixer,
		tailwindcss,
		...process.env.NODE_ENV === `production`
      ? [purgecss]
      : [],
	]
};

```

执行build，输出日志如下，可以看到最终打包耗时 `3min` 左右，还有继续优化的空间，会在后续的文章中分享。

```shell
 SMP  ⏱
General output time took 3 mins, 15.86 secs

 SMP  ⏱  Plugins
TerserPlugin took 1.21 secs
DllReferencePlugin took 0.784 secs
ProvidePlugin took 0.058 secs

 SMP  ⏱  Loaders
vue-loader, and
mini-css-extract-plugin, and
css-loader, and
postcss-loader, and
sass-loader, and
cache-loader, and
vue-loader took 3 mins, 4.059 secs
  module count = 320
css-loader, and
vue-loader, and
postcss-loader, and
sass-loader, and
cache-loader, and
vue-loader took 3 mins, 4.029 secs
  module count = 160
vue-loader, and
mini-css-extract-plugin, and
css-loader, and
postcss-loader, and
cache-loader, and
vue-loader took 3 mins, 3.71 secs
  module count = 190
```

## 参考

[tailwindcss docs](https://tailwindcss.com/docs/controlling-file-size)
[purgecss](https://github.com/FullHuman/purgecss)
[Setting up Tailwind CSS with Vue.js](https://markus.oberlehner.net/blog/setting-up-tailwind-css-with-vue/)