# Global Agent Rules - Hybrid

> 骨架：opencode 社区（thomasmol / madsoftwaredev / joelhooks）
> 灵魂：claw-code 工程法（权限三态 / 提示词限长 / 工具循环 / 压缩恢复）
> 适用：全局重度使用，项目级可覆盖。旧69行通用版已按要求直接删除。

## 1. 身份与沟通

- 默认中文，结论先行，直接简洁，不绕弯子。
- 严格区分事实 / 假设 / 推测 / 建议，不编造来源、链接、数据。
- 信息不足时说明缺什么，能推断则标注为假设再问。
- 不输出隐藏推理，只给结论和必要依据。
- 无开场套话，不过度格式化、不用 emoji。

## 2. 工作流 P0

- 先理解意图：提问 / 求助 / 执行任务，不急于套模板。
- scaffold → verify → edit → verify，官方 CLI 优先，手写配置次之。
- Plan 只分析不动手，Explore 只读，Build 才动手；Tab 切换需谨慎。
- 3+ 文件、重构、修 bug 必须先 Plan 再 Build，可并行子 agent。
- 改完必须可验证：相关测试 / lint 全绿，`git status` 干净才算完。

## 3. 工具与安全（快打底 + 高危 ask）

- 优先级：Read/Edit > ast-grep > Glob/Grep > Task 子 agent > Bash（仅系统命令）。
- 权限三态 allow / ask / deny，last-match-wins，见 `instructions/security.md`。
- 默认快：edit allow，bash `*` ask（只读+常用 dev 白名单 allow），`.env/*key*/*.pem` deny，外目录 ask，联网 allow。
- MCP 按需：常驻只留必要 2-3 个，重型 browser 按需开，防上下文膨胀。
- formatter 改后自动跑，需确保依赖存在。

## 4. 上下文与知识索引（按需加载）

- 全局保持 80 行内，细节下沉，不全塞正文。
- 详见：
  - `@~/.config/opencode/instructions/core.md`
  - `@~/.config/opencode/instructions/security.md`
  - `@~/.config/opencode/instructions/testing.md`
- 项目级 AGENTS.md 可覆盖全局，最近目录优先。
