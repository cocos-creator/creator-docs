# 缓存管理器

> 文：Santy-Wang

对于 Web 平台，资源在下载之后由浏览器来管理缓存，不需要由引擎进行管理。但在某些非 Web 平台，比如微信小游戏，这类平台具备文件系统，但没有实现资源的缓存机制，需要由引擎实现一套缓存机制用于管理从网络上下载下来的资源，包括缓存资源，清理缓存资源，查询缓存资源等功能。

在 v2.4 Creator 在所有非 Web 平台上实现了缓存管理机制，你可以通过 `cc.assetManager.cacheManager` 进行访问。

## 资源下载流程

当需要下载资源时，会进行一系列检查步骤如下：

1. 判断该资源是否在游戏包内，如果在则直接使用

2. 查询该资源是否在缓存中，如果在则直接使用

3. 查询该资源是否在临时目录中，如果在则直接使用（原生平台将跳过此步）

4. 下载远程资源到临时目录，直接使用临时目录的资源（原生平台将直接下载资源到缓存目录）

5. 后台缓慢将临时目录中的资源缓存到缓存目录中（原生平台将跳过此步）

6. 当缓存空间占满后，此时会使用 LRU 算法删除比较久远的资源（原生平台将跳过此步，你可以手动调用清理）

## 查询缓存文件

缓存管理器提供了 `getCache` 接口以查询所有缓存的资源，你可以通过传入资源原路径查询该资源的缓存路径，例如：

```js
    cc.resources.load('images/background', cc.Texture2D, function (err, texture) {
        var cachePath = cc.assetManager.cacheManager.getCache(texture.nativeUrl);
        console.log(cachePath);
    });
```

## 查询临时文件

当资源被下载到本地之后，可能会以临时文件的形式存储到临时目录。缓存管理器提供了 `tempFiles` 以查询所有下载到临时目录的资源，你可以通过传入资源原路径进行查询，例如：

```js
    cc.assetManager.loadRemote('http://example.com/background.jpg', function (err, texture) {
        var tempPath = cc.assetManager.cacheManager.getTemp(texture.nativeUrl);
        console.log(tempPath);
    });
```

## 缓存资源

缓存管理器中提供了一些控制参数用于控制资源的缓存：

1. `cacheManager.cacheDir` 控制了缓存资源的存储目录。
2. `cacheManager.cacheInterval` 控制缓存单个资源的周期，默认为 500 ms 缓存一次。
3. `cacheManager.cacheEnabled` 控制是否缓存资源，默认为缓存资源。

另外，你可以通过指定可选参数 `cacheEnabled` 来覆盖全局设置，例如：

```js
    cc.assetManager.loadRemote('http://example.com/background.jpg', { cacheEnabled: true }, callback);
```

## 清理缓存

缓存管理器提供了三个接口清理缓存资源，分别是 `removeCache`, `clearCache` , `clearLRU` 接口，`removeCache` 用于清理单个缓存资源，`clearCache` 用于清理所有缓存资源，请慎重使用，`clearLRU` 用于清理较早使用的资源，`clearLRU` 会在小游戏平台存储空间满了后自动调用。

`removeCache` 需要提供资源的原路径进行清除，例如：

```js
cc.assetManager.loadRemote('http://example.com/background.jpg', function (err, texture) {
    cc.assetManager.cacheManager.removeCache(texture.nativeUrl);
});
```
