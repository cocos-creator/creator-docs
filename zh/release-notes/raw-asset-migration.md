# v1.10 资源升级指南

> 目前 v1.10 仍然未正式发布，测试版可到 [论坛发布帖](https://forum.cocos.com/t/cocos-creator-v1-10-0-4-10/58534) 下载

在 v1.10 之前的 [获取和加载资源](../scripting/load-assets.md) 文档中，我们有提到过 Creator 的资源分成了 [Asset](../scripting/load-assets.md#asset) 和 [RawAsset](../scripting/load-assets.md#raw-asset) 两种。当时这样划分主要是为了尽量重用已有的 Cocos2d-x 基础模块，并且降低 Cocos2d-x 用户的上手门槛。不过我们一直仍希望把 RawAsset 全部替换成标准的 Asset，随着 Creator 这两年的发展，是时候进行一轮重构了。重构后可以简化编辑器和引擎对资源的处理方式，减小发布后的 settings.js 文件体积，同时提升用户的开发体验。

为了带来平滑的升级体验，我们将在一段时间内保留对 RawAsset 的兼容。v1.10 开始，会先支持调整后的新的 API，并且对已经废弃的用法给出警告，请用户根据警告和本文的说明对代码进行调整，升级到新的用法。等到 v2.1 以上的某个版本，才会全面移除对 RawAsset 的兼容。

对 **美术策划** 而言，项目中的所有资源，例如场景、动画、Prefab 都不需要修改，也不用升级。<br>
对 **程序** 而言，影响主要体现在原先在代码中用 URL 字符串表示的资源，需要都改为直接引用加载后的实际资源对象。Creator 中和资源操作有关的 API 也将由传入资源 URL 改为传入资源对象。本文将详细介绍有关内容。

## 常见问题

### 我需要升级吗？

如果有下列情况，你需要升级：
 - 你在自己的游戏代码中直接声明了这些类型：`cc.Texture2D`, `cc.RawAsset`, `cc.AudioClip` 和 `cc.ParticleAsset`
 - 你扩展了引擎或编辑器，定义了新的直接继承自 `cc.RawAsset` 的类型

如果有下列情况，可能需要升级：
 - 你在自己的游戏代码中直接调用了 `cc.audioEngine` 或 `cc.textureCache`
 - 你使用 `cc.loader` 加载了远程服务器上的文本、粒子

### 我其实并不确定要升级哪些东西？

Cocos Creator 是一个非常重视兼容性的引擎，代码中所有需要升级的部分，都会在编辑器或者游戏运行时给出警告。只要你根据错误信息调整了 API 用法就可以了。

### 旧项目有对资源管理做一些特殊操作，我不敢乱改，怎么办？

RawAsset 调整为 Asset，本质上无非就是从引擎层面把字符串转变成对象。只要你保证跟引擎交互时，所使用的是对象即可，原先项目内部想要继续使用字符串还是可以的。你要做的无非就是，从引擎获取对象后先转换为字符串，传字符串给引擎前先转换为对象。

 - Asset 转字符串<br>
对于 Texture2D, RawAsset, AudioClip 和 ParticleAsset 类型的资源来说，可以直接通过 `.nativeUrl` 获得原有的 URL。如果无法获取则说明这是其它类型的 Asset 对象，其它类型的对象本来就不用升级，所以不用修改。

	```js
	var url = this.file.nativeUrl || this.file;
	```

 - 字符串转 Asset

	```js
	cc.loader.loadRes(musicURL, cc.AudioClip, function (err, audioClip) {
	    cc.log(typeof audioClip);  // 'object'
	});
	```

## 升级步骤

 - 重命名旧版本 Cocos Creator 所在目录，然后安装新版本 Cocos Creator。这样新旧两个版本就能共存。
 - **备份好旧版本的工程后**，使用新版 Cocos Creator 打开原有工程，Creator 将对有影响的资源重新导入，第一次升级时会稍微多花一点时间，导入完毕后就会打开编辑器主窗口。
 - 工程打开后，可能会出现一堆黄色的警告，警告一般都不会影响游戏的预览发布，但是强烈建议都尽快解决。下面我将对常见的警告信息进行讲解。

![](raw-asset-migration/warning.png)

### "Please change the definition of property 'audio_bgMusic' in class 'FOO'..."

这是一类升级过程中最常见的错误，其中 FOO 是你声明的 CCClass 的类名，也就是文件名，audio_bgMusic 是你的属性名。你需要在 `FOO.js` 中找到 audio_bgMusic 定义的地方，然后根据后面的提示信息进行修改。后面的提示信息假设为：

#### "the use of declaring a property in CCClass as a URL has been deprecated..."

这句话的意思是，你在声明 `audio_bgMusic` 属性的时候，指定了 `url` 类型，现在已经不支持使用 `url` 了。通过查找项目中的 `FOO.js`，你能找到原先类似这样的定义方式：

	```
    // FOO.js

    audio_bgMusic: {
        default: ***,
        url: cc.AudioClip,
    },
	```

将 url 改为 type，并且确保 default 为 null。

	```
    audio_bgMusic: {
        default: null,
        type: cc.AudioClip,  // use 'type:' to define Asset object directly
    },
	```

这样一来，游戏场景加载后，audio_bgMusic 就会是一个 AudioClip 类型的对象，而不是原先的 audio 字符串。为了确保游戏逻辑不会出错，请继续在项目中全局搜索 audio_bgMusic，确保你不会对这个变量做任何字符串相关的 substring、replace 等调用，否则请先通过 audio_bgMusic.nativeUrl 得到真实的 URL。

注意：如果你原先定义的类型是 `cc.RawAsset`，除了修改 url 为 type，连带类型也应该改为 `cc.Asset`。<br>
假如原来是：

	```
	// 声明时
    manifest: {
        default: ***,
        url: cc.RawAsset
    },

    // 访问时
    this._am = new jsb.AssetsManager(this.manifest, storagePath);
	```

请修改为：

	```
	// 声明时
    manifest: {
        default: null,
        type: cc.Asset
    },

    // 访问时
    this._am = new jsb.AssetsManager(this.manifest.nativeUrl, storagePath);
	```

#### "properties in CCClass can not be abbreviated if they are of type RawAsset"

除了前面提到的警告信息，你还有可能看这个警告。这句话的意思是，你在声明 `audio_bgMusic` 属性的时候，使用了将来有可能引起歧义的简便形式，这些简写暂时被废弃了，等到大部分项目都平滑升级上去后，才会重新支持。通过查找项目中的 `FOO.js`，你能找到原先类似这样的定义方式：

	```
    audio_bgMusic: cc.AudioClip,
	```

你需要参照前面的修改方式，使用 type + default 进行完整声明：

	```
    audio_bgMusic: {
        default: null,
        type: cc.AudioClip,
    },
	```

这样一来，游戏场景加载后，`audio_bgMusic` 就会是一个 AudioClip 类型的对象，而不是原先的 audio 字符串。相关的注意事项都和前面一致，这里不再追述。

### "textureCache.addImage(url) - The type of the url should be string, not Texture2D..."

这个警告一般是如下代码引起的：

	```
	// 按照上面的文档升级后的写法
    tex: {
        default: null,
        type: cc.Texture2D,
    },

    // 原先获取 texture 的代码
	var texture = cc.textureCache.addImage(this.tex);
	```

这个警告的意思是，当你调用 `addImage` 时，你传入的已经是一个 Texture2D 对象了，所以直接使用这个对象就行，不需要再做加载。因为升级后的 `tex` 就已经是一个 Texture2D 了。也就是说你只要：

	```
	var texture = this.tex;
	```

### "Since 1.10, `cc.audioEngine.play` accept cc.AudioClip instance directly, not a URL string..."

这个警告一般是这种代码引起的：

	```
	var url = cc.url.raw('resources/bg.mp3');
	cc.audioEngine.play(url);
	```

请改成：

	```
	cc.loader.loadRes('resources/bg', cc.AudioClip, function (err, clip) {
		cc.audioEngine.play(clip);
	});
	```

## 其它更新

### 新增了 `cc.TextAsset` 用于加载文本文件

从 1.10 开始，常见的文本格式，如 `.txt, .plist, .xml, .json, .yaml, .ini, .csv, .md`，都会导入为 `cc.TextAsset`。


### 其余未知类型默认也全都导入为 `cc.Asset`

对从编辑器导入的未知类型的文件来说，原先都是导入为无类型的 `cc.RawAsset`，现在会导入为 `cc.Asset`。声明方法和上面一样，将 `url: cc.RawAsset` 改为 `type: cc.Asset`，访问时也同样采用 `asset.nativeUrl` 这样的方式获得原始 url 即可。

## 反馈问题

如果还有其它升级问题，可点击页面右下角的"提交反馈"，直接到本文档的 GitHub 仓库提交 issue，或者在 [论坛发布帖](https://forum.cocos.com/t/cocos-creator-v1-10-0-4-10/58534) 里回复。
