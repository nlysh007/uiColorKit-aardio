# uiColorKit

`uiColorKit` 是面向 aardio `plus` 控件的颜色工具库，提供常用 CSS Color 4 格式处理、颜色选择器和弹出式颜色输入控件。所有公开颜色数值统一使用 GDI+ 的 `0xAARRGGBB`。

## 模块

| 模块 | 职责 |
|---|---|
| `uiColorKit.format` | CSS、HEX、aardio 颜色解析、格式化与色彩空间转换 |
| `uiColorKit.picker` | SV、色相、透明度及多格式字段选择器 |
| `uiColorKit.input` | 绑定 plus 控件，点击后弹出共享 picker 的颜色输入控件；支持默认绘制或 draw=false 自定义绘制 |

顶层 `uiColorKit` 只汇总以上三个子模块，不重复导出子模块函数。
## 安装扩展库

```aardio
import ide
ide.installLib("uiColorKit","https://github.com/nlysh007/uiColorKit-aardio/releases/latest/download/uiColorKit.tar.lzma")
```
## 快速开始

```aardio
import win.ui;
import uiColorKit;

var winform = win.form(text="uiColorKit";right=560;bottom=400)
winform.add(
    picker={cls="plus";left=20;top=20;right=320;bottom=380;z=1};
    colorInput={cls="plus";left=340;top=20;right=540;bottom=60;z=2}
)

var color = uiColorKit.format.parse("oklch(0.628 0.258 29.2 / 80%)");
var picker = uiColorKit.picker.create(winform.picker,{
    color=color;
    colorMode="oklch";
    showAlpha=true;
    theme="dark";
});
var colorInput = uiColorKit.input.create(winform.colorInput,{
    color=color;
    showAlpha=true;
    theme="dark";
/*
    // draw=false 不绘制默认 InputView 外观；onDraw 可绘制自定义外观
    draw=false;
    onDraw = function(graphics,rc,view){
        var brush = gdip.solidBrush(view.getColor());
        graphics.fillRoundRect(brush,0,0,rc.width,rc.height,8);
        brush.delete();
    }
*/
});

picker.onChange = function(view,color){
    colorInput.setColor(color,true);
};
colorInput.onChange = function(view,color){
    picker.setColor(color,true);
};

winform.onDestroy = function(){
    picker.dispose();
    colorInput.dispose();
};

winform.show();
win.loopMessage();
```

`silent=true` 的设置方法不触发 `onChange`，适合多个控件之间同步。

## Format API

`uiColorKit.format` 是纯逻辑模块，不依赖窗口。

### 解析与输出

| API | 说明 |
|---|---|
| `parse(value)` | 解析数值颜色、CSS、HEX 或 aardio 颜色字符串；失败返回 `null` |
| `toHex(color, withAlpha)` | 输出 `#RRGGBB` 或 `#RRGGBBAA` |
| `parseAardio(value)` | 解析 `0xRRGGBB` 或 `0xAARRGGBB` 字符串 |
| `toAardio(color)` | 输出 `0xRRGGBB`；存在透明度时输出 `0xAARRGGBB` |
| `toCss(color, space, withAlpha)` | 输出指定色彩空间的浏览器兼容 CSS；未知空间回退 HEX |

`parse` 支持：

- `#RGB`、`#RGBA`、`#RRGGBB`、`#RRGGBBAA`。
- 传统及现代 `rgb()`、`rgba()`、`hsl()`、`hsla()`，包括 `/ alpha`。
- `oklab()`、`oklch()`、`lab()`、`lch()`。
- `transparent`。
- `0xRRGGBB`、`0xAARRGGBB` 及数值颜色。

CSS 八位 HEX 是 `RRGGBBAA`，aardio 八位颜色是 `AARRGGBB`，两者通道顺序不同。非法文本、分量缺失、多余分量或重复 alpha 均返回 `null`。

```aardio
import uiColorKit;

var format = uiColorKit.format;
var color = format.parse("rgb(51 102 153 / 50%)");
var hex = format.toHex(color,true);          // #33669980
var css = format.toCss(color,"oklch");
var value = format.toAardio(color);          // 0x80336699
var r,g,b,a = format.getRgb(color);          // alpha 为 0..1
```

### 构造与拆分

构造函数：`rgb`、`hsl`、`hsv`、`hsb`、`cmyk`、`oklab`、`oklch`、`lab`、`lch`。

拆分函数：`getRgb`、`getHsl`、`getHsv`、`getCmyk`、`getOklab`、`getOklch`、`getLab`、`getLch`。

其他函数：`getAlpha`、`setAlpha`、`getRgbValue`、`fmtTrim`、`fmtPct`。输出精度配置公开为 `format.PREC`。

构造函数的 alpha 范围是 `0..255`；`get*` 返回的 alpha 与 `setAlpha` 参数范围是 `0..1`。

## Picker API

### 创建

- `uiColorKit.picker.create(ctrl, options)`：绑定 `plus` 控件并创建可视选择器。
- `uiColorKit.picker.createModel(options)`：创建无控件模型。
- `uiColorKit.picker.themes`：内置 `dark`、`light` 主题表。
- `uiColorKit.picker.modes`：`hex`、`aardio`、`rgb`、`hsl`、`oklab`、`oklch`、`lab`、`lch`。

`hex` 模式使用 CSS HEX；`aardio` 模式使用 `0xRRGGBB` 或 `0xAARRGGBB`。

### 选项

| 字段 | 默认值 | 说明 |
|---|---:|---|
| `color` | `0xFF3B82F6` | 初始数值颜色或 CSS 字符串 |
| `colorMode` | `"hex"` | 初始字段模式 |
| `showAlpha` | `true` | 是否显示透明度条与字段 |
| `showHue` | `true` | 是否显示色相条 |
| `showInfo` | `true` | 是否显示颜色字段 |
| `theme` | `"dark"` | `dark`、`light` 或自定义主题表 |
| `radius` | `10` | 面板圆角半径 |
| `pad` | `10` | 内容边距 |
| `gap` | `10` | 控件间距 |
| `hueWidth` | `16` | 色相条宽度 |
| `barHeight` | `16` | 透明度条高度 |
| `fontName` | 系统默认 | 字段字体名称 |
| `fontSize` | `12` | 字段字体大小 |

### 方法

| 方法 | 说明 |
|---|---|
| `getColor()` / `setColor(color, silent)` | 读取或设置 `0xAARRGGBB` |
| `getHex()` / `setHex(hex, silent)` | 读取或设置 CSS HEX |
| `getRgb()` | 返回 `red, green, blue` |
| `getHsv()` / `setHsv(h, s, v, silent)` | 读取或设置 HSV |
| `getAlpha()` / `setAlpha(alpha, silent)` | 读取或设置 `0..1` 透明度 |
| `getColorText()` | 按当前模式输出完整颜色文本 |
| `copyColor()` / `copyHex()` | 复制颜色文本到剪贴板 |
| `getColorMode()` / `setColorMode(mode)` | 读取或切换模式 |
| `cycleColorMode(back)` | 循环模式，`back=true` 时反向 |
| `beginFieldEdit(field)` / `endFieldEdit(apply)` | 开始或结束字段编辑 |
| `isEditing()` | 是否正在编辑字段 |
| `setTheme(theme)` / `setOptions(options)` | 更新主题或批量选项 |
| `refresh()` / `redraw()` | 刷新字段与重绘控件 |
| `dispose()` | 释放编辑框、字体及绘图资源 |

### 事件

- `onChange(view,color)`：颜色变化。
- `onInput(view,region,...)`：拖拽中的分量变化。
- `onCommit(view,color)`：拖拽结束或字段应用。
- `onCopy(view,text,mode)`：复制颜色文本后通知。
- `onModeChange(view,mode)`：模式切换后通知。

无控件模型同样应在结束时调用 `dispose()`。

## Input API

### 创建

- `uiColorKit.input.create(ctrl, options)`：绑定一个 `plus` 控件并返回输入视图。
- `uiColorKit.input.createModel(options)`：创建不绑定控件的输入模型；模型不能打开弹窗，但可用于颜色状态与格式化逻辑。
- `uiColorKit.input.themes`：内置 `dark`、`light` 主题表。

`input` 只支持 `plus` 控件，不适配 `button` 等普通 ctrl。可视 input 使用手型鼠标；单击或按 Enter/空格打开选择器，Esc 取消，弹窗失活或正常关闭时提交。多个 input 共享一个弹窗，每次打开都会同步当前视图的颜色、主题、透明度和颜色模式。弹窗按输入控件所在显示器的工作区定位，下方空间不足时自动显示在字段上方。

### 时序与关闭语义

Input 使用非模态共享弹窗。弹窗打开后，拖拽或字段编辑通过 `onChange(view,color)` 即时通知；输入视图会同步当前颜色，但本次会话仍可取消。

- `close()` 或 `close(true)`：提交待选颜色，依次触发 `onCommit(view,color)` 和 `onClose(view,color)`。
- `close(false)` 或按 Esc：取消本次提交通知，不触发 `onCommit`，但仍触发 `onClose(view,color)`；颜色在 `onChange` 时已同步到视图，不会自动回滚到打开前的值。
- 点击弹窗外部、弹窗失活、再次点击当前输入控件，通常都会按提交方式关闭。
- `onOpen(view,color)` 在弹窗显示并完成颜色同步后触发；`onClose` 每次关闭都会触发。

组件内部用 `time.tick()` 保护窗口显示、隐藏和焦点切换产生的重复事件：打开后的约 100 ms 内忽略弹窗失活，打开后的约 350 ms 内忽略宿主窗体点击，关闭后的约 300 ms 内忽略重复鼠标或键盘触发。这些是内部防抖窗口，不是可配置的超时 API，也不应作为精确计时保证。

### 选项

| 字段 | 默认值 | 说明 |
|---|---:|---|
| `color` | `0xFF3B82F6` | 初始颜色，可指定 `0xAARRGGBB` 或 CSS 颜色字符串 |
| `theme` | `"dark"` | `dark`、`light` 或自定义主题表 |
| `showAlpha` | `true` | 是否允许透明度；设为 `false` 时颜色强制不透明 |
| `colorMode` | `"hex"` | 弹出 picker 的初始字段模式 |
| `radius` | `8` | 输入控件圆角半径 |
| `fontName` | 系统默认 | 输入文字字体 |
| `fontSize` | `13` | 输入文字大小 |
| `popupWidth` | `320` | 弹窗宽度 |
| `popupHeight` | `380` | 弹窗高度 |
| `draw` | `true` | 是否绘制默认 InputView 外观 |
| `noDraw` | — | `draw=false` 的兼容别名 |

### 方法

| 方法 | 说明 |
|---|---|
| `attach(ctrl)` | 将模型绑定到 `plus` 控件并返回自身 |
| `getColor()` | 获取当前 `0xAARRGGBB` 颜色 |
| `setColor(color, silent)` | 设置颜色；支持数值或 CSS 字符串，`silent=true` 不触发 `onChange` |
| `getHex()` | 获取 CSS HEX；透明度启用且不透明度不足时包含 alpha |
| `setHex(hex, silent)` | 以 CSS HEX 设置颜色 |
| `getRgb()` | 获取 `red, green, blue` |
| `setTheme(theme)` | 设置主题并刷新输入控件 |
| `setOptions(options)` | 批量更新颜色、主题、模式、尺寸及绘制选项 |
| `open()` | 打开共享颜色选择弹窗 |
| `close(commit)` | 关闭弹窗；省略或 `true` 提交，`false` 取消 |
| `isOpen()` | 判断当前视图是否打开弹窗 |
| `dispose()` | 取消并关闭弹窗、解除控件事件、释放资源 |

### 绘制

- `draw=true`：组件接管默认 InputView 绘制，并设置交互所需的 plus 属性。
- `draw=false`：保留点击、键盘、颜色同步和提交行为，不绘制默认外观，也不修改 plus 现有文字与样式。
- `onDraw(graphics,rc,view)`：仅在 `draw=false` 时调用的自定义绘制回调；回调创建的 GDI+ 资源由调用方负责释放。
- `setOptions({onDraw=...})`：动态替换自定义绘制回调。

### 事件

- `onChange(view,color)`：颜色即时变化。
- `onCommit(view,color)`：关闭时提交。
- `onOpen(view,color)`：弹窗打开并完成颜色同步后触发。
- `onClose(view,color)`：弹窗关闭时触发，无论提交还是取消都会触发。

窗口销毁时应调用每个可视输入视图的 `dispose()`；最后一个视图释放后共享弹窗与 picker 缓存会销毁。

## 主题与资源

- `theme` 可传 `"dark"`、`"light"` 或自定义主题表。
- 自定义主题会从基础主题合并，未指定字段自动继承。
- `showAlpha=false` 时颜色强制为不透明。
- 窗口销毁时必须调用可视 picker 和 input 的 `dispose()`。


## 致谢

- OKLab / OKLCH：[Björn Ottosson — Oklab](https://bottosson.github.io/posts/oklab/)
- 语法与区间：[CSS Color Module Level 4](https://www.w3.org/TR/css-color-4/)
- 交互语义：Chrome DevTools / Figma 取色器
