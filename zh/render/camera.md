# Camera 摄像机

摄像机（Camera）组件在制作卷轴或是其他需要移动屏幕的游戏时比较有用，在没有 Camera 组件的情况下，卷轴游戏都是通过移动场景节点或是游戏根节点来实现的，这样将会导致大量节点的矩阵都需要重新计算，效率自然会有所降低，而使用 camera 是直接将 camera 的矩阵信息在渲染阶段统一处理，将会比移动节点来移动屏幕更加高效。

Camera 组件提供了两个属性来供用户设置：

- targets - 指定 camera 会拍摄哪些节点，即 camera 会影响哪些节点。
- zoomRatio - 指定 camera 的缩放比例, 值越大显示的图像越大。

Camera 组件将会随着他依附的节点进行移动，可以想象成我们举着摄像机跟随着摄像机的节点移动，而这个摄像机只会拍摄他的 targets 目标，Camera 组件拍摄的范围即是设备屏幕大小。

## 实例

我们用一个场景实例来解释 Camera 组件怎么使用。

假设我们在做一个物理游戏，需要 physics 节点和 tiled map 节点做背景，hero 做主角，我们的 camera 需要跟随 hero 来移动。

![camera-1](./camera/camera-1.png)

这里我们需要新建一个空节点，重命名为 camera，将该节点作为摄像机，使用一个单独的节点作为摄像机节点会更灵活。当然我们也可以直接将 Camera 组件添加到 hero 节点上，但是这样 camera 的位置就只能和 hero 节点重叠在一起了，不能做到缓慢跟随之类的效果。

选择 **层级管理器** 中的 camera 节点，点击 **属性检查器** 下面的 `添加组件 -> 添加其他组件 -> Camera` 按钮，即可添加 Camera 组件到节点上。

<img src="./camera/camera-2.png" style="width:50%;height:50%"></img>

在这里 Camera 组件添加了三个节点到 targets 上，即我们需要摄像机拍摄这三个节点。并且我们还添加了一个 `camera-control` 脚本组件，这个组件的作用主要是移动 camera 节点跟随 hero 节点。

实例可在 [物理系统示例](https://github.com/2youyou2/physics-example) 中的 tiled 示例中找到。

也可以参考 [Camera Demo](https://github.com/cocos-creator/demo-camera)，包含了 camera 的使用示例。

**注意** ：

当我们使用 camera 时，如果使用到了物理系统或者碰撞系统这些会内置渲染节点的系统，需要调用相关的 api 将他们的渲染节点也添加到camera 上。

```javascript
cc.director.getPhysicsManager().attachDebugDrawToCamera(camera);
cc.director.getCollisionManager().attachDebugDrawToCamera(camera);
```
