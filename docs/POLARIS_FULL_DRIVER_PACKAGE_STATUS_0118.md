# Polaris 完整驱动包状态（0118）

当前工作树不能诚实地生成“触摸、GPU、USB、相机、Wi‑Fi、蓝牙、NFC 全部正常”的可刷入包。原因是代码层次和可验证工件还没有达到这个声明所需的门槛。

| 功能 | 当前代码 | 能否放进完整功能包 |
|---|---|---|
| 触摸 | GPIO33/34 bit-bang I²C、实验性 RMI4/F12 HID；0118 只降低轮询延迟并改善丢帧处理 | 只能作为候选；仍未接入 GPIO125/GIC IRQ，也没有 0118 编译工件 |
| USB | 旧 SWEET 路径有 CDC-ECM/调试日志和端点 bring-up | 不能当作完整 USB 外设、MTP 或 USB 主机驱动 |
| GPU | A630 固件元数据、只读探针和禁用的 bring-up gate | 没有 PAS/GMU/HFI、GPUCC/GDSC、SMMU、命令环、fence 或 iOS 用户态渲染后端 |
| Wi‑Fi | WCN3998/WCN3990 资源识别和 WLFW board-data 编码 | 没有 SNOC/ICNSS、QMI/QRTR、固件启动、数据面或 iOS 网络接口 |
| 蓝牙 | QCA H4/固件报文编码原型 | 没有 UART/SlimBus 传输、固件/NVM 下载、HCI 控制器或 BlueTool 适配 |
| 相机 | CCI 命令格式和传感器文件清单 | 没有供电、CCI 总线访问、CSIPHY/CSID/IFE、DMA 或 iOS 相机服务 |
| NFC | NCI 1.x 编解码原型和 PN553 节点探针 | 没有 QUP I²C、VEN/IRQ 时序、固件下载或 NFC daemon 桥接 |

## 当前可验证事实

- 0118 触控补丁可干净应用到 0117 的 XNU 验证树，应用后的临时树通过 `git diff --check`。
- 补丁序列在工作树中连续到 0118。
- 本地没有 Apple-Clang/XNU 0118 编译结果，也没有新的 boot 镜像；现有 0115 镜像不能被重命名为 0118。
- 现有 0115 刷机包的 manifest 已明确把 GPU、Wi‑Fi、蓝牙、相机、NFC 和触摸标为未验证/非功能驱动。

## 完整包的硬门槛

只有同时满足下列条件，打包脚本才应允许生成“可刷入候选包”：

1. 0118 及后续补丁在真实 xnu-8796/macOS 构建树中通过 Apple-Clang 链接。
2. boot 镜像中的 kernel、ADT、shim 和固件哈希均可追溯，且不是旧 0115 工件。
3. 每个硬件后端至少有一次独立的 `fastboot boot` 真机日志：触摸事件、GPU 固件/命令完成、USB 枚举、Wi‑Fi 扫描/收发、蓝牙 HCI、相机 chip-ID/单帧、NFC NCI 版本/轮询。
4. 失败时脚本必须停止，不能把诊断镜像当成完整驱动包，也不能自动再次刷写 userdata。

在这些条件满足前，只能生成“诊断候选包”，不能声称所有硬件正常。
