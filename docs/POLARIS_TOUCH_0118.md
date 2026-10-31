# Polaris 触控响应候选 0118

这个补丁针对实机反馈“触控可以使用，但很不灵敏”。它只调整现有的实验性触控上报路径，不代表已经完成 Polaris 的正式触控驱动。

## 修改内容

- GPIO33/34 bit-bang I²C 的 HID 轮询间隔由 20 ms 降到 8 ms。
- 收到合法的 RMI4 F12 联系帧后，第一帧立即发送 DOWN，不再强制等待第二帧。
- 已按下的触点遇到较大的坐标变化时继续发送新坐标，不再合成错误的 UP/DOWN。
- 连续的 I²C/帧解析失败最多容忍两帧后再释放触点；累计 32 次失败才停止该实验线程。
- 启动日志会打印 `poll=8ms immediate-down`，便于确认补丁路径已运行。

## 已知边界

- 当前仍是 GPIO33/34 的 bit-bang I²C 轮询，没有接入 GPIO125 对应的 Synaptics IRQ，也没有注册真实 GIC 中断。
- F12 数据布局和 8 字节触点槽位仍属于实验性兼容路径；多指和异常帧没有完成真机验证。
- 没有 Apple-Clang/XNU 完整编译结果、GitHub Actions 构建结果或新的 boot 镜像，也没有刷写或重启手机。

## 本地验证

- `0118-polaris-touch-latency.patch` 可干净应用到 0117 的 XNU 验证树。
- 应用后的源文件通过 `git diff --check`。
- 静态检查确认轮询常量、首帧立即下发、大位移连续上报和坏帧容忍分支均存在。

下一步应先用 Apple-Clang/Actions 编译并做一次 `fastboot boot` 临时启动测试；在此之前不要把旧镜像重命名为 0118，也不要再次刷写 userdata。
