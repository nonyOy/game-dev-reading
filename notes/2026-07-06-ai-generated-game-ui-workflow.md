# 2026-07-06：AI 生图做游戏 UI 的使用方法、注意点和制作流程

今天这篇不是分析某个具体游戏，而是整理一个很实际的问题：怎么用 AI 生图辅助游戏 UI 美术，以及为什么现在直接让 AI 出图，经常达不到出品级别。

我现在遇到的核心问题是：AI 很容易生成“看起来像游戏 UI 的图”，但它不理解游戏 UI 资源真正要怎么用。比如让它生成血条，它经常会直接给一张完整的血条成品图：有框、有红色填充、有高光、有数字、有装饰，甚至还带背景。但游戏里真正需要的不是这一整张图，而是一套可以拆分、拉伸、遮罩、换状态、随数值变化的组件。

这就是 AI 生图做游戏 UI 最大的误区：

```text
AI 擅长生成视觉印象。
游戏 UI 需要的是可实现的资源系统。
```

如果把 AI 当成“最终出图工具”，结果通常会很不稳定；如果把 AI 当成“风格探索、造型草案、纹理灵感、局部装饰生成工具”，它会有用很多。

## 一、游戏 UI 不是一张图，而是一组可工作的零件

很多 AI 图像模型理解的 UI，更接近“UI 概念图”或“游戏截图”。它会把所有东西混在一张画面里，因为从视觉上这样最完整。

但游戏项目里，UI 需要拆成组件。

以血条为例，一条真正能进游戏的血条通常不应该只有一张整图，而应该至少考虑这些层：

```text
血条底板：血量为空时的背景槽。
外框：包住血条的装饰边框，可以保持不变。
血量填充：随血量比例横向缩放或裁切。
延迟血条：受伤后慢慢减少的第二层颜色。
高光：让填充有体积感，但要能跟随填充变化。
遮罩：控制填充区域形状，避免超出边框。
左右端盖：如果血条有特殊端头，不能被横向拉变形。
中段拉伸区：可以平铺或九宫格拉伸的区域。
刻度线：如果需要显示分段血量，最好单独做。
状态特效：中毒、护盾、低血量、无敌、破防等状态可以叠加。
数值文字：通常不应该烘焙在图片里，而应该由字体系统渲染。
头像或图标位：如果血条旁边有角色头像，要独立于血条本体。
```

AI 一次性生成的“完整血条”往往没有这些层。它只是画了一个像血条的东西。看图可以，进游戏就会出问题。

常见问题包括：

- 血量填充和外框粘在一起，不能单独裁切。
- 左右端头被画死，横向缩放会变形。
- 高光和阴影跟外框混在一起，无法随血量变化。
- 文字、数字、图标被烘焙在图里，后期不能改。
- 背景不透明，抠图成本高。
- 细节太碎，小尺寸下看不清。
- 透视角度不正，无法作为平面 UI 资源使用。
- 同一套 UI 的按钮、面板、血条风格不一致。

所以第一条原则是：不要向 AI 要“一张游戏 UI 图”，而是要“某个 UI 组件的视觉方案”，然后自己拆成可用资源。

## 二、AI 生图适合做什么，不适合做什么

AI 适合做的事情：

```text
探索风格方向
生成 moodboard 灵感
尝试不同材质和装饰语言
快速比较几种按钮、面板、血条造型
生成纹理、高光、边框、角花等局部素材
给图标提供轮廓和气质参考
帮助突破空白画布
```

AI 不适合直接负责的事情：

```text
最终切图
精确九宫格
严格像素对齐
多状态组件统一
多语言文本排版
图标语义准确性
按钮点击热区
血条填充逻辑
Unity/Unreal/Cocos 中的适配
整套 UI 系统的一致性
```

换句话说，AI 可以帮你找“这套 UI 长什么样”，但不能替你决定“这套 UI 怎么工作”。

真正的出品级 UI，必须经过人工整理、拆分、重绘、规范化和引擎测试。

## 三、提示词要从“画面描述”改成“资源描述”

如果提示词是：

```text
一个很酷的幻想游戏血条，红色，金属边框，华丽，高质量
```

AI 很可能会生成一张完整 HUD 截图，甚至带角色、背景、文字和杂乱装饰。

更好的思路是把提示词改成资源生产语言。

比如血条，可以这样写：

```text
fantasy RPG health bar UI component, front view, flat 2D game asset, separated frame and empty inner slot, no numbers, no text, no character, no background, centered on transparent background, clean silhouette, readable at small size, ornate but simple, gold metal frame, dark empty groove, horizontal layout, production game UI asset
```

如果模型不支持透明背景，也要尽量要求：

```text
plain solid background, easy to cut out, no shadows outside the asset, no full HUD screenshot
```

还可以加负面要求：

```text
no complete screen mockup, no in-game screenshot, no text, no numbers, no player avatar, no background scene, no perspective, no random icons, no filled red bar merged with frame
```

这个变化很重要。

以前是在要求 AI “画一个好看的血条”。现在是在要求它“给我一个可以拆成游戏资产的血条组件”。两者差别很大。

不过也要承认：即使提示词写得很清楚，AI 仍然不一定真的给你分层文件。它最多让画面更接近可拆分状态。真正分层还是要靠自己做。

## 四、血条应该怎么让 AI 辅助制作

以血条为例，我觉得比较靠谱的流程不是让 AI 直接出最终图，而是这样：

```text
1. 先画结构草图
2. 列出需要的层
3. 让 AI 生成 6 到 12 个风格方向
4. 选其中 1 到 2 个作为参考
5. 人工重绘成干净结构
6. 拆成外框、底槽、填充、高光、端盖、遮罩等资源
7. 放进引擎里测试血量从 0% 到 100% 的变化
8. 回到 PSD/Figma/PS 调整切图
```

血条的资源拆分可以这样设计：

```text
hp_bar_frame.png
  不随血量变化，负责外框和装饰。

hp_bar_empty.png
  空槽底色，永远显示在最下面。

hp_bar_fill.png
  红色血量填充，可以横向裁切或缩放。

hp_bar_fill_highlight.png
  填充高光，可以跟着 fill 一起被遮罩裁切。

hp_bar_damage_delay.png
  受伤延迟层，通常是黄色、橙色或白色。

hp_bar_mask.png
  控制填充形状，尤其是不规则边缘时需要。

hp_bar_left_cap.png
hp_bar_right_cap.png
hp_bar_center.png
  如果需要可变长度，左右端盖和中段最好拆开。

hp_bar_tick.png
  分段刻度线，最好单独叠加。

hp_bar_low_hp_glow.png
  低血量闪烁或边缘警告特效。
```

这样做的好处是，血条不是一张死图，而是一个可以被程序控制的组件。

如果只做一整张图，就会遇到这些问题：

```text
血量减少时，红色填充不能自然变短。
血条变长时，左右装饰被拉扁。
低血量状态只能重新换整图。
受伤延迟效果做不了。
不同角色血条长度不同会崩。
```

所以血条这种 UI，不能只追求第一眼漂亮，必须先保证结构正确。

## 五、按钮、面板、图标也要按“组件”思路做

血条的问题，在按钮和面板上也一样存在。

一个按钮通常至少要有这些状态：

```text
normal：普通状态
hover / selected：鼠标悬停或手柄选中
pressed：按下状态
disabled：不可用状态
locked：未解锁状态
new：有新内容提示
```

如果 AI 只生成一个漂亮按钮，没有这些状态，那它还不是完整游戏资源。

一个面板也不能只是一张背景图。要考虑：

```text
是否需要九宫格拉伸？
四个角是否要保持不变？
边框是否能平铺？
中间区域是否能承载大量文本？
标题区和内容区是否分开？
关闭按钮、标签页、滚动条是否同风格？
```

图标也类似。AI 生成图标时经常画得很细、很写实，但游戏图标需要的是小尺寸下能识别。

图标要检查：

```text
32px 能不能看懂？
轮廓是否清楚？
正负形是否干净？
是否和其他图标同角度、同光源、同描边？
是否避免了没意义的小碎片？
是否有透明背景？
是否能做灰态、稀有度框、选中态？
```

所以 AI 生成按钮、面板、图标时，也要先问它要“组件”，不要问它要“截图”。

## 六、出品级 UI 的判断标准

我现在做的 UI 不能出品，问题大概率不是“画得不够华丽”，而是没有经过出品级检查。

一套游戏 UI 至少要过这些标准：

```text
可读性：玩家一眼知道重要信息在哪里。
可操作性：按钮、选中态、禁用态、反馈都清楚。
可拆分：资源能被程序控制，而不是一张死图。
可适配：不同分辨率、语言、数值长度下不崩。
可复用：按钮、面板、标题、图标有统一规则。
可扩展：以后新增功能时能继续沿用这套系统。
可维护：PSD/Figma/工程资源命名清楚，别人能接手。
可实现：动效和切图不是只在概念图里好看。
可测试：放进引擎后在真实尺寸下仍然成立。
```

AI 生图最大的问题，是它通常只满足第一项的一部分：看起来有视觉冲击。

但出品级 UI 是一个系统，不是一张漂亮图。

## 七、AI 生图的正确制作流程

我觉得以后可以固定成这个流程：

### 1. 先写 UI 需求，不先写提示词

先明确这个 UI 是什么功能。

比如血条需求：

```text
用途：玩家主角血量显示
位置：战斗界面左上角
尺寸：约 360x42
变化：血量 0% 到 100%，受伤有延迟条
状态：普通、低血量、中毒、护盾
风格：暗黑幻想，金属边框，红色液体填充
技术：Unity UI，填充层用 Image Filled 或 Mask 裁切
文本：数值文字由字体系统生成，不烘焙进图
```

如果这个需求没写清楚，AI 就会乱发挥。

### 2. 画最低成本结构草图

哪怕画得很丑，也要先画结构。

草图只需要说明：

```text
外框在哪里？
填充在哪里？
文字在哪里？
图标在哪里？
哪些地方可以拉伸？
哪些地方必须保持不变？
```

这个草图比提示词更重要。AI 负责风格，人负责结构。

### 3. 生成风格探索图

这一阶段不要追求最终图，只看方向。

可以一次生成多组：

```text
方案 A：暗黑金属
方案 B：手绘羊皮纸
方案 C：魔法水晶
方案 D：粗厚卡通
方案 E：科幻全息
```

每个方案只选优点，比如：

```text
A 的边框好
B 的纹理好
C 的高光好
D 的轮廓清楚
E 的状态反馈好
```

不要迷信 AI 的整张图。它可能局部很好，整体不能用。

### 4. 人工重绘和规范化

选定方向后，要人工重绘。

重绘时做这些事：

```text
拉直透视
清理杂乱细节
统一左右对称或刻意非对称
拆开外框和填充
补齐被 AI 画糊的边缘
整理透明背景
确定九宫格区域
重新画可读的图标
去掉 AI 生成的假文字
```

这一步决定能不能出品。

AI 图像不要直接进项目。至少要经过一次人工整理。

### 5. 建立源文件分层

PSD、Figma、Illustrator 或其他源文件里，层命名要清楚。

比如：

```text
HPBar/
  frame
  empty_slot
  fill_red
  fill_highlight
  damage_delay
  mask
  low_hp_glow
  tick_marks
  icon_slot
  text_placeholder
```

如果源文件层乱成一团，后面改一次颜色或尺寸会很痛苦。

### 6. 导出游戏资源

导出时要按引擎需求来，不要只导出一张展示图。

例如：

```text
PNG 透明背景
外框和填充分开
按钮状态分开
九宫格资源留足边缘
图标统一尺寸
命名包含用途和状态
保留 @2x 或高分辨率源文件
```

命名可以这样：

```text
ui_hp_frame.png
ui_hp_slot_empty.png
ui_hp_fill_red.png
ui_hp_fill_poison.png
ui_hp_fill_shield.png
ui_hp_damage_delay.png
ui_hp_glow_low.png
ui_btn_primary_normal.png
ui_btn_primary_selected.png
ui_btn_primary_pressed.png
ui_btn_primary_disabled.png
```

### 7. 放进引擎里验证

不进引擎，就不知道 UI 能不能用。

血条至少测试：

```text
100% 血量
75% 血量
50% 血量
25% 血量
5% 血量
0% 血量
受伤延迟层
低血量闪烁
不同分辨率
不同 UI 缩放
```

按钮至少测试：

```text
普通
选中
按下
禁用
长文字
手柄焦点
鼠标悬停
点击反馈
```

面板至少测试：

```text
短文本
长文本
滚动内容
不同语言
不同屏幕比例
九宫格拉伸
边角是否变形
```

### 8. 回到美术文件修正

引擎测试后，一定会发现问题。

常见修正包括：

```text
线条太细，小尺寸看不见。
装饰太多，抢信息。
按钮选中态不明显。
填充层边缘穿帮。
九宫格拉伸后纹理变丑。
文字区域不够。
颜色和场景背景撞在一起。
透明边缘有脏像素。
```

这一步不是返工，而是 UI 从概念走向产品的必要过程。

## 八、给 AI 的提示词模板

### 血条外框

```text
fantasy RPG health bar frame, 2D game UI asset, front view, empty inner slot, ornate metal frame, clean readable silhouette, no red fill, no text, no numbers, no character portrait, no background scene, centered, transparent background if possible, designed for slicing into left cap, center stretch area, right cap
```

### 血条填充

```text
red liquid health bar fill texture, 2D game UI asset, horizontal strip, no frame, no text, no background, clean highlight, readable at small size, seamless horizontal stretch, transparent background if possible
```

### 按钮组件

```text
fantasy RPG button UI component set, normal selected pressed disabled states, 2D game asset, front view, no text, no icons, same shape and proportions, clean edges, readable at small size, transparent background if possible
```

### 面板背景

```text
fantasy RPG UI panel background, 2D game asset, front view, clean center area for text, ornate corners, simple border, suitable for 9-slice scaling, no text, no icons, no character, no background scene, transparent background if possible
```

### 图标

```text
game inventory icon, simple readable silhouette, 2D painted style, centered object, no background, no text, no border, consistent lighting, readable at 32 pixels, transparent background if possible
```

### 负面提示词

```text
full HUD screenshot, complete game screen, background scene, character, numbers, text, random symbols, perspective view, blurry edges, overly detailed, tiny unreadable details, merged layers, filled bar merged with frame, watermark
```

这些提示词不能保证直接出最终资产，但能明显减少 AI 往“整张截图”方向跑。

## 九、特别要注意的坑

### 1. 不要让 AI 生成最终文字

AI 生成的文字经常是乱码，也不方便本地化。按钮文字、数值、标题、任务描述都应该由字体系统来做。

AI 图里如果出现文字，最好只当构图参考，后面全部删掉重排。

### 2. 不要把光效和功能层混死

比如血条高光，如果直接和外框画在一起，就不能跟着血量裁切。高光应该和填充层绑定，或者单独做遮罩。

### 3. 不要让装饰影响识别

很多 AI 图看起来细节丰富，但缩小到游戏里会糊成一团。UI 不是插画，第一目标是让玩家快速看懂。

### 4. 不要只看静态图

UI 是动态使用的。血条会变化，按钮会按下，面板会拉伸，图标会变灰，提示会闪烁。只看静态图，很容易误判质量。

### 5. 不要混用太多 AI 风格

今天生成一批暗黑金属，明天生成一批二次元水晶，后天生成一批卡通木头，单看都不错，放一起就会散。要先定风格规范。

### 6. 不要直接模仿某个商业游戏

可以学习《女神异闻录》《Metaphor》《暗黑破坏神》《原神》等游戏的 UI 方法，但不要让 AI 直接“生成某某游戏风格的按钮”。这样容易变成低质量仿制，也有版权和品牌风险。

更好的做法是拆方法：

```text
学习它的色彩主从。
学习它的形状语言。
学习它的信息层级。
学习它的动效节奏。
不要复制它的具体图形。
```

### 7. 不要忽略引擎限制

如果项目用 Unity、Unreal、Cocos 或自研引擎，UI 资源的实现方式不同。比如 Unity 里 Image Filled、Mask、Sprite 9-slice、Animator、Canvas 缩放都会影响切图方式。

美术资源必须和实现方式对齐。

## 十、一个更靠谱的 AI UI 制作管线

我以后可以按这个管线来做：

```text
需求定义
  确定功能、尺寸、状态、适配、引擎实现方式。

结构草图
  先画出信息层级和拆分结构。

风格参考
  找 3 到 5 个参考，但只分析方法，不直接复制。

AI 风格探索
  生成多个方向，只选造型、材质、装饰灵感。

人工重绘
  清理结构，统一规则，补齐 AI 画错的地方。

组件拆分
  外框、底图、填充、遮罩、状态、图标、文字全部分开。

源文件整理
  PSD/Figma 分层命名，建立颜色、字体、描边、阴影规范。

切图导出
  按引擎需要导出 PNG、九宫格、图标、状态图。

引擎验证
  测数值变化、缩放、分辨率、交互状态、动效。

回修迭代
  根据真实运行效果调整可读性和美术细节。
```

这条管线的重点是：AI 只在“风格探索”和“局部素材”阶段参与，不直接越过结构和实现。

## 十一、血条的具体出品检查表

以后做血条，可以用这个检查表：

```text
是否有单独外框？
是否有单独空槽？
是否有单独填充层？
填充层能不能从 0% 到 100% 正常变化？
左右端头会不会被拉伸变形？
是否需要九宫格或三段式切分？
高光是否跟随填充变化？
受伤延迟条是否能单独显示？
低血量状态是否有单独特效？
数值文字是否由字体系统渲染？
是否有中毒、护盾、破防等状态扩展空间？
透明边缘是否干净？
在真实游戏尺寸下是否还能看清？
放到复杂背景上是否还能读？
```

如果这些问题答不上来，就说明它还只是概念图，不是出品资源。

## 十二、最重要的结论

AI 生图做游戏 UI，不能按“我要一张漂亮图”的方式用。

应该按这个逻辑用：

```text
人负责功能结构。
人负责组件拆分。
人负责风格规范。
AI 负责提供视觉方向和局部灵感。
人再把它整理成能进引擎的资源。
```

我现在做的 UI 不够出品，很可能不是因为 AI 不够强，而是前面缺了“资源拆分”和“引擎验证”这两步。

以后做任何 UI，都先问：

```text
这个东西在游戏里怎么变化？
它需要哪些状态？
哪些部分能拉伸？
哪些部分必须固定？
哪些文字不能烘焙？
哪些层要交给程序控制？
```

只要开始这样想，AI 生成的图就不会再被当成最终成品，而会变成一块可以被加工的原材料。

## 关键词

- AI 生图
- 游戏 UI
- UI 美术
- 血条
- 组件拆分
- 九宫格
- 切图
- 透明背景
- 提示词
- 制作流程
- 出品级 UI
- Unity UI
- 资源规范
