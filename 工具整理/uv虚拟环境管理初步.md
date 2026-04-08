## installation
```bash
# Windows (PowerShell) 
iwr https://astral.sh/uv/install.ps1 -usebasicparsing | iex 
# macOS/Linux 
curl -Ls https://astral.sh/uv/install.sh | sh 
# 验证 
uv --version
```

[官方文档](https://docs.astral.sh/uv/getting-started/installation/)

## 初始化
``` bash
# 方式1：新建项目目录+初始化 
uv init my_ai_project 
cd my_ai_project 
# 方式2：已有目录，直接初始化 
cd your_project_dir 
uv init
```
## 核心文件介绍

- `pyproject.toml`：项目配置 + 依赖声明（替代 `requirements.txt`）
- `uv.lock`：依赖锁文件（首次 `uv sync`/`uv run` 自动生成）
- `.venv/`：虚拟环境（自动创建，默认在项目根目录）
## 安装依赖

```bash
# 安装生产依赖（写入 pyproject.toml + 生成/更新 uv.lock） 
uv add numpy pandas requests openai 
# 安装开发依赖（测试/格式化/类型检查，--dev 标记） 
uv add --dev pytest black ruff mypy 
# 查看依赖树 
uv tree
```
## 同步以及运行代码

```bash
# 1. 一键同步：按 uv.lock 安装所有依赖，清理多余包 
uv sync
# 2. 直接运行（无需手动激活虚拟环境！） 
uv run python main.py 
uv run pytest tests/
# 3. 导出requirements
uv export > requirements.txt
```

## 关于git

```bash
# 提交配置+锁文件，保证团队一致 
git add pyproject.toml uv.lock 
git commit -m "feat: init project with uv"
```