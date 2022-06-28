---
title: 用React手写简单的低代码编辑器
date: 2022-06-28 23:06:03
tags: [前端, react]
---

> 参考教程来自超——级温柔可爱的越越老师：https://juejin.cn/post/7101148854371745823#heading-9
>
> GitHub地址：https://github.com/ERUIHNIYHBKBNF/mini-lowcode

## 概述

技术选型：React, TypeScript, React-Dnd

低代码编辑器，拖拖拽拽，生成页面，通过一定格式的数据存储，通过一定的方式解析成可见页面。

主要由三个部分组成：组件区、画布区、属性编辑区。            

牛逼一点的比如drawio长这样：

<img src="https://cdn.jsdelivr.net/gh/ERUIHNIYHBKBNF/picapica@main/frontend/2022062701.webp" width="600px">

## 数据格式定义

自由约定就好了叭，这边简单用json来描述画布的内容。（/src/mock/editorData.json）：

```json
{
  "projectId": "xxx",
  "projectName": "xxx",
  "author": "xxx",
  "data": [
    {
      "id": "1", // 每个组件都有个唯一标识
      "type": "text", // 组件类型，这个是文本框
      "data": "xxxxxx", // 文本组件的data就是文本内容
      "color": "#000000", // 以下是css，这里使用的是absolute定位
      "size": "12px",
      "width": "100px",
      "height": "100px",
      "left": "100px",
      "top": "100px"
    },
    {
      "id": "2",
      "type": "image",
      "data": "http://xxxxxxx", // 图片/视频组件的data就是对应资源的url
      "width": "100px",
      "height": "100px",
      "left": "100px",
      "top": "100px"
    },
    {
      "id": "3",
      "type": "video",
      "data": "http://xxxxxxx",
      "width": "100px",
      "height": "100px",
      "left": "100px",
      "top": "100px"
    }
  ]
}
```

## 项目搭建

简单绘制一下页面，分为左中右三个部分：

<img src="https://cdn.jsdelivr.net/gh/ERUIHNIYHBKBNF/picapica@main/frontend/2022062703.webp" width="750px">

文件结构也很普通qwq：

<img src="https://cdn.jsdelivr.net/gh/ERUIHNIYHBKBNF/picapica@main/frontend/2022062702.webp" width="500px">

## 编写一个简单的可拖动组件

React DnD，react官方开发的拖拽库，https://react-dnd.github.io/react-dnd/

搬运文档里提供的一些概念：

* Items and Types：React DnD使用数据，而不是视图

  > When you drag something across the screen, we don't say that a component, or a DOM node is being dragged. Instead, we say that an *item* of a certain *type* is being dragged.

  * item：使用数据（一般是js对象）来描述一个可拖动对象，类似上面json
  * type：拖动元素的类型，例如上方json中的 text, image等

  * drag sources：可拖动的item

  * drop targets：可以被放置item的元素

  > The types let you specify which drag sources and drop targets are compatible.

* Monitors：记录dnd相关组件的state，例如正在拖动/已放置。通过定义一个collect函数，允许开发者在dnd状态改变时，即时修改组件的props

  例如这样定义一个collect来即时更新一个棋子的部分props：

  ```js
  collect: (monitor) => ({
    highlighted: monitor.canDrop(), // 拖动时高亮
    hovered: monitor.isOver(), // 鼠标指向时表现hover
  })
  ```

安装依赖：

```bash
npm install react-dnd react-dnd-html5-backend
```

在根组件中引入Provider：

```tsx
<DndProvider backend={HTML5Backend}>
  <div className="App">...</div>
</DndProvider>
```

补充定义一些组件类型（/src/consts/types.ts）：

```ts
export enum COMPONENT_TYPE {
  TEXT = 'text',
  VIDEO = 'video',
  IMAGE = 'image',
  AUDIO = 'audio',
  CARD = 'card',
}
```

编写一个可拖动的组件（/src/components/textComponent.tsx）：

```tsx
import React from 'react';
import { useDrag } from 'react-dnd';
import { COMPONENT_TYPE } from '../../consts/types';
import './style.css';

export default function TextComponent() {
  const [, drag] = useDrag(() => ({
    type: COMPONENT_TYPE.TEXT,
    collect: monitor => ({
      isDragging: !!monitor.isDragging(),
    }),
  }));
  
  return (
    <div className="textComponent" ref={drag}>
      文字组件
    </div>
  );
}
```

在左侧面板中引入这个可拖动组件，就可以在整个窗口范围内拖动了。

## 通过中间画板渲染元素

先添加一组mock数据假装是已保存的画布（/src/mock/drawData/json）：

```json
{
  "data": [
    {
      "id": "text-1",
      "type": "text",
      "data": "我是 1 号文字",
      "color": "#FF0000",
      "size": "12px",
      "width": "100px",
      "height": "20px",
      "left": "100px",
      "top": "100p"
    },
    {
      "id": "text-2",
      "type": "text",
      "data": "我是 2 号文字",
      "color": "#00FF00",
      "size": "12px",
      "width": "100px",
      "height": "20px",
      "left": "100px",
      "top": "150p"
    },
    {
      "id": "text-3",
      "type": "text",
      "data": "我是 3 号文字",
      "color": "#0000FF",
      "size": "12px",
      "width": "100px",
      "height": "20px",
      "left": "100px",
      "top": "200p"
    }
  ]
}
```

添加一些类型，用于标识右侧面板展示哪些配置项（/src/consts/types.ts）：

```ts
export enum RIGHT_PANEL_TYPE {
  NONE = 'none',
  TEXT = 'text',
  VIDEO = 'video',
  IMAGE = 'image',
  CARD = 'card',
}
```

在App.tsx中新增两个state，并将其传给中间画板：

```tsx
const [drawPanelData, setDrawPanelData] = useState([...drawData.data]);
const [rightPanelType, setRightPanelType] = useState(RIGHT_PANEL_TYPE.TEXT);

<MidPanel data={drawPanelData} setRightPanelType={setRightPanelType} />
```

在MidPanel中渲染元素（核心就是一个generateContent）（/src/pages/midPanel）：

```jsx
import React from "react";
import { RIGHT_PANEL_TYPE } from "../../consts/types";
import './style.css';

type DrawPanelProps = {
  data: any;
  setRightPanelType: Function;
}

export default function MidPanel({data, setRightPanelType}: DrawPanelProps) {
  const generateContent = () => {
    const ret = [];
    for (const item of data) {
      switch (item.type) {
        case 'text':
          ret.push(
            <div
              key={item.id}
              onClick={() => {
                console.log(`clicked: item ${item.id}`);
                setRightPanelType(RIGHT_PANEL_TYPE.TEXT);
              }}
              style={{
                color: item.color,
                fontSize: item.size,
                width: item.width,
                height: item.height,
                left: item.left,
                top: item.top,
                position: 'absolute',
                backgroundColor: '#bbbbbb'
              }}
            >
              {item.data}
            </div>
          );
          break;
      }
    }
    return ret;
  }
  
  return (
    <div className="midPanel"> 
      <div>
        <h1>MidPanel</h1>
      </div>
      {/* 顺带把这个盒子定位改成relative，方便内部组件用(absolute)height和left确认位置，定住宽高方便之后摆放 */}
      <div>
        {generateContent()}
      </div>
    </div>
  );
}
```

## 右侧属性编辑面板

首先要知道编辑的是哪个元素，在app.tsx和midPanel.tsx中新增state相关：

```tsx
const [rightRanelElementId, setRightRanelElementId] = useState('');
// 对应组件内也做一些修改，这里不放代码了qwq
<MidPanel
  data={drawPanelData}
  setRightPanelType={setRightPanelType}
  setRightRanelElementId={setRightRanelElementId}
/>
<RightPanel
  type={rightPanelType}
  data={drawPanelData}
  elementId={rightRanelElementId}
  setDrawPanelData={setDrawPanelData}
/>
```

新增type方便写ts（/src/consts/types）：

```ts
export type ElementType = {
  id: string;
  type: string;
  [prop: string]: any;
};
```

然后开始编写右侧面板，类似中间面板的方式（/src/pages/rightPanel）：

```tsx
export default function RightPanel({
  type,
  data,
  elementId,
  setDrawPanelData
}: RightPanelProps) {
  // 找到要修改的元素以将其原始值展示到右侧
  const findCurrentElement = (id: string) => {
    return data.find((item: ElementType) => item.id === id);
  }
  // 修改id元素key属性newData值
  const changeElementData = (id: string, key: string, newData: any) => {
    const element = findCurrentElement(id);
    if (element) {
      element[key] = newData;
      setDrawPanelData([...data]);
    }
  }

  const generateRightPanel = () => {
    if (type === RIGHT_PANEL_TYPE.NONE) {
      return <div>属性编辑区</div>;
    }
    switch (type) {
      case RIGHT_PANEL_TYPE.TEXT:
        if (!elementData) {
          return <div>属性编辑区</div>;
        }
        const inputDomObject: Array<HTMLInputElement> = []; // 保存输入框的DOM便于更新时获取其值

        return (
          <div key={elementId}>
            <div>文字元素</div>
            <br />
            <div className="flex-row-space-between text-config-item">
              <div>文字内容:</div>
              <input
                defaultValue={elementData.data}
                ref={(element) => {
                  inputDomObject[0] = element!;
                }}
                type="text"
              ></input>
            </div>
			{/* 各项属性表单... */}
            <br />
            <button
              onClick={() => {
                changeElementData(elementId, 'data', inputDomObject[0].value);
				// ......
              }}
            >
              确定
            </button>
          </div>
        );
    }
  };

  return (
    <div className="rightPanel">
      <div>
        <h1>RightPanel</h1>
      </div>
      <div className="rightFormContainer">
        {generateRightPanel()}
      </div>
    </div>
  );
}
```

基本的样子已经有了：

<img src="https://cdn.jsdelivr.net/gh/ERUIHNIYHBKBNF/picapica@main/frontend/2022062801.webp" width="800px">

## 中间接收并生成元素

掏出之前写的（/src/components/testComponent）这个地方，可以发现允许drag的同时指定了type：

```tsx
const [_, drag] = useDrag(() => ({
  type: COMPONENT_TYPE.TEXT
}));
```

在midPanel中useDrop，接受TEXT类型元素，并计算相对位置保存进data：

```tsx
const containerRef = React.useRef<HTMLDivElement>(null); // 分给midPanel容器便于计算坐标
const [, drop] = useDrop(() => ({
  accept: COMPONENT_TYPE.TEXT, // drop接受的type
  drop: (_, monitor) => {
    const { x, y } = monitor.getSourceClientOffset()!; // 相对屏幕左上角的位置
    // 计算相对容器左上角的位置
    const [currentX, currentY] = [x - containerRef.current!.offsetLeft, y - 75];
    setData([
      ...data,
      {
        id: `text-${Date.now()}`,
        type: 'text',
        data: '我是新建的文字',
        color: '#000000',
        size: '12px',
        width: '100px',
        height: '20px',
        left: `${currentX}px`,
        top: `${currentY}px`
      }
    ]);
  }
}));
// 以及对要接受drop的元素指定ref为drop
<div className="midItemsContainer" ref={drop}>
  {generateContent()}
</div>
```

**注意为App下的MidPanel元素添加key值**：这样新增元素时才能让react认为存在更新，这里直接用item的个数作为key值。

```tsx
<MidPanel
  key={`${drawPanelData.length}`}
  data={drawPanelData}
  setRightPanelType={setRightPanelType}
  setRightRanelElementId={setRightRanelElementId}
  setData={setDrawPanelData}
/>
```

## 可拖动调整位置

以上操作就可以完成一个简易的低代码编辑器啦，不过我们仍然可以利用这个小项目来练习一下刚刚所学的知识，例如再写个图片组件、布局组件之类的，或者让中间已经放置的元素仍然可以被拖动。毕竟放上不能再挪位置就很不合理qwq

我们先把midPanel渲染的文章组件拿出来，并且使它可拖动（/src/pages/midPanel）：

```tsx
type TextComponentDropedProps = {
  item: ElementType;
  setRightPanelType: Function;
  setRightRanelElementId: Function;
};

function TextComponentDroped({
  item,
  setRightPanelType,
  setRightRanelElementId
}: TextComponentDropedProps) {
  const [, drag] = useDrag(() => ({
    type: COMPONENT_TYPE.TEXT_DROPED,
    item: { id: item.id }, // 这里把id传进去以便后面drop接收
  }));
  return (
    <div
      onClick={() => {
        console.log(`clicked: item ${item.id}`);
        setRightPanelType(RIGHT_PANEL_TYPE.TEXT);
        setRightRanelElementId(item.id);
      }}
      style={{
        color: item.color,
        fontSize: item.size,
        width: item.width,
        height: item.height,
        left: item.left,
        top: item.top,
        position: 'absolute',
        backgroundColor: '#bbbbbb'
      }}
      ref={drag}
    >
      {item.data}
    </div>
  );
}
```

仍然是在midPanel中接收，但这种类型的元素drop时只是去更新data而不是创建新的元素（/src/pages/midPanel）：

```tsx
const [, drop] = useDrop(() => ({
  accept: [COMPONENT_TYPE.TEXT, COMPONENT_TYPE.TEXT_DROPED], // drop可接受的type中增添新的类型
  drop: (_, monitor) => {
    const { x, y } = monitor.getSourceClientOffset()!;
    // 计算相对容器左上角的位置
    const [currentX, currentY] = [x - containerRef.current!.offsetLeft, y - 75];
    switch (monitor.getItemType()) {
      case COMPONENT_TYPE.TEXT:
        setData([
          ...data,
          {
            id: `text-${Date.now()}`,
            type: 'text',
            data: '我是新建的文字',
            color: '#000000',
            size: '12px',
            width: '100px',
            height: '20px',
            left: `${currentX}px`,
            top: `${currentY}px`
          }
        ]);
        return;
      case COMPONENT_TYPE.TEXT_DROPED:
        // 这里直接使用leftPanel的changeElementData
        changeElementData((monitor.getItem() as { id: string }).id, 'left', `${currentX}px`);
        changeElementData((monitor.getItem() as { id: string }).id, 'top', `${currentY}px`);
        return;
    }
  }
}));
```

这样就ok啦qwq
