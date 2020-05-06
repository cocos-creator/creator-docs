# fillColor

The `fillColor` property sets or returns the color used for the fill.

## Example

```javascript
var ctx = node.getComponent(cc.Graphics);
ctx.fillColor = new cc.Color().fromHEX('#0000ff');
ctx.rect(20,20,250,200);
ctx.fill();
```

![](graphics/fillColor.png)

<hr>

Return to [Graphics Component Reference](../../components/graphics.md)
