# @babel/preset-env 问题

我的项目是 nuxt项目，但是npm i 一直无法成功，报错

```
 Cannot find module '@babel/preset-env/lib/utils'
```

找到@babel/preset-env,发现里面并没有utils文件，所以可能是preset-env版本的问题

在网上找到一些解决方案

## 方案

"@babel/preset-env": "^7.12.17"=> "@babel/preset-env": "~7.12.17"

![image-20240123143557587](https://pikachu-2022-1305579406.cos.ap-nanjing.myqcloud.com/markdown/image-20240123143557587.png)

并没有解决问题，我把node_modules删除，清除缓存，npm install 报出了另一个错误

```
Cannot find module '@babel/plugin-proposal-async-generator-functions'
```



找到preset-env这个文件，我在其package.json中找到了`'@babel/plugin-proposal-async-generator-functions'`这个依赖，但是我在node_modules文件夹下并没有找到下载的依赖，所以也就是npm install 没有把这些依赖下载下来

接下来我就 单独下载这个preset-env依赖

```
npm i --save-dev @babel/preset-env@7.12.17
```

下载完后，重新`npm run dev`,成功运行，先完成开发任务了，接下来看为什么会出现依赖,如何才能做到一步到位！