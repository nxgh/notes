#less

[[Css 颜色]]

## Less 循环

```less
.cm-header(@i) when( @i <= 5) {
  .cm-header-@{i} {
    @var: ~'var(--h@{i}-color)';
    --header-color: @var;
  }
  .cm-header((@i + 1));
}

.cm-header(1);
```

## less 函数
```less
@primary-color: hsla(209, 100%, 55%, 1);
```
### saturate & desatruate
改变元素中颜色饱和度
```less
@primary-color: hsla(209, 50%, 55%, 50%);
div {
  --color-saturate-10: saturate(@primary-color, 10%);
  --color-saturate-20: saturate(@primary-color, 20%);
  --color-saturate-30: saturate(@primary-color, 30%);
  --color-desaturate-10: desaturate(@primary-color, 10%);
  --color-desaturate-20: desaturate(@primary-color, 20%);
  --color-desaturate-30: desaturate(@primary-color, 30%);
}
```
编译后
```css
div {
  --color-saturate-10: hsla(209, 60%, 55%, 0.5);
  --color-saturate-20: hsla(209, 70%, 55%, 0.5);
  --color-saturate-30: hsla(209, 80%, 55%, 0.5);
  --color-desaturate-10: hsla(209, 40%, 55%, 0.5);
  --color-desaturate-20: hsla(209, 30%, 55%, 0.5);
  --color-desaturate-30: hsla(209, 20%, 55%, 0.5);
}
```

### lighten / darken
改变颜色亮度
```less
@primary-color: hsla(209, 50%, 55%, 50%);
div {
  --color-lighten-10: lighten(@primary-color, 10%);
  --color-lighten-20: lighten(@primary-color, 20%);
  --color-lighten-30: lighten(@primary-color, 30%);
  --color-darken-10: darken(@primary-color, 10%);
  --color-darken-20: darken(@primary-color, 20%);
  --color-darken-30: darken(@primary-color, 30%);
}
```
编译后
```css
div {
  --color-lighten-10: hsla(209, 50%, 65%, 0.5);
  --color-lighten-20: hsla(209, 50%, 75%, 0.5);
  --color-lighten-30: hsla(209, 50%, 85%, 0.5);
  --color-darken-10: hsla(209, 50%, 45%, 0.5);
  --color-darken-20: hsla(209, 50%, 35%, 0.5);
  --color-darken-30: hsla(209, 50%, 25%, 0.5);
}
```

### fade / fadein /fadeout
- `fadein()`  降低颜色的透明度
- `fadeout()` 增加颜色透明度
- `fade()` 设置颜色透明度
改变颜色不透明度
```less
@primary-color: hsla(209, 50%, 55%, 50%);
div {
  --color-fade-10: fade(@primary-color, 10%);
  --color-fade-20: fade(@primary-color, 20%);
  --color-fade-30: fade(@primary-color, 30%);
  --color-fadein-10: fadein(@primary-color, 10%);
  --color-fadein-20: fadein(@primary-color, 20%);
  --color-fadein-30: fadein(@primary-color, 30%);
  --color-fadeout-10: fadeout(@primary-color, 10%);
  --color-fadeout-20: fadeout(@primary-color, 20%);
  --color-fadeout-30: fadeout(@primary-color, 30%);
}
```
编译后
```css
div {
  --color-fade-10: hsla(209, 50%, 55%, 0.1);
  --color-fade-20: hsla(209, 50%, 55%, 0.2);
  --color-fade-30: hsla(209, 50%, 55%, 0.3);
  --color-fadein-10: hsla(209, 50%, 55%, 0.6);
  --color-fadein-20: hsla(209, 50%, 55%, 0.7);
  --color-fadein-30: hsla(209, 50%, 55%, 0.8);
  --color-fadeout-10: hsla(209, 50%, 55%, 0.4);
  --color-fadeout-20: hsla(209, 50%, 55%, 0.3);
  --color-fadeout-30: hsla(209, 50%, 55%, 0.2);
}
```
### spin
任意方向旋转颜色的色相角度（hue angle）
```less
@primary-color: hsla(209, 100%, 55%, 50%);
div {
  color: spin(@primary-color, 10);
  color: spin(@primary-color, 20);
  color: spin(@primary-color, 30);
}
```
编译后
```css
div {
  color: hsla(219, 100%, 55%, 0.5);
  color: hsla(229, 100%, 55%, 0.5);
  color: hsla(239, 100%, 55%, 0.5);
}
```
### mix
根据比例混合两种颜色，包括计算不透明度
```less
@primary-color: rgba(119, 189, 255, 0.5);
@primary-color-2: rgba(121, 255, 166, 0.5);
div {
  --color-mix: mix(@primary-color, @primary-color-2);
}
```
```css
div {
  --color-mix: rgba(120, 222, 211, 0.5);
}
```
### tint / shade
- `tint()` 用于混合颜色与白色
- `shade()`  将颜色与黑色混合

```less
@primary-color: rgba(119, 189, 255, 0.5);
div {
  --color-tint-10: tint(@primary-color, 10%);
  --color-tint-20: tint(@primary-color, 20%);
  --color-tint-30: tint(@primary-color, 30%);
  --color-shade-10: shade(@primary-color, 10%);
  --color-shade-20: shade(@primary-color, 20%);
  --color-shade-30: shade(@primary-color, 30%);
}
```
```css
div {
  --color-tint-10: rgba(153, 206, 255, 0.55);
  --color-tint-20: rgba(177, 217, 255, 0.6);
  --color-tint-30: rgba(196, 226, 255, 0.65);
  --color-shade-10: rgba(89, 142, 191, 0.55);
  --color-shade-20: rgba(68, 108, 146, 0.6);
  --color-shade-30: rgba(52, 83, 112, 0.65);
}
```
### greyscale
完全移除颜色的饱和度，与`desaturate(@color,100%)`函数效果相同
### contrast
选择两种颜色相比较，得出哪种颜色的对比度最大就倾向于对比度最大的颜色
```less
@primary-color: rgba(119, 189, 255, 0.5);
@primary-color-2: rgba(121, 255, 166, 0.5);
div {
  --color-contrast: contrast(@primary-color, @primary-color-2);
}
```
编译后
```css
div {
  --color-contrast: rgba(121, 255, 166, 0.5);
}
```
