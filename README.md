# DFLegacy-Winlator

<a href="#"><img src="https://img.shields.io/badge/Android-Winlator-3DDC84" alt="Winlator" /></a>
<a href="#"><img src="https://img.shields.io/badge/Direct3D-8%20%E2%86%92%209-0078D4" alt="D3D8 to D3D9" /></a>
<a href="#许可证"><img src="https://img.shields.io/badge/license-0BSD-2E7D32" alt="0BSD" /></a>

在 Android（[Winlator](https://github.com/brunodev85/winlator)）与 ARM Linux（Wine + Box64）上运行
**DFLegacy 60 版单机端**的适配层与部署说明。

配套源码仓库：[Yoshino-Loli/DFLegacyAot](https://github.com/Yoshino-Loli/DFLegacyAot)

---

## 为什么需要这个

DFLegacy 的客户端是 2008 年的 32 位 Windows 程序，**只认 Direct3D 8**。
而 Android 上的 DXVK 只支持 D3D9 及以上，直接跑会黑屏、花屏或崩溃。

本仓库提供一层 `d3d8.dll`（[d3d8to9](https://github.com/crosire/d3d8to9) 包装），
把 D3D8 调用逐条翻译成等价的 D3D9 调用：

```
DNF.exe  →  d3d8.dll (d3d8to9)  →  D3D9  →  DXVK  →  Vulkan  →  Adreno / Mali
```

这样就绕开了 Wine 自带的 `wined3d → OpenGL` 路径，在 Android 上能走 Vulkan 后端。

## 使用

### 1. 放置文件

把 `d3d8/d3d8.dll` 复制到**游戏目录根下**，与 `DNF.exe` 同级：

```
Dnf60\
├─ DNF.exe
├─ d3d8.dll          ← 放这里
├─ ijl15.dll
├─ DFLegacy.Server.exe
└─ ...
```

> **Windows 本机不需要这个文件。** 原生的 D3D8 路径在 Windows 上工作正常，
> 放进去反而会改变渲染路径。只在 Wine / Winlator 环境下使用。

### 2. Winlator 容器配置

| 选项 | 建议值 | 说明 |
| --- | --- | --- |
| Box86 / Box64 | **两个都开** | 客户端与服务端都是 32 位 |
| DXVK 版本 | 2.x（不行就换 1.10.x） | 1.x 对老 D3D9 更宽容 |
| 显卡驱动 | Turnip（骁龙）/ VirGL（其他） | 骁龙 8 系优先 Turnip |
| Windows 版本 | Windows 7 | 客户端是 2008 年的，不需要更高 |
| 分辨率 | 与屏幕匹配，保持 4:3 | 例如 1600×1200 |
| 环境变量 | `WINEDLLOVERRIDES=d3d8=n,b` | 确保加载本地 d3d8.dll |

### 3. 启动

在 Winlator 里直接运行 `DNF.exe`。

客户端会通过 `ijl15.dll` 的启动引导自动拉起服务端（无窗口），
约 1~2 秒后进入登录界面。

## 已知限制

| 项目 | 状态 | 说明 |
| --- | --- | --- |
| `TerSafe.dll` | ✅ 无害 | 腾讯 2007 年的安全模块，会加载但连不上后端，静默失效 |
| `ijl15.dll` 的 WIC 解码 | ⚠️ 未验证 | 依赖 Windows Imaging Component 解 JPEG，Wine 9 下可用但**未在真机验证** |
| 服务端自动拉起 | ⚠️ 可能失败 | 依赖 `JOB_OBJECT_LIMIT_KILL_ON_JOB_CLOSE`，Wine 支持不完整。失败时手动启动 `DFLegacy.Server.exe` 即可 |
| 退出联动清理 | ⚠️ 同上 | 若 Job Object 不生效，关游戏后服务端会残留，需手动结束 |
| 反作弊 | ✅ 无影响 | 客户端没有任何内核级保护，`ijl15.dll` 的 39 处代码补丁 + IAT 钩子在 Wine 下同样有效 |

## 性能预期

DNF 是 2D 精灵渲染，GPU 压力很小，瓶颈在 Box86 的 CPU 翻译。

| 设备档位 | 预期帧数 |
| --- | --- |
| 骁龙 8 Gen 1 / 8 Gen 2 | 25~40 fps |
| 骁龙 870 / 778G | 15~25 fps |
| 更早的 SoC | 大概率不可玩 |

启动阶段要索引 `Script.pvf` 里的 48,351 个脚本文件，**冷启动可能长达 1~3 分钟**，
不要以为卡死了。

## 不包含什么

**游戏客户端本体不在本仓库，也不提供下载。**
`DNF.exe`、`Script.pvf`、`ImagePacks2` 等资源的权利归原权利人（Neople / 腾讯）所有，
请自行从合法途径获取并确保使用行为的合法性。

本仓库只包含：

- `d3d8/d3d8.dll` —— d3d8to9 包装层（BSD-3-Clause，见同目录 LICENSE）
- 配置模板
- 部署文档

## 目录结构

```
d3d8/     d3d8to9 包装层（BSD-3-Clause）
config/   Config.ini 与 server.json 的模板
docs/     详细说明
```

## 许可证

本仓库的文档与配置模板采用 **0BSD**。
`d3d8/d3d8.dll` 及其源码来自 [crosire/d3d8to9](https://github.com/crosire/d3d8to9)，
采用 **BSD-3-Clause**，版权归 Patrick Mours 所有，详见 `d3d8/LICENSE.md`。
