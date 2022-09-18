# 5 种瀑布流场景的实现

我报名参加金石计划 1 期挑战——瓜分 10 万奖池，这是我的第 4 篇文章，[点击查看活动详情](https://s.juejin.cn/ds/jooSN7t "https://s.juejin.cn/ds/jooSN7t")

## 背景

本文介绍 5 种瀑布流场景的实现，**大家可以根据自身的需求场景进行选择**。

5 种场景分别是：

- 纵向+高度排序
- 纵向+高度排序+根据宽度自适应列数
- 横向
- 横向+高度排序
- 横向+高度排序+根据宽度自适应列数

我已经将这 5 种场景的实现封装成 npm 包，npm 包地址：<https://www.npmjs.com/package/react-masonry-component2>，可以直接在 React 项目中安装使用。

## 介绍

瀑布流，是比较流行的一种网站[页面布局](https://baike.baidu.com/item/%E9%A1%B5%E9%9D%A2%E5%B8%83%E5%B1%80?fromModule=lemma_inlink)，视觉表现为参差不齐的多栏布局，随着页面[滚动条](https://baike.baidu.com/item/%E6%BB%9A%E5%8A%A8%E6%9D%A1/7166861?fromModule=lemma_inlink)向下滚动，这种布局还会不断加载[数据块](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E5%9D%97/107672?fromModule=lemma_inlink)并附加至当前尾部。

下图就是一个瀑布流布局的示意图：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0bc735c3dc32429680a801b2a6b820b9~tplv-k3u1fbpfcp-watermark.image?)

## 纵向+高度排序

纵向+高度排序指的是，每列按照纵向排列，往高度最小的列添加内容，如下图所示。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d55b1ff1db04d55afd2ae128d146af6~tplv-k3u1fbpfcp-watermark.image?)

实现纵向+高度排序瀑布流的方法是 **CSS 多列布局**。

### 1. 多列布局介绍

[多列布局](https://www.runoob.com/css3/css3-multiple-colu1ns.html)指的是 CSS3 可以将文本内容设计成像报纸一样的多列布局，如下实例:

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7da202ae1bf4857851aa5a363ffae6b~tplv-k3u1fbpfcp-watermark.image?)

CSS3 的多列属性:

- `column-count`：指定了需要分割的列数；
- `column-gap`：指定了列与列间的间隙；
- `column-rule-style`：指定了列与列间的边框样式；
- `column-rule-width`：指定了两列的边框厚度；
- `column-rule-color`：指定了两列的边框颜色；
- `column-rule`：是 column-rule-\* 所有属性的简写；
- `column-span`：指定元素跨越多少列；
- `column-width`：指定了列的宽度。

### 2. 实现思路

瀑布流实现思路如下：

- 通过 CSS `column-count` 分割内容为指定列；
- 通过 CSS `break-inside` 保证每个子元素渲染完再换行；

### 3. 实现代码

```css
.css-column {
  column-count: 4; //分为4列
}

.css-column div {
  break-inside: avoid; // 保证每个子元素渲染完在换行
}
```

### 4. 直接使用 npm 包

[npm - react-masonry-component2](https://www.npmjs.com/package/react-masonry-component2) 的使用方法：

```tsx
import { Masonry } from "react-masonry-component2";

export const MyComponent = (args) => {
  return (
    <Masonry direction="column">
      <div></div>
      <div></div>
      <div></div>
    </Masonry>
  );
};
```

在线预览：<https://632339a3ed0b247d36b0fa3c-njrsmzdcdj.chromatic.com/?path=/story/%E5%B8%83%E5%B1%80-masonry-%E7%80%91%E5%B8%83%E6%B5%81--%E7%BA%B5%E5%90%91%E5%B8%83%E5%B1%80>

## 纵向+高度排序+根据宽度自适应列数

在纵向+高度排序的基础上，按照宽度自适应列数。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb268b95e34f4c5d8672ee4d2e4573d8~tplv-k3u1fbpfcp-watermark.image?)

### 1. 实现思路

- 监听 resize 方法，根据屏幕宽度得到该宽度下应该展示的列数

### 2. 实现代码

```ts
import { useCallback, useEffect, useMemo, useState } from "react";

import { DEFAULT_COLUMNS_COUNT } from "../const";

export const useHasMounted = () => {
  const [hasMounted, setHasMounted] = useState(false);
  useEffect(() => {
    setHasMounted(true);
  }, []);
  return hasMounted;
};

export const useWindowWidth = () => {
  const hasMounted = useHasMounted();
  const [width, setWidth] = useState(0);

  const handleResize = useCallback(() => {
    if (!hasMounted) return;
    setWidth(window.innerWidth);
  }, [hasMounted]);

  useEffect(() => {
    if (hasMounted) {
      window.addEventListener("resize", handleResize);
      handleResize();
      return () => window.removeEventListener("resize", handleResize);
    }
  }, [hasMounted, handleResize]);

  return width;
};

export const useColumnCount = (columnsCountBreakPoints: {
  [props: number]: number;
}) => {
  const windowWidth = useWindowWidth();
  const columnCount = useMemo(() => {
    const breakPoints = (
      Object.keys(columnsCountBreakPoints as any) as unknown as number[]
    ).sort((a: number, b: number) => a - b);
    let count =
      breakPoints.length > 0
        ? columnsCountBreakPoints![breakPoints[0]]
        : DEFAULT_COLUMNS_COUNT;

    breakPoints.forEach((breakPoint) => {
      if (breakPoint < windowWidth) {
        count = columnsCountBreakPoints![breakPoint];
      }
    });

    return count;
  }, [windowWidth, columnsCountBreakPoints]);

  return columnCount;
};
```

动态定义 `style columnCount`，实现根据屏幕宽度自适应列数：

```tsx
const { columnsCountBreakPoints } = props;
const columnCount = useColumnCount(columnsCountBreakPoints);
return (
  <div className={classNames(["masonry-column-wrap"])} style={{ columnCount }}>
    {children}
  </div>
);
```

### 3. 直接使用 npm 包

[npm - react-masonry-component2](https://www.npmjs.com/package/react-masonry-component2) 的使用方法：

```tsx
import { Masonry } from "react-masonry-component2";

export const MyComponent = (args) => {
  return (
    <Masonry
      direction="column"
      columnsCountBreakPoints={{
        1400: 5,
        1000: 4,
        700: 3,
      }}
    >
      <div></div>
      <div></div>
      <div></div>
    </Masonry>
  );
};
```

在线预览：<https://632339a3ed0b247d36b0fa3c-njrsmzdcdj.chromatic.com/?path=/story/%E5%B8%83%E5%B1%80-masonry-%E7%80%91%E5%B8%83%E6%B5%81--%E7%BA%B5%E5%90%91%E5%B8%83%E5%B1%80>

## 横向

横向瀑布流指的是，每列按照横向排列，如下图所示。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74c452c957b342c0b8dbb966d3fc7846~tplv-k3u1fbpfcp-watermark.image?)

实现横向瀑布流的方法是**CSS 弹性布局**。

### 1. 弹性布局介绍

弹性布局，是一种当页面需要适应不同的屏幕大小以及设备类型时确保元素拥有恰当的行为的布局方式。

引入弹性盒布局模型的目的是提供一种更加有效的方式来对一个容器中的子元素进行排列、对齐和分配空白空间。

CSS3 的弹性布局属性：

- `flex-dicreation`：指定了弹性子元素的排列方式；
- `justify-content`：指定了弹性布局的主轴对齐方式；
- `align-items`：指定了弹性布局的侧轴对齐方式；
- `flex-wrap`：指定了弹性子元素的换行方式；
- `align-content`：指定弹性布局各行的对齐方式；
- `order`：指定弹性子元素的排列顺序；
- `align-self`：指定弹性子元素的纵向对齐方式；
- `flex`  属性用于指定弹性子元素如何分配空间；
  - `auto`: 计算值为 1 1 auto
  - `initial`: 计算值为 0 1 auto
  - `none`：计算值为 0 0 auto
  - `inherit`：从父元素继承
  - `[ flex-grow ]`：定义弹性盒子元素的扩展比率。
  - `[ flex-shrink ]`：定义弹性盒子元素的收缩比率。
  - `[ flex-basis ]`：定义弹性盒子元素的默认基准值。

### 2. 实现思路

瀑布流实现思路如下：

- CSS 弹性布局对 4 列按横向排列，对每一列内部按纵向排列。

### 3. 实现代码

瀑布流实现代码如下：

```tsx
<div className={classNames(["masonry-flex-wrap"])}>
  <div className="masonry-flex-wrap-column">
    <div></div>
    <div></div>
    <div></div>
    <div></div>
  </div>
  <div className="masonry-flex-wrap-column">
    <div></div>
    <div></div>
    <div></div>
    <div></div>
  </div>
</div>
```

```css
.masonry-flex-wrap {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-content: stretch;

  &-column {
    display: "flex";
    flex-direction: "column";
    justify-content: "flex-start";
    align-content: "stretch";
    flex: 1;
  }
}
```

### 4. 直接使用 npm 包

[npm - react-masonry-component2](https://www.npmjs.com/package/react-masonry-component2) 的使用方法：

```tsx
import { Masonry } from "react-masonry-component2";

export const MyComponent = (args) => {
  return (
    <Masonry
      columnsCountBreakPoints={{
        1400: 5,
        1000: 4,
        700: 3,
      }}
    >
      <div></div>
      <div></div>
      <div></div>
    </Masonry>
  );
};
```

在线预览：<https://632339a3ed0b247d36b0fa3c-njrsmzdcdj.chromatic.com/?path=/story/%E5%B8%83%E5%B1%80-masonry-%E7%80%91%E5%B8%83%E6%B5%81--%E6%A8%AA%E5%90%91%E5%B8%83%E5%B1%80>

## 横向+高度排序

横向+高度排序指的是，每列按照横向排列，往高度最小的列添加内容，如下图所示。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17f765ac5082442299f1617b0cb7d963~tplv-k3u1fbpfcp-watermark.image?)

高度排序就需要用 JS 逻辑来做了。

### 1. 实现思路

- JS 将瀑布流的列表按高度均为分为指定列数，比如瀑布流为 4 列，那么就要把瀑布流列表分成 4 个列表

### 2. 实现代码

```ts
export const getColumnsSortWithHeight = (
  children: React.ReactNode,
  columnCount: number
) => {
  const columns: {
    height: number;
    children: React.ReactNode[];
  }[] = Array.from({ length: columnCount }, () => ({
    height: 0,
    children: [],
  }));

  React.Children.forEach(children, (child: React.ReactNode, index) => {
    if (child && React.isValidElement(child)) {
      if (index < columns.length) {
        columns[index % columnCount].children.push(child);
        columns[index % columnCount].height += child.props.height;
        return;
      }

      const minHeightColumn = minBy(columns, (a) => a.height) as {
        height: number;
        children: React.ReactNode[];
      };
      minHeightColumn.children.push(child);
      minHeightColumn.height += child.props.height;
    }
  });

  return columns;
};
```

### 3. 直接使用 npm 包

[npm - react-masonry-component2](https://www.npmjs.com/package/react-masonry-component2) 的使用方法：

```tsx
import { Masonry, MasonryItem } from "react-masonry-component2";

export const MyComponent = (args) => {
  return (
    <Masonry
      sortWithHeight
      columnsCountBreakPoints={{
        1400: 5,
        1000: 4,
        700: 3,
      }}
    >
      <MasonryItem height={200}>
        <div></div>
      </MasonryItem>
      <MasonryItem height={300}>
        <div></div>
      </MasonryItem>
      <MasonryItem height={400}>
        <div></div>
      </MasonryItem>
    </Masonry>
  );
};
```

在线预览：<https://632339a3ed0b247d36b0fa3c-njrsmzdcdj.chromatic.com/?path=/story/%E5%B8%83%E5%B1%80-masonry-%E7%80%91%E5%B8%83%E6%B5%81--%E6%A8%AA%E5%90%91%E5%B8%83%E5%B1%80%E9%AB%98%E5%BA%A6%E6%8E%92%E5%BA%8F>

## 横向+高度排序+根据宽度自适应列数

根据宽度自适应列数的做法和纵向场景一致，都是监听 resize 方法，根据屏幕宽度得到该宽度下应该展示的列数，这里不做赘述。

### 1. 直接使用 npm 包

[npm - react-masonry-component2](https://www.npmjs.com/package/react-masonry-component2) 的使用方法：

```tsx
import { Masonry } from "react-masonry-component2";

export const MyComponent = (args) => {
  return (
    <Masonry
      sortWithHeight
      direction="column"
      columnsCountBreakPoints={{
        1400: 5,
        1000: 4,
        700: 3,
      }}
    >
      <div></div>
      <div></div>
      <div></div>
    </Masonry>
  );
};
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9e8f7a49e38457fa43f72cfe214eb50~tplv-k3u1fbpfcp-watermark.image?)

在线预览：<https://632339a3ed0b247d36b0fa3c-njrsmzdcdj.chromatic.com/?path=/story/%E5%B8%83%E5%B1%80-masonry-%E7%80%91%E5%B8%83%E6%B5%81--%E6%A8%AA%E5%90%91%E5%B8%83%E5%B1%80%E9%AB%98%E5%BA%A6%E6%8E%92%E5%BA%8F>

## 小结

本文介绍了 5 种瀑布流场景的实现：

- 纵向+高度排序
- 纵向+高度排序+根据宽度自适应列数
- 横向
- 横向+高度排序
- 横向+高度排序+根据宽度自适应列数

感兴趣的同学可以到[项目源码](https://github.com/jiaozitang/react-masonry-component2)查看完整实现代码。

也可以下载 <https://www.npmjs.com/package/react-masonry-component2> 直接使用。

希望能对你有所帮助，感谢阅读～

别忘了点个赞鼓励一下我哦，笔芯 ❤️

---
