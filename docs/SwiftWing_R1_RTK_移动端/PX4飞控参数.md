<div class="rtk-guide" aria-hidden="true"></div>

# PX4 飞控参数

本项目经典 PX4 模式对应仍接收旧 Fix2 ECEF 航向扩展的 v1.17.0 路径。较新 PX4 main 已移除此接收方式，不能把「版本更高」理解为必然兼容。AUTO 识别到 PX4 家族也不等于精确版本已验证。[S1][S2]

先导出飞控参数备份。飞控通过 USB 或遥测连接 QGC 后，进入 **飞行器设置 / Vehicle Setup → 参数 / Parameters**，搜索下面的完整参数名，核对当前值后修改并保存；这里不是 SwiftWing RTK 设备参数列表。按重启提示完成后重新读回。以下按 CAN1、单个目标 GNSS 实例说明；CAN2 或多 GPS 系统需核对实际接线和 GPS 实例，勿把板型默认值直接覆盖到另一架飞机。

## 一、参数配置

| 飞控参数 | 设置/核对值 | 作用 |
| --- | --- | --- |
| UAVCAN_ENABLE | 仅传感器用 2；原有 3 则保留 3 | 启用 DroneCAN 自动配置；保留原有 CAN 电调功能 |
| UAVCAN_BITRATE | 1000000 | 与移动端 CAN 一致 |
| UAVCAN_SUB_GPS | 1 | 接收 CAN GNSS |
| EKF2_GPS_CTRL | 原值 7 时改 15 | 在位置/高度/速度基础上增加 bit3 GPS yaw |
| UAVCAN_PUB_RTCM | 1 | 将地面站注入差分发布到 CAN |
| UAVCAN_PUB_ARM | 1 | 发布解锁状态，供移动端灯态等功能使用 |
| EKF2_GPS_YAW_OFF | 核对为 0 | 板端已输出机头角，避免重复安装偏角补偿 |
| EKF2_GPS_POS_X | ANT1 相对重心 X，米 | 前方为正 |
| EKF2_GPS_POS_Y | ANT1 相对重心 Y，米 | 右方为正 |
| EKF2_GPS_POS_Z | ANT1 相对重心 Z，米 | 下方为正 |

EKF2_GPS_CTRL 是位掩码。15 适用于原值为 7 且需要增加 GPS 航向的本项目配置；若原来启用不同组合，应在原位掩码上增加 bit3，先核对所用版本定义，不机械覆盖其他选项。[S3]

例如 ANT1 在重心前 0.30 m、左 0.05 m、上 0.10 m，则 X=+0.30、Y=-0.05、Z=-0.10。这里不是两天线间距；ANT2 在后 60 cm 不能写成 X=0.60。此版本不要求存在 SENS_GNSS0_HDG，也不应为找不到该参数而套用其他版本名称。已有 UAVCAN_SUB_GPS_R=1 可保留，但相对位置订阅本身不是航向融合开关。[S3]

本项目是单颗双天线 UM982 加固定地面基站，正常差分使用 `RTCMStream`，不需要为此机械启用双接收机移动基线用的 `UAVCAN_PUB_MBD`。板载终端 R69 由硬件装配决定，也不使用其他节点的 `CANNODE_TERM` 参数控制。[S2]

## 二、保存与验证顺序

1. 保存飞控参数，按 QGC 提示重启飞控；建议本次配置完成后飞控和移动端一起正常重新上电。
2. 在 QGC 确认移动端的卫星数与定位数据持续更新，操作见[差分与 Wi-Fi 后台](差分与Wi-Fi后台.md#qgc-gps)。
3. 按[RTK 航向确认](起飞前检查.md#rtk-heading)在 MAVLink 检测中查看 `GPS_RAW_INT.yaw`，对照实际机头，再比较 `GLOBAL_POSITION_INT.hdg`。
4. 无桨地面整体缓慢转动机体与双天线，再回正，核对方向一致，没有 180° 反向或 90° 错装。

<div class="admonition warning">
<p class="admonition-title">注意</p>
<p>本板没有外置磁罗盘。保留并校准实际使用的内置磁罗盘，不照抄 CUAV 带外置罗盘产品的禁磁步骤。启用 GPS yaw 与选择磁罗盘是两件事；保留罗盘并不能保证任意 GNSS 航向异常都无缝回退，须通过当前 PX4 版本和实机日志验证。</p>
</div>

## 参考资料

阅读顺序参考了[零一 OneRTK UM982 的 PX4 教程](https://docs.01aero.cn/docs/0NcgNwij)：先启用 CAN、设置安装位置，再确认航向。本板使用独立的 ESP32 固件与参数定义，板端安装角用 `GPS_YAW_OFFSET`，经典 PX4 飞控端核对 `EKF2_GPS_YAW_OFF=0`；不要套用零一外设固件、UART 命令或把参数写入错误的 Component。

- [S1：PX4 v1.17.0 DroneCAN GNSS 接收源码](https://github.com/PX4/PX4-Autopilot/blob/v1.17.0/src/drivers/uavcan/sensors/gnss.cpp)
- [S2：PX4 v1.17 DroneCAN 配置说明](https://docs.px4.io/v1.17/en/dronecan/index)
- [S3：PX4 v1.17.0 GNSS/EKF 参数定义](https://github.com/PX4/PX4-Autopilot/blob/v1.17.0/src/modules/ekf2/params_gnss.yaml)
- [S4：PX4 v1.17 参数参考](https://docs.px4.io/v1.17/en/advanced_config/parameter_reference)
