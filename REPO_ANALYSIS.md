# 30dayOS 仓库分析报告

## 1. 仓库定位

该仓库是《30天自制操作系统》配套源码/工具集的完整打包版本，核心目标是按天递进构建一个从引导扇区到图形、多任务、文件系统、应用生态的教学型 x86 操作系统（Haribote OS）。

从内容形态看，它更像“教学发行包”而不是“现代工程仓库”：

- 含完整阶段快照（`projects/01_day` 到 `projects/30_day`）。
- 含专用工具链与模拟器（`tolset/z_tools`，大量 `.exe`）。
- 含附加源码与历史资源（`omake`）。
- 顶层直接给出可启动镜像（`haribote.img`）。

## 2. 结构总览

仓库根目录主要由四块组成：

- `projects/`：主学习区，30 天分阶段代码。
- `tolset/`：教学工具链（汇编器、编译器、镜像工具、QEMU 等）。
- `omake/`：补充源码、工具源码、许可证和额外样例。
- 顶层镜像与说明：`haribote.img`、`readme.txt`。

### 2.1 规模特征（当前仓库快照）

- 天数目录：30
- 阶段目录：204
- `Makefile` 数量：685
- `.c` 文件：3041
- `.nas` 文件：1379
- `.bat` 文件：2163
- 仓库体积：约 `98M`（其中 `projects` 约 `57M`，`omake` 约 `27M`，`tolset` 约 `3.6M`）

结论：这是一个“多快照 + 工具集 + 二进制资源”型仓库，天然较大，不适合只按现代源码仓库方式理解。

## 3. 学习与代码组织模式

`projects` 采用“天 + 小版本”演进：

- 例如 `03_day/harib00a ... harib00j`
- 到后期 `30_day/harib27a ... harib27f`

这种组织方式的价值：

- 每个小版本是可独立构建的里程碑，便于对照书中章节。
- 可通过目录 diff 观察每一步新增内容。

### 3.1 最终阶段（`30_day/harib27f`）结构

可分成两层：

- 内核层：`haribote/`
  - 典型模块：`bootpack.c`、`memory.c`、`mtask.c`、`file.c`、`window.c`、`console.c` 等。
- 应用层：`hello3/`、`calc/`、`invader/`、`mmlplay/`、`gview/` 等。
  - 每个应用通常有独立 `Makefile`，共用 `app_make.txt` 规则。
- API/库层：`apilib/` + `apilib.h`
  - `apilib.h` 定义 `api_putchar/api_openwin/api_getkey/api_fopen` 等系统调用式接口。

这说明最终形态已经是“内核 + 用户程序 + API 库”的微型生态。

## 4. 构建与运行链路

## 4.1 总体链路

在后期阶段（如 `harib27f`）中，构建链路大体是：

1. `nask.exe` 构建 IPL 和汇编对象。
2. `cc1.exe` + `gas2nask.exe` 把 C 转为 NASM 风格汇编再装配。
3. `obj2bim.exe` / `bim2hrb.exe` / `bim2bin.exe` 生成内核与应用格式。
4. `edimg.exe` 打包 FAT12 镜像。
5. `qemu/qemu-win.bat` 启动镜像运行。

## 4.2 关键入口

- 阶段总控：`projects/30_day/harib27f/Makefile`
  - `run`：打包镜像并调用 QEMU
  - `full`：先编内核与全部应用，再打包
  - `run_full`：全量构建后运行
- 内核构建：`projects/30_day/harib27f/haribote/Makefile`
- 应用通用规则：`projects/30_day/harib27f/app_make.txt`
- QEMU 启动脚本：`tolset/z_tools/qemu/qemu-win.bat`

## 4.3 典型使用方式（Windows 原生）

在某个阶段目录（例如 `projects/30_day/harib27f`）下：

```bat
make.bat run
```

或全量：

```bat
make.bat run_full
```

说明：`make.bat` 实际转发到 `..\z_tools\make.exe`，并依赖 `copy/del/cmd.exe` 语义，明显偏 Windows。

## 5. 技术特征与实现重点

从最终阶段源码可见，仓库覆盖了 OS 入门的完整闭环：

- 引导与加载：`ipl09.nas`（FAT12 引导扇区逻辑）
- 中断/PIC/PIT：`int.c`、`timer.c`
- 输入设备：`keyboard.c`、`mouse.c`
- 内存管理：`memory.c`
- 图层窗口系统：`sheet.c`、`window.c`
- 多任务：`mtask.c`
- 控制台与命令：`console.c`
- 文件系统读取：`file.c`（FAT 解析、文件加载）
- 应用接口：`apilib.h` + `apilib/*.nas`

教学价值在于“从裸机到可交互桌面系统”的连续演进，而不是单点模块性能或工业级健壮性。

## 6. 兼容性与工程风险

## 6.1 平台耦合（最明显）

仓库默认依赖 Windows 工具链：

- 大量 `.exe` 工具
- `copy/del` 命令
- `.bat` 驱动流程

在 Linux/macOS 直接执行通常会失败，需要 Wine/兼容层或自行改造 Makefile。

## 6.2 编码问题

注释与部分文本明显是 Shift-JIS/GBK 历史编码语境，UTF-8 终端下会出现乱码；`com_mak.txt` 还专门使用 `sjisconv.exe`。若要二次开发，建议先制定统一编码策略。

## 6.3 二进制资产较多

仓库包含大量预编译工具、镜像、资源文件；可复现实验，但会增加审计、跨平台迁移与 CI 自动化成本。

## 7. 建议的上手路径

如果你是第一次接触这个仓库，建议按下面路径：

1. 先在 `01_day -> 03_day` 跑通“镜像构建 + QEMU 启动”。
2. 跳到 `10_day` 左右看 C/汇编混合构建规则。
3. 重点阅读 `30_day/harib27f/haribote` 的模块划分。
4. 再看 `apilib.h` 与应用目录，理解用户程序接口。
5. 最后对比 `harib27a~f`，观察最终阶段增量（更高效）。

## 8. 一句话结论

这是一个结构清晰但平台年代感很强的教学型 OS 仓库：最适合“跟书逐步学习 + 对照阶段差异”，不适合直接当现代跨平台工程模板使用。
