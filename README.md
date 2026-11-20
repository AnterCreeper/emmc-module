# eMMC Module

本项目为四层 eMMC 模块 PCB，采用 BGA-153 eMMC 和 GB042-34S 连接器。适配 Radxa ROCK 3/4/5 系列、ODROID N2/C4/XU4 系列、Orange Pi 5 Plus/5 Pro/3B/4 Pro、Pine64 RockPro64 等使用 GB042 34-pin eMMC 接口的板卡。

## 文件

| 文件 | 内容 |
| --- | --- |
| `emmc-module.kicad_pro` | 工程入口，KiCad 9 |
| `emmc-module.kicad_sch` | 原理图 |
| `emmc-module.kicad_pcb` | 四层 PCB |
| `emmc-module.kicad_sym` | 项目符号库 |
| `emmc-module.pretty/` | GB042 连接器封装 |
| `emmc-module-schematic.pdf` | 原理图导出 |
| `docs/GB040.pdf` | GB040 系列 0.4 mm BTB 连接器规格书（aitendo） |

## 设计说明

### NC 球位复用

本设计利用 BGA-153 的 NC 球位增加扇出：

| U1 球位 | 网络 |
| --- | --- |
| B1 | DAT3 |
| B7 | DAT7 |
| C1 | VDDI |
| C3 | DAT4 |
| C5 | DAT6 |
| C8 | DAT7 |
| D2 | DAT4 |
| D4 | GND |
| N1 | GND |
| N3 | VCCQ |
| P1 | GND |
| P2 | GND |

U1 的 C5 指 BGA 球位；原理图中另有一个去耦电容 C5 接 VCCQ，两者不是同一个器件。

这些球位在不同 eMMC 料号上可能是 NC、RFU 或厂商自定义功能。更换料号前按 datasheet 核对，不能只凭 BGA-153 封装相同就判断兼容。

### 连接器

J1 为 GB042-34S-H10-E3000，属于 GB040 系列，规格书见 `docs/GB040.pdf`：

- VCCQ：20、21
- VCC：22、23
- RST_N：12，板上没有上拉电阻
- DS：15
- DAT0/1/2/3/4/5/6/7：6、29、27、10、4、2、33、31
- CLK：8；CMD：25
- 其余引脚为 GND

### 去耦

- C1：0.1 uF，VDDI
- C2、C4、C5：0.1 uF，VCCQ
- C3、C6：0.1 uF，VCC

### 投板叠层

投板使用 JLC04081H-3313 叠层：四层、0.8 mm。信号线宽 0.1524 mm，按 50 Ω 单端阻抗设计。

## 验证

已完成打板，成品通过 Radxa eMMC Reader 的实机读写测试。

在 KiCad 9.0.2 下执行：

```bash
kicad-cli pcb drc --format json --severity-all -o /tmp/emmc-module-drc.json emmc-module.kicad_pcb
```

结果为 0 条违规、0 个未连接项。工程中 `starved_thermal` 和 `silk_edge_clearance` 设为 ignore。

加 `--schematic-parity` 会报告 12 条 `net_conflict`，对应上表 12 个球位。这是原理图符号与 PCB 网络的有意差异，不是连接错误；不要把这些焊盘改回 unconnected 来消除告警。
