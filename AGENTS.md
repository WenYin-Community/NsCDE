# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 项目概述

NsCDE（Not so Common Desktop Environment）是一个基于 FVWM 的复古 CDE 风格 UNIX 桌面环境，使用 Korn Shell (ksh93)、Python 3、FvwmScript 和 C 语言构建。

## 构建与安装

```sh
# 标准 autoconf/automake 流程
./autogen.sh        # 需要 autoconf/automake，首次或修改 configure.ac 后使用
./configure         # 检测依赖：ksh93、FVWM、python3、X11 headers 等
make
make install        # 默认安装到 /usr/local，需要 root 权限
```

`./configure --prefix=/opt/NsCDE` 可自定义安装路径。`--with-python-shebang=` 可指定 Python 解释器。

**构建产物**：`configure` 脚本处理 `.in` 文件生成最终配置（`@KSH@`、`@PREFIX@`、`@VERSION@` 等占位符被替换）。`Makefile.am` 驱动所有子目录安装。

## 核心架构

### 入口与启动流程
- `bin/nscde.in` → 主启动脚本（ksh），设置环境变量后 `exec fvwm` 启动
- `data/fvwm/Main.fvwmconf.in` → FVWM 主配置入口，加载所有 `.fvwmconf` 模块
- 用户配置目录：`$HOME/.NsCDE`（`$FVWM_USERDIR`）

### 目录结构关键说明
| 目录 | 作用 |
|---|---|
| `data/fvwm/` | FVWM 配置文件（`.fvwmconf`），包括 Style、Colorset、Menus、Functions 等模块化配置 |
| `data/palettes/` | `.dp` 文件，CDE 色板定义，由 Color Style Manager 消费 |
| `data/config_templates/` | Xdefaults、Xscreensaver、Dunst、Rofi 等配置模板 |
| `data/defaults/` | 默认 FrontPanel 布局、子面板、WSM 配置 |
| `lib/python/` | Python 3 主题引擎：`Theme.py`（GTK/Qt 颜色主题生成）、`MotifColors.py`（颜色转换）、`ThemeGtk.py`（GTK 专用） |
| `lib/scripts/` | FvwmScript GUI 应用程序：ColorMgr、FontMgr、BackdropMgr、StyleMgr 等 Style Manager |
| `lib/fvwm-modules/` | FVWM 模块 |
| `nscde_tools/` | ksh 辅助脚本（`.in` 文件），如 `colormgr`、`backdropmgr`、`chtheme`、`bootstrap` 等 |
| `src/` | C 程序源码：`colorpicker`（颜色拾取器）、`pclock`（时钟）、`XOverrideFontCursor` |
| `po/` | 国际化翻译文件（目前仅克罗地亚语 `.hr.po`） |
| `pkg/` | 打包配置（Debian、RPM、pacman） |
| `patches/` | FVWM2 补丁 |

### `.in` 文件机制
大量文件使用 `.in` 后缀（`nscde.in`、`Theme.py.in`、`*.fvwmconf.in` 等），这是 autoconf 模板文件。`configure` 运行时将 `@VARIABLE@` 占位符替换为实际值（如 `@KSH@` → `/bin/ksh`、`@NSCDE_DATADIR@` → 安装路径）。修改 `.in` 文件后需重新运行 `./configure` 生效。

### 关键运行时路径（安装后）
- `$NSCDE_ROOT`（默认 `/usr/local`）
- `$NSCDE_DATADIR` = `$NSCDE_ROOT/share/NsCDE` — 数据文件
- `$NSCDE_LIBDIR` = `$NSCDE_ROOT/lib/NsCDE` — Python 库和 FvwmScript
- `$NSCDE_TOOLSDIR` = `$NSCDE_ROOT/libexec/NsCDE` — ksh 工具脚本
- `$FVWM_DATADIR` = `$NSCDE_DATADIR/fvwm` — FVWM 配置
- `$FVWM_USERDIR` = `$HOME/.NsCDE` — 用户配置（首次启动由 `bootstrap` 脚本初始化）

## 开发关键点

- **Shell 脚本必须使用 ksh93**（AT&T Korn Shell），非 bash/sh。工具脚本在 `nscde_tools/` 中
- **FvwmScript** 是 FVWM 内置的 GUI 脚本语言，`lib/scripts/` 下的文件是 GUI 管理器程序
- **Python 依赖**：`PyQt5`（或 `PyQt4`）、`PyYaml`、`pyxdg`、`psutil`
- **C 组件**编译需要 `libX11-devel`、`libXext-devel`、`libXpm-devel`
- **国际化**：使用 gettext，`.po` 文件位于 `po/`，通过 `msgfmt` 编译为 `.mo`
- **版本信息**：如果在 git 仓库中运行 `configure`，版本号会附加 git describe 信息
- **FVWM2 与 FVWM3**：两者都支持，`FrontPanel.fvwm2.fvwmconf` 和 `FrontPanel.fvwm3.fvwmconf` 分别处理

## 常用操作

```sh
# 重新生成 autotools 文件
./autogen.sh

# 检查构建配置
./configure 2>&1 | tail -20   # 查看配置摘要

# 仅编译 C 组件
make -C src

# 仅编译翻译文件
make -C po

# 清理构建产物
make clean
make distclean   # 连 configure 生成的文件一起清理
```
