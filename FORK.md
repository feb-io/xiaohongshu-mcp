# feb-io/xiaohongshu-mcp 维护说明

本仓库 fork 自 [xpzouying/xiaohongshu-mcp](https://github.com/xpzouying/xiaohongshu-mcp)（Apache-2.0），内容 = 上游 `main` + 下表补丁。

## 补丁清单

| 补丁 | 改动文件 | 上游状态 | 合入本仓库 |
|---|---|---|---|
| [PR #839](https://github.com/xpzouying/xiaohongshu-mcp/pull/839)（commit `f908738`）：搜索等结果注水而非等 DOM 静止，修 `search_feeds` 60s 超时 | `xiaohongshu/search.go` | 未合并（2026-10-27） | merge commit `dda757f` |

按 Apache-2.0 第 4(b) 条声明：上表所列文件已在本仓库中修改。

## 同步上游

只用 merge，不 rebase、不 force-push。新 clone 先加一次 upstream（禁止误推上游）：

```bash
git remote add upstream https://github.com/xpzouying/xiaohongshu-mcp.git
git remote set-url --push upstream DISABLED
```

之后每次同步：

```bash
git fetch upstream --tags
git merge upstream/main
```

- 上游已合并某补丁（或等价修复）：merge 后从补丁清单删掉该行。
- 冲突：以上游为准，重新判断补丁是否还需要；需要就按新代码重写并重新验收。

## 构建

与上游 release 参数一致，版本号写成 `<上游版本>+feb.<n>`：

```bash
CGO_ENABLED=0 go build -ldflags "-s -w -X main.version=v2.5.5+feb.1" -o xiaohongshu-mcp-darwin-arm64 .
```

不要推 `v*` 形式的 tag：会触发 `tag-release` / `docker-release` 工作流发布公开 release。

## 验收（每次同步后、部署前）

1. `go vet ./... && go test ./...` 通过。
2. 专用账号只读试跑 `search_feeds`：3 个关键词 × 3 轮，间隔 60–90 秒随机。
   - 0 次超时 / 500；
   - 每次结果与上一次（不同关键词）搜索的重叠 ≈ 0，即没有读到上一页的旧结果；
   - 无风控信号（300011/300012/300015、461/471、验证码、账号异常），结束时仍在登录状态。
