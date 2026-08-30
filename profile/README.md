# HDU-py-515 · BCI-VR

BCI-VR 是一个面向脑机接口（BCI）与虚拟现实（VR）实验的本地开发平台。项目按职责拆分为 Web、业务 API、实时通知、算法、共享协议、基础设施和 Unity 客户端；它以本地开发与实验运行为中心，目前不包含服务器自动部署流程。

## 架构

```text
Web (React/Vite)
  ├── REST ───────────────> API Management (Spring Boot)
  ├── SSE <─────────────── Notification Service (Spring Boot)
  └── Metrics / logs ────> Prometheus / Loki / Grafana

API Management ──────────> ML Engine (local Python processes)
API Management ──────────> Unity VR client
API / Notification / ML ─> Common contracts
All services ────────────> MySQL and local data directories
```

## 仓库

| 仓库 | 职责 |
| --- | --- |
| [BCI-VR-web](https://github.com/HDU-py-515/BCI-VR-web) | React/Vite 工作台、API 调用、SSE 与可观测性界面 |
| [BCI-VR-api-management](https://github.com/HDU-py-515/BCI-VR-api-management) | Spring Boot REST API 与本机实验编排 |
| [BCI-VR-notification-service](https://github.com/HDU-py-515/BCI-VR-notification-service) | 进程内事件网关与 SSE 推送 |
| [BCI-VR-common-proto](https://github.com/HDU-py-515/BCI-VR-common-proto) | Protobuf、JSON Schema、稳定事件主题与多语言 SDK |
| [BCI-VR-ml-engine](https://github.com/HDU-py-515/BCI-VR-ml-engine) | EEG 采集、预处理、训练与推理 |
| [BCI-VR-infra](https://github.com/HDU-py-515/BCI-VR-infra) | MySQL、Compose、观测、schema 与本机启动脚本 |
| [BCI-VR-client-unity](https://github.com/HDU-py-515/BCI-VR-client-unity) | Unity XR/VR 客户端 |

## 快速开始

### 环境要求

- Docker Desktop
- PowerShell 7
- JDK 17 与 Maven 3.9+
- Node.js 20 LTS
- Python 3.10+（运行 ML 时）
- Unity `2022.3.51f1c1`（运行 VR 时）

将所有 BCI-VR 仓库克隆到同一个父目录后：

```powershell
cd BCI-VR-infra
Copy-Item dev/.env.example dev/.env
# 在 dev/.env 中设置 MYSQL_ROOT_PASSWORD
.\scripts\Start-BCI-Stack.ps1
```

默认地址：

| 服务 | 地址 |
| --- | --- |
| Web | <http://127.0.0.1:3000/> |
| API | <http://127.0.0.1:8080/api/health> |
| Notification | <http://127.0.0.1:8090/actuator/health> |
| Prometheus | <http://127.0.0.1:9090/> |
| Loki | <http://127.0.0.1:3100/> |
| Grafana | <http://127.0.0.1:3001/> |

停止本地栈：

```powershell
.\scripts\Stop-BCI-Stack.ps1
```

如需完全容器化的本地服务：

```powershell
.\scripts\Start-Docker-Stack.ps1 -Build
```

ML 批处理与协议工具是可选 profile：`-WithBatch`、`-WithTools`。接入真实 EEG 硬件、LSL、Neuracle 或 Unity 实时推理时，建议使用本机环境。

## 开发流程

1. 从 `main` 创建功能分支。
2. 修改前确认公共协议、API 路由与下游消费者。
3. 运行对应仓库 README 中的验证命令。
4. 推送分支并创建 PR；轻量 CI 会在 PR 阶段执行。
5. 检查通过后合并到 `main`；Java、Web、协议与基础设施按各自 workflow 继续验证，服务镜像在适用仓库发布到 GHCR。

共享契约采取向后兼容的演进方式：只新增字段编号，不复用已发布字段、枚举值或事件主题。不要提交密钥、本机绝对路径、实验数据、模型包、数据库卷、Unity 缓存或构建产物。

## CI 说明

每个业务仓库都有轻量 PR CI。ML 的 PR 检查刻意不安装 PyTorch、SciPy、MNE 等大型依赖；完整依赖层在 Docker 镜像构建中缓存。Unity 当前执行项目完整性检查，不执行云端 Player 构建。

部分跨仓库 Java 工作流需要 `REPO_ACCESS_TOKEN` 读取私有 Common 仓库。该 Secret 仅配置在需要它的仓库中，绝不写入源码或文档。
