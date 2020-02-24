# 常见问题（FAQ）

### 在 Mac 平台安装 Creator 后，打开工程，窗口无法显示出来。

把 CocosCreator.app 从 dmg 中拖到应用程序里，再打开。

### 如何配置构建后生成的目录结构以及自动修改构建后的文件内容？

可以分别执行 `Editor.Builder.on(“build-finished”, callback)` 和 `Editor.Builder.on(“before-change-files”, callback)` 来监听并对文件做出修改。<br>
详见 [定制项目构建流程](../publish/custom-project-build-template.md)。

### 构建后，组件关联的对象找不到，或者警告找不到类型？

指定模块可能被剔除了，请检查一下 `菜单栏 -> 项目 -> 项目设置` 中的模块设置。

### 如何从 CDN 加载资源？

在 `main.js` 里修改 `libraryPath` 和 `rawAssetsBase` 参数为任意路径。

### 浏览器里如何截屏？

截屏方法：监听 `cc.Director.EVENT_AFTER_DRAW` 事件，在回调中调用 `cc._canvas.toDataURL()` ，就可以获得 base64 格式的截屏。

具体内容可参考 [官方范例](https://github.com/cocos-creator/example-cases/tree/master/assets/cases/07_capture_texture) 中 **07_capture_texture** 关于三种不同平台如何截屏并保存的测试例。

### 在安卓第三方 App 自带的 WebView 上触摸时，引擎报错。

找到 `main.js` 中调用 `cc.view.enableAutoFullScreen` 的地方，将传入参数改为 `false`。

### 帧动画每帧的图片大小不一样的话，播放动画时会发生偏移。

请参考：[自带位置信息的序列帧动画](../asset-workflow/trim.md#%E8%87%AA%E5%B8%A6%E4%BD%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%E7%9A%84%E5%BA%8F%E5%88%97%E5%B8%A7%E5%8A%A8%E7%94%BB)。

### 图片放大后，边缘白边/黑边问题。

这是由于线性过滤时，颜色通道和背景色做了插值，比较简单的办法是使用预乘。预乘需要将 Sprite 组件的混合模式设置成 `SRC: ONE, DST: ONE_MINUS_SRC_ALPHA`，然后对原图做处理。

原图有两种处理方式，一种在图片导出时，在颜色通道下面添加黑色背景，透明通道保持不变。另一种是在代码中对 texture 调用 `texture.update({premultiplyAlpha: true})`，如果调用失败，有可能是旧版 Creator，旧版需要改用 `texture.handleLoadedTexture(true)`。

### 如何用插件控制 Prefab？

**创建 Prefab**：

```js
Editor.Ipc.sendToPanel('scene', 'scene:create-prefab', node.uuid, 'db://assets/xxx/xxx.prefab');
```

**进入 Prefab 编辑模式**：

```js
Editor.Ipc.sendToAll('scene:enter-prefab-edit-mode', assetUuid);
```

**保存 Prefab**：

```js
Editor.Ipc.sendToPanel('scene', 'scene:apply-prefab', node.uuid);
```

### 1.8.2 的版本可能出现 JSC 崩溃问题。

请升级到 1.8.2 之后的版本。

### 微信开放数据加载头像时提示 wx.request 找不到。

加载图片时 url 若缺失 .png 之类的后缀，`cc.loader.load` 需要改成传入 `{ url: url, type: "png" }`。

具体内容可参考 [官方范例](https://github.com/cocos-creator/example-cases/tree/master/assets/cases/dragonbones) 中的 **dragonBones/DragonMesh** 测试例。

### 如何从服务器远程加载 DragonBones ？

#### 加载文本格式的 DragonBones 资源

```js
var animNode = new cc.Node();
animNode.parent = cc.find('Canvas');
var dragonDisplay = animNode.addComponent(dragonBones.ArmatureDisplay);

var image = 'http://localhost:7456/res/raw-assets/eee_tex-1529064342.png';
var ske = 'http://localhost:7456/res/raw-assets/eee_ske-1529065642.json';
var atlas = 'http://localhost:7456/res/raw-assets/eee_tex-1529065642.json';
cc.loader.load(image, (error, texture) => {
    cc.loader.load({ url: atlas, type: 'txt' }, (error, atlasJson) => {
        cc.loader.load({ url: ske, type: 'txt' }, (error, dragonBonesJson) => {
            var atlas = new dragonBones.DragonBonesAtlasAsset();
            atlas.atlasJson = atlasJson;
            atlas.texture = texture;

            var asset = new dragonBones.DragonBonesAsset();
            asset.dragonBonesJson = dragonBonesJson;

            dragonDisplay.dragonAtlasAsset = atlas;
            dragonDisplay.dragonAsset = asset;

            dragonDisplay.armatureName = 'box_anim';
            dragonDisplay.playAnimation('box_anim', 0);
        });
    });
});
```

#### 加载二进制格式的 DragonBones 资源

```js
var animNode = new cc.Node();
animNode.parent = cc.find('Canvas');
var dragonDisplay = animNode.addComponent(dragonBones.ArmatureDisplay);

var image = 'http://localhost:7456/res/raw-assets/eee_tex-1529064342.png';
var ske = 'http://localhost:7456/res/raw-assets/eee_ske-1529065642.dbbin';
var atlas = 'http://localhost:7456/res/raw-assets/eee_tex-1529065642.json';
cc.loader.load(image, (error, texture) => {
    cc.loader.load({ url: atlas, type: 'txt' }, (error, atlasJson) => {
        cc.loader.load({ url: ske, type: 'bin' }, (error, dragonBonesBin) => {
            var atlas = new dragonBones.DragonBonesAtlasAsset();
            atlas.atlasJson = atlasJson;
            atlas.texture = texture;

            var asset = new dragonBones.DragonBonesAsset();
            asset._nativeAsset = dragonBonesBin;

            dragonDisplay.dragonAtlasAsset = atlas;
            dragonDisplay.dragonAsset = asset;

            dragonDisplay.armatureName = 'box_anim';
            dragonDisplay.playAnimation('box_anim', 0);
        });
    });
});
```

### 如何从服务器远程加载 Spine

#### 加载文本格式的 Spine 资源

```js
var spineNode = new cc.Node();
var skeleton = spineNode.addComponent(sp.Skeleton);
this.node.addChild(spineNode);

var image = "http://localhost/download/spineres/1/1.png";
var ske = "http://localhost/download/spineres/1/1.json";
var atlas = "http://localhost/download/spineres/1/1.atlas";
cc.loader.load(image, (error, texture) => {
    cc.loader.load({ url: atlas, type: 'txt' }, (error, atlasJson) => {
        cc.loader.load({ url: ske, type: 'txt' }, (error, spineJson) => {
            var asset = new sp.SkeletonData();
            asset.skeletonJson = spineJson;
            asset.atlasText = atlasJson;
            asset.textures = [texture];
            asset.textureNames = ['1.png'];
            skeleton.skeletonData = asset;
        });
    });
});
```

#### 加载二进制格式的 Spine 资源

```js
var spineNode = new cc.Node();
var skeleton = spineNode.addComponent(sp.Skeleton);
this.node.addChild(spineNode);

var image = "http://localhost/download/spineres/1/1.png";
var ske = "http://localhost/download/spineres/1/1.skel";
var atlas = "http://localhost/download/spineres/1/1.atlas";
cc.loader.load(image, (error, texture) => {
    cc.loader.load({ url: atlas, type: 'txt' }, (error, atlasJson) => {
        cc.loader.load({ url: ske, type: 'bin' }, (error, spineBin) => {
            var asset = new sp.SkeletonData();
            asset._nativeAsset = spineBin;
            asset.atlasText = atlasJson;
            asset.textures = [texture];
            asset.textureNames = ['1.png'];
            skeleton.skeletonData = asset;
        });
    });
});
```

### 如何自定义或者直接禁用编辑器自带的 uglify？

自定义引擎完成后，打开 `engine/gulp/util/utils.js` 脚本，在最下面有一个 `uglify` 函数，可以根据需求自行修改其中的参数。如果想要完全跳过 `uglify` 操作，可以直接将 `uglify` 部分中的内容替换成：

```js
const Es = require('event-stream');
return Es.through();
```

### 如何在插件中创建 AssetDB 资源

在主进程中，使用 `Editor.assetdb.create(url, data, callback)`。第一个参数是 `db://assets/xxx.png`，第二个参数是 .png 文件读取出来的 buffer。

### 如何在插件中重新刷新 AssetDB 中的资源

`Editor.assetdb.refresh()` 提供了一个手动刷新资源库的方法。

### Creator 打包 APK 提交到 Google Play 失败，提示 API 等级最低 26？

Google Play 声明 2018 年 8 月开始，新提交的应用必须使用 api level 26 及以上版本编译。Android 构建时三个版本号的设置如下：

- compileSdkVersion 编译版本：编译 Java 代码时使用的 SDK 版本，和支持的最低版本无关。可以设置为 26/27/28。
- minSdkVersion：支持的最小版本，决定编译出的应用最小支持的 Android 版本。建议设置为 16（对应 Android 4.1）。
- targetSdkVersion：和运行时的行为有关，建议设置与 compileSdkVersion 一致，也可以设置为 22，避免 [运行时权限的问题](https://developer.android.com/training/permissions/requesting?hl=zh-cn)。

  ![](introduction/compile_version.png)

### 物理刚体的速度为什么会被限制

`b2_maxTranslation` 是引擎脚本 box2d.js 中用于限制刚体速度变化率的参数，并且默认值为 2。开发者可以通过修改这个值来改变刚体的最大速度值。box2d.js 在引擎目录中的相对路径是 **./engine/external/box2d/box2d.js**，找到并修改 `b2_maxTranslation` 的参数赋值之后，请参考 [引擎定制工作流程](https://docs.cocos.com/creator/manual/zh/advanced-topics/engine-customization.html#12-%E5%AE%89%E8%A3%85%E7%BC%96%E8%AF%91%E4%BE%9D%E8%B5%9619) 编译引擎即可。

### 场景加载报错：`typeError: children[i]._onBatchCreated is not a function`

这是由于用于记录场景的 json 文件中某个 `_children` 的值变成了 null，将其改成能够正常被读取的数值即可。

### VideoPlayer 播放视频显示黑屏

HTML 只支持 H.264 编码格式的 MP4，建议使用音视频格式转换工具输出 AVC(H264) 编码格式的 MP4 视频。具体可参考文章 https://blog.csdn.net/keji_123/article/details/77717849

### 打开项目后，未出现编辑器，仅显示 CocosCreator 图标

删除项目中 `local` 文件夹的 `local.json` ，之后重启项目即可。

### 运行预览后，Creator 默认的调试信息显示不清晰

CCProfiler.js 脚本在引擎目录中的相对路径是：**./engine/cocos2d/core/utils/profiler/CCProfiler.js**，找到 `cc.profiler` 对象后在其中加入如下代码，之后参考 [引擎定制工作流程](https://docs.cocos.com/creator/manual/zh/advanced-topics/engine-customization.html#12-%E5%AE%89%E8%A3%85%E7%BC%96%E8%AF%91%E4%BE%9D%E8%B5%9619) 编译引擎即可。

```
/*
 * @example
 * cc.profiler.setDebugInfoColor({ r: 255, g: 0, b: 0, a: 255 });
 */
setDebugInfoColor(colorObj) {
    //需要等待 _rootNode 节点激活才能修改调试信息颜色
    if (!_rootNode) {
        setTimeout(()=>{
            this.setDebugInfoColor(colorObj);
        }, 100)
    }
    else {
        let leftNode = _rootNode.getChildByName("LEFT-PANEL");
        let rightNode = _rootNode.getChildByName("RIGHT-PANEL");
        if (leftNode && rightNode && typeof colorObj === "object") {
            leftNode.color = rightNode.color = new cc.Color(colorObj);
        }
    }
}
```

### Widget 组件改变的坐标值当前帧不刷新

```
widget.updateAlignment();
```

在需要立即刷新节点坐标值的地方执行这句代码即可。

### 多点触控监听，假设 A、B 两点，按住 B 点，重复点击 A 点之后，手指离开 B 点时不响应 ‘touchend’ 事件

在项目中添加插件脚本并对 `cc.macro.TOUCH_TIMEOUT` 重新赋予一个更大的值即可解决这个问题。你可以通过阅读 [API 文档](https://docs.cocos.com/creator/api/zh/classes/macro.html#touchtimeout) 了解这个参数的定义。

### 动态修改 material 贴图

材质贴图接收的是 `cc.Texture2D` 对象，所以我们可以在 **属性检查器** 面板添加一个 `cc.Texture2D` 对象。例如：

```
goldTexture: {
    default: null,
    type: cc.Texture2D
}
```

最后直接调用材质系统的 `setProperty` 来修改贴图：

```
this.material.setProperty("diffuseTexture", this.goldTexture);
```

具体内容可参考 [范例](https://github.com/cocos-creator/example-cases) 工程中的 `custom_material` 场景

### 取消定时器失败，定时器仍然运行

this.unschedule(callBack, target) 接收的参数必须与 this.schedule(callBack, target) 一致。其中 callBack 必须是同一函数对象，而 target 也必须接收同一执行环境对象。如果传入的参数不同那么就不能正常停止 schedule。

### 如何远程加载图集

参考 [范例](https://github.com/cocos-creator/load-remote-plist)。
