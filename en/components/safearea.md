# SafeArea component reference

This component is used to adjust the layout of current node to respect the safe area of a notched mobile device such as the iPhone X.

It is typically used for the top node of the UI interaction area. For specific usage, refer to the official [example-cases/02_ui/16_safeArea/SafeArea.fire](https://github.com/cocos-creator/example-cases).

The concept of safe area is to give you a fixed inner rectangle in which you can safely display content that will be drawn on screen.

You are strongly discouraged from providing controls outside of this area. But your screen background could embellish edges.

![Renderings](./safearea/renderings.png)

This component internally uses the API `cc.sys.getSafeAreaRect();` to obtain the safe area of the current iOS or Android device and implements the adaptation by using the Widget component and set anchor.