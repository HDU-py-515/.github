# HDU-py-515

BCI-VR 是一个面向脑机接口与 VR 实验的本地开发平台。组织内的仓库按职责拆分，业务服务、算法、协议、运行环境和客户端彼此独立，通过 API、事件流和共享协议协作。

## 仓库职责

| 仓库 | 作用 |
| --- | --- |
| [BCI-VR-web](https://github.com/HDU-py-515/BCI-VR-web) | React/Vite 网页入口，提供系统总览、受试者、采集、处理、训练、日志和 VR 启动界面 |
| [BCI-VR-api-management](https://github.com/HDU-py-515/BCI-VR-api-management) | Spring Boot API，负责受试者、Session、采集、处理、训练和 VR 相关业务 |
| [BCI-VR-notification-service](https://github.com/HDU-py-515/BCI-VR-notification-service) | 服务间事件网关，以及向 Web 推送状态变化的 SSE 服务 |
| [BCI-VR-common-proto](https://github.com/HDU-py-515/BCI-VR-common-proto) | Protobuf 协议中心、事件主题、Java/Python/TypeScript SDK 和协议校验工具 |
| [BCI-VR-ml-engine](https://github.com/HDU-py-515/BCI-VR-ml-engine) | EEG 数据采集、预处理、训练和实时推理 |
| [BCI-VR-infra](https://github.com/HDU-py-515/BCI-VR-infra) | 开发环境配置、Docker Compose、数据库脚本、观测组件和启动编排 |
| [BCI-VR-client-unity](https://github.com/HDU-py-515/BCI-VR-client-unity) | Unity VR 客户端工程；Unity 缓存和构建产物不提交 |

## 一键启动开发环境

环境要求：Docker Desktop、PowerShell 7、Node.js LTS、JDK 17、Maven 3.9+。首次使用请确保 Docker Desktop 已登录并处于运行状态。

在 `BCI-VR-infra` 目录执行：

```powershell
.\scripts\Start-BCI-Stack.ps1
```

也可以直接双击项目根目录的 `start-bci-stack.cmd`，或使用桌面的“启动 BCI-VR”快捷方式。启动内容包括：

- Docker MySQL 8.4
- common-proto 共享 SDK 构建
- API 管理服务（8080）
- 通知服务（8090）
- React Web（3000）
- Prometheus（9090）、Loki（3100）和 Grafana（3001）

网页地址：<http://127.0.0.1:3000/>

停止全部本地服务：

```powershell
.\scripts\Stop-BCI-Stack.ps1
```

## 仅启动 Docker 服务

如果只需要容器化依赖和观测组件：

```powershell
cd BCI-VR-infra
.\scripts\Start-Docker-Stack.ps1 -Build
```

ML 批处理和协议工具是按需启动的：

```powershell
.\scripts\Start-Docker-Stack.ps1 -Build -WithBatch -WithTools
```

硬件采集仍建议使用本机 Python 环境，以便访问 LSL、Neuracle 和 Unity。

## 单独开发仓库

Web：

```powershell
cd BCI-VR-web
corepack pnpm install --frozen-lockfile
corepack pnpm dev
```

API 和通知服务需要先安装 common-proto：

```powershell
cd BCI-VR-common-proto
mvn install

cd ..\BCI-VR-api-management
mvn spring-boot:run

cd ..\BCI-VR-notification-service
mvn spring-boot:run
```

ML Engine：

```powershell
cd BCI-VR-ml-engine
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
pip install -e .
Copy-Item .env.example .env
python scripts/validate_environment.py
```

Unity：使用 Unity Hub 打开 `BCI-VR-client-unity`。当前工程版本为 `2022.3.51f1c1`；如需实时推理，请同时准备 ML Engine 的推理进程。

## 本地数据与日志

数据库数据保存在 Docker 命名卷中，EEG 原始数据、处理结果、模型包和运行日志均属于本机开发数据，不进入 GitHub。各服务日志可在对应仓库的 `logs/` 目录查看，并由本地观测组件采集到 Loki。

## 开发约定

- 所有仓库默认分支为 `main`。
- 修改共享协议时，只新增字段编号，不复用已发布编号。
- 合并前运行对应仓库测试；API、通知服务和 Web 的 CI 会在合并前执行校验。
- 不提交密钥、本机绝对路径、Unity 缓存、构建产物和实验数据。
