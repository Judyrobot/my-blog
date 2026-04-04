---
title: "基于 Unstructured + PaddleOCR + DeepSeek-OCR 的智能 PDF 解析技术路线复盘"
date: 2026-04-04T20:00:00+09:00
draft: false
tags: ["PDF解析", "OCR", "Unstructured", "PaddleOCR", "DeepSeek-OCR", "YOLOX"]
---

# 基于 Unstructured + PaddleOCR + DeepSeek-OCR 的智能 PDF 解析技术路线复盘

最近终于把一条完整的智能 PDF 解析链路跑通了。整个方案从最开始的 `fast` 策略文本提取，到后面的 `hi_res`、版面检测、Paddle OCR、表格图块导出，再到 DeepSeek-OCR 对表格进行结构化增强，基本把一条复杂 PDF 的解析链完整打透了。

最终效果比预期要好，尤其是 **YOLOX 在版面检测阶段对表格区域的提取效果明显优于 detectron2_onnx**，这一点对后续表格结构化质量提升非常关键。

这篇文章主要做一个完整复盘，梳理这个项目的核心技术路线、关键模块分工、踩坑点，以及最终可落地的方案。

---

# 一、项目目标

这个项目的核心目标不是单纯做“PDF 转文本”，而是：

- 对复杂 PDF 做结构化解析
- 识别标题、正文、图片、表格等版面元素
- 对复杂表格做更高质量的结构化重建
- 最终输出统一 JSON，方便后续检索、知识库入库和下游问答

所以本质上，这是一个 **多模型协同的智能文档解析系统**，而不是一个简单的 OCR 脚本。

---

# 二、整体技术路线

整个系统采用的是分阶段流水线：

1. **数据接入层**
   - 支持本地 PDF
   - 支持数据库二进制 PDF

2. **文档解析主链**
   - 使用 `unstructured.partition.pdf.partition_pdf`
   - `fast` 模式用于基础文本提取
   - `hi_res` 模式用于复杂版面解析

3. **版面检测（Layout Detection）**
   - 使用 `detectron2_onnx` / `yolox`
   - 负责识别页面中的 `Title`、`Text`、`Table`、`Image` 等区域

4. **OCR 识别**
   - 使用 `Paddle OCR`
   - 对版面检测后的区域做文字识别

5. **表格局部增强**
   - 提取 `Table` 元素对应的图块
   - 调用 `DeepSeek-OCR` 做 markdown / HTML 风格重构

6. **统一结构化输出**
   - 将所有元素统一转换成分页 JSON
   - 表格节点额外输出 `table_parts`

一句话概括就是：

> Unstructured 负责文档拆块，Paddle OCR 负责识字，DeepSeek-OCR 负责复杂表格重构。

---

# 三、三类核心组件分别做什么

## 1. Unstructured：文档拆块与主解析框架

`unstructured` 在这个项目中扮演的是 **主解析框架** 的角色。

它的作用不是简单 OCR，而是负责：

- 读取 PDF
- 根据策略选择 `fast` 或 `hi_res`
- 对页面进行版面分析
- 产出统一的元素列表 `elements`

这些元素可能包括：

- `Title`
- `NarrativeText`
- `UncategorizedText`
- `Image`
- `Table`

也就是说，`unstructured` 给整个系统提供了统一的“文档结构骨架”。

---

## 2. Paddle OCR：基础文字识别

Paddle OCR 负责把检测到的区域中的文字真正识别出来。

它主要解决的问题是：

- 某块图像区域里到底写了什么字
- 对扫描版 PDF 或图片型区域进行文字提取

在我的方案里，Paddle OCR 是主 OCR 引擎。

简单说：

- layout model 负责“框哪里”
- Paddle OCR 负责“读里面的字”

---

## 3. DeepSeek-OCR：表格增强与结构重构

DeepSeek-OCR 并不是全文主 OCR 引擎。

在我的代码里，它只处理 **Table 元素对应的图块**，主要用来做：

- 表格重构
- markdown 生成
- HTML table 生成
- 更强的表格结构表达

所以它更像是：

> 面向复杂表格的视觉语言增强模块

这一步的设计思路很重要：**不是全量调用大模型，而是只对高价值的复杂表格局部调用**，从而平衡效果和成本。

---

# 四、为什么要分成这三层

如果只靠一种方法，问题会很多：

## 只用 fast
优点：
- 快
- 依赖少

缺点：
- 对扫描版 PDF 很差
- 复杂表格几乎无能为力

## 只用 OCR
优点：
- 扫描件也能处理

缺点：
- 没有版面结构
- 表格经常变成一坨碎文本

## 只用大模型看整页
优点：
- 理论上能力强

缺点：
- 成本高
- 延迟高
- 全量处理不经济

所以合理方案就是：

- `Unstructured + Layout Model` 做版面结构
- `Paddle OCR` 做基础文本识别
- `DeepSeek-OCR` 只增强复杂表格

这也是这个项目最核心的架构思想。

---

# 五、核心数据流

整个数据流大概是这样的：

## 第一步：读取 PDF
项目封装了统一的 `PdfSource`，支持：

- `LocalPdfSource`
- `SqlServerPdfSource`

这样上层逻辑不依赖 PDF 来源。

## 第二步：调用 partition_pdf
通过 `partition_pdf(...)` 进入主解析链。

- `fast`：偏向原生文本提取
- `hi_res`：走版面检测 + OCR

## 第三步：拿到 elements
主链输出的核心中间结果是 `elements`，每个 element 都是一个版面元素。

例如：

- 标题
- 正文
- 图片
- 表格

## 第四步：对 Table 做局部增强
如果开启 `use_table_llm=True`，就只遍历 `Table` 元素：

- 读取 `image_base64`
- 还原成表格裁图
- 送进 `DeepSeek-OCR`
- 返回 markdown / HTML 风格结果

## 第五步：统一转成 JSON
最终所有元素进入统一 JSON 输出：

- 普通文本节点：`page + index + type + text`
- 表格节点：`page + index + type + text + table_parts`

---

# 六、table_raw 和 table_content 的区别

这是整个项目里一个非常重要的点。

## 1. table_raw
表示：

- 这张表格没有被成功重构成真正二维表
- 只有一段文本化的表格内容

本质上是兜底结果。

适合：
- 保底留存信息
- 关键词检索
- 排查识别效果

## 2. table_content
表示：

- 这张表格已经被成功重构成结构化表格
- 有二维数组 `list_table`
- 有 `html_table`

适合：
- 后续表格抽取
- 转 DataFrame
- 转 Excel/CSV
- 知识库结构化入库

所以可以简单理解成：

> `table_raw` 是兜底文本  
> `table_content` 是真正的结构化表格结果

---

# 七、为什么 YOLOX 效果更好

这一轮调试里，我最明显的体感就是：

**YOLOX 对版面检测的实际效果明显更好，尤其是在表格区域提取上。**

之前用 `detectron2_onnx` 时，常见问题包括：

- 表格漏检
- 表格切碎
- 表格区域不完整

切到 `yolox` 之后，效果明显改善：

- 表格召回更好
- 表格图块更完整
- 导出的表格裁图更接近真实表格区域
- 后续 DeepSeek-OCR 处理结果也更稳定

这说明在我的文档场景下，**layout model 的选择会直接影响下游 OCR 与表格结构化质量**。

也就是说：

> 版面检测效果越好，后面的 OCR 和 DeepSeek-OCR 发挥空间就越大。

---

# 八、调试过程中的关键方法

这个项目能跑通，一个很重要的经验就是：

> 不要只看最终 JSON，一定要看中间产物。

我中间重点做了这些调试：

## 1. 打印 hi_res 参数
确认：

- strategy 是否真的为 hi_res
- 是否真的使用了 yolox
- 是否真的启用了 Paddle OCR
- 离线缓存路径是否正确

## 2. 检查 HF 本地缓存
由于服务器不能联网，所以需要提前把版面检测模型下载好，并放进 Hugging Face cache 目录。

## 3. 导出表格图块
把 `Table` 元素对应的裁图落盘，直接看：

- 表格检测框准不准
- 是否漏表
- 是否切碎

这一步非常有帮助。

## 4. 单独保存 DeepSeek-OCR 输出
把每张表格生成的 markdown 单独写成 `.md` 文件，方便和表格裁图做一一对照。

---

# 九、踩过的主要坑

这个项目跑通之前，环境问题占了大量时间。

## 1. Windows 下 hi_res 环境不稳定
主要涉及：

- `unstructured`
- `unstructured-inference`
- `unstructured-paddleocr`
- `poppler`
- 本地 detection 模型缓存
- Hugging Face 离线模式

## 2. Paddle OCR 版本兼容
Paddle、PaddleOCR、桥接层之间并不是任意最新版都能平滑兼容。

## 3. numpy / pandas 二进制冲突
这个是典型老坑，报错通常类似：

- `numpy.dtype size changed`
- `binary incompatibility`

## 4. unstructured_paddleocr 桥接包
即使 `paddleocr` 本体能用，也不代表 `unstructured` 内部 OCR agent 一定能用，因为它还依赖桥接层。

---

# 十、项目亮点总结

如果后面要拿去面试，我会总结成这几个亮点：

## 1. 多模型协同
不是单模型硬吃全场，而是：

- layout model 做版面检测
- Paddle OCR 做基础识别
- DeepSeek-OCR 做表格增强

## 2. 工程化分层
系统分成：

- 数据接入层
- 配置层
- 解析主链
- 表格增强层
- JSON 输出层

可维护性和扩展性都更强。

## 3. 成本控制
DeepSeek-OCR 不是全量调用，而是只对 `Table` 局部调用，兼顾精度和效率。

## 4. 实际落地能力
解决了：

- Windows 环境部署
- 离线模型缓存
- OCR 适配层
- 依赖冲突
- 调试中间产物

---

# 十一、最终结论

这次项目跑通之后，我对整个技术路线的理解更加清晰了：

- `unstructured` 负责搭建文档解析主链和统一元素结构
- `layout model` 决定版面检测质量
- `Paddle OCR` 提供基础文字识别
- `DeepSeek-OCR` 负责复杂表格增强
- `YOLOX` 在我当前场景下明显优于 detectron2_onnx

最终系统不是简单地“抽文字”，而是实现了一条：

> 从 PDF 到结构化 JSON 的多阶段智能解析路线

这套方案已经具备较强的工程落地价值，后续如果继续优化，我会重点关注：

- 更好的自定义 layout model
- 表格结构重建的稳定性
- JSON 输出规范进一步统一
- 下游知识库和问答系统接入

---

# 十二、后续优化方向

后面如果继续做，我认为可以沿着这几个方向迭代：

## 1. 自定义版面检测模型
如果现有 YOLOX 仍有漏检场景，可以考虑训练自己的版面检测模型，并接入 `unstructured-inference`。

## 2. 表格后处理增强
对 `table_content` 做进一步清洗，例如：

- 自动识别表头
- 对齐列
- 处理合并单元格
- 标准化输出 schema

## 3. 文档类型自适应
不同类型 PDF：

- 技术说明书
- 发票
- 合同
- 论文

适合的解析策略不同，可以做策略路由。

## 4. 接入下游应用
最终 JSON 可以继续用于：

- 向量化检索
- RAG
- 表格问答
- 知识库入库

