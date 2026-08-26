# Python pytest 与 Allure 方案

当目标项目没有既定测试框架且 Python 合适时使用本方案。

## 依赖和 pytest 配置

按需选择：

```text
pytest
allure-pytest
playwright
requests
python-socketio[client]
```

示例 `pytest.ini`：

```ini
[pytest]
testpaths = test_publish
python_files = test_*.py *_test.py
addopts = -ra --tb=short --alluredir=report/allure-results --clean-alluredir
markers =
    integration: 依赖本地或外部服务
    e2e: 端到端测试
    external_side_effect: 会创建或修改外部数据
```

项目已有测试目录名时使用实际名称。

## fixture 和门禁

- session fixture：配置、日志、认证状态、共享客户端。
- function fixture：独立数据、页面、临时文件和可回收资源。
- 使用 marker 和显式 CLI 开关跳过副作用测试。
- CLI 开关只放行已在 Data 中启用的案例，不把所有保留场景全部执行。
- 失败截图/附件应在 `pytest_runtest_makereport` 中按可用 driver 获取。

## 数据驱动

默认使用一个经过 schema 校验的 JSON 文件作为数据源。只有用户明确需要逐行编辑、表格维护或外部导入时，才选择 JSON Lines、CSV 或 Excel；不要把 TXT/JSON 作为双数据源，也不要在 pytest 启动时无必要地转换配置。数据层应提供：

- UTF-8 编码。
- schema/version。
- 字段白名单和类型校验。
- 唯一案例 ID。
- 默认值、profile 和案例覆盖的确定优先级。
- 敏感键递归拒绝。
- 原子写入和失败时旧文件保护。

参数化 ID 使用稳定案例 ID。关闭案例用 `pytest.param(..., marks=pytest.mark.skip(...))` 保留在报告中。

## 分阶段日志

阶段应贴近业务：数据加载、前置状态、素材/输入校验、参数生成、客户端选择、连接、执行、进度、最终结果和清理。

日志内容：

- 案例 ID、任务 ID、步骤、脱敏选项、状态码、进度和业务结果 ID。
- 不记录 Cookie/Token/localStorage 值、密码、签名和完整请求 payload。
- 对第三方/生产进程输出再次过滤，不能假设上游已脱敏。

## Allure

用 `feature`、`story`、动态标题和 `step` 表达业务层级。附件只放脱敏参数摘要、结果摘要、必要日志和失败截图。

测试结束后可自动生成 HTML：

```text
allure generate <results-dir> --output <report-dir> --clean --name <report-name> --lang zh
```

实现要求：

- `pytest_sessionfinish` 使用 `trylast=True`，确保测试结果已写完。
- `--collect-only` 时不生成。
- 提供单次关闭 HTML 的开关。
- 使用 `shutil.which("allure")` 和参数数组调用，不用 `shell=True`。
- 校验 CLI 返回码和 `index.html`。
- 报告生成失败要明确显示，但不能覆盖更重要的原测试失败信息。
- 默认不自动打开浏览器。

## 合同测试

至少对自建基础设施添加测试：

- 配置读取和 CLI 覆盖。
- 数据读写、转换、非法字段和敏感字段。
- 客户端 payload 和事件映射。
- 超时、失败状态和资源清理。
- 日志脱敏。
- Allure 生成命令、清理、名称、语言和 CLI 缺失。
