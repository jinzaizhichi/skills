# Graphviz DOT — 图表目录与覆盖账本

| 项 | 值 |
|---|---|
| 引擎 | `@viz-js/viz` **3.29**（WASM，内置 **Graphviz 15.1.1**） |
| 文档 | https://graphviz.org/doc/info/（属性 / 形状 / 箭头 / 颜色 / 语言）· https://graphviz.org/gallery/ |
| fence | ` ```dot `（本仓库也接受 ` ```graphviz `；文档统一用 `dot`） |
| 调研日期 | 2026-09-21 |
| 状态 | 属性与形状全表已枚举；**关键边界已实机验证**（§5）；示例待阶段 1b |
| 定位 | **自动布局型关系图**：依赖 / 调用 / 影响分析 / 层级 / 因果 / 状态机 / 族谱；用 HTML-like label 做表格化节点 |

---

## 1. 文档入口

| 入口 | URL | 用途 |
|---|---|---|
| **属性全表（权威）** | https://graphviz.org/doc/info/attrs.html | graph / node / edge / cluster 属性全集（含 Only 标注与默认值） |
| **节点形状** | https://graphviz.org/doc/info/shapes.html | 三类形状：polygon-based / record-based / HTML-like labels |
| 箭头形状 | https://graphviz.org/doc/info/arrows.html | `arrowType` 取值 |
| 颜色 | https://graphviz.org/doc/info/colors.html | X11 名称 + Brewer `colorscheme` |
| DOT 语言 | https://graphviz.org/doc/info/lang.html | 语法、HTML strings、escString |
| 布局引擎 | https://graphviz.org/docs/layouts/ | dot / neato / fdp / sfdp / circo / twopi / osage / patchwork |
| 输出格式 | https://graphviz.org/docs/outputs/ | svg / png / plain / canon… |
| **Gallery** | https://graphviz.org/gallery/ | 官方示例（§4 场景基线） |

---

## 2. 属性全表：按用途分组（收录策略）

> 官方属性约 130 个（含大量仅 neato/fdp/sfdp 可用的物理参数）。我们**不做全量文档化**：
> 只收录与 T0/T1 场景相关的 ~30 个，其余在 `engines/dot.md` 给出「按需查 attrs.html」的指引。

| 分组 | 属性 | 说明 / 官方要点 |
|---|---|---|
| **布局与方向** | `layout` | 选择布局引擎；**"takes precedence over the -K flag or the actual command name"**（§5 已实测） |
| | `rankdir` `ranksep` `nodesep` | `TB/BT/LR/RL`；rank 间距、同 rank 节点间距（**dot only**） |
| | `rank` | 子图内 `same/min/source/max/sink` 约束（**dot only**） |
| | `constraint` | `false` 时该边不参与层级排序（**dot only**，做"回边"必备） |
| | `newrank` `TBbalance` | 全局统一 rank（忽略 cluster 的递归排序）/ 浮动节点对齐到 min/max rank（**dot only**） |
| | `minlen` `weight` | 边跨越的最小 rank 数 / 权重（权重越大越短越直，**整数**） |
| **聚类** | `cluster`（subgraph）| `cluster_` 前缀或 `cluster=true`；`clusterrank` `local/global/none`（**dot only**） |
| | `compound` + `lhead`/`ltail` | 允许**簇间边**并把边裁剪到簇边界（**dot only**） |
| | `pencolor` `bgcolor` `style=filled` | 簇边框色 / 底色 / 渐变底 |
| **节点形状** | `shape` | 见 §3（polygon / record / HTML-like） |
| | `fixedsize` `width` `height` `margin` | `fixedsize=true` 强制尺寸；`shape` 关键字下标签可溢出 |
| | `regular` `peripheries` `orientation` `sides` `skew` `distortion` | 多边形几何控制 |
| **样式** | `style` | 节点 8 值：`filled` `invisible` `diagonals` `rounded` `dashed` `dotted` `solid` `bold`（可逗号组合，**后写的胜**） |
| | `penwidth` `color` `fillcolor` `fontcolor` `fontname` `fontsize` | 线宽 / 描边 / 填充 / 字体 |
| | `colorscheme` + `color="7"` | Brewer / X11 配色命名空间 |
| | `bgcolor="red:blue"` + `gradientangle` | 渐变填充（`style=radial` 为径向） |
| **边** | `dir` `arrowhead` `arrowtail` `arrowsize` | `forward/back/both/none` + 箭头样式 |
| | `headlabel` `taillabel` `label` `xlabel` `decorate` | 边上/端点标签；`xlabel` 外置标签（`forcelabels` 强制放置） |
| | `headport` `tailport` | 指定连到节点的哪个 port（配合 record/HTML label 的 `PORT`） |
| | `color="red:blue"` | 双向关系用**平行样条**两色同绘（官方推荐替代双箭头） |
| | `splines` `radius` | `true/spline` `false/line` `polyline` `curved` `ortho` `none`；`radius` 给 ortho 边加圆角（**≥14.1.0，本引擎 15.1.1 可用**） |
| | `concentrate` | 合并多重边 |
| **文本与编码** | `charset`（默认 UTF-8）· escString：`\N` 节点名 / `\E` 边名 / `\G` 图名 / `\n \l \r` 换行对齐 | 官方示例里直接用 emoji 也 OK（§5 实测） |
| **导出交互（SVG）** | `class` `stylesheet` `id` `URL` `href` `target` `tooltip` | 导出为 SVG 时的链接与 tooltip（静态图场景价值有限，T2） |
| **仅其它引擎** | `overlap` `sep` `mode` `model` `Damping` `maxiter` `start` `root` `mindist` `oneblock` `repulsiveforce` … | 仅 `neato/fdp/sfdp/circo/twopi` 生效 → 只有显式写 `layout=` 时才涉及 |

---

## 3. 节点形状（三类）

**① polygon-based（约 60 个）**

- 基础：`box` `rect` `rectangle` `square` `ellipse` `oval` `circle` `point` `egg` `triangle` `diamond` `trapezium` `parallelogram` `house` `pentagon` `hexagon` `septagon` `octagon` `doublecircle` `doubleoctagon` `tripleoctagon` `invtriangle` `invtrapezium` `invhouse` `Mdiamond` `Msquare` `Mcircle` `star` `none` `underline` `plaintext` `plain`
- 语义化：`cylinder`（存储）`note` `tab` `folder` `box3d` `component`
- 分子生物学专用 20 个（`promoter` `cds` `terminator` …）→ **不收录**（无适用场景）
- 关键语义：`plain` = `plaintext` + `width=0 height=0 margin=0`（**HTML-like label 的标配**）；`rect`/`rectangle` 是 `box` 的同义；`none` 是 `plaintext` 的同义

**② record-based（`record` / `Mrecord`）**

- 用 `label="<f0> left|<f1> mid|<f2> right"` 的 `|`/`{}` 语法切分字段，`<portName>` 定义端口
- 顶层方向受 `rankdir` 影响（TB/BT → 字段横向；LR/RL → 纵向）
- 官方提示：**已被 HTML-like labels 大幅取代**；相邻同 rank 的 record 节点之间用带端口/标签的边有已知问题

**③ HTML-like labels（官方推荐做法）**

- 写法：`label=<...>`（尖括号而非引号），常配 `shape=none/plaintext`（或 `shape=plain`）
- 元素：`<TABLE>` `<TR>` `<TD>` `<FONT>` `<BR/>` `<IMG>` `<B>` `<I>` `<U>` `<O>` `<SUB>` `<SUP>` `<S>` `<HR/>` `<VR/>`
- 单元格属性：`COLSPAN` `ROWSPAN` `BGCOLOR`（可双色渐变）`ALIGN` `BALIGN` `VALIGN` `BORDER` `CELLBORDER` `CELLPADDING` `CELLSPACING` `COLOR` `FIXEDSIZE` `HEIGHT` `WIDTH` `PORT` `SIDES` `STYLE(ROUNDED/RADIAL)` `HREF` `TARGET` `TITLE/TOOLTIP` `ID` `GRADIENTANGLE` `COLUMNS/ROWS="*"`
- 转义规则：字面量中的 `"` `&` `<` `>` 需转义；`<FONT>`/`<B>` 直接包裹 `<TABLE>` 前后不能有空格（官方明确报错示例）
- 字体标记 `<B>/<I>/<U>/<S>/<SUB>/<SUP>` 与 `<HR/>/<VR/>` 在 **cairo / svg 渲染器**可用 → 我们用 SVG，**全部可用**

**④ 用户自定义形状（PostScript / 位图）** → ❌ 不收录（需外部文件，离线不可靠）

---

## 4. 场景基线（官方 Gallery 归类）

| 分区 | 示例 | 对应我们的场景 |
|---|---|---|
| Directed | git（分支模型）、bazel（构建依赖）、ninja、go-package、Linux kernel diagram | 依赖图 / 调用图 / 构建图 |
| | cluster、siblings、family tree、unix family tree、racehorse pedigree | 簇分组的层级 / 族谱 |
| | fsm、traffic lights（neato） | 状态机 |
| | data structures、math parse tree、parsing tree、UML class demo | 结构 / 语法树（表格化节点） |
| | neural network、pprof CPU profile、program profile、world dynamics | 计算图 / 性能剖析 |
| | switch network、SDH | 网络设备关系 |
| Neato | ER（实体关系数据模型）、softmaint（模块依赖）、process、philosophers、transparency | 力导向关系图（写 `layout=neato`） |
| Undirected | grid、fdp clusters、sfdp 大图 | 无向关系 / 大图 |
| Twopi | network map、mind map of happiness、Pandora 网络地图 | 放射状网络（**§5 待复核**） |
| Gradient | graph/cluster/node 渐变、linear/radial angle、table cell 渐变 | 视觉强化（T2） |

---

## 5. 本仓库实现边界（**已实机验证**，`@viz-js/viz` 3.29 = Graphviz 15.1.1）

| 项 | 结论 | 证据 |
|---|---|---|
| 内置版本 | **Graphviz 15.1.1** | `viz.graphvizVersion` |
| 可用引擎 | circo · dot · fdp · neato · nop* · osage · patchwork · sfdp · twopi | `viz.engines` |
| **`layout=` 属性** | ✅ **生效**，且 **8 个引擎全部可用**：neato · fdp · sfdp · circo · twopi · osage · patchwork 与 dot 坐标均不同（`nop` 不做布局，不收录） | 同图 plain 输出的节点坐标逐一对比（见 `notes.md` §3） |
| `layout=twopi` | ✅ 可用（**上一轮的“可疑”是测试图选得不好**——线性链在 twopi 下本就是一条线） | 换成带环的图后坐标与 dot 明显不同 |
| `radius`（ortho 圆角，≥14.1.0） | ✅ 可用（版本 15.1.1） | 渲染成功（len=1307） |
| HTML-like label（`<TABLE>`） | ✅ 可用 | 渲染成功（len=919） |
| `record` 形状 | ✅ 可用 | 渲染成功（len=999） |
| 渐变填充（`fillcolor="red:blue"` + `gradientangle`） | ✅ 可用 | 渲染成功（len=1068） |
| emoji / UTF-8 标签 | ✅ 可用（默认 `charset=UTF-8`） | `"🍔" -> "💩"` 渲染成功 |
| 外部资源 | ❌ `image=` / `shapefile=` / `<IMG SRC>` 指向本地文件：离线/导出不可靠 | 文档明示「Only paths to local resources are supported」 |
| 交互属性（`URL`/`tooltip`/`target`） | T2：静态图无意义（导出 SVG 可留链接） | 文档标注 svg/map only |

> ⚠️ **修正一条旧结论**：计划早期写的「dot 渲染器未传 engine ⇒ 示例不得依赖 neato/circo」**不成立** ——
> 源码内 `layout=` 属性优先级高于引擎参数，且**实测 8 个引擎全部可用**（选哪个取决于图的形态：
> 层级→dot、力导向→neato/fdp、大图→sfdp、环形→circo、放射→twopi、簇式→osage、拼贴→patchwork）。

**渲染器另注**（`src/renderers/dot-renderer.ts`）：渲染时强制 `bgcolor=transparent`；深色文档会注入
`graph/node/edge` 的 fontcolor/color 前置属性（**用户代码里显式写的属性仍然后覆盖**，因为前缀在前）。

---

## 6. 覆盖账本（处置）

| 类别 | 官方规模 | 收录 | 不收录（理由） |
|---|---|---|---|
| 属性 | ~130 | ~30（§2 表格） | 其余为 neato/fdp/sfdp 物理参数、写属性（`bb` `lp` `rects`…）、调试属性（`showboxes`）→ 在 `engines/dot.md` 给指引 |
| 节点形状 | polygon ~60 + record 2 + HTML labels | 常用 12 个 + record + HTML-like 全支持 | 分子生物学 20 个、用户自定义形状 |
| 箭头类型 | 20+（arrows.html） | 常用 8 个（`normal` `vee` `dot` `odot` `diamond` `box` `none` `empty`） | 其余按需查文档 |
| 样式值 | 节点 8 / 边 & 簇若干 | 全部 8 个（成本低、收益高） | — |
| Gallery 场景 | ~45 例 | 挑 ~12 例转成示例（依赖 / 簇 / 状态机 / 族谱 / 数据表 / ER / 网络 / 渐变） | 领域专用（SDH、Linux kernel 全图）不做示例，仅作复杂案例参考 |

---

## 7. 工单（下一轮）

1. ~~`layout=twopi` 复核~~ ✅ 已完成：8 个引擎全部可用（见 §5）。
2. **`concentrate` / `compound+ltail/lhead` / `samehead`** 三个“高级排版”实机验证（文档标注 dot only，我们正是 dot 路径）。
3. **深色主题交互复核**：注入的默认颜色与用户显式颜色同时存在时的最终表现（需实渲染截图，不能只看代码顺序）。
4. **多引擎示例选型**：为 neato/fdp/sfdp/circo/twopi 各配一个场景（力导向关系、大图、环形关系、放射网络）。
5. **示例选型**：把 §4 的 ~12 个场景落成具体示例文件（含 `source:` 指向 Gallery 对应页）。

---

## 8. 引用

- 属性：https://graphviz.org/doc/info/attrs.html · 形状：https://graphviz.org/doc/info/shapes.html
- 箭头：https://graphviz.org/doc/info/arrows.html · 颜色：https://graphviz.org/doc/info/colors.html
- Gallery：https://graphviz.org/gallery/
- 本仓库实现：`src/renderers/dot-renderer.ts`（仅用于边界）；引擎：`@viz-js/viz` 3.29（Graphviz 15.1.1）
