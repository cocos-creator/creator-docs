# Label Component Reference

Label components are used to display a piece of text, text can be system fonts, TrueType fonts or BMFont fonts and art numbers, in addition, label also has typesetting function.

![label-property](./label/label-property.png)

Click the **Add Component** button at the bottom of the **Properties** panel and select **Label** from **Renderer Component** to add the Label component to the node.

## Label Properties

| Properties |   Function Explanation
| -------------- | ----------- |
|String| Text content character string.
|Horizontal Align| Horizontal alignment pattern of the text. The options are LEFT, CENTER and RIGHT.
|Vertical Align| Vertical alignment pattern of the text. The options are TOP, CENTER and BOTTOM.
|Font Size| Font size of the text.
|SpacingX | The spacing between font characters, only available in BMFont.
|Line Height| Line height of the text.
|Overflow| Layout pattern of the text. Currently supports CLAMP, SHRINK and RESIZE_HEIGHT. See [Label Layout](#label-layout) below for more detailed information.
|Enable Wrap Text| Enable or disable the text line feed. (which takes effect when the Overflow is set to CLAMP or SHRINK)
|Font | Designate the font file needed for rendering the text. If the system font is used, then this attribute can be set to null.
|Font Family| Font family name, takes effect when using system font.
|Cache Mode| The text cache mode (New in v2.0.9). Takes effect only for **system font** or **ttf** font, BMFont do not require this optimization. And includes **NONE**, **BITMAP**, **CHAR** three modes. See [Cache Mode](#cache-mode-new-in-v209) below for details.
|  Use System Font | Boolean value, choose whether to use the system font or not.

## Label Layout

| Properties |   Function Explanation
| -------------- | ----------- |
|CLAMP| The text size won't zoom in or out as the Bounding Box size changes. When Wrap Text is disabled, parts exceeding the Bounding Box won't be shown according to the normal character layout. When Wrap Text is enabled, it will try to wrap the text exceeding the boundaries to the next line. If the vertical space is not enough, any not completely visible text will also be hidden.
|SHRINK| The text size will zoom in or out (it won't zoom out automatically, the maximum size that will show is specified by Font Size) as the Bounding Box size changes. When Wrap Text is enabled, if the width is not enough, it will try to wrap the text to the next line before automatically adapting the Bounding Box's size to make the text show completely. If Wrap Text is disabled, then it will compose according to the current text and zoom automatically if it exceeds the boundaries. **Note**: This mode may takes up more CPU resources when the label is refreshed.
|RESIZE_HEIGHT| The text Bounding Box will adapt to the layout of the text. The user cannot manually change the height of text in this status; it is automatically calculated by the internal algorithm.

## Cache Mode (New in v2.0.9)

| Properties |   Function Explanation
| -------------- | ----------- |
|  NONE  | Defaults, the entire text in label will generate a bitmap
| BITMAP | After selection, the entire text in the Label will still generate a bitmap, but will try to participate in [Dynamic Atlas](../advanced-topics/dynamic-atlas.md). As long as the requirements of Dynamic Atlas are met, the Draw Call will be merged with the other Sprite or Label in the Dynamic Atlas. Because Dynamic Atlas consume more memory, **this mode can only be used for Label with infrequently updated text**. **Note**: Similar to NONE, BITMAP will force a bitmap to be generated for each Label component, regardless of whether the text content is equivalent. If there are a lot of Labels with the same text in the scene, it is recommended to use CHAR to reuse the memory space.
|  CHAR  | The principle of CHAR is similar to BMFont, Label will cache text to the global shared bitmap in "word" units, each character of the same font style and font size will share a cache globally. Can support frequent modification of text, the most friendly to performance and memory. However, there are currently restrictions on this model, which we will optimize in subsequent releases:<br>1. **This mode can only be used for font style and fixed font size (by recording the fontSize, fontFamily, color, and outline of the font as key information for repetitive use of characters, other users who use special custom text formats need to be aware). And will not frequently appear with a huge amount of unused characters of Label.** This is to save the cache, because the global shared bitmap size is 2048*2048, it will only be cleared when the scene is switched. Once the bitmap is full, the newly appearing characters will not be rendered. <br>2. Cannot participate in dynamic mapping (multiple labels with CHAR mode enabled can still merge Draw Call in the case of without interrupting the rendering sequence).

**Note**：

- Cache Mode has an optimized effect for all platforms.
- The BITMAP mode replaces the original Batch As Bitmap option, and old projects will automatically migrate to this option if Batch As Bitmap is enabled.
- The **RenderTexture** module in the **Project -> Project Settings -> Module Config** panel cannot be removed when using the cache mode.

## Detailed Explanation

By dragging the TTF font file and BMFont font file into the `Font` attribute in the **Properties** panel the Label component can alter the rendering font type. If you want to stop using a font file, you can use the system font again by checking `Use System Font`.

If you want to use LabelAtlas, you should create a LabelAtlas font asset at first. Please refer to [LabelAtlas asset](../asset-workflow/label-atlas.md) for more information.

### UI Rendering Batch Processing Guide

For more info of this feature, please refer to [UI Rendering Batch Processing Guide](../advanced-topics/ui-auto-batch.md) topic.

---

Continue on to read about [Spine Component Reference](spine.md).
