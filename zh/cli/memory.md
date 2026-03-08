

  CLI 命令

  
# memory

管理语义内存索引与搜索。由活动内存插件提供（默认：`memory-core`；设置 `plugins.slots.memory = "none"` 以禁用）。相关：

-   内存概念：[Memory](../concepts/memory.md)
-   插件：[Plugins](../tools/plugin.md)

## 示例

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory index --force
openclaw memory search "meeting notes"
openclaw memory search --query "deployment" --max-results 20
openclaw memory status --json
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

## 选项

`memory status` 和 `memory index`：

-   `--agent `：限定到单个智能体。不指定此选项时，这些命令会为每个已配置的智能体运行；如果未配置智能体列表，则回退到默认智能体。
-   `--verbose`：在探测和索引期间输出详细日志。

`memory status`：

-   `--deep`：探测向量和嵌入可用性。
-   `--index`：如果存储是脏的，则运行重新索引（隐含 `--deep`）。
-   `--json`：打印 JSON 输出。

`memory index`：

-   `--force`：强制完全重新索引。

`memory search`：

-   查询输入：传递位置参数 `[query]` 或 `--query `。
-   如果两者都提供，`--query` 优先。
-   如果两者都未提供，命令将出错退出。
-   `--agent `：限定到单个智能体（默认：默认智能体）。
-   `--max-results `：限制返回的结果数量。
-   `--min-score `：过滤掉低分匹配项。
-   `--json`：打印 JSON 结果。

注意：

-   `memory index --verbose` 会打印每个阶段的详细信息（提供者、模型、来源、批处理活动）。
-   `memory status` 包含通过 `memorySearch.extraPaths` 配置的任何额外路径。
-   如果实际活动的内存远程 API 密钥字段配置为 SecretRefs，该命令将从活动网关快照中解析这些值。如果网关不可用，命令将快速失败。
-   网关版本偏差说明：此命令路径需要支持 `secrets.resolve` 的网关；较旧的网关会返回未知方法错误。

[logs](./logs.md)[message](./message.md)