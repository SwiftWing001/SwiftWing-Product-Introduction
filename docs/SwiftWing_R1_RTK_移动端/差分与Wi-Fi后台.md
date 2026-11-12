# 差分与 Wi-Fi 后台

## 一、推荐链路

```text
USB 方案：基站原生 USB → SwiftWing QGC → 飞控 MAVLink → CAN → 移动端 UM982
无线方案：基站 Wi-Fi/UDP 或 TCP → SwiftWing QGC → 飞控 MAVLink → CAN → 移动端 UM982
```

移动端与飞控使用同一 CAN 总线；基站不需要也接这条总线。基站需要固定放置并完成其测量/坐标配置，测试过程中不能随手移动；本版本没有新增搬移保护。

基站与 QGC 电脑/手机连接同一局域网，基站 UDP 目标填 QGC 所在主机的实际 IPv4，端口默认 13320，保持 QGC 差分接收端口一致。电脑 IP 变化后目标也需核对；网络防火墙允许所用入口。TCP 方案由 QGC 客户端连接基站 IP 和默认端口 2101。

在 QGC **NTRIP/RTK** 页启用 **UDP RTCM Input → Enable UDP RTCM input**，核对 **UDP RTCM input port=13320**；新配置默认开启，但会保留用户以前保存的关闭状态。TCP 等显式来源在 **Serial and Network RTCM Relay** 中选择。确认有效 RTCM 帧、CRC 结果及新鲜数据。普通「UDP MAVLink 通信连接」不是原始 RTCM 输入；**UDP Forwarding** 是向外转发，不是接收入口。USB 方案使用本版 QGC 自动识别，不应再由另一个串口软件同时打开同一 RTCM 接口。

当前支持的基站固件在有效 QGC USB 会话下优先 USB 输出；不只凭插着一根 USB 线就认为已经向 QGC 输出差分。基站可关闭 Wi-Fi 省电，接收状态以 QGC 有效帧和移动端 RTCM 龄为准。

飞控已通过 MAVLink 连接地面站后，QGC 再向飞控转发；PX4 需在 [PX4 飞控参数](PX4飞控参数.md) 中启用 UAVCAN_PUB_RTCM。移动端 RTCM_SRC_ID=0 自动选择 CAN 来源，这里的源通常是飞控 CAN 节点，不是基站设备编码。通过网页核对 RTCM 帧增长、龄期与位置解，不能只看基站正在发送。

<div class="admonition warning">
<p class="admonition-title">注意</p>
<p>当前移动端没有直接接收局域网 UDP/TCP 差分的功能；其 Wi-Fi 后台用于管理和调试，不能绕过飞控 CAN 链路当无线 RTCM 入口。原始 RTCM USB 串口也是输出用途，不是向移动端写差分的入口。</p>
</div>

## 二、打开移动端网页后台

移动端在 CAN 正常交互时可关闭 Wi-Fi 省电，搜不到热点并不一定是故障。USB 配置仍可用。在 QGC 所选移动端区域点击 **启用无线后台**，等待真实联网状态和地址出现，再点击 **用默认浏览器打开后台**。没有实际 IP 时不猜测或沿用另一台设备旧地址。

需要配置局域网时，在该设备网络区域填写 SSID 和密码、保存局域网设置；最多保存 5 组，可选择已存网络或自动选网。设备名后缀来自硬件身份，不是随机每次生成。默认热点密码为 12345678；热点后台仅在 AP 确已开启、主机连上该热点时使用 `http://192.168.4.1/`。

使用 LAN 后台时，地址为设备实际取得/配置的 LAN IP，不能把 4.1 热点地址当局域网地址。完成调试后可关闭无线高级维护，恢复 CAN 工作时的省电策略。关闭射频前确认已经下载所需日志，避免误判后台失联。
