# Provider 连接稳定性排查结论（2026-02-14）

## 测试范围
- Provider: `rightcode`
- Endpoint: `https://right.codes/codex/v1`
- 客户端: `codex-cli 0.101.0`

## 关键结果
1. 沙箱内执行 `codex exec`：
   - `no_mcp` 6/6 失败
   - `with_mcp` 6/6 失败
   - 失败特征一致：`stream disconnected before completion`
   - 记录文件：
     - `debug/provider-stability-20260214-164904.md`
     - `debug/provider-stability-20260214-164904.log`

2. 非沙箱直连 provider（HTTP 探测）：
   - `GET /models` 10/10 成功（200）
   - 说明 DNS/TCP/TLS 与基础可达性正常

3. 非沙箱执行 `codex exec`（禁用 MCP）
   - 4/4 成功，均返回 `ok`
   - 无断流

4. 非沙箱执行 `codex exec`（启用 Zotero MCP 工具调用）
   - 3/3 成功完成
   - Provider 流式响应稳定

## 结论
- `right.codes` provider 在当前环境下并非“本体不稳定”。
- 之前出现的 `stream disconnected before completion`，主要与“沙箱内运行路径”相关，而不是 MCP 接入本身。
- 换言之：**这是执行环境（sandbox/network policy）问题，不是 provider 稳定性问题**。

## 建议动作（按优先级）
1. 对需要稳定联网与流式输出的任务，优先在非沙箱路径执行（或允许对应命令提升权限）。
2. 保留 `zotero` MCP 配置不变；它不是本次断流根因。
3. 若后续仍偶发断流，再做 provider 侧重试策略：
   - CLI 级别重试（指数退避）
   - 减少单次响应体积（更短 prompt / 降低输出长度）

## 额外观察（与 provider 稳定性无关）
- Zotero 工具返回中出现 `WinError 10061`（无法连接本地 Zotero API）是另一条独立问题，通常表示 Zotero 本地 API 服务未开启或 Zotero 未运行。
