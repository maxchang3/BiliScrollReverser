<p align="center">
  <img src="./assets/logo.gif" width="50%">
</p>

# 哔哩哔哩触控板滚动反转
# BiliScrollReverser

> 自然滚动下，音量调节也应该符合直觉。

优化 B 站视频音量调节在触控板上的体验，同时保留传统鼠标滚轮原有的操作方向。安装脚本后，在 B 站视频全屏界面中，使用触控板向下滚动将降低音量（未安装时为升高），切换回鼠标时则无需改变操作习惯。

## 背景

在触控板上，滚动方向通常遵循「自然滚动」原则：手指向上推动，页面随之展示下方内容，如同推动一张纸。传统鼠标滚轮的操作语义则恰好相反：向上滚动，表示返回页面上方。<sup>*</sup>

浏览网页时，两种逻辑可以自然共存；但音量不是一张可以被推动的「纸」，它通常只有一套与设备无关的方向语义：**向上升高，向下降低**。B 站播放器按照传统鼠标滚轮的方向处理音量，因此换成自然滚动的触控板后，操作方向便会显得颠倒。

如果只使用触控板，直接反转播放器接收到的所有滚动事件就足够了。真正的问题出现在鼠标与触控板混用时：无条件反转虽然修正了触控板，却会让原本正常的鼠标滚轮变成反向。

因此，本脚本会尝试判断滚动事件来自触控板还是鼠标，仅反转触控板的音量调节方向。由于浏览器没有提供可靠的输入设备识别 API，这种判断只能基于 `WheelEvent` 的数值特征进行推测。脚本提供「简单」与「矫正」两种策略，以适应不同的系统、浏览器和硬件环境。

<sub>* 本文假设触控板采用「自然滚动」，传统鼠标滚轮采用「非自然滚动」。macOS 默认会统一设置二者，不少用户会借助 Scroll Reverser 或鼠标厂商提供的驱动分别配置。Magic Mouse 的交互方式更接近触控板，不属于这里所说的传统滚轮鼠标。</sub>

## 安装

[[Greasyfork](https://greasyfork.org/zh-CN/scripts/432783)]
[[Github Release](https://github.com/MaxChang3/BiliScrollReverser/releases/latest/download/BiliScrollReverser.user.js)]
[[Github Pages](https://maxchang3.github.io/BiliScrollReverser/BiliScrollReverser.user.js)]

## 使用建议

初始化时提供两个选项：**「简单」** 与 **「矫正」**。

### 简单模式
粗暴地认为 `deltaY` 值低于 `100` 的为触控板。仅对于细微调节适用。移动过快时，会导致判断中间出错。

### 矫正模式
根据提示滚动鼠标滚轮选择最小整数 `deltaY`。除去这个数值外，如果是浮点数则为鼠标，整数值则为触控板。

**对于 macOS（Chrome/Safari），推荐使用「矫正模式」。（或如果你发现你的设备表现类似于上述情况）**

**对于未开启平滑滚动的情况下的 Windows 设备、所有平台的 Firefox 浏览器下，目前暂时推荐使用「简单模式」。** 

> [!WARNING]
> 网页缩放时，deltaY 的表现可能会有所不同，建议在 100% 缩放下进行矫正与使用。（后面会优化）

## 实现原理

Hook `EventTarget.prototype.addEventListener` 拦截对应的 `mousewheel` 事件。~~（为什么不用 `wheel`！）~~

判断是否为触控板，添加代理拦截 [wheelDelta](https://developer.mozilla.org/en-US/docs/Web/API/Element/mousewheel_event) 值，取相反数（这里直接取 [deltaY](https://developer.mozilla.org/en-US/docs/Web/API/WheelEvent/deltaY) 后做一定计算处理，他与 `wheelDelta` 正负相异）后返回。


## 触控板判断逻辑

> [!WARNING]
> 由于硬件、浏览器差异，可能会有完全不同的表现。如有问题，欢迎反馈！
> 
> 目前的逻辑是判断出鼠标，反过来得知触控板，鼠标测试硬件均为 Logitech MX Anywhere2s。

### macOS <sup>[1]</sup><sup>[2]</sup>

1. 鼠标滚轮推动下的 `deltaY` 存在一个**最小整数值**，除去这个值之外的其他值大概率为浮点数，并且小数点后较为复杂。形如：`235.867919921875`。

2. 触控板大部分情况下为整数，存在为浮点数的触控板，但是一般也不会很复杂。形如：`2.5`。

因此，采取人工矫正的方式，使用鼠标的最低刻度推动获取一个最低的整数 `deltaY`。除去这个值外的所有数值，都根据是否为整数<sup>[3]</sup>进行判断，是则为触控板，否则为鼠标。目前经过一段时间测试，效果良好。


### Windows<sup>[4]</sup>

1. 未开启平滑滚动的情况下：通过不同的力度，仅能得到几个整数值。最低为 100，高至 600。

2. 开启平滑滚动的情况下：数值为分布在 1 左右的浮点数。

因此，对于 Windows 目前推荐使用简单模式。



<sub>[1] 仅 Chrome / Safari，Firefox 下全部为整数值。</sub>

<sub>[2] 仅有限测试于 macOS 13.2.1（MacBook Air M1）。</sub>

<sub>[3] 由于 2 的原因，对 `deltaY` 做乘 2 处理避免这种情况。目前测试设备有限，未来可能会有所变动。</sub>

<sub>[4] 仅有限测试于 Windows 11（LEGION R7000）。</sub>
