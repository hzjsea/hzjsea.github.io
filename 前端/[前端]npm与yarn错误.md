---
title: 前端_npm与yarn错误
date: 2021-05-10 16:15:10
categories:  [vue]
tags: [vue]
---


<!--more-->


# npm与yarn错误

记录一次npm与yarn的错误，之前尝试了用yarn做vue脚手架工具的主要下载项，于是我修改了根目录下面的`.vuerc`
```bash
{
  "useTaobaoRegistry": true,
  "latestVersion": "4.5.13",
  "lastChecked": 1620633351418,
  "packageManager": "yarn"
}
```

于是导致我每次`vue create demo`创建项目的时候，进入文件并运行时会报错，
```bash
cd demo  && npm run serve or yarn run serve
```
怎么重装都没用，这是因为yarn搞的鬼

<div style='font-size:20px;color:red'>解决办法是换回npm</div>

1. 卸载yarn `npm unintall -g yarn`
2. 修改.vuerc 
```bash
{
  "useTaobaoRegistry": true,
  "latestVersion": "4.5.13",
  "lastChecked": 1620633351418,
  "packageManager": "npm"
}
```
3. 删掉demo `rm -r demo`
4. 重新创建 `vue create demo && cd demo && npm run serve`


<div style='font-size:20px;color:red'>或者在创建项目的时候，指定下载工具</div>

```bash
vue create demo -m npm
```