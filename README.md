# ThingsBoard IoT 平台自动化测试（接口 + UI）

基于 **Docker 本地部署**的 ThingsBoard 社区版（物联网平台），用 Python + pytest 搭建的 **接口 + UI** 自动化测试项目。

一句话概括：对物联网平台的核心功能（登录认证、设备管理、遥测上报、异步告警、仪表盘）做接口 + UI 的自动化回归。

---

## 量化结果一览

| 维度 | 成果 |
|------|------|
| 功能自动化 | **40 条**用例（接口 32 + UI 8），含等价类 / 边界值 / 负面场景 |
| 全量回归 | 约 **1 分钟**（实测 71 秒） |
| 报告 | Allure 可视化测试报告 |

---

## 技术栈

| 分类 | 技术 |
|------|------|
| 语言 | Python 3.11 |
| 测试框架 | pytest + allure-pytest |
| 接口请求 | requests |
| UI 自动化 | pytest-playwright（POM 模式） |
| 被测系统 | ThingsBoard 3.9.1（Docker 部署，锁版本） |

---

## 目录结构

```
thingsboard-autotest/
├── config/setting.py           # 全局配置：地址、账号、轮询参数
├── common/api_client.py        # 登录 + 统一发请求（X-Authorization 认证）
├── api/                        # 接口业务层
│   ├── device_api.py           #   设备增删查
│   ├── telemetry_api.py        #   遥测上报 / 查询
│   └── alarm_api.py            #   告警规则 + 告警查询轮询
├── pages/                      # UI 页面对象（POM）
├── testcases/
│   ├── conftest.py             # 公共 fixture（登录、各 API 对象）
│   ├── api/                    # 接口用例（32 条）
│   └── ui/                     # UI 用例（8 条）
├── docker/docker-compose.yml   # ThingsBoard 本地部署配置
└── requirements.txt
```

---

## 核心设计

1. **四层分层框架**：`config`（配置）→ `common`（通用）→ `api`/`pages`（业务）→ `testcases`（用例），职责清晰、易维护。
2. **X-Authorization 认证**：ThingsBoard 特有认证头，登录拿 JWT 后统一加 `X-Authorization: Bearer <token>`（非标准 `Authorization`）。
3. **异步数据校验**：遥测 / 告警由规则链异步处理，采用「轮询 + 超时上界」（2 秒轮询、60 秒上限）等待数据写入，而非上报后立刻断言。
4. **POM + UI/API 交叉验证**：页面对象模型降低维护成本；用「UI 操作 + API 查询」规避虚拟滚动列表 DOM 断言不稳的问题。
5. **负面 / 边界用例**：覆盖 400 / 401 / 404 等错误码，接入 Allure 出报告。

---

## 环境准备

1. **启动 ThingsBoard**（Docker）：
   ```bash
   docker compose -f docker/docker-compose.yml up -d
   ```
   默认账号：`tenant@thingsboard.org` / `tenant`（网页 http://localhost:8080）

2. **安装依赖**：
   ```bash
   python -m pip install -r requirements.txt
   python -m playwright install chromium
   ```

## 运行测试

```bash
# 全部测试（40 条）
python -m pytest -v

# 生成并打开 Allure 报告
python -m pytest --alluredir=reports/allure-results
allure generate reports/allure-results -o reports/allure-report --clean
allure open reports/allure-report
```

> Windows 可直接双击根目录 `run_allure.bat` 一键跑完并打开报告。
