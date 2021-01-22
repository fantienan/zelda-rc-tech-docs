# 重构过程中产生的遗留问题

### map/tool/featureTool/addTool.js

> 因为涉及到改 openlayers 源码，所以重构后没有用 SnapDraw 类，待后期找到修改 openlayers 源码的方法在优化。

原来的

```js
function _drawGeometry(type) {
  _interaction_draw = new ol.interaction.SnapDraw({
    ...
  });
  ...
}
```

重构后的

```js
_drawGeometry(type) {
	this._interaction_draw = new Draw({
		...
	});
	...
}
```
