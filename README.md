# Markora

**原生 macOS 图像标注工作台 · 本地 AI 辅助 · 面向目标检测的数据集工作流**  
**A native macOS image annotation workspace with local AI assistance.**

[中文介绍](#中文介绍) · [English](#english) · [模型兼容 / Model compatibility](#model-compatibility) · [开发与构建 / Build](#build) · [联系 / Contact](#contact)

---

## 中文介绍

Markora 将图片浏览、几何标注、AI 候选审核和数据集整理放在一个 Mac 工作台中。你可以逐张手动标注，也可以让本地检测模型先生成候选，再修正位置、类别和遗漏目标，最后导出训练所需的数据。

它适合普通照片中的目标检测样本制作、已有标注整理，以及小型团队或个人的数据集审核。图片、标注与模型推理均在本机处理，无需创建账户。界面支持**简体中文与英语**。

> 本文对应当前源码功能，工程版本为 **1.1.5（Build 3）**。商店中实际可下载版本以 Mac App Store 页面为准。Markora 是独立应用，不是 LabelMe 官方产品。

### 功能概览

| 模块 | 能力 |
| --- | --- |
| 图片工作台 | 文件夹浏览、缩略图、名称搜索、审核状态筛选、缩放和平移 |
| 手动标注 | 矩形、多边形、圆、点、直线、折线、自由闭合轮廓、旋转框绘制 |
| 编辑操作 | 对象移动、顶点修改、多选、批量改标签、复制、删除、撤销与重做 |
| 连续标注 | 完成并下一张、下一张未完成、复制上一张、跨图重复粘贴标注模板 |
| 类别管理 | 文件夹类别统计、批量重命名与合并、数字键快速切换类别 |
| 本地 AI | 内置目标检测、兼容 Core ML / ONNX 模型导入、单张或整批预标注 |
| 候选审核 | 框和顶点修正、改类别、接受或丢弃、低置信度优先审核 |
| 前景辅助 | 点击或框选前景生成轮廓候选，支持人工修正 |
| 数据交换 | LabelMe JSON、YOLO 检测、COCO 检测、Pascal VOC 检测 |
| 质量检查 | 疑似重复框、空白未完成图片等检查线索 |
| 大文件夹 | 后台目录扫描、缩略图内存与磁盘缓存、异步图片加载、下一张预加载 |
| 界面 | 深蓝三栏布局、蓝青渐变线框启动动画、中英文切换 |

### 快速开始

1. 在 **macOS 14 或更新版本**上打开 Markora。
2. 选择包含图片的文件夹。支持 PNG、JPEG、TIFF、BMP、HEIC。
3. 设置类别，选择矩形或其他工具，在画布上绘制对象；也可以先运行 AI 检测。
4. 检查所有对象，修正错误框并补齐漏检。编辑会自动保存到图片旁的同名 JSON。
5. 点击“完成并进入下一张”。如果一张图包含多个对象，请在全部检查后再标记完成。
6. 按需要导出 YOLO、COCO 或 Pascal VOC 检测数据集。

仓库中的 [Demo](Demo/) 提供示例文件。普通“打开文件夹”只浏览该目录直接包含的图片；整套 YOLO 导入另行处理数据集中的子目录。

### 标注工具与坐标

- **矩形、圆、直线**：按住并拖动绘制。
- **多边形、折线**：逐点点击，Return 或双击结束；多边形可点击起点闭合。
- **自由轮廓**：沿物体边缘拖动，松开后自动闭合，保存为多边形。
- **旋转框**：依次确定起点、边终点和宽度位置，保存为四点多边形。
- **选择工具**：拖动对象或顶点进行修正；按住 ⌘ 或 Shift 点击可多选。

所有标注坐标基于原图像素。自由轮廓是采样折线，不是贝塞尔曲线；旋转框暂不使用原生 OBB 数据格式。

### 减少重复操作

**快捷类别。** 画布获得焦点后，数字键 `1–9` 和 `0` 对应前十个类别；映射显示在界面中。本次打开文件夹期间，新类别追加到映射末尾。

**跨图复用。** 可复制上一张的全部标注，也可把选中对象保存为本次会话的模板，连续粘贴到多张图片。没有选中对象时，复制模板会复制当前全部标注。跨图复制要求图像尺寸一致，保留目标图已有对象，复制后需要调整位置。换文件夹会清空模板。

**整目录改名。** 扫描文件夹后，可批量修改类别名称或合并到已有类别。原 JSON 备份到 `.markora-backups`；受影响图片会重新标为未完成。

**按审核状态推进。** “下一张未完成”只跳转，不会自动将当前图标为完成。编辑已完成图片会取消其完成状态。无目标图片必须主动确认完成，才作为已审核负样本。

### 智能标注与人工审核

内置 **YOLOv3 Tiny FP16 通用 80 类模型**，可直接离线使用。选择模型类别、设置置信度后，可以检测当前图片或批量检测文件夹。

检测结果首先进入待审核候选区。你可以修正框、修改类别、接受或丢弃候选；接受后才写入正式标注。批量检测支持停止后续跑，并跳过已完成人工审核的图片。候选和来源指纹保存在 `.markora-candidates`，用于恢复工作和识别过期结果。

低置信度排序用于安排审核顺序，不能保证发现所有错误。内置模型只识别自身支持的类别；小目标、遮挡和复杂背景仍可能漏检。前景轮廓辅助使用 Apple Vision，不自动判断语义类别，也不是 SAM 的交互式分割实现。轮廓只保留外边界，不表示孔洞。

质量检查会提示同类高度重叠框（IoU ≥ 0.90）和未完成且无对象的图片等问题。这些是**待检查线索**，不是能够理解所有物体的漏标识别器。

### 数据格式与训练

| 格式 | 导入 | 导出 | 范围 |
| --- | --- | --- | --- |
| LabelMe JSON | 支持 | 支持 | 六种基础形状；自由轮廓和旋转框存为 polygon；可导出嵌入图片的 JSON 副本 |
| YOLO 检测 | 单图 TXT、整套目录 | 支持 | 5 列归一化检测框；整套导入需提供按类别 ID 排序的类别列表 |
| COCO 检测 | 支持 | 支持 | bbox 和类别；不导入 segmentation、关键点或 RLE mask |
| Pascal VOC 检测 | 支持 | 支持 | XML 检测框；使用 1 起点、包含末端像素的坐标约定 |

检测数据集导出只使用已完成图片。多边形、圆和旋转框转换为轴对齐外接框；点、直线、折线会被拒绝导出，不会悄悄丢弃。

YOLO 导出包含图片、labels、`data.yaml`、`classes.txt` 和 `manifest.json`，自动进行约 80% / 20% 的训练与验证划分，要求至少两张已完成图片且存在有效对象。该划分不按类别分层，也不识别连拍或同源样本；训练前请根据数据来源复核分组。移动导出目录后，需要更新 `data.yaml` 中的绝对路径。

COCO / VOC 导出不自动拆分训练与验证集。VOC 使用惯例目录名 `JPEGImages`，实际图片统一为 PNG，XML 中的文件名与之对应。整套导入生成新的可编辑项目及来源清单，不修改源数据集。

**PyCharm 是开发环境，不决定标注格式。** 训练程序若实现 LabelMe JSON 读取，就能使用 JSON；通常的 YOLO 检测训练使用导出的 `data.yaml` 和 TXT。Markora 不包含训练环境，也不会自动训练模型。

<a id="model-compatibility"></a>
### 自定义模型兼容范围 / Model compatibility

| 模型 | 当前支持 / Current support |
| --- | --- |
| Core ML `.mlmodel` / `.mlpackage` / `.mlmodelc` | 图像输入、类别列表、NMS 检测输出，能够通过 Vision 返回目标检测结果 / Image input, class labels and NMS detection output recognized by Vision |
| YOLO `.onnx` | 下述固定形状、原始检测张量约定 / Fixed-shape raw detection tensors described below |
| PyTorch `.pt` | 不能直接运行，须在训练环境转换 / Convert in your training environment; no direct execution |

**ONNX 要求 / Requirements:**

- 自包含文件，不大于 512 MB；不使用外部权重。 / Self-contained file up to 512 MB; no external weights.
- 单个 float32 RGB 输入 `[1,3,H,W]`；固定 H/W，范围 32–2048。 / One fixed-size float32 RGB input.
- 单个固定 float32 输出：YOLOv8/11 风格 `[1,4+nc,N]`，或含 objectness 的 `[1,N,5+nc]`。 / One fixed float32 output in either supported layout.
- 包含 `task=detect` 和从 0 开始连续编号的 `names` 类别元数据。 / Detection task metadata and contiguous zero-based class names.
- 预处理为 RGB 0–1、NCHW、等比例缩放并以灰色 114 补边。 / RGB 0–1 NCHW with aspect-preserving resize and 114-gray letterboxing.

不支持任意 ONNX 架构、动态形状、已内置 NMS 的 ONNX 输出、end-to-end / NMS-free 布局、分类、分割、姿态、OBB 或量化输入。文件扩展名相同不代表兼容。 / Arbitrary architectures, dynamic shapes, post-NMS or end-to-end outputs, classification, segmentation, pose, OBB and quantized inputs are not supported.

对于兼容的 Ultralytics 检测权重，可在已有训练环境中运行： / For compatible Ultralytics detection checkpoints, run in your existing training environment:

```sh
# ONNX
python Scripts/export_detector.py /absolute/path/best.pt --format onnx --imgsz 640

# Core ML
python Scripts/export_detector.py /absolute/path/best.pt --format coreml --imgsz 640
```

该环境需具备匹配的 Ultralytics 和导出依赖。App 本身不执行 Python。导出成功仍需通过应用的模型兼容检查；你的训练权重和实际数据应单独验证。 / The script requires matching Ultralytics/export dependencies. The app does not run Python. Validate the exported model in the app and evaluate it on your own data.

### 常用快捷键 / Keyboard shortcuts

| 快捷键 / Shortcut | 操作 / Action |
| --- | --- |
| ⌘O | 打开文件夹 / Open folder |
| ⌘S | 保存 / Save |
| ⌘E | 导出 JSON 副本 / Export JSON copy |
| ⌘Z / ⇧⌘Z | 撤销 / 重做 / Undo / Redo |
| V / R / P / C / L / K / H / F | 选择 / 矩形 / 多边形 / 圆 / 线 / 点 / 平移 / 自由轮廓；Select / Rectangle / Polygon / Circle / Line / Point / Pan / Freehand |
| 1–9 / 0 | 画布聚焦时选择类别 / Select class when canvas is focused |
| ⌘D | 复制选中对象 / Duplicate selection |
| ⇧⌘C / ⇧⌘V | 复制 / 粘贴标注模板 / Copy / Paste annotation template |
| ⌘Return | 完成并下一张 / Mark reviewed and advance |
| ⌘[ / ⌘] | 上一张 / 下一张 / Previous / Next image |
| ⇧⌘] | 下一张未完成 / Next unreviewed image |
| ⌘0 | 适合窗口 / Fit to window |
| Return / Esc | 完成 / 取消草稿 / Finish / Cancel draft |
| Delete | 删除草稿末点或选中对象 / Delete last draft point or selected objects |

滚动平移；双指捏合或 ⌘滚动缩放。 / Scroll to pan; pinch or ⌘-scroll to zoom.

### 性能与数据保护

图片目录和审核索引后台扫描，缩略图按需读取并缓存，切图采用异步加载；快速连续切换时仅应用最后一次选择。下一张图片在设定的像素预算内预加载，避免大图预加载占用过多内存。

默认缩略图缓存约为内存 32 MB / 600 张、磁盘 128 MB / 5000 张；原图缓存最多 2 张 / 96 MB。缓存为有界、定期清理策略，不是应用总内存上限。实际体验取决于图片尺寸、磁盘速度和设备配置。

标注采用自动保存与原子写入，切图前检查保存结果，并检测外部 JSON 修改。支持的 JSON 扩展字段会保留；不支持的形状不会静默删除。保存冲突应先解决或导出副本，再继续切图。普通图片目录不能同时包含 `a.jpg` 和 `a.png` 这类共享同一 JSON 名称的文件。

### 隐私

- 图片、标注和模型推理在本机处理，应用没有网络访问功能。
- 无需账号，不包含广告、行为分析或用户追踪。
- 通过系统文件选择器授权访问文件，使用 macOS App Sandbox。
- 标注、候选、批量改名备份、导入模型、缩略图缓存和语言偏好保存在本地。
- 使用 iCloud 等同步目录时，文件同步由相关服务及其设置决定。

完整政策：[中文](Markora/PrivacyPolicy.txt) · [English](Markora/PrivacyPolicy-en.txt)。

### 当前边界与后续方向

目前仅支持 macOS 14+，不提供 Windows/Linux 版本；不包含视频标注、像素掩膜编辑、贝塞尔控制柄、原生 OBB 数据交换、SAM 提示分割、开放词汇识别或在线协作。天文风格的视觉设计不代表具备 FITS 科学影像处理能力。

后续可重点完善更多模型结构适配、分割与 OBB、质量审核和大型数据集工作流。这些是优化方向，不是当前功能或发布时间承诺。

---

## English

**Markora is a native macOS workspace for image annotation and object detection dataset preparation.** Browse a folder, draw and edit shapes, review predictions from local models, and export data for your training pipeline—all in one desktop app.

The interface supports English and Simplified Chinese. Images, annotations and inference stay on your Mac; no account is required. This README describes source version **1.1.5 (Build 3)**. The version available through the Mac App Store may differ. Markora is an independent application, not an official LabelMe product.

### Highlights

- **Native workspace:** a thumbnail library, central canvas, and annotation/AI inspector in a dark blue interface.
- **Flexible geometry:** rectangles, polygons, circles, points, lines, polylines, closed freehand outlines and rotated rectangles. Freehand outlines and rotated rectangles are stored as polygons.
- **Faster editing:** vertex editing, multi-selection, bulk relabeling, duplication, undo/redo and numeric class shortcuts.
- **Continuous annotation:** mark reviewed and advance, jump to the next unreviewed image, copy the previous image's annotations, or paste a reusable template across equal-sized images.
- **Folder-level organization:** class counts, batch rename/merge with backups, filename search and review filters.
- **Local AI:** a bundled YOLOv3 Tiny FP16 detector with 80 classes, compatible Core ML/ONNX imports, confidence controls and batch prediction.
- **Reviewable suggestions:** adjust boxes or vertices, change labels, accept/reject predictions and prioritize low-confidence images.
- **Foreground assistance:** point/box-guided Apple Vision foreground outlines. This does not provide semantic classification or SAM prompting, and contours do not represent holes.
- **Dataset interchange:** LabelMe JSON plus YOLO, COCO and Pascal VOC object detection data.
- **Large-folder support:** background indexing, thumbnail caches, asynchronous loading and bounded next-image preloading.
- **Launch experience:** a skippable blue-to-cyan wireframe animation that respects Reduce Motion.

### Getting started

1. Run Markora on macOS 14 or later and select an image folder.
2. Choose a class and draw objects, or run local AI detection to generate suggestions.
3. Correct predictions and add missed objects. Edits are saved beside the image as a same-basename JSON file.
4. Mark the image reviewed once all objects have been checked, then advance.
5. Export reviewed data in the format required by your training code.

Supported image extensions include PNG, JPEG, TIFF, BMP and HEIC. The normal folder browser lists direct children; whole-dataset YOLO import separately supports nested dataset directories. See [Demo](Demo/) for sample files.

### Dataset compatibility

| Format | Import / Export | Notes |
| --- | --- | --- |
| LabelMe JSON | Both | Six basic shape types; freehand/rotated shapes use polygons. Portable JSON copies may embed image data. |
| YOLO detection | Both | Five-column normalized boxes. Full import requires an explicitly supplied class list in class-ID order. |
| COCO detection | Both | Bounding boxes and categories; no segmentation, RLE masks or keypoint import. |
| Pascal VOC detection | Both | XML boxes using one-based inclusive coordinates. |

Detection exports convert polygons and circles into axis-aligned bounding boxes and reject points/lines/polylines rather than silently omitting them. Only reviewed images are exported. An empty image becomes a reviewed negative only after explicit confirmation.

YOLO export produces images, labels, `data.yaml`, `classes.txt` and a source manifest. It requires at least two reviewed images and one valid object, and creates an approximately 80/20 train/validation split. The split is not stratified or source-group aware: review it for related images and leakage before training. Update the absolute dataset path in `data.yaml` after moving the export.

COCO/VOC exports do not automatically split train/validation data. VOC keeps the conventional `JPEGImages` directory name but stores PNG files referenced by the XML. Whole-dataset imports create a new editable project without modifying the source dataset.

PyCharm is an IDE, not an annotation format. Use LabelMe JSON only if your training code parses it, or export the detection format expected by your pipeline. Markora does not train models or bundle a Python training environment.

### Models and review workflow

See the shared [model compatibility section](#model-compatibility) for exact input/output requirements and conversion commands. **Direct `.pt` execution and arbitrary ONNX architectures are not supported.** The bundled detector cannot recognize arbitrary custom categories.

Predictions remain separate suggestions until accepted. Batch processing can be stopped and resumed; reviewed images are skipped. Suggestions and source fingerprints are stored under `.markora-candidates` to resume review and invalidate stale results. Low-confidence ordering helps prioritize review but does not measure annotation completeness.

Quality checks flag issues such as highly overlapping same-class boxes (IoU ≥ 0.90) or unreviewed images with no objects. They are review hints, not a guarantee that missing objects have been detected.

### Efficiency, performance and safety

Use `1–9` / `0` to select the first ten classes while the canvas is focused. Multi-select with ⌘/Shift-click. Annotation templates can be pasted repeatedly onto equal-sized images; existing objects are retained, and copied coordinates need review. Templates are session-only and cleared when switching folders. See the shared [shortcut table](#常用快捷键--keyboard-shortcuts).

Folder-level renaming creates `.markora-backups` and resets affected images to unreviewed. Editing a reviewed image also clears its review status. Saving is automatic and atomic, with external JSON conflict detection and save checks before navigation. Unsupported shapes are rejected; supported documents preserve extra fields. Images such as `a.jpg` and `a.png` cannot coexist in the same working folder because they share the sidecar name `a.json`.

Background indexing, memory/disk thumbnail caches and bounded prefetching reduce repeated work. Cache budgets do not cap total application memory, and responsiveness depends on image size, storage and hardware. The app does not continuously monitor every external filesystem change.

### Privacy and limitations

Markora has no network access functionality, account requirement, advertising, analytics or user tracking. File access is granted through the system picker and constrained by App Sandbox. Annotation files, model copies, candidate records, rename backups, caches and preferences are local. Third-party folder synchronization is controlled by the corresponding service. Read the [English privacy policy](Markora/PrivacyPolicy-en.txt).

The current app is macOS-only. It does not support video annotation, raster mask editing, Bézier handles, native OBB interchange, SAM prompting, open-vocabulary recognition or online collaboration. Its astronomy-inspired design does not imply FITS processing support. Future model compatibility, segmentation and review improvements are directions rather than promised releases.

<a id="build"></a>
## 开发与构建 / Build

项目使用 SwiftUI、AppKit、Apple Vision / Core ML 和 ONNX Runtime。需要 macOS、支持 macOS 14 部署目标的 Xcode，以及本机可用的开发签名配置。 / The project uses SwiftUI, AppKit, Apple Vision/Core ML and ONNX Runtime. Build on macOS with an appropriate Xcode installation and your own development signing configuration.

1. 打开 / Open `Markora.xcodeproj`.
2. 选择 / Select the `Markora` scheme.
3. 在 Signing & Capabilities 中选择自己的开发团队；分发时配置自己的 Bundle ID。 / Select your development team and configure an appropriate bundle identifier for distribution.
4. 编译运行 / Build and run.

```sh
# Universal Release; configure your team in Xcode first.
xcodebuild -project Markora.xcodeproj -scheme Markora \
  -configuration Release -derivedDataPath build \
  CODE_SIGN_IDENTITY='Apple Development' \
  'ARCHS=arm64 x86_64' ONLY_ACTIVE_ARCH=NO build

# Regression tests
./Scripts/test.sh
```

ONNX Runtime 动态库需要随应用嵌入并正确签名。开发构建不等同于可直接提交的 App Store 发行包。 / The ONNX Runtime library must be embedded and correctly signed. A development build is not an App Store submission package.

### 测试 / Tests

测试覆盖标注读写、撤销重做、保存冲突、审核流程、数据集导入导出、候选编辑、效率操作、缓存与后台索引、中英文资源及 ONNX 推理。

The test suite covers annotation round trips, undo/redo, save conflicts, review state, dataset interchange, candidate editing, productivity actions, caching/indexing, localization and native ONNX execution.

ONNX 测试使用仓库中确定性的测试图，验证预处理、坐标、类别、阈值、NMS、模型恢复和错误拒绝；它们不是检测精度基准，也不替代真实模型及生产数据验证。 / Deterministic ONNX fixtures verify runtime behavior, not detection accuracy. Validate real models and production datasets separately.

### 仓库结构 / Repository layout

```text
Markora/
├── Markora.xcodeproj/   Xcode project
├── Markora/            App, canvas, models, localization and privacy text
├── Scripts/            Tests, conversion and developer utilities
├── Tests/              Regression tests and deterministic fixtures
├── Vendor/             ONNX Runtime and third-party notices
├── Demo/               Example annotation files
├── Design/             Design assets
├── ReleaseKit/         Release preparation materials
└── docs/               Archived development notes
```

[历史开发记录](docs/DEVELOPMENT-HISTORY.md)仅供追溯，不代表当前功能、版本号或商店状态。 / [Archived development notes](docs/DEVELOPMENT-HISTORY.md) are historical and may contain superseded statements.

## 许可与致谢 / Licensing and acknowledgements

- 内置检测模型说明 / Bundled detector notices: [Markora/Models/NOTICE.txt](Markora/Models/NOTICE.txt) and [Darknet-LICENSE.txt](Markora/Models/Darknet-LICENSE.txt).
- ONNX Runtime 来源及许可 / Runtime provenance and licenses: [SOURCE.txt](Vendor/onnxruntime/SOURCE.txt), [LICENSE](Vendor/onnxruntime/LICENSE), [ThirdPartyNotices.txt](Vendor/onnxruntime/ThirdPartyNotices.txt).
- 自定义模型的使用与分发遵循模型自身条款。 / Custom models retain their own usage and distribution terms.

当前仓库未声明 Markora 应用源码的开源许可证；第三方组件许可不自动适用于应用源码。需要复制、修改或再分发应用源码时，请联系开发者确认授权。 / No open-source license is currently declared for the Markora application source. Third-party licenses do not license the app itself; contact the developer regarding reuse or redistribution.

<a id="contact"></a>
## 联系与反馈 / Contact

**开发者 / Developer:** 胡冬生（Hu Dongsheng）  
**邮箱 / Email:** [hudongsheng356@gmail.com](mailto:hudongsheng356@gmail.com)

反馈时请附上应用版本、macOS 版本、重现步骤和预期行为。模型问题请提供模型格式、任务类型及错误信息；图片或模型文件仅在你愿意且有权分享时提供。

For bug reports, include the app version, macOS version, steps to reproduce and expected behavior. For model issues, include the format, task and error message. Share images or weights only when you are comfortable and authorized to do so.
