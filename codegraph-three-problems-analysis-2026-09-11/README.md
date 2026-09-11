# CodeGraph 三个核心问题分析包

本目录汇总 CodeGraph 当前三个问题的分享报告，以及基于 Django 项目的六个完整查询示例。

## 内容

- [深入浅出的分析报告](./codegraph-three-problems-sharing-guide.md)
- [六个查询示例与结果索引](./query-examples/codegraph-1.6.0/README.md)

三个评估方向：

1. 返回内容的信息价值与 token 消耗；
2. 自然语言、噪声和拼写错误查询的召回效果；
3. 证据充分后的停止与重复结果缩减。

## 数据说明

- CodeGraph 版本：1.6.0
- 测试项目：Django，3,019 个索引文件
- 查询方式：通过 STDIO MCP 直接调用 `codegraph_explore`
- 结果形式：每个测试用例一个 Markdown 文件，保留完整原始返回和评估指标
- 测试记录时间：2026-09-10
- 打包时间：2026-09-11

查询结果可能包含 Django 源码片段，仅用于测试、分析和版本对比。
