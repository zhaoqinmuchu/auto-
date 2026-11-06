# ChanScanner · 缠论图表分析

ChanScanner 是一个运行在 Windows 上的桌面缠论分析工具：导入通达信导出的 TXT 行情或联网拉取单只 A 股行情，在 PyQt6 界面里计算并绘制笔、线段、中枢、买卖点和 MACD，支持图表缩放/平移/框选等交互，并能把当前线段导出为通达信公式文本（`.tni`）。

界面入口为 `App/ashare_bsp_scanner_gui.py`，缠论计算内核在仓库根目录的 `Chan.py` 及其它模块中。

## 功能

- **本地行情导入**：通达信 TXT（日线、分钟线、无年份的秒级 K 线），支持跨月、跨年日期推断。
- **单只股票联网分析**：1 / 5 / 15 / 30 / 60 分钟使用腾讯财经分钟接口，日线使用 baostock；只分析当前输入的股票，不做全市场扫描。
- **缠论元素绘制**：K 线、合并 K 线、笔、线段、线段中枢、线段买卖点、MACD。
- **图表交互**：滚轮缩放、左键拖动平移、右键拖动框选放大、双击恢复视图、十字光标，以及 K 线 / 笔 / 线段 / 线段中枢 / 线段买卖点 / MACD 六个显示开关。
- **通达信公式导出**：将当前线段导出为 GBK 编码的文本，默认扩展名 `.tni`；复制到通达信公式编辑器使用，不是已验证的二进制公式安装包。

## 数据来源

| 周期 | 来源 |
| --- | --- |
| 1 / 5 / 15 / 30 / 60 分钟 | 腾讯财经接口（分钟数据最多约最近 320 根） |
| 日线 | baostock |

在线分析依赖网络与行情服务可用性；接口不可达时程序会给出失败提示。

## 快速开始

### 从源码运行

需要 Python 3.11+（开发环境为 Python 3.14）与 Windows 桌面环境。

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-gui.txt
.\.venv\Scripts\python.exe .\App\ashare_bsp_scanner_gui.py
```

请在仓库根目录启动程序，模块按当前工作目录导入。

### 打包为 EXE

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements-build.txt
.\.venv\Scripts\python.exe -m PyInstaller .\ChanScanner.spec
```

产物在 `dist\ChanScanner\`。程序是目录式打包，运行 `ChanScanner.exe` 时必须保留同目录下的 `_internal`，不能只复制单个 EXE。

## 使用要点

1. **导入通达信 TXT**：在通达信导出时保留原始编码和表头，不要用电子表格另存，以免时间、价格或分隔符被改写。
2. **填写数据所属年月**：填入最后一根 K 线所属的年月（如 `2026-01`）。无年份的秒级数据靠这个基准向前推断跨月、跨年日期；导入历史数据时建议明确填写。
3. **看图**：初始显示最近约 500 根 K 线，可缩放查看更早的数据（显示范围不等于只读取了这些数据）。
4. **导出公式**：点击“导出通达信线段公式”，保存为新文件，再按程序提示复制到通达信公式编辑器。

## 固定分析参数

界面不提供缠论参数修改入口，`get_chan_config()` 固定以下关键参数：

| 参数 | 固定值 | 说明 |
| --- | --- | --- |
| `bi_strict` | `False` | 使用新笔（非严格模式） |
| `bi_fx_check` | `"half"` | 分型有效性校验方式 |
| `cal_bi_zs` | `False` | 不计算笔级别中枢 |
| `cal_bi_bsp` | `False` | 不计算笔级别买卖点 |

其余参数保持框架默认值。详细说明见 [PROJECT_GUIDE.md](PROJECT_GUIDE.md)。

## 目录结构

| 路径 | 内容 |
| --- | --- |
| `App/` | PyQt6 界面：TXT 导入、单股分析、图表交互与公式导出 |
| `Chan.py`、`ChanConfig.py` | 缠论计算入口与配置 |
| `Bi/`、`Seg/`、`ZS/`、`BuySellPoint/` | 笔、线段、中枢、买卖点计算 |
| `KLine/`、`Combiner/`、`Common/`、`Math/` | K 线与合并、公共结构与指标 |
| `DataAPI/` | 通达信 TXT、腾讯分钟线、baostock 日线 |
| `Plot/` | 绘图组件 |
| `Debug/` | 4 个离线日期回归脚本 |
| `TDX/` | 手写通达信公式源文件 |
| `tools/` | 验收与检查脚本 |
| `docs/` | 维护、验证与验收记录 |

## 文档

- [PROJECT_GUIDE.md](PROJECT_GUIDE.md)：功能、参数、使用流程与验收入口
- [docs/VERIFICATION.md](docs/VERIFICATION.md)：验证记录
- [docs/MAINTENANCE.md](docs/MAINTENANCE.md)：开发环境、构建、备份与恢复
- [docs/RAZOR_ACCEPTANCE.md](docs/RAZOR_ACCEPTANCE.md)：2026-11-06 精简改动的验收记录
- [docs/DEBUG_INDEX.md](docs/DEBUG_INDEX.md)：离线回归脚本索引

## 许可与致谢

本项目以 MIT 许可证发布，`LICENSE` 中的原始版权声明予以保留。

缠论计算框架来自开源项目 **chan.py**（作者 Vespa314）。本项目在其基础上做个人化定制：收敛为单只股票分析、固定 GUI 参数、精简演示与调试脚本，并补充了验收与维护文档。上游 README 中的演示图片和部分文档仍保留在仓库中，仅作历史存档，不参与程序运行。

## 免责声明

本项目仅供学习和研究使用，不构成任何投资建议。行情数据来自第三方接口，可能存在延迟、缺失或错误；据此进行的任何交易决策与损失由使用者自行承担。
