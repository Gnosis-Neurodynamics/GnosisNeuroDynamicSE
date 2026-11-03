# GnosisNeuroDynamicSE

静息态头皮 EEG 信号质量评估 Notebook，提供数据读取、预处理与 ICA、伪差占比、高低频异常、频带功率、频带幅值及 PSD 通道差异的调用示例，并展示通道柱状图、地形图、标准分布图和三组质量雷达图。

- [信号质量评估算法示例.ipynb](信号质量评估算法示例.ipynb)：按顺序运行的示例与参数入口。
- [算法计算方法.md](算法计算方法.md)：窗口、滤波、阈值、统计公式及雷达图归一化方法。

## 环境与依赖

使用 **64 位 CPython 3.12**，支持 Windows x86-64 和 Linux x86-64。安装命令均在本仓库根目录执行，并使用已激活的 Python 3.12 环境。

公共依赖由 `requirements-common.txt` 和对应平台的 requirements 文件固定版本，主要包括 NumPy、SciPy、pandas、MNE、mne-icalabel、scikit-learn、ONNX Runtime、Matplotlib、pyEDFlib 和 Jupyter。安装时需要访问公共 Python 包索引，或自行准备相同平台的完整离线依赖包。

`requirements_gnosis` 提供三个自研包的 Windows、Linux wheel：

| 包 | 版本 | 用途 |
| --- | --- | --- |
| eegaux | 2.0.9 | EEG 读取、预处理与绘图 |
| eegext | 2.1.9 | 数据读写支持 |
| SignalQualityEval | 3.0.0 | 质量评估算法与兼容绘图入口 |

根据平台选择 `cp312-cp312-win_amd64.whl` 或 `cp312-cp312-linux_x86_64.whl`。Notebook 从安装环境导入已编译的算法模块。

## Windows 安装与启动

在仓库根目录打开 PowerShell。先确认当前解释器版本为 Python 3.12，再一次性安装公共依赖与本平台 wheel。

```powershell
python --version

python -m pip install -r requirements-windows.txt `
    ./requirements_gnosis/eegext-2.1.9-cp312-cp312-win_amd64.whl `
    ./requirements_gnosis/eegaux-2.0.9-cp312-cp312-win_amd64.whl `
    ./requirements_gnosis/signalqualityeval-3.0.0-cp312-cp312-win_amd64.whl
```

检查依赖，注册当前环境为 Jupyter 内核，再打开主 Notebook。

```powershell
python -m pip check

python -m ipykernel install --user --name gnosis-se312 --display-name "GnosisNeuroDynamicSE Python 3.12"

python -m notebook "信号质量评估算法示例.ipynb"
```

## Linux 安装与启动

在仓库根目录打开终端并激活 Python 3.12 环境。安装公共依赖和 Linux wheel。

```bash
python --version

python -m pip install -r requirements-linux.txt \
    ./requirements_gnosis/eegext-2.1.9-cp312-cp312-linux_x86_64.whl \
    ./requirements_gnosis/eegaux-2.0.9-cp312-cp312-linux_x86_64.whl \
    ./requirements_gnosis/signalqualityeval-3.0.0-cp312-cp312-linux_x86_64.whl
```

检查依赖并注册当前环境，随后启动 Jupyter。

```bash
python -m pip check

python -m ipykernel install --user --name gnosis-se312 --display-name "GnosisNeuroDynamicSE Python 3.12"

python -m notebook "信号质量评估算法示例.ipynb"
```

## 数据与运行参数

准备自己的 EDF 或 BDF 数据。默认读取仓库根目录下的 `data/example.edf`。也可以在启动 Jupyter 前通过 `EEG_DATA_PATH` 指定数据，或直接修改 Notebook 的 `data_path`。

PowerShell 中设置输入文件后启动：

```powershell
$env:EEG_DATA_PATH = "./data/example.edf"

python -m notebook "信号质量评估算法示例.ipynb"
```

Bash 中设置输入文件后启动：

```bash
export EEG_DATA_PATH="./data/example.edf"

python -m notebook "信号质量评估算法示例.ipynb"
```

打开 Notebook 后选择 **GnosisNeuroDynamicSE Python 3.12** 内核，检查参数，然后从首个单元格开始顺序运行。

| 参数 | 当前设置 |
| --- | --- |
| 分析区间 | `pick_start=0.0`，`pick_stop=None`，结束点取记录末尾 |
| 通道 | FP1、FP2、FZ、F3、F4、F7、F8、CZ、C3、C4、PZ、P3、P4、T3、T4、T5、T6、O1、O2 |
| 目标采样率 | 256 Hz，实际取原采样率与 256 Hz 的较小值 |
| 预处理带通 | 1–70 Hz，随后进行共同平均参考 |
| ICA | `use_ica=True`，5 个成分 |
| ICA 剔除规则 | 标签为 `eye blink`、`heart beat` 或 `muscle artifact`，且分类置信值达到 0.8 |
| 异常分析窗口 | `abnormal_win=1.0` s |
| 波形展示 | `pick_times=(100.0, 110.0)`，按数据时长调整 |
| 频域统计范围 | 1–40 Hz，使用全局伪差区间之外持续时间大于 10 s 的片段 |

`pick_start`、`pick_stop` 相对于原始记录起点，`pick_times` 相对于裁剪后的数据起点。`pick_times` 控制波形展示区间，完整裁剪片段用于统计。数据应包含所选通道，预处理频带上限应低于处理后采样率的一半。ICA 开关在预处理代码块中修改。

三组雷达图分别展示综合异常、五个频带相对功率偏离度和五个频带波幅偏离度，计算与归一化方法见[算法计算方法](算法计算方法.md)。

## 验证与许可

当前发布副本完成结构与代码语法检查。安装后请运行 `python -m pip check`，并使用自己的 EEG 数据完整执行 Notebook，核对所选时间、通道和输出结果。

Notebook 示例和说明文档采用 [MIT License](LICENSE)。`requirements_gnosis` 中的自研二进制 wheel 由权利人另行授权，使用与分发须遵循提供方的授权条款。第三方依赖遵循各自许可证。
