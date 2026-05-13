---
name: sts2-ritsulib-mod
description: Build and maintain Slay the Spire 2 RitsuLib mods. Use when working on STS2 mod projects, RitsuLib APIs, Slay the Spire 2 modding, 杀戮尖塔2模组, creating cards, relics, characters, events, monsters, potions, ancients, timelines, enchantments, encounters, patching base-game behavior, querying RitsuLib docs/API, locating original game implementations, or scaffolding RitsuLib mod projects.
---

# STS2RitsuLibMod Skill

为 Slay the Spire 2 + RitsuLib Mod 开发提供通用的源码发现、获取、索引和查询能力。

## 跨平台支持

支持 Windows、macOS 和 Linux，使用 PowerShell Core 脚本实现跨平台兼容。

| 平台 | Steam 发现 | 游戏反编译 | 说明 |
|------|-----------|-----------|------|
| Windows | ✅ 注册表 | ✅ ilspycmd | 完全支持 |
| macOS | ✅ 默认路径 | ✅ ilspycmd | 完全支持 |
| Linux | ✅ 默认路径 | ✅ ilspycmd | 完全支持 |

**前提条件**：安装 PowerShell Core 和 .NET SDK

## 触发条件

- 用户提到 STS2 mod、RitsuLib、杀戮尖塔2模组
- 用户要求创建卡牌、遗物、角色、事件、怪物、药水、先古、时间线、附魔、遭遇
- 用户需要 patch 原版游戏行为
- 用户需要查找 RitsuLib API 或文档

## 核心能力

1. **自动发现与获取**：定位游戏 DLL、RitsuLib 源码，必要时自动反编译/克隆
2. **索引构建**：为游戏源码、RitsuLib 文档和 API 建立可查询索引
3. **智能查询**：按内容名、类型、功能域、本地化键查找原版实现
4. **Mod 脚手架**：基于 RitsuLib 模板生成 Mod 项目结构
5. **项目创建**：使用 NuGet 模板 `STS2.RitsuLib.ModTemplate` 创建标准 Mod 项目

## 配置与路径解析

### 配置优先级

```
1. Skill 全局配置 config.json
2. 缓存目录中已有的资源
3. 自动发现 Steam 安装目录
4. 自动拉取 RitsuLib 和教程仓库
5. 自动反编译游戏 DLL
```

### 全局配置文件

`config.json`（位于 Skill 根目录）：

```json
{
  "gameDll": "",
  "gameSourceRoot": "",
  "ritsulibRoot": "",
  "tutorialsRoot": "",
  "localizationLang": "zhs"
}
```

- 留空则自动发现
- **自动发现的结果会自动保存到配置文件**，下次无需重新发现
- 所有项目共享此配置
- `localizationLang`：本地化语言（zhs=简中, eng=英文, zht=繁中）

## RitsuLib 获取

- 若未找到本地 RitsuLib，则 clone `https://github.com/BAKAOLC/STS2-RitsuLib.git` 到 skill 缓存目录
- 若缓存已存在，则使用 `git fetch` + `git pull --ff-only` 更新
- 文档索引源：`Docs/pages/guide/`
- 源码索引源：仓库内所有 C# 文件

## 教程文档获取

- 从 `https://github.com/GlitchedReme/SlayTheSpire2ModdingTutorials.git` 克隆教程仓库
- 主要读取 `RitsuLib` 目录下的教程内容
- 若缓存已存在，则使用 `git fetch` + `git pull --ff-only` 更新

## 游戏源码获取

- 自动读取 Steam 注册表和 `libraryfolders.vdf`，查找 app `2868840` 的安装目录
- 定位 `data_sts2_windows_x86_64\sts2.dll`
- 无现成源码时用 ILSpy CLI 反编译：`ilspycmd -p -o <output> <sts2.dll>`
- 输出到缓存目录，不写入游戏目录
- 若 `ilspycmd` 不存在，提示安装：`dotnet tool install -g ilspycmd`

## 索引与查询

索引只保存导航信息（文件路径、行号、类型签名），不复制大段源码。源码按需读取原文件。

### 索引类型

| 索引 | 内容 | 用途 |
|------|------|------|
| `ritsulib-docs` | RitsuLib 文档标题、路径、摘要 | 查找 RitsuLib 功能说明 |
| `ritsulib-api` | RitsuLib 公共类型、方法、属性签名 | 查找 RitsuLib API |
| `sts2-content` | 原版卡牌、遗物、角色、事件等类名和路径 | 查找原版内容实现 |
| `sts2-localization` | 本地化键值对 | 查找游戏文本 |
| `tutorials` | 教程标题、路径、摘要 | 查找 Mod 开发教程 |

### 内置索引

项目内置了以下索引（位于 `indexes/` 目录），用户无需运行脚本即可查询：

- `ritsulib-api.json` - RitsuLib API 索引
- `ritsulib-docs.json` - RitsuLib 文档索引
- `tutorials.json` - 教程索引
- `sts2-content.json` - 游戏内容索引
- `sts2-localization.json` - 游戏本地化索引

### 查询命令

```powershell
# 查询原版卡牌
pwsh -File scripts/query-index.ps1 -Type card -Name "Abrasive"

# 查询 RitsuLib API
pwsh -File scripts/query-index.ps1 -Type ritsulib-api -Keyword "AddCard"

# 查询本地化
pwsh -File scripts/query-index.ps1 -Type localization -Key "ABRASIVE"

# 查询教程
pwsh -File scripts/query-index.ps1 -Type tutorial -Keyword "卡牌"
```

### 更新索引

当 RitsuLib 或教程有更新时，运行以下命令同步索引：

```powershell
pwsh -File scripts/update-indexes.ps1
```

## 创建 Mod 项目

使用 NuGet 模板创建标准 RitsuLib Mod 项目：

```powershell
# 创建新 Mod 项目
pwsh -File scripts/create-mod.ps1 -Name "MyAwesomeMod"

# 指定输出目录
pwsh -File scripts/create-mod.ps1 -Name "MyMod" -OutputDir "D:/Projects"
```

模板基于 [STS2.RitsuLib.ModTemplate](https://www.nuget.org/packages/STS2.RitsuLib.ModTemplate/)，包含：
- 示例角色、卡牌、遗物
- Godot 场景资源
- 自动配置的构建流程

创建后需要：
1. 复制 `local.props.template` 为 `local.props`
2. 编辑 `local.props` 设置游戏安装目录等路径
3. 运行 `dotnet build` 构建项目

## Mod 编写流程

### 0. 创建项目（如果是新项目）

```powershell
pwsh -File scripts/create-mod.ps1 -Name "MyMod"
```

### 1. 初始化阶段

每次任务开始时：
```
1. 调用 discover-roots.ps1 发现所有路径
2. 缺失 RitsuLib 时调用 acquire-ritsulib.ps1
3. 缺失教程仓库时调用 acquire-tutorials.ps1
4. 缺失游戏源码时调用 decompile-sts2.ps1
5. 调用 build-index.ps1 刷新过期索引
```

### 2. 查询阶段

根据用户需求查询：
- **原版内容参考**：用户提到原版卡牌/遗物/角色时，先通过索引定位，再读取反编译源码和本地化
- **RitsuLib API**：用户提到 RitsuLib 功能时，先读文档，再读源码确认 API 签名
- **教程参考**：用户需要学习 Mod 开发时，查询相关教程
- **Patch 目标**：需要修改原版行为时，读取目标方法和调用方

### 3. 编码阶段

- 优先使用 RitsuLib 的内容注册、模板、生命周期、本地化、存档、设置和 patching API
- 只有 RitsuLib/原生 API 无稳定入口时才写 Harmony patch
- patch 前必须读取目标方法和调用方

### 4. 构建验证

```powershell
dotnet build
```

确认：
- manifest 依赖 `STS2-RitsuLib`
- 不污染游戏目录
- 反编译源码不随 Mod 分发

## 关键原则

- **不修改游戏安装目录**
- **不提交反编译源码到 Mod 项目**
- **不把反编译源码复制进 Mod 项目**
- **用户路径优先于自动发现**
- **索引轻量化，源码按需读取**

## 缓存目录结构

```
~/.claude/skills/STS2RitsuLibModSkill/cache/
├── ritsulib/                    # RitsuLib 克隆
├── tutorials/                   # 教程仓库克隆
├── decompiled/sts2/             # 反编译的游戏源码
└── indexes/                     # 索引文件
    ├── ritsulib-docs.json
    ├── ritsulib-api.json
    ├── sts2-content.json
    ├── sts2-localization.json
    └── tutorials.json
```

## 配置文件格式

`.sts2-mod-builder.json`（放在 Mod 项目根目录，所有字段均可选）：
```json
{
  "gameDll": "D:/SteamLibrary/steamapps/common/Slay the Spire 2/data_sts2_windows_x86_64/sts2.dll",
  "ritsulibRoot": "D:/Projects/STS2-RitsuLib",
  "gameSourceRoot": "D:/Projects/sts2-decompiled",
  "tutorialsRoot": "D:/path/to/SlayTheSpire2ModdingTutorials",
  "modRoot": "."
}
```

**注意**：如果不提供配置文件或路径为空，Skill 会自动：
- 从 Steam 注册表发现游戏 DLL
- 从 GitHub 克隆 RitsuLib 和教程仓库
- 使用 ILSpy 反编译游戏 DLL
