# Windows 11电脑上的最省事安装顺序（若实验组采用Windows）

## 选择默认运行环境
推荐实验负责人统一：Windows 11 + WSL2 Ubuntu 24.04 + Docker Desktop（WSL集成）。不要在一台机器上混杂Windows版Python与Ubuntu版Python来运行同一次实验。

## 步骤一：WSL2
以管理员打开Windows PowerShell，执行：

```powershell
wsl --install -d Ubuntu-24.04
```

完成后按系统提示重启，首次进入Ubuntu设置本地Linux用户名和密码。
参考：https://learn.microsoft.com/windows/wsl/install

## 步骤二：Ubuntu命令行安装Git、Python和解压工具
在Ubuntu终端执行：

```bash
sudo apt update
sudo apt install -y python3 python3-venv git unzip curl
python3 --version
git --version
```

Ubuntu 24.04默认Python 3.12。若终端中`python`命令不存在，使用`python3`替代；或经管理员批准安装`python-is-python3`。

## 步骤三：先不安装Docker，运行D1
在Ubuntu终端进入ZIP所在目录（Windows下载目录通常在 `/mnt/c/Users/<WindowsUser>/Downloads`）：

```bash
unzip IEEEAccess_CICD_Team_HandsOn_Starter_V2.zip
cd CI_CD_Experiment_QuickStart_V2
python3 tools/first_day_check.py
```

看到`D1 CHECK: SUCCESS`，第一天入门任务就完成了。

## 步骤四：Docker Desktop与WSL集成

1. 从Docker官网安装Docker Desktop for Windows：https://docs.docker.com/desktop/setup/install/windows-install/
2. Docker Desktop → Settings → Resources → WSL Integration → 勾选已安装的Ubuntu发行版。
3. 回到Ubuntu终端执行：

```bash
docker version
docker compose version
```

4. 命令成功后再执行操作手册的D2三容器流程。

**权限阻塞处理**：若企业安全策略不允许安装Docker Desktop，可以由CI工程师直接把代码推到实验GitHub仓库，在GitHub托管Runner上运行D2/D3；但是本机容器证据标为`NOT_RUN`，不能填写“通过”。

## 步骤五：GitHub仓库与GitHub CLI

1. 用团队的专门测试账号创建不含任何真实数据的公开测试仓库。
2. GitHub网站 https://github.com/ 登录后可操作Actions界面。
3. GitHub CLI可选：安装说明 https://cli.github.com/；首次登录`gh auth login`，确认`gh auth status`。
4. 如果本地GitHub网站因网络问题不可达，应由团队先确认合法可用的内部GitLab/Jenkins和既有签名工具，再重新审核B0/B1/B2工具能力与公平比较配置；**不能把离线自检当作真实GitHub工作流**。

## 常见出错

- `docker: command not found`：WSL integration未配置，或Docker Desktop未启动。
- `error during connect`：Docker引擎尚未启动。
- `python: command not found`：在Ubuntu中尝试`python3`。
- Git push权限报错：应核验仓库写权限和GitHub认证方式；不要把Token写进Git仓库或截图。
- 工作流无法运行：确认`.github/workflows/`三个YAML均已push到默认分支main。
