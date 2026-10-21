# CI/CD 证据闭合实验团队：**先跑起来** V2.0

> **定位**：可运行的非生产教学与工程脚手架，帮助团队从 0 搭出 B0、B1、B2 的最小技术链路。
> **重要**：本项目中的 `tools/prototype_contract.py` 仅实现有限演示检查，**不是论文冻结 Algorithm 1**，不是独立 Oracle，不得直接将其运行数字写进论文。

## 0. 一小时内跑通的最小验证（不需要 GitHub/Docker）

工具仅需 Python 3.12 或 3.13，终端位于本工程根目录。

**推荐只运行一条命令**，自动完成单测、构建和三个场景，并生成 `runs/first_day_report.json`：

```bash
python tools/first_day_check.py
```

若需逐步排错，执行：

```bash
python -m unittest discover -s tests -v
python tools/build.py
python tools/run_local.py --scenario N01
python tools/run_local.py --scenario A101
python tools/run_local.py --scenario A201
```

预期 **N01 B0=PASS，B2_PROTOTYPE=CLOSED**；**A101 B0=PASS，B2_PROTOTYPE=OPEN**；**A201 B0=PASS，B2_PROTOTYPE=OPEN，状态 NFI**。三次均生成 `runs/LOCAL-.../` 证据包。B1此时必须标为 **NOT_RUN**，不能用本地SHA-256充当来源证明。

## 1. 第二步：实际 Docker 容器三环境

先在自己电脑安装Docker Engine/Desktop+Compose，并确保 `docker version` 与 `docker compose version`成功。

```bash
python tools/build.py
docker build -t closure-pilot:local -f infra/Dockerfile .
docker image inspect closure-pilot:local --format '{{.Id}}'
docker compose -f infra/compose.yml up -d
python tools/check_stages.py --out runs/docker-N01
python tools/record_manifest.py runs/docker-N01
docker compose -f infra/compose.yml down
```

**预期**：三个容器真实运行HTTP服务，Dev/QA/Staging均得到200；同一镜像**本地config ID**被复用。Docker `.Id` 不是发布到注册表的OCI manifest digest，论文不能写错。

真实失败样本（**预期命令退出码非0**）：

```bash
QA_FAIL_HEALTH=1 docker compose -f infra/compose.yml up -d --force-recreate
python tools/check_stages.py --out runs/docker-B01
python tools/record_manifest.py runs/docker-B01
docker compose -f infra/compose.yml down
```

## 2. 第三步：GitHub Actions B0

1. 创建 **全新公开测试仓库**，不要上传真实业务日志、密钥或患者/员工数据。
2. 将本解压目录**所有文件（包含 `.github/` 隐藏目录）**提交到仓库main。
3. 打开GitHub网页仓库 → **Actions** → 选择`B0 - Modern CI and Staged Gates` → **Run workflow** → `N01`。
4. 查看工作流绿灯及每一步日志；下载该run的artifact（`b0-<runid>`）。
5. 再运行`B01`；QA的真实HTTP健康检查返回503，**整个工作流应红灯**，仍必须保留并下载该run的artifact。
6. 打开PR改动（即使只是README），确认`pull_request`会自动触发B0。

不要在未实际运行前说B0完成。`ubuntu-24.04`是runner版本约束；实际镜像版本需从run日志记录。

## 3. 第四步：B1用GitHub真实来源证明

官方文档（实时规则以GitHub为准）：
- https://docs.github.com/en/actions/how-tos/secure-your-work/use-artifact-attestations/use-artifact-attestations
- https://cli.github.com/manual/gh_attestation_verify

仓库→Actions→`B1 - Real GitHub Artifact Attestation`→Run workflow。此流程使用`actions/attest@v4`真正签发证明、再用`gh attestation verify`验证原始构件；修改构件一个字节后再次验证应失败。**如果计划/权限不支持，截图记录失败并标为B1 N/A，不得标为漏检**。

## 4. 第五步：B2练习与正式A2必须区分

仓库→Actions→`B2 - Experimental Evidence Adapter`→N01 / A101 / A201。此工作流先重跑B0阶段检查、**真实签发与验证B1证明**，再添加教学B2证据适配。下载 `evidence/raw/inventory.jsonl`、`dispositions.jsonl`、`case/case.json`、`results/b2_prototype.json`。

实验队可以先用它理解结构：分开保存的事件清单 → 处置/状态 → cutoff → 后继义务 → 失败码。实际论文B2 **必须替换为经S15/S20核对的新A2实现、完整Schema/收集证明及独立Oracle**。当前脚手架中的 raw inventory 与 disposition 虽然分别保存，但主要来自同一教学采集器，**不构成独立上游来源的A1完整性证明**；正式实验必须增加来源级inventory和独立覆盖证明。

## 5. 什么时候可以开始E2正式对照？

- B0正常/失败都真实跑通、日志全部归档。
- B1签发与验证**真实运行成功**，且记录工具能力及策略（不能只装命令行）。
- 论文A2新双时点实现通过六类边界反例；旧107/16包保持独立冻结。
- 独立Oracle标签先冻结，再让实验工程师运行E2。
- 三组都用相同基础服务、commit、规则明确；B2增加的采集字段和开发成本另算。
- 预注册场景、重复次数、失败重跑规则及证据manifest。

详见 `docs/` 和《从零开工工程操作手册V2.0.docx》。

- Modification 1
