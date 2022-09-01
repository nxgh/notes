## backdrop-filter 和 filter 

### 语法
- `backdrop-filter` 和 `filter` 语法一模一样
- `backdrop-filter`是让当前元素所在区域后面的内容模糊灰度或高亮
- `filter` 是让当前元素自身模糊灰度或高亮
```css
backdrop-filter: none;
/* URL方式外链SVG filter */
backdrop-filter: url(zxx.svg#filter);
/* <filter-function>值 */
backdrop-filter: blur(2px);
backdrop-filter: brightness(60%);
backdrop-filter: contrast(40%);
backdrop-filter: drop-shadow(4px 4px 10px blue);
backdrop-filter: grayscale(30%);
backdrop-filter: hue-rotate(120deg);
backdrop-filter: invert(70%);
backdrop-filter: opacity(20%);
backdrop-filter: sepia(90%);
backdrop-filter: saturate(80%);
```

### 滤镜

| 滤镜          | 释义   |
|-------------|------|
| blur        | 模糊   |
| brightness  | 亮度   |
| contrast    | 对比度  |
| drop-shadow | 投影   |
| grayscale   | 灰度   |
| hue-rotate  | 色调变化 |
| invert      | 反相   |
| opacity     | 透明度  |
| saturate    | 饱和度  |
| sepia       | 褐色   |


`filter` 和 `backdrop-filter` 有性能问题，使用`transform: translate3d(0, 0, 0)`;开启硬件加速后，会有改善