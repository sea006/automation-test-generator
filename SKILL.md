---
name: automation-test-generator
description: Generate or extend maintainable automation test scripts from requirements and an existing codebase. Use for Web, API, desktop, Electron/Socket, or mixed workflows that need test architecture, data-driven cases, logging, execution gates, and reports; prefer the repository's framework, or Python pytest and Allure when no standard exists.
---

# 自动化测试脚本生成

从需求和目标项目代码生成可运行、可维护、可诊断的自动化测试脚本，并完成无副作用验证。测试应覆盖真实业务合同，但不能用测试代码替代生产实现。

## 必须保持的边界

- 先读需求、代码、配置和现有测试，再选测试层级和工具；不要按模板猜测接口、事件或页面结构。
- 优先沿用仓库已有框架、fixture、测试目录和命名。只有项目未建立标准时才默认使用 Python + pytest + Allure。
- 默认只新增或修改测试代码及测试配置。业务代码修改需要用户单独明确授权。
- 集成和端到端测试必须调用项目正式入口，不能复制生产算法或增加测试专用业务捷径来制造通过结果。
- 默认执行不得产生不可逆外部副作用。发布内容、付款、发送消息、删除数据等操作必须获得本轮明确授权，并限制为授权数量。
- 密码、Cookie、Token、密钥、个人数据和完整敏感 payload 不得写入源码、测试 Data、日志或报告。
- 只清理测试创建的资源和进程，不关闭用户已有服务，不删除不属于本次测试的数据。

## 工作方式

1. 阅读 [references/workflow.md](references/workflow.md)，把需求转换为范围、测试层级、依赖、数据和通过条件。
2. 阅读 [references/frameworks.md](references/frameworks.md)，根据目标系统选择 Web、API、桌面/Socket 或混合实现；只读取适用部分。
3. 建立职责清晰的测试结构：配置、模型、数据、fixture、驱动/客户端、用例、日志和报告分离。
4. 先实现正常流程的最小闭环，再按风险增加边界、失败、状态迁移、恢复和安全用例。
5. 测试数据默认直接维护在经 schema 校验的 JSON 文件中，并作为唯一数据源；只有需求明确要求逐行编辑或表格导入时才选择 JSON Lines、CSV 或 Excel。不要无故建立 TXT/JSON 双数据源或运行时转换链路。运行时凭证通过环境变量、密钥系统或 Git 忽略的状态文件注入。
6. 使用稳定标识、显式等待和最终业务状态判断成功，避免固定睡眠、仅检查按钮点击或只断言 HTTP 200。
7. 若采用 pytest + Allure，阅读 [references/pytest-allure.md](references/pytest-allure.md) 并实现自动结果清理、附件脱敏和 HTML 生成。
8. 执行前阅读 [references/safety-verification.md](references/safety-verification.md)。先运行无副作用回归，再按授权决定是否运行外部副作用流程。

## 交付要求

- 说明生成或修改的测试文件、覆盖范围、配置入口和执行命令。
- 报告实际运行的通过、失败、跳过数量，以及未运行部分和原因。
- 外部副作用测试只报告脱敏业务证据和创建的资源标识。
- 明确测试替身覆盖与真实集成覆盖的边界，不把 mock 通过描述为完整业务通过。
- 保留用户现有工作树修改，不清理或回退无关文件。
