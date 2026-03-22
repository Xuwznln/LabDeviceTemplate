# LabDeviceTemplate

Uni-Lab-OS 外部设备包模板仓库。Fork 本仓库即可快速创建你自己的设备驱动包。

**创建时间**: 2026-03

## 功能

- 提供标准的设备包目录结构
- 包含一个示例计数设备 (`counting_device.py`)
- 内置 GitHub Actions CI，自动验证注册表

## 快速开始

### 1. Fork 本仓库

点击右上角 **Fork** 按钮，创建你自己的仓库副本。

### 2. 修改包名

将 `device_package_example/` 目录重命名为你的设备包名称，例如 `my_lab_devices/`。

同时更新 `pyproject.toml` 中的包名和描述：

```toml
[project]
name = "my_lab_devices"
description = "我的实验室设备包"
```

### 3. 编写设备代码

参考 `device_package_example/counting_device.py` 示例，使用 `@device` 装饰器编写你的设备类：

```python
from unilabos.registry.decorators import device, action, topic_config

@device(
    id="my_device",
    category=["custom"],
    description="我的自定义设备",
)
class MyDevice:
    def __init__(self, device_id=None, config=None, **kwargs):
        self.device_id = device_id or "my_device"
        self.data = {}

    @action(description="执行操作")
    def do_something(self, param: str = "") -> dict:
        return {"success": True}

    @property
    @topic_config()
    def status(self) -> str:
        return self.data.get("status", "idle")
```

### 4. 本地开发与测试

```bash
# 创建 conda 环境并安装 unilabos（需要 ROS2 完整环境）
mamba create -n unilab python=3.11.14 -c conda-forge -y
mamba activate unilab
mamba install uni-lab::unilabos -c uni-lab -c robostack-staging -c conda-forge -y

# 验证注册表（check mode，会自动检测并安装 requirements.txt 中的依赖）
unilab --check_mode --devices ./device_package_example --external_devices_only

# 启动服务（带实验图）
unilab --devices ./device_package_example --external_devices_only -g graph.json
```

> **依赖自动安装**: unilabos 在启动时会自动检测 `--devices` 目录下的 `requirements.txt`，缺失的包会通过 `uv`（优先）或 `pip` 自动安装。

### 5. CI 验证

Push 代码后，GitHub Actions 会自动运行 `--check_mode` 验证你的设备定义是否正确。

## 目录结构

```
├── README.md                     # 本文件
├── requirements.txt              # Python 依赖
├── pyproject.toml                # 包配置（支持 pip install -e .）
├── .github/
│   └── workflows/
│       └── check_registry.yml    # CI 自动验证
├── device_package_example/       # 设备包（重命名为你的包名）
│   ├── __init__.py
│   └── counting_device.py        # 示例设备
└── .gitignore
```

## 装饰器参考

| 装饰器 | 用途 | 示例 |
|---|---|---|
| `@device(id=..., category=[...])` | 标记设备类 | `@device(id="my_pump", category=["pump_and_valve"])` |
| `@action(...)` | 标记动作方法 | `@action(description="启动泵")` |
| `@topic_config()` | 标记状态属性（配合 `@property`） | 见示例代码 |
| `@not_action` | 排除公共方法（不作为动作） | `@not_action` |
| `@always_free` | 标记为不受排队限制的动作 | `@always_free` |

## 自动发现规则

- 带 `@action` 装饰器的方法 → 注册为**动作**
- 不带 `@action` 的公共方法 → 自动注册为 `auto-{方法名}` 动作
- `@property` + `@topic_config()` → 注册为**状态属性**
- `_` 开头的方法/属性 → 不会被扫描
- `@not_action` 标记的方法 → 不会被注册为动作

## License

MIT
