# gif工具（loop-gif-maker）

把插画拆成同尺寸透明PNG拖进来，逐层加动画，一键出循环GIF

四种常用动画——位移、旋转、缩放呼吸、扫光——逐层叠加，浏览器里直接编码出 GIF，**不需要 ffmpeg、不需要后端、不需要联网**。

[打开工具 →](https://abububu903.github.io/loop-gif-maker/) · [English](#english)

---

## 打开就用

👉 **[abububu903.github.io/loop-gif-maker](https://abububu903.github.io/loop-gif-maker/)**

不用安装、不用注册。想本地跑：下载 `index.html` 双击打开即可——**页面里没有任何 CDN 引用，所有依赖都打包在这一个文件内，完全离线可用**。

## 三步跑通

1. 把透明 PNG 拖进左侧「画布素材」（或点击选择）。**第一张切片的尺寸就是画布基准尺寸**，后续切片必须与它同宽同高，否则会被弹窗拦下、不入列。列表里越靠上的层越在前
2. 选中一个层，在右侧「位移 / 旋转 / 缩放呼吸 / 扫光」里填参数
3. 点「导出 GIF」

第一次上手建议直接点 **文件 → 载入测试用例**，里面是一个 5 层示例（背景 + 扫光 + 呼吸 + 旋转 + 位移），四个动画同时开着，改参数立刻能看出各自的作用。

## 四种动画和它们的参数

| 动画 | 参数 | 说明 |
| --- | --- | --- |
| **位移** | X振幅、Y振幅、相位 | 正弦往复。相位用来让多层错开节奏，不同时「齐步走」 |
| **旋转** | 圈数、锚点层 | 圈数支持小数（0.25 步进）；锚点层选哪个层就绕哪个层转 |
| **缩放呼吸** | 振幅、锚点 | 锚点可选「中心」「底部居中」或素材自身；底部居中适合做原地呼吸的小人 |
| **扫光** | 启用、强度、带宽、斜率、亮下限、亮上限 | 一条高光带扫过。亮上下限控制作用在哪个亮度区间，避免把暗部也扫亮 |

每层面板底部有「清除该层动画」，一键撤掉这一层的全部动画。

## 第一张切片决定画布尺寸

「画布尺寸」取的是**第一张素材的实际像素尺寸**，不是右侧「导出设置」里的宽 / 高——那两个字段只控制导出 GIF 的尺寸，默认 300×300。

所以拖第二张之前先想清楚第一张是谁。要整批换尺寸，弹窗里给了正规做法：点左栏「清空」删掉全部素材，再拖入新的第一张切片。

尺寸不一致时不会静默错位，会直接弹窗跳过并列出每张的 `文件名：宽×高`。
另外，PNG 解码失败（文件损坏、不是真 PNG）走的是另一条提示，红色 toast「这几张 PNG 读不出来：xxx」，不弹窗。

## 导出设置

| 项 | 范围 |
| --- | --- |
| 宽 / 高 | ≥ 16 px |
| 帧数 | ≥ 2 |
| 时长 ms | ≥ 20，步进 10 |

### ⚠️ 时长为什么下限是 20 ms

GIF 的延时字段单位是百分之一秒，本工具的编码器按「不早于 2cs」写入。
如果你填低于 20 ms，用 PIL 之类的命令行工具转出来的文件会按 1cs 写，**同一个动画速度差一倍**。
所以这里直接不让填，不是功能缺失。

## 工程文件

**文件 → 保存文件（含素材）** 会把 PNG 素材以 base64 一起写进一个 JSON，
下次用 **导入文件（JSON）** 原样恢复，素材不用重新拖。想改成品尺寸或参数，不用重画。

## 已知边界

- **GIF 格式本身限 256 色**，本工具用中位切分（median cut）量化，**不做抖动（dithering）**。渐变和照片类素材会有色带，插画类没问题
- **调色板只按第一帧统计一次**，后续帧复用。如果你的动画在后面的帧里才出现新颜色，那些颜色会被映射到最接近的已有色。规避办法：让第一帧就包含全部会出现的颜色
- 默认导出 **300×300、32 帧、50 ms/帧**，其中 32 帧属于高流畅度设置
- 帧数 × 尺寸太大时编码会明显变慢，纯前端单线程

## 免责

作者：abububu903

## 文件说明

| 文件 | 作用 |
| --- | --- |
| `index.html` | 工具本体，界面 + 动画数学 + GIF 编码器全在里面（约 94 KB，含内嵌测试用例） |
| `README.md` | 你正在看的这个说明页，GitHub 自动展示在仓库首页 |

---

## English

Turn layered transparent PNGs into a looping GIF, entirely in the browser. Four animations — translate, rotate, scale-breath and a shimmer sweep — applied per layer, then encoded to GIF by a built-in median-cut + LZW encoder. **No ffmpeg, no backend, no network.**

- **Live page:** <https://abububu903.github.io/loop-gif-maker/> — or download `index.html` and open it. There are zero CDN references; everything ships in that one file and it works fully offline.
- **Workflow:** drop transparent PNGs into the layer list (top of the list = foreground), pick a layer, set its animation, export. Start with **File → Load test case** for a working 5-layer example with all four animations active.
- **The first slice sets the canvas size.** Every later slice must match it exactly — mismatched files are rejected with a dialog listing each offending `name: WxH`, never silently misaligned. The 宽/高 fields in the export panel only size the output GIF (default 300×300). To change the batch size, clear all assets and drop a new first slice.
- **Frame delay floor is 20 ms on purpose.** GIF stores delay in hundredths of a second and this encoder never writes below 2cs; a CLI tool like PIL will write 1cs, so the same animation plays twice as fast. The floor keeps the two in agreement.
- **Project files:** *Save file (with assets)* writes a JSON with the PNGs embedded as base64; *Import* restores the whole scene.
- **Limits:** GIF caps at 256 colours, quantised by median cut with **no dithering** — gradients band. The palette is built from **frame 1 only** and reused, so colours that first appear in later frames get mapped to the nearest existing entry; make sure frame 1 already contains them. Encoding runs on the main thread, so large frame counts get slow. Defaults are 300×300, 32 frames, 50 ms per frame.

*By abububu903.*
