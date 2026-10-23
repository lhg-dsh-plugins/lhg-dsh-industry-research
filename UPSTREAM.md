# UPSTREAM

本仓库是上游插件的一个修复版 fork，发布于 GitHub 组织 `lhg-dsh-plugins`。

| 项 | 值 |
|---|---|
| 上游仓库 | https://github.com/PerryLink/dsh-industry-research |
| 固定 SHA | `1523dfa222ad0cf2489d346be7c36cef570f9ad4` |
| 上游版本 | 0.3.23（`package.json`；注：上游 `src/version.ts` 的 `VERSION` 为 `0.3.21`，与 package.json 存在版本戳漂移，fork 已将两者一并对齐到 0.3.24） |
| 许可证 | Apache-2.0（见仓库根目录 `LICENSE`，与上游逐字一致） |
| 拉取日期 | 2026-10-23 |
| 原作者 | PerryLink / dsh-industry-research contributors |

仓库身份核对：任务给定 `perrylink/dsh-industry-research`，`gh api` 返回的 `full_name` 为
`PerryLink/dsh-industry-research` —— 仅大小写之差，为同一仓库，无改名/重定向。

fork 落点：https://github.com/lhg-dsh-plugins/lhg-dsh-industry-research
本版相对上游的唯一逻辑改动见 `BUGFIX.md`；除该修复与版本/仓库地址同步外，其余文件与上游固定 SHA 的内容一致。


---

## 修复版版权

本修复版的修改部分 © 2026 lhg-dsh-plugins（刘洪光），修改内容见 [BUGFIX.md](./BUGFIX.md)；原始代码版权归原作者所有，见仓库内 LICENSE 与上方出处。
