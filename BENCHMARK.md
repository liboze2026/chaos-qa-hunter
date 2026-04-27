# A/B Benchmark: chaos-qa-hunter vs gstack/qa-only

> 同一被测项目上，并行跑两个 bug-hunt skill，对比方法论覆盖与发现差异。
> 目的：找出 `chaos-qa-hunter` 当前 SKILL 的系统性盲点 → 反哺 SKILL.md。

## 方法

- **被测对象**：一个静态前端 HTML demo 集合（~7400 LOC，5 主 demo + 20 子 chamber，纯 HTML/CSS/JS，无 build step、无 server）。
- **执行**：两个独立 subagent 在同一 repo 上并行运行，互不可见。
- **共同约束**：
  - 不修改任何源代码
  - 文件路径 + 行号 + 复现步骤齐全
  - 不能真启动浏览器（被测对象是纯 demo）

## 结果摘要

| 指标 | chaos-qa-hunter | gstack/qa-only |
|---|---|---|
| Bug 总数 | 40 | 26 |
| Severity 分布 | 2C / 5H / 13M / 20L | 0C / 6H / 12M / 8L |
| 报告体量 | 591 行 | 204 行 |
| 估算覆盖率 | ~78%（function 80% / branch 74%） | ~70% vs 真浏览器 |
| Compute 用时 | ~5.7 min | ~5.3 min |

## 重叠分析

- 重叠 ≈ **25%**（双方都抓到 XSS 大类、A/D 键位问题、autoplay 无停止）
- 独家 ≈ **75%**（强互补，不可互换）

## chaos-qa-hunter 独家发现（白盒深挖）

- 6 条 race condition（setTimeout、pointer 双输入、Web Animations onfinish 泄漏）
- 14 条状态机 bug（finishRound 状态残留、queue 双调用、reset 后访问悬空 DOM）
- Math.atan2 视角跳变、超大 pointer delta 导致 90° 翻转
- `__lockscreen.decide` 暴露给 window 可绕过 dragging 状态
- 时钟 padStart 缺失（"9:1" 而非 "09:01"）

## qa-only 独家发现（项目级 hygiene）

- **orphan 目录**：一整个子模块对公开入口不可达
- **空目录被服务**：返回 404
- **文件克隆漂移**：665 行文件几乎完全克隆，未来必漂移
- **跨 demo 一致性破坏**：键 `A` 在不同 demo 含义反转（"允许" vs "禁止"）
- **a11y 违规**：`viewport user-scalable=no`（WCAG 1.4.4）

## 结论 → SKILL.md 改动

`chaos-qa-hunter` 的 7 攻击向量是**单文件深度**模式，对**项目级表面缺陷**系统性失明。
最小补丁：

1. **Phase 1**：新增 "公开入口图" 与 "克隆候选" 两份清单
2. **Phase 3**：新增 **3.8 项目级 Hygiene 攻击** 子章节
   - 3.8.1 orphan / 死链
   - 3.8.2 文件克隆漂移
   - 3.8.3 跨组件 UX / 协议一致性
   - 3.8.4 a11y / WCAG 快查
3. **Phase 5**：95% 判定加第 6 条（项目 hygiene 至少跑一轮）
4. **Phase 6**：第 5 轮焦点改为 hygiene 扫描
5. **末尾**：加 "适配场景" 节，明确建议与黑盒/真浏览器 skill 配合

## 双方共同盲点（仍需第三方工具补）

- 真浏览器 visual diff / Core Web Vitals
- 真触屏 vs 鼠标的并发 race
- Console error / 404 / CSP 实测
- Performance trace（FCP / TTI / long task）
- Screen reader / a11y tree 实测
