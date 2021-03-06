<!--
 index: 3
 title: 升级指南
 resource:
   jsFiles:
     - ${url.g6}
-->

# 概述

该文档是 G6 升大版本时的升级指南文档，如果有你觉得描述的不够清晰的地方，升级有困难的话，敬请[联系我们](/about/index.html)！

## G6 1.1.X -> G6 1.2.X

* [移除了两个内置网图布局](#_移除了两个内置网图布局)
* [拼写错误](#_拼写错误)

### 移除了两个内置网图布局

经过实践检验 `G6.Layouts.Force` &&  `G6.Layouts.Flow` 这两个布局，十分鸡肋。1.2.0 我们从源码中移除了这两个布局。其相关功能可以使用 [layout.d3.force--力导布局](../api/plugins.html#_layout.d3.force--力导布局) && [layout.dagre--统一分层布局](../api/plugins.html#_layout.dagre--统一分层布局)

### 拼写错误

该版本修复了一下拼写错误，错误的写法将在下个版本移除，敬请悉知，修复如下：

* 错误 -> 正确
* G6.Global.nodeAcitvedBoxStyle -> G6.Global.nodeActivedBoxStyle
* G6.registNode -> G6.registerNode
* G6.registEdge -> G6.registerEdge
* G6.Layout -> G6.Layouts

## G6 0.X -> G6 1.X

* [原来的 Graph 既是现在的 Net](#_原来的-graph-既是现在的-net)
* [新的锚点描述](#_新的锚点描述)
* [再见无穷尽的xxxable](#_再见无穷尽的xxxable)
* [调整初始化视口应该用fitView](#_调整初始化视口应该用fitview)
* [统一输入输出 save && read](#_统一输入输出-save-amp-amp-read)

### 原来的 Graph 既是现在的 Net

Graph 在 G6 1.0 中已经沉为抽象类，用户不应该直接实例化 Graph 使用。1.0 中 Net 基本含 0.X Graph 所有属性、方法。从 0.X 迁 1.X 的用户应将所有的 Graph 改为 Net。

### 新的锚点描述

锚点在关系图中无非是一个至关重要的概念，如果 0.X 中你使用了自定义锚点，应参看下文。

过去的锚点，我们定义了上（0）右（1）下（2）左（3），四个边，然后在通过第二个参数确定锚点的位置，如左图所示。这样不仅理解起来复杂，而且还有一个硬伤 —— 锚点无法定义到图形上！在新的锚点机制里，我们以左上角为（0，0）点，将锚点的范围拓展至了整个图形的包围盒平面内，如右图所示：

<img src="https://zos.alipayobjects.com/rmsportal/dpLPhjRBQTvESnCydHwA.png" style = "width: 500px;"/>

这样我们不仅能把锚点定义到图形包围盒边上，还能定义到包围盒内图，以做到`锚点`跟图形精确关联，如下图所示：

<img src="https://zos.alipayobjects.com/rmsportal/lneevqEqTkjvGYJPKoiK.png" style = "width: 500px;"/>


### 再见无穷尽的xxxable

诸如：`selectable`、`dragable`、`zoomable`，在 1.0 后，应用行为模式的交互插拔机制实现，比如：

0.X 中

```js
const graph = new Graph({
  zoomable: false
});
```

等于 1.X 中

```js
const net = new Net();
net.removeBehaviour(['wheelZoom']);
```


行为、模式的插拔组装，参见[Demo](../demo/other/mode.html);

### 调整初始化视口应该用fitView

0.x 中有时我们会`render`结束后，调用`autoZoom`方法，让图适应画布。1.X 中这种方法被废弃，需要在初始化图时配置`fitView`，以决定初始化时画布的视口。[Graph API](../api/graph.html)，fitView 属性。


<button id='autoZoom'>自动缩放 autoZoom</button>&nbsp;&nbsp;<button id='autoSize'>自动画布尺寸 autoSize</button>&nbsp;&nbsp;<button id='restore'>还原</button>
<div id='fitView'></div>

```js-
var Matrix = G6.Matrix.Matrix3;
var autoZoom = $('#autoZoom');
var autoSize = $('#autoSize');
var restore = $('#restore');
// 第三步：设置数据
var data = {
  "nodes": [
    {
      "x": 140,
      "y": 210,
      "id": "node1"
    },
    {
      "x": 270,
      "y": 210,
      "id": "node2"
    }
  ],
  "edges": [
    {
      "source": "node1",
      "id": "edge1",
      "target": "node2"
    }
  ]
};
// 第四步：配置G6画布
var net = new G6.Net({
    id: 'fitView', // 容器ID
    width: 500,    // 画布宽
    height: 500    // 画布高
  });
// 第五步：载入数据
net.source(data.nodes, data.edges);
// 第六步：渲染关系图
net.render();
autoZoom.on('click', function(){
  net.clear();
  net.set('fitView', 'autoZoom');
  net.source(data.nodes, data.edges);
  net.render();
});
autoSize.on('click', function(){
  net.clear();
  net.set('fitView', 'autoSize');
  net.source(data.nodes, data.edges);
  net.render();
});
restore.on('click', function(){
  net.changeSize(500, 500);
  net.updateMatrix(new Matrix());
  net.clear();
  net.set('fitView', null);
  net.source(data.nodes, data.edges);
  net.render();
});
```

### 统一输入输出 save && read

在使用关系图开发的过程中，我们常常需要导出导入数据，以满足关系图的存储、分享、编辑的功能。0.X 中 `graph.save` 和 `graph.srouce` 并没有严格对照。 G6 1.0 后，我们对这部分的需求做了总结。在 Graph 定义了一对相互对照的接口，read <==> save，后面用户应该严格通过这两个该接口导入导出数据。

#### 读入数据 -- read

```js
/**
 * 导入数据
 * @param  {Object}  data   通过save导出的数据
 */
graph.read(data);
```

#### 保存数据 -- save

```js
/**
 * 导出数据
 * @return  {Object}  data  导出的数据
 */
graph.save();
```
