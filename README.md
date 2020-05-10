# 性能优化

## 网络层面
 1. 尽可能减少http请求。
    * 压缩资源，合并图片，开启Gzip
    * 资源预加载，图片懒加载，在页面刚加载的时候可以只加载第一屏，当用户继续往后滚屏的时候才加载后续的图片。
 2. 利用好浏览器最大的并发请求数，可以利用cdn加大并发数
 3. 合理的利用缓存()
 4. 使用 HTTP / 2.0
    
### 懒加载
*  懒加载就是将不关键的资源延后加载。
  懒加载的原理就是只加载自定义区域（通常是可视区域，但也可以是即将进入可视区域）内需要加载的东西。对于图片来说，先设置图片标签的 src 属性为一张占位图，将真实的图片资源放入一个自定义属性中，当进入自定义区域时，就将自定义属性替换为 src 属性，这样图片就会去下载资源，实现了图片懒加载。
  懒加载不仅可以用于图片，也可以使用在别的资源上。比如进入可视区域才开始播放视频等等。

### 预加载
  * 在开发中，可能会遇到这样的情况。有些资源不需要马上用到，但是希望尽早获取，这时候就可以使用预加载。
  预加载其实是声明式的 fetch ，强制浏览器请求资源，并且不会阻塞 onload 事件，可以使用以下代码开启预加载
  ```html
<link rel="preload" href="http://example.com">
```
  预加载可以一定程度上降低首屏的加载时间，因为可以将一些不影响首屏但重要的文件延后加载，唯一缺点就是兼容性不好。

## CDN
静态资源尽量使用 CDN 加载，由于浏览器对于单个域名有并发请求上限，可以考虑使用多个 CDN 域名。对于 CDN 加载静态资源需要注意 CDN 域名要与主站不同，否则每次请求都会带上主站的 Cookie。

### 选择合适的缓存策略
对于大部分的场景都可以使用强缓存配合协商缓存解决，但是在一些特殊的地方可能需要选择特殊的缓存策略

对于某些不需要缓存的资源，可以使用 Cache-control: no-store ，表示该资源不需要缓存
对于频繁变动的资源，可以使用 Cache-Control: no-cache 并配合 ETag 使用，表示该资源已被缓存，但是每次都会发送请求询问资源是否更新。
对于代码文件来说，通常使用 Cache-Control: max-age=31536000 并配合策略缓存使用，然后对文件进行指纹处理，一旦文件名变动就会立刻下载新的文件。

## 使用 Webpack 优化项目
* 对于 Webpack4，打包项目使用 production 模式，这样会自动开启代码压缩
* 使用 ES6 模块来开启 tree shaking，这个技术可以移除没有使用的代码
* 优化图片，对于小图可以使用 base64 的方式写入文件中
* 按照路由拆分代码，实现按需加载
* 给打包出来的文件名添加哈希，实现浏览器缓存文件

## 代码层面来优化
  a. 对于长列表使用虚拟列表（react-virtualized），或分片渲染（requestAnimationFrame ）；
  b. Javascript中的数据访问包括直接量 (字符串、正则表达式 )、变量、对象属性以及数组，其中对直接量和局部变量的访问是最快的，对对象属性以及数组的访问需要更大的开销。当出现以下情况时，建议将数据放入局部变量： 
　　1. 对任何对象属性的访问超过 1次 
　　2. 对任何数组成员的访问次数超过 1次 
　　3. 另外，还应当尽可能的减少对对象以及数组深度查找。
  c. DOM 和 CSSOM 通常是并行构建的，所以 CSS 加载不会阻塞 DOM 的解析。然而，由于 Render Tree 是依赖于 DOM Tree 和 CSSOM Tree 的，
所以他必须等待到 CSSOM Tree 构建完成，也就是 CSS 资源加载完成(或者 CSS 资源加载失败)后，才能开始渲染。因此，CSS 加载会阻塞 Dom 的渲染。
由此可见，对于 CSSOM 缩小、压缩以及缓存同样重要，我们可以从这方面考虑去优化。
d. 减少回流和重绘
e. 事件委托 ,优点：1. 大量减少内存占用，减少事件注册。2. 新增元素实现动态绑定事件
f. 防抖和节流，防抖是延迟执行（搜索框掉接口搜索数据），节流是一段时间内只执行一次（点击按钮请求接口，等接口返回才可以再点击）

# es6
1. 对象里的函数可以简写，即不加function，但是简写的函数不能当做构造函数使用。
2.  ES2020 引入了“链判断运算符”（optional chaining operator）?.
```javascript
const firstName = message?.body?.user?.firstName || 'default';
```
3.读取对象属性的时候，如果某个属性的值是null或undefined，有时候需要为它们指定默认值。常见做法是通过||运算符指定默认值。

```javascript
const headerText = response.settings.headerText || 'Hello, world!';
const animationDuration = response.settings.animationDuration || 300;
const showSplashScreen = response.settings.showSplashScreen || true;
```
上面的三行代码都通过||运算符指定默认值，但是这样写是错的。开发者的原意是，只要属性的值为null或undefined，默认值就会生效，
但是属性的值如果为空字符串或false或0，默认值也会生效。
为了避免这种情况，ES2020 引入了一个新的 Null 判断运算符??。它的行为类似||，但是只有运算符左侧的值为null或undefined时，才会返回右侧的值。
```javascript
const headerText = response.settings.headerText ?? 'Hello, world!';
const animationDuration = response.settings.animationDuration ?? 300;
const showSplashScreen = response.settings.showSplashScreen ?? true;
```

## Set
1.ES6 提供了新的数据结构 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。
```javascript
// 去除数组的重复成员
[...new Set(array)]
```
向 Set 实例添加了两次NaN，但是只会加入一个。这表明，在 Set 内部，两个NaN是相等的。
需要特别指出的是，Set的遍历顺序就是插入顺序。这个特性有时非常有用，比如使用 Set 保存一个回调函数列表，调用时就能保证按照添加顺序调用。

## Map
ES6 提供了 Map 数据结构。它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说，Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。如果你需要“键值对”的数据结构，Map 比 Object 更合适。
需要特别注意的是，Map 的遍历顺序就是插入顺序。


