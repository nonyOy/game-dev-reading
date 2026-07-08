# 2026-07-08：Unity 项目里游戏 UI 资源和 Prefab 的组织方法

前几篇已经连续写了游戏 UI 美术、AI 生图、切图和引擎落地。今天继续往更实际的方向走：如果项目用 Unity，UI 资源应该怎么组织，Prefab 应该怎么搭，哪些东西交给美术，哪些东西交给程序，哪些地方最容易变乱。

我现在越来越觉得，UI 出品质量不只取决于单张图好不好看。很多 UI 最后看起来不专业，是因为资源进入项目以后没有被当成“系统”管理。

比如一个血条，最开始可能只是几张 PNG：

```text
血条框
血条底
血条红色填充
低血量闪光
```

但放进 Unity 以后，它会变成更多东西：

```text
Sprite 导入设置
九宫格边界
Image 组件
Mask 或 Filled Image
Canvas 层级
Prefab
脚本引用
动画状态
字体文本
图集打包
分辨率适配
运行时数据绑定
```

如果这些都没有规范，UI 就会从“看着还不错”变成“谁都不敢改”。

所以这篇重点整理：Unity 项目里，游戏 UI 从资源到 Prefab，应该怎么组织才比较稳。

## 一、Unity UI 不能只按图片管理，要按功能模块管理

很多人整理 UI 资源时，会按图片类型放：

```text
Textures/
Sprites/
Icons/
Buttons/
Panels/
```

这样不是完全不行，但如果项目稍微变大，就会有一个问题：资源和功能脱节。

比如背包界面需要：

```text
背包面板
物品格子
物品图标
稀有度框
数量文本
筛选按钮
滚动条
详情弹窗
装备按钮
```

如果这些东西分散在很多目录里，后期改背包会很难找。更好的方式是按模块管理，同时保留公共资源。

可以这样组织：

```text
Assets/
  Game/
    UI/
      Common/
        Sprites/
        Prefabs/
        Fonts/
        Materials/
        Animations/
      HUD/
        Sprites/
        Prefabs/
        Animations/
      Inventory/
        Sprites/
        Prefabs/
        Animations/
      Battle/
        Sprites/
        Prefabs/
        Animations/
      Shop/
        Sprites/
        Prefabs/
        Animations/
      Dialog/
        Sprites/
        Prefabs/
        Animations/
```

公共按钮、通用面板、通用弹窗、通用字体放在 `Common`。某个界面专用的资源放在自己的模块目录。

这样做的好处是：

- 改背包时主要看 `Inventory`。
- 改战斗 HUD 时主要看 `HUD` 或 `Battle`。
- 公共按钮可以统一维护。
- 不会所有 UI 图都堆在一个巨大文件夹里。
- 新人接项目时能按功能找资源。

目录结构本身就是协作工具。

## 二、Sprite 导入设置是 UI 质量的一部分

UI 图片导进 Unity 后，不是直接能用就完了。导入设置会直接影响清晰度、内存、九宫格和显示效果。

常见设置要注意：

```text
Texture Type：Sprite (2D and UI)
Sprite Mode：Single 或 Multiple
Pixels Per Unit：保持项目统一
Mesh Type：Full Rect 通常更适合 UI
Compression：重要 UI 不要压到出现脏块
Filter Mode：看风格决定 Point 或 Bilinear
Wrap Mode：一般 Clamp，平铺纹理才考虑 Repeat
Alpha Is Transparency：透明边缘要检查
```

如果是像素风 UI，Filter Mode 可能要用 Point，避免边缘糊掉。如果是手绘或高清 UI，Bilinear 通常更自然。

如果是面板和按钮，九宫格边界要在 Sprite Editor 里设置好。这个边界不是随便拖的，它决定哪些地方可以拉伸，哪些角必须保持不变。

九宫格边界设置错了，会出现：

```text
边角被拉长
装饰变形
边框厚度不一致
纹理被拉糊
按钮变宽后中间花纹奇怪
```

所以 UI 资源导入 Unity 后，要把导入设置也当成制作流程的一部分，而不是程序随手处理。

## 三、Prefab 是 UI 的真正交付单位

UI 美术资源进 Unity 后，最终给项目使用的不应该只是 PNG，而应该是 Prefab。

比如一个按钮，不应该只交：

```text
button_normal.png
button_selected.png
button_pressed.png
button_disabled.png
```

更应该交一个已经搭好的：

```text
Button_Primary.prefab
```

里面包括：

```text
Image 背景
Text 或 TMP_Text
Button 组件
状态切换
选中态节点
禁用态表现
音效触发位
必要的 Layout 设置
```

这样项目里使用按钮时，不需要每个人重新搭一遍。

Prefab 的价值是把“资源”和“使用方式”绑定起来。否则同一套按钮图，不同人搭出来的效果可能完全不一样：

```text
有人文字居中。
有人文字偏上。
有人禁用态只改透明度。
有人选中态加描边。
有人按钮点击区域比图小。
有人忘了关 Raycast Target。
```

这些小差异累积起来，UI 就会失去统一感。

所以 UI 的交付单位应该是：

```text
Sprite：原始视觉资源
Prefab：可复用组件
Scene / Panel Prefab：完整界面
```

不是只交图片。

## 四、UI Prefab 要分三层：基础组件、组合组件、完整界面

Unity UI 很容易出现一个问题：所有东西都直接堆在一个巨大界面 Prefab 里。这样短期最快，长期很难维护。

更合理的结构是三层：

```text
基础组件
  Button
  Panel
  IconSlot
  ProgressBar
  Toggle
  Tab

组合组件
  ItemCell
  SkillButton
  PlayerHpBar
  CurrencyDisplay
  QuestListItem

完整界面
  InventoryPanel
  BattleHUD
  ShopPanel
  SettingsPanel
```

比如 `SkillButton` 不是简单按钮，它可能包含：

```text
按钮底图
技能图标
冷却遮罩
冷却数字
快捷键提示
锁定图标
选中框
红点
```

它应该是一个组合组件，而不是每个界面里复制一套节点。

这样做的好处是：

- 改技能按钮样式时，只改一个 Prefab。
- 所有技能按钮状态一致。
- 程序绑定逻辑更稳定。
- 美术回修改动更集中。
- 新界面可以复用已有组件。

如果没有这三层，项目后期很容易到处复制 UI 节点。复制越多，维护越痛。

## 五、血条在 Unity 里可以做成标准组件

血条是最适合做成标准 Prefab 的 UI。

一个比较完整的血条 Prefab 可以这样：

```text
UI_HPBar
  EmptySlot        Image
  DamageDelay      Image
  FillRoot         RectTransform
    Fill           Image
    FillHighlight  Image
  Frame            Image
  LowHpGlow        Image
  ShieldLayer      Image
  ValueText        TMP_Text
```

核心逻辑：

```text
Fill 显示当前血量。
DamageDelay 显示受伤延迟。
ShieldLayer 显示护盾。
LowHpGlow 在低血量时显示或播放动画。
Frame 永远在最上层。
ValueText 用字体系统显示，不烘焙在图片里。
```

填充方式可以选：

```text
Image Type: Filled
RectMask2D 裁切
调整 RectTransform 宽度
Shader 遮罩
```

如果血条是规则矩形，`Image Type: Filled` 或调整宽度就够。如果血条边缘是不规则形状，遮罩更稳。

血条 Prefab 应该暴露这些可配置项：

```text
最大值
当前值
是否显示数字
是否启用延迟条
低血量阈值
普通颜色
中毒颜色
护盾颜色
延迟条速度
```

这样血条就不是一堆图，而是一个能被复用的 UI 组件。

玩家血条、Boss 血条、怪物血条可以共享基础逻辑，只换资源和尺寸。

## 六、字体尽量统一用 TMP，并建立文本样式

Unity 里做 UI 文本，最好统一用 TextMeshPro。它对清晰度、描边、阴影、材质、中文字体资产管理都更稳。

但问题是：如果每个文本都手动调字号、颜色、描边，项目还是会乱。

应该建立文本样式规范：

```text
Title_Large
Title_Medium
Body_Normal
Body_Small
Button_Label
Number_Main
Number_Damage
Hint_Text
Disabled_Text
```

每种样式明确：

```text
字体
字号
颜色
描边
阴影
行距
对齐方式
最大宽度
是否自动缩放
```

游戏 UI 里数字尤其重要。血量、金币、伤害、倒计时、等级，都应该有统一的数字风格。

文字最容易暴露 UI 是否粗糙。常见问题：

- 同一界面里字号太多。
- 文本描边颜色不统一。
- 数字和中文风格不匹配。
- 按钮文字没有垂直居中。
- 长文本被遮挡。
- 英文换行很丑。
- 字体资产缺字。

所以字体不是最后补上去，而是 UI 系统的一部分。

## 七、Canvas 和层级要有规则

Unity UI 的 Canvas 层级如果没有规则，会很快混乱。

可以按用途分层：

```text
Canvas_HUD
  战斗血条、技能按钮、小地图、任务追踪

Canvas_Window
  背包、商店、设置、角色面板

Canvas_Popup
  确认框、奖励弹窗、提示弹窗

Canvas_Top
  Loading、系统遮罩、断线提示、全局 Toast
```

这样可以避免弹窗被 HUD 挡住，或者奖励界面被普通窗口盖住。

如果所有 UI 都放在一个 Canvas 下，可能会带来：

```text
排序混乱
重绘范围过大
查找节点困难
弹窗层级冲突
动效互相影响
```

Canvas 不是越多越好，也不是越少越好。原则是：高频变化的 UI 和低频静态 UI 尽量不要互相拖累。

比如战斗 HUD 中血条和倒计时经常变化，可以和静态窗口分开。弹窗和系统遮罩也应该有独立层级。

## 八、Raycast Target 要认真关

Unity UI 里一个很容易忽略的问题是 Raycast Target。

很多 Image 默认会勾选 Raycast Target。如果一个只是装饰的 Image 也接收射线，它可能挡住下面的按钮，导致点击异常。

所以 UI Prefab 里要有规则：

```text
能被点击的按钮、拖拽区域、输入框：开启 Raycast Target。
纯装饰图片、边框、高光、背景纹理：关闭 Raycast Target。
文本如果不需要交互：关闭 Raycast Target。
```

这个问题非常实际。很多“按钮点不到”“点击穿透异常”“滚动列表拖不动”，最后都和射线层级有关。

出品级 UI 不只是看起来好看，还要点起来稳定。

## 九、动效不要直接散在节点上，要有管理方式

UI 动效可以用 Animator、Tween 插件、Timeline、代码插值等方式实现。无论用哪种，都要避免一个问题：动效散落在各个节点上，没有统一规则。

常见动效状态：

```text
Open
Close
Show
Hide
Selected
Unselected
Pressed
Disabled
RewardIn
Warning
```

如果是常用组件，比如按钮、弹窗、红点、奖励图标，可以做统一动效。

比如弹窗：

```text
打开：背景遮罩淡入，面板从 0.95 缩放到 1，透明度从 0 到 1。
关闭：透明度淡出，面板轻微缩小。
```

按钮：

```text
按下：缩放到 0.96，快速回弹。
选中：外框亮起或轻微呼吸。
禁用：停止动效，降低饱和度。
```

动效时间也要有规范：

```text
高频按钮反馈：0.08 到 0.15 秒。
普通弹窗开关：0.15 到 0.25 秒。
奖励演出：0.4 秒以上可以接受，但要能跳过。
```

UI 动效不是装饰，它要让操作更清楚。如果动效让玩家等，就要谨慎。

## 十、图集和资源加载要提前考虑

UI 图很多，如果完全散图加载，可能会增加 Draw Call、内存和管理成本。Unity 项目里通常要考虑图集或 Addressables。

不同项目做法不一样，但原则类似：

```text
同一界面常一起出现的图，适合放进同一图集。
全局常用资源，可以放 Common 图集。
大弹窗或活动界面资源，可以单独分包。
不会同时出现的资源，不一定要塞进同一图集。
超大背景图和小图标不要随便混在一起。
```

图集不是越大越好。太大的图集可能浪费内存，也会让小改动牵动大资源。

如果项目用 Addressables，要考虑：

```text
界面打开前是否预加载？
关闭界面后是否释放？
公共资源是否常驻？
活动资源是否可热更？
字体和材质是否被重复打包？
```

这部分属于程序和技术美术的交界。UI 美术至少要知道：资源怎么组织会影响加载和性能。

## 十一、Prefab 绑定要避免硬拖乱拖

Unity UI 经常会出现一个问题：脚本上拖了很多引用，但命名不清楚，层级一改就丢。

比如一个背包格子脚本可能有：

```text
Image icon;
Image frame;
TMP_Text countText;
GameObject selected;
GameObject locked;
GameObject redDot;
Button button;
```

这些引用要清楚命名，并且 Prefab 层级也要稳定。

不要让脚本依赖模糊路径，比如：

```text
transform.GetChild(3).GetChild(1)
```

这种写法只要美术调整层级就会坏。

更稳的方式是：

```text
明确 SerializeField 字段。
Prefab 内引用拖好。
关键节点命名清楚。
美术调整前知道哪些节点被脚本引用。
```

UI 美术和程序要有一个共识：不是所有节点都能随便删。被脚本引用的节点要稳定，或者在改动时同步程序。

## 十二、UI Prefab 要有验收场景

只在 Prefab 预览里看 UI 不够。最好有一个专门的 UI 测试场景。

测试场景可以包含：

```text
所有按钮状态
所有血条状态
弹窗打开关闭
背包格子不同数量
技能按钮冷却
长文本和短文本
不同语言占位
不同分辨率模拟
亮背景和暗背景
```

这个场景不一定进正式包，但对开发很有用。

它可以快速回答：

- 新按钮状态是否统一？
- 新字体是否缺字？
- 新血条在低血量是否穿帮？
- 新面板九宫格是否正常？
- UI 在深色背景和浅色背景上是否都能看清？

如果没有测试场景，每次都要进正式流程里看，效率很低。

## 十三、UI 修改要走回归检查

UI 很容易改一处影响很多处。

比如改了通用按钮：

```text
背包按钮变了。
商店按钮变了。
任务按钮变了。
弹窗按钮变了。
设置按钮变了。
```

如果只看当前界面，很可能漏掉别处的问题。

所以通用组件修改后，要做回归检查：

```text
这个 Prefab 被哪些界面引用？
改尺寸后文字还放得下吗？
改颜色后禁用态还清楚吗？
改九宫格后长按钮是否正常？
改动效后高频操作是否变慢？
```

Unity 里 Prefab 的复用是优势，但也意味着改动会扩散。越通用的组件，越要谨慎。

## 十四、一个比较稳的 Unity UI 制作流程

我以后可以按这个流程做：

```text
1. 需求拆解
   确定界面功能、数据、状态、交互、分辨率要求。

2. 美术稿
   画出风格和信息层级，但不要把所有动态内容烘焙成死图。

3. 资源拆分
   拆出外框、底图、填充、图标、状态、遮罩、特效。

4. Unity 导入
   设置 Sprite、透明、压缩、九宫格、Filter Mode。

5. 基础 Prefab
   搭按钮、面板、血条、图标槽、滚动项等基础组件。

6. 组合 Prefab
   搭技能按钮、物品格子、玩家状态栏、货币条等组合组件。

7. 界面 Prefab
   搭完整界面，接 Layout、Scroll、Grid、Canvas 层级。

8. 脚本绑定
   稳定引用关键节点，不用脆弱的 Child Index。

9. 状态接入
   普通、选中、按下、禁用、冷却、红点、低血量、空数据。

10. 动效接入
   打开、关闭、点击、选中、奖励、警告，统一节奏。

11. 适配测试
   测不同分辨率、不同安全区、长文本、多语言占位。

12. 真实场景测试
   放到战斗、背包、商店等实际场景里看。

13. 回修改进
   根据引擎效果回到源文件和 Prefab 修正。
```

这条流程的重点是：不要把 Unity 当成最后摆一下的地方。Unity 里的 Prefab 搭建，本身就是 UI 制作的一部分。

## 十五、我现在最应该养成的习惯

如果目标是做出能出品的游戏 UI，我觉得自己现在最该养成几个习惯：

```text
画 UI 前先想组件。
出图前先想拆分。
导入 Unity 后检查 Sprite 设置。
按钮至少补齐四个状态。
血条一定测试 0% 到 100%。
面板一定测试九宫格拉伸。
文字一定测试长文本。
Prefab 命名和层级要清楚。
纯装饰节点关闭 Raycast Target。
通用组件改动后做回归检查。
```

这比单纯多生成几张漂亮 UI 图更重要。

一句话总结：

```text
游戏 UI 的质量，不在单张图结束。
它在 Unity 里的 Sprite、Prefab、状态、适配、动效和测试里继续决定。
```

## 可以尝试的实践

可以在 Unity 里做一个小型 UI 组件库，不做完整游戏界面，只做基础组件。

目标：

```text
Button_Primary
Button_Secondary
Panel_Normal
Panel_Popup
HPBar_Player
HPBar_Boss
IconSlot_Item
SkillButton
Toast_Message
```

每个组件都要做到：

```text
有清楚 Prefab 名字。
有正常状态和异常状态。
有真实文本测试。
有拉伸测试。
有暗背景和亮背景测试。
有简单动效。
能被另一个界面复用。
```

做完以后再搭一个 `UI_TestScene`，把所有组件摆进去。

这个练习的价值不在于画多少界面，而在于建立一套可以复用的 UI 资产工作方式。

## 关键词

- Unity UI
- UGUI
- Prefab
- Sprite
- 九宫格
- TextMeshPro
- Canvas
- Raycast Target
- UI 组件库
- 血条
- 按钮状态
- 图集
- UI 测试场景
- 制作流程
