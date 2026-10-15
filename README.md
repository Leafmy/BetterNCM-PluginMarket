# BetterNCM-PluginMarket

**BetterNCM v3 的插件市场源。** 一个纯静态目录：索引 + 插件包 + 预览图，没有任何服务端逻辑。

这个仓库是 [`Leafmy/BetterNCM-v3`](https://github.com/Leafmy/BetterNCM-v3) 里 `market/` 目录的
独立发布副本，格式与上游 BetterNCM 的插件市场（`BetterNCM/Plugin-Market`，`PluginMarket` 0.8.4）
**完全一致**，因此可以被上游插件市场直接消费，也可以被 BetterNCM v3 的插件市场作为在线源读取。

## 目录结构

```
plugins.json     索引：顶层数组，上游契约 + 少量附加字段
market.json      源元数据（本项目用，上游忽略）
catalog.json     索引的人工维护输入（审阅信息：预览图/仓库/许可证/时间戳/备注）
plugins/         插件包，命名 <slug>-<version>.plugin
previews/        预览图（SVG）
```

## 怎么用

| 用途 | baseURL |
|---|---|
| GitHub raw | `https://raw.githubusercontent.com/Leafmy/BetterNCM-PluginMarket/main/` |
| jsDelivr（有 CDN 缓存） | `https://cdn.jsdelivr.net/gh/Leafmy/BetterNCM-PluginMarket@main/` |

在 BetterNCM v3 的管理面板里：**插件市场 → 源设置 → 添加自定义源**（填入上面的 baseURL），
再勾选「信任自定义源」；或者把 `tools/market_host.mjs` 的 `DEFAULT_SOURCES` 里对应条目改成
`enabled: true`。

> **本机网络提醒**：这台机器上 `raw.githubusercontent.com` 与 `cdn.jsdelivr.net` 的直连都被阻断
> （走系统代理才通），而客户端注入页面的后端 Node `fetch` **不走系统代理**。所以在本机
> 该源会一直处于「正在后台读取」状态；这是网络问题，不是索引问题。要在本机用在线源，
> 需要给 Node 设代理（`HTTPS_PROXY`）或改用可达的镜像（Gitee、对象存储等）。

## 索引契约

`plugins.json` 是顶层数组，每个条目：

| 字段 | 说明 |
|---|---|
| `slug` `name` `version` `author` | 身份与展示；`slug` 一旦发布不再更改 |
| `file` `file-url` | 包文件名与相对下载路径（`file-url` **含 `plugins/` 前缀**，上游约定） |
| `preview` | 相对本目录的预览图路径 |
| `requirements` `loadAfter` `incompatible` | 依赖（安装时前序递归）、加载顺序提示、互斥（消费方自动补双向） |
| `publish_time` `update_time` `stars` | 排序与「新插件」角标；`publish_time` 还决定综合排序的加成 |
| `sha256` `size` | 包的真实摘要与字节数；**安装前会被校验**，失败则不落盘 |
| `bundled` `notes` | 是否随发行包分发、上架备注 |

完整契约、安装流水线与市场行为见主仓库的
[`docs/21-plugin-market.md`](https://github.com/Leafmy/BetterNCM-v3/blob/main/docs/21-plugin-market.md)。

## 内容

| 插件 | 版本 | 作者 | 许可证 |
|---|---|---|---|
| `LyricBar` | 0.3.6 | solstice23 | MIT（再分发） |
| `MaterialYouTheme` | v1.0.4 | long45343 | GPL-3.0（再分发） |
| `RefinedNowPlaying` | 3.0.2 | SUlTlUS / MiHN / solstice23 谱系 | MIT（再分发） |
| `NCM-Font-Manager` | 0.1.3 | BetterNCM-v3（本项目自研） | GPL-3.0-or-later |

前三个是**再分发**：包内容取自各自上游发布包，只改了 `manifest.json`（来源声明与 slug 归一化）。
`NCM-Font-Manager` 是自研插件，源码在主仓库 `plugins/src/NCM-Font-Manager/`。

## 重建

索引不是手写的，由主仓库的脚本从 `catalog.json` + 真实插件包推导，并逐条校验
slug / version / sha256 / size 与包一致：

```powershell
py tools\market\build_market.py           # 重建 market/
py tools\market\build_market.py --check    # 校验一致性
```

改动流程：在主仓库改 `market/catalog.json` 或插件包 → 重建 → 把 `market/` 的内容同步到本仓库。

## 上架准则

沿用上游 BetterNCM 的标准（见主仓库 [`market/README.md`](https://github.com/Leafmy/BetterNCM-v3/blob/main/market/README.md)）：
唯一的 slug、一张预览图、必须开源、包内只保留运行必需文件、标注源码仓库与反馈渠道、
尽量与其它插件不冲突。提交前请用
`py tools\pack_plugin.py <源码目录> -o plugins --runtime-only` 打包。

## 许可证

本仓库内容随 [`Leafmy/BetterNCM-v3`](https://github.com/Leafmy/BetterNCM-v3) 以
**GNU GPL-3.0-or-later** 分发；三个再分发插件各自遵循其上游许可证（MIT / GPL-3.0 / MIT），
`NCM-Font-Manager` 为本项目自研。详情见各插件在主仓库中的 `NOTICE.md` 与
`THIRD-PARTY-NOTICES.md`。
