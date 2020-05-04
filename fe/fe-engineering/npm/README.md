# 什么是前端工程化？

前端工程化要解决的问题：前端工程师的开发效率

前端工程化是根据前端开发业务的特点，对前端的技术选型、开发流程、代码规范、构建发布等等，进行流程规范化。以此来提高前端工程师的开发效率和代码质量。


# 为什么会出现如此众多的前端技术？
早期前端开发很简单，写写脚本切切图。但是随着前端（浏览器）的能力不断增强，业务方给的需求也是难度不断增加。

前端开发人员在使用`HTML`、CSS、JavaScript 这原生三剑客进行开发的时候发现，效率很低。于是出现了各种模板引擎，less、sass、stylus，typescript等中间语言，通过转译到原生语言来提升开发效率。于是也就有了构建工具。

再比如代码修改之后，我们要看到最新的效果需要刷新浏览器，就有了`webpack-dev-middleware` 结合 webpack watch mode 在内存打包，然后通知浏览器自动刷新的功能。还不够，我们又发现刷新之前的state会丢失，又有了能不能不丢失state进行局部刷新？HMR就又来了。等等等。


# npm
```json
{
  "dependencies": {
    "A": "1.0.0",
    "B": "1.0.0",
    "C": "1.0.0",
  }
}
```
假设 `AC` 都依赖了相同的一个仓库 `D@1.0.0`, `B` 依赖了 `D@2.0.0`

## node_modules在npm2.x的行为
`npm2.x` 的做法很简单，就是下载对应的仓库，然后递归安装仓库的依赖即可。

这样做的优点：
+ 结构清晰

```sh
├── node_modules
│   ├── A@1.0.0
│   │   └── node_modules
│   │   │   └── D@1.0.0
│   ├── B@1.0.0
│   │   └── node_modules
│   │   │   └── D@2.0.0
│   └── C@1.0.0
│      └── node_modules
│           └── D@1.0.0

```
缺点：
+ 有可能会下载同一个包多次，造成冗余
+ 目录层级嵌套过深

## npm3.x的行为
`npm3.x` 采用扁平化的方式来组织 `node_modules` : 

在 `install` 的过程中，根据 `package.json` 的依赖字段，依次安装依赖。如果发现依赖的依赖是未安装过的包，则会将其安装到第一级目录下。<br>
后续在安装其它依赖的过程中，如果遇到一级目录下已存在的包并且符合版本约束，则不会重新下载。而如果是相同的包，但是版本不同，则和 `npm2.x` 的行为一直，安装到依赖包的 `node_modules` 下。<br>

```sh
├── node_modules
│   ├── A@1.0.0
│   ├── B@1.0.0
│   │   └── node_modules
│   │   │   └── D@2.0.0
│   ├── C@1.0.0
│   └── D@1.0.0
```
这还不算完。如果在 `npm install` 之后又下载一个包 `E`(依赖`D@2.0.0`)，这个时候， `npm` 会发现一级目录下不存在，会把E的依赖安装到 `E` 的`node_modules`下。也就是这样：
```sh
├── node_modules
│   ├── A@1.0.0
│   ├── B@1.0.0
│   │   └── node_modules
│   │   │   └── D@2.0.0
│   ├── C@1.0.0
│   ├── D@1.0.0
│   └── E@1.0.0
│       └── node_modules
│           └── D@2.0.0
```
依然没完，如果这个时候再安装一个 `F`(依赖`D@1.0.0`), `npm` 会发现一级目录下已经存在，所以就只会安装F在一级目录中不存在的其他依赖，不会再次安装`D@1.0.0`了。

```sh
├── node_modules
│   ├── A@1.0.0
│   ├── B@1.0.0
│   │   └── node_modules
│   │   │   └── D@2.0.0
│   ├── C@1.0.0
│   ├── D@1.0.0
│   ├── E@1.0.0
│   │     └── node_modules
│   │       └── D@2.0.0
│   └── F@1.0.0
```

`3.x` 在 `2.x` 的基础上进行了一些优化，减少了一部分的包冗余问题，但是依然会存在下载重复依赖的问题（比如 `BF` 都依赖了 `D@2.0.0`，但是缺下载了两遍）。

> 当 `AC` 依赖的 `D` **升级成2.0.0的前提下**，如果不想重新下载，可以执行 `npm dedupe`， 把 `D@2.0.0` 重定向到一级目录下。

## npm5.x 新增 package-lock.json
`npm5.x` 对 `node_modules` 的组织方式和 `npm3.x` 相同，都是采用扁平化的方案，最大的不同点在于 `5.x` 新出了一个 `package-lock.json` 文件。

> `npm` 为了让开发者在安全的前提下使用最新的依赖包，在 `package.json` 中通常做了锁定大版本的操作，这样在每次 `npm install` 的时候都会拉取依赖包大版本下的最新的版本。这种机制最大的一个缺点就是当有依赖包有小版本更新时，可能会出现协同开发者的依赖包不一致的问题。

`package-lock.json` 文件精确描述了 `node_modules` 目录下所有的包的树状依赖结构，每个包的版本号都是完全精确的。
```json
{
  "version": "4.29.6",
  "resolved": "xxx",
  "integrity": "sha1-Zr8OyL7uTUafi1mNOYj/nY2Q6VU=",
  "dev": true,
  "requires": {

  }
}
```
+ resolved：安装源
+ integrity：验证包是否有效的hash
+ dev：是否是顶级依赖（也就是说是不是package.json里的依赖字段里的依赖）
+ requires
+ dependencies

换句话说，`package-lock.json` 和 `node_modules` 的目录结构是一一对应的，有了 `package-lock.json` 就能够保证每次 `npm instal` 生成的 `node_modules` 是一致的。
> 所以在开发一个应用的时候，我们应该是把 `package-lock.json` 放到仓库中的。但是在开发一个第三方库的时候则不应该（npm在publish的时候也默认不会发布），因为库一般是被其他项目所依赖的，不写死有利于复用一些已经加载过的包，而如果库依赖的版本号是精确的，那就会大大增加包的冗余性。


## npm script
首先科普两个知识点： 
+ 软链接：又称符号链接，一种特殊的可执行文件。其包含有一条以*绝对路径*或者*相对路径*的形式指向其它文件或者目录的引用。在对链接文件进行读或写操作的时候，系统会自动把该操作转换为对源文件的操作，但删除链接文件时，系统仅仅删除链接文件，而不删除源文件本身。
+ terminal 执行指令的时候，会跟环境变量 `$PATH` 中的路径，以此查找到和指令同名的可执行文件，然后执行。


