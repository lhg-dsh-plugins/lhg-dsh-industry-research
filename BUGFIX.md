# BUGFIX — 0.3.24

## Bug 标题

`isIsoDate` 接受不存在的日历日期：`2026-02-30`、非闰年的 `02-29`、`04-31` 被当作真实日期通过全部校验，并被 `Date.parse` 静默滚动成另一天参与「非未来」判断。

## 证据链

- 基线（上游固定 SHA `1523dfa`）：`pnpm test` 142/142 全绿，`pnpm typecheck` 退出 0；上游 open issue 无 bug 报告（仅 4 个 dependabot 依赖升级 PR），故走静态定位 + 最小复现路线。
- 复现脚本（纯 Node 直接加载 `src/chain.ts`，无外部服务）修复前输出（exit 1）：

  ```
  FAIL isIsoDate('2026-02-30'): got true, expected false
  FAIL isIsoDate('2023-02-29'): got true, expected false
  FAIL isIsoDate('2026-04-31'): got true, expected false
  FAIL validateStatus flags 2026-02-30: got false, expected true
  FAIL validateChainMap flags 2023-02-29: got false, expected true
  ```

  对照组同时通过：`2026-13-01` 被拒、`2024-02-29`（闰年）合法、`2026-08-19T00:00:00.000Z` 合法。
- `git stash` 只回退实现复跑，复现 exit 1 逐字一致；恢复修复后 exit 0、全项通过。

## 根因

`src/chain.ts` 的 `isIsoDate` 此前实现为「格式正则 + `Date.parse` 非 NaN」：

- 正则 `/^\d{4}-\d{2}-\d{2}(?:[T ].*)?$/` 只约束位数，不约束月/日范围；
- V8 的 `Date.parse` 对日溢出做归一化而非报错：`2026-02-30` → 2026-03-02、`2023-02-29` → 2023-03-01、`2026-04-31` → 2026-05-01，均返回有效时间戳。

于是与该函数自述的「a real ISO-8601 date」矛盾：不存在的日期被判合法。下游 `validateStatus`（链节点与公司卡的 `statusAsOf`）、`validateCompanyCard` 与 `validateDeliveryContract`（指标 `asOf`）的非未来判断又直接用 `Date.parse` 比较，写错的 asOf 不仅不报错，还按滚动后的日期参与判断——一个「fail loud、每个数值可回溯」的审计型插件在此静默放行坏日期。

## 改动文件

- `src/chain.ts`（唯一逻辑改动，+15/−1）：`isIsoDate` 在原正则与 `Date.parse` 检查之后，再按字符串写定的年/月/日分量做真实性校验（月份 1–12、日期不超过当月天数、2 月闰年感知）。
- 契约同步（非逻辑改动）：`package.json` 版本 0.3.23 → 0.3.24 且 `repository.url` 改指 fork 落点；`src/version.ts` 的 `VERSION` 对齐到 0.3.24（上游基线自身漂移：package.json 为 0.3.23 而 VERSION 为 0.3.21，非本版修复目标，fork 按发版口径一并对齐，特此披露）；`CHANGELOG.md` 新增 0.3.24 条目；`README.md` 顶部加 fork 头（原文一字未删）；新增 `UPSTREAM.md` / 本文件。`LICENSE` 与上游逐字一致。pnpm-lock 无根包版本字段，无待同步项。

## 复验结果

- 复现脚本：修复前 exit 1（5 项失败）→ 修复后 exit 0（9 项全过）。
- `pnpm test`：142/142 通过，与基线逐项一致，零回归。
- `pnpm typecheck`：退出 0；`pnpm lint`（oxlint）：0 warnings / 0 errors。
- ready/ 副本内独立复跑复现脚本：exit 0。

## 验证边界

- 本版只校验日期分量（年/月/日）的真实性。时间部分的溢出（如 `T24:00:00`）仍按 `Date.parse` 语义处理：ISO 8601 本就允许 24:00 表示一日终点，未做改动。
- 日期分量按字符串写定值校验，与时区无关：带时区偏移的时间戳（如 `2026-08-19T00:30:00+05:00`）写定日期合法即合法，不受 UTC 换算影响。
- 未调用任何真实外部服务；全部验证为离线纯函数复现 + 上游自带测试套件。
