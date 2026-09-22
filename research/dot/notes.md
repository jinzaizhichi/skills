# Graphviz DOT 文档要点摘录（notes）

> 抓取日期：2026-09-21 · 引擎：`@viz-js/viz` 3.29（内置 **Graphviz 15.1.1**）

## 1. 文档结构（查漏基线）

```
docs/info/attrs.html     属性全表（Graphs / Nodes / Edges / Clusters）× 类型 / 默认值 / 最小值 / Only 标注
docs/info/shapes.html    三类形状：polygon-based · record-based · HTML-like labels（含元素与属性全表）
docs/info/arrows.html    arrowType 取值
docs/info/colors.html    X11 颜色名 + Brewer colorscheme
docs/info/lang.html      DOT 语法、HTML strings、escString、端口
docs/info/command.html   命令行（-K 选引擎、-n/-s 等）
docs/layouts/            dot · neato · fdp · sfdp · circo · twopi · osage · patchwork
docs/outputs/            svg · png · plain · canon · xdot …
gallery/                 Directed / Neato / Undirected / Twopi / Gradient 五个分区
```

## 2. 原文要点（直接引述）

**`layout` 属性（关键）**

> “Specifies the name of the layout engine to use, such as `dot` or `neato`. … This attribute takes
> precedence over the `-K` flag or the actual command name.”

⇒ 源码内即可切布局，**不必依赖调用方传 engine**（我们用 viz-js 的默认调用，因此这条很关键）。

**`rank`（dot only）**

> `same` 所有节点同 rank；`min`/`source` 最小 rank；`max`/`sink` 最大 rank（最小 rank = 最上或最左）。

**`constraint`（dot only）**

> “If false, the edge is not used in ranking the nodes.”（画"回边/反馈边"时避免层级被拉乱的必备属性）

**`clusterrank`（dot only）**

> `local` 时名字以 `cluster` 开头的子图会被单独布局后作为一个整体嵌入，并画出带标签的外框。

**`splines`**

> `true/spline` 样条绕开节点；`false/line` 直线段；`none/""` 不画边；`polyline`；`ortho` 轴对齐折线（**不处理 ports，dot 下也不处理边标签**）；`curved` 弧线。

**`radius`（≥14.1.0）**

> “Controls the radius of rounded corners on orthogonal edges … only has an effect when `splines=ortho` is set.”

**`record` 的定位**

> “The record-based shape has largely been superseded and greatly generalized by HTML-like labels. That is,
> instead of using `shape=record`, one might consider using `shape=none`, `margin=0` and an HTML-like label.”

**`plain` 的语义**

> “`shape=plain` is shorthand for `shape=none width=0 height=0 margin=0` … the size of the node is totally
> determined by the label.”（HTML-like label 的标配写法）

**HTML-like labels**

> “If the value of a label attribute … is given as an HTML string, that is, delimited by `<...>` rather than
> `"..."`, the label is interpreted as an HTML description.”

> 字体标记（`<B> <I> <U> <SUB> <SUP>`、`<S>`）与 `<HR/> <VR/>` “are currently only available via the
> **cairo and svg renderers**” —— 我们正是 SVG，因此全部可用。

> ⚠️ 官方坑：「`< <U><TABLE>…</TABLE></U>>` is not legal」—— `<FONT>/<B>` 直接包 `<TABLE>` 时前后多一个空格就报语法错误。

**边颜色双色**

> `color="red:blue"`：无分号时为并行样条（一条边画两色），官方建议用它替代双箭头表达"对向关系"。

**无向图的方向歧义**

> 无向边 `A -- B` 第一次出现时按"左尾右头"解析；`taillabel` 之类属性会因此挂错端 → **需要端点语义时改用有向图 + `dir=none`**。

**字符串/编码**

> `charset` 默认 `UTF-8`；但 “It is not possible to use HTML-like labels in combination with Big-5 encoding.”

**外部资源**

> `image` / `shapefile`：“**Only paths to local resources are supported**.”（无远程 URL 支持）

## 3. 实机验证结果（`@viz-js/viz` 3.29，Graphviz 15.1.1）

用 `renderString(src, {format:'plain'})` 对比坐标，用 `{format:'svg'}` 验证可渲染性：

| 探针 | 结果 |
|---|---|
| `graphvizVersion` | `15.1.1` |
| `engines` | circo dot fdp neato nop nop1 nop2 osage patchwork sfdp twopi |
| `layout=neato` | ✅ 生效（坐标与 dot 完全不同：环状散布 vs 竖排） |
| `layout=circo` | ✅ 生效（节点落在一个圆环上） |
| **全部 8 个引擎** | ✅ 均生效（补测，见下表）：neato · fdp · sfdp · circo · twopi · osage · patchwork 与 dot 坐标均不同；`nop` 不做布局（节点全在 0,0） |
| `splines=ortho` + `radius=8` | ✅ 渲染成功 |
| HTML `<TABLE>` label | ✅ 渲染成功 |
| `shape=record` + 端口 | ✅ 渲染成功 |
| `fillcolor="red:blue"` + `gradientangle=45` | ✅ 渲染成功 |
| emoji 节点名 | ✅ 渲染成功 |

⇒ **修正旧结论**：此前计划里写的「示例不得依赖 neato/circo」不成立；正确口径是
「默认 dot 布局；需要其它排布时用源码内 `layout=` 属性，**8 个引擎全部可用**」。

**补充实测（同一图 `a->b; b->c; c->d; d->a; a->c` 的节点坐标）**

| 引擎 | 形态 |
|---|---|
| dot（默认） | 分层竖排（x 随层级变化） |
| neato | 力导向散布（stress majorization） |
| fdp | 力导向（类似 neato，参数不同） |
| sfdp | 大图力导向（多尺度） |
| circo | 圆环 |
| twopi | 放射（以 root 为中心） |
| osage | 簇式分组聚集 |
| patchwork | 方块拼贴（面积映射） |
| nop | 不做布局（保坐标，全部 0,0）→ ❌ 不收录 |

## 4. 对本 skill 的直接含义

1. **DOT 的差异化不在"画什么"而在"怎么排"**：`rank` / `constraint` / `cluster` / `rankdir` / `minlen` / `weight`
   这一组是其它引擎（plantuml 的类图、mermaid）给不了或很难给的精细控制 → 示例应集中展示这些。
2. **HTML-like label 是"表格化节点"的唯一手段**（记录字段、多列指标、带底色的分区）→ 值得 2–3 个示例。
3. **能换布局**：关系网络/力导向用 `layout=neato`；环形关系用 `layout=circo`（两者已验证）。
4. **不要碰**：外部图片、用户自定义形状、分子生物形状、写属性（`bb/lp/rects`）、调试属性（`showboxes`）。
5. **深色主题**：渲染器会注入默认颜色，但用户显式属性在后、仍生效 → 示例若要自定义配色，直接写 `color`/`fontcolor` 即可。

## 5. 待确认清单（与 catalogue §7 工单对应）

- [ ] `layout=twopi` 是否真的可用（放射状）
- [ ] `fdp` / `sfdp` / `osage` / `patchwork` 抽查
- [ ] `concentrate`、`compound+ltail/lhead`、`samehead` 实机验证
- [ ] 深色主题下"注入颜色 vs 用户颜色"的最终渲染效果（需截图确认）
- [ ] SVG 导出中的 `URL`/`tooltip`/`class`/`stylesheet` 是否值得收录（T2 判断）
