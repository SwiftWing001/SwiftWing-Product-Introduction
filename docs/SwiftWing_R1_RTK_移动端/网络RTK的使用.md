<div class="rtk-guide" aria-hidden="true"></div>

# 网络 RTK 的使用

网络 RTK 通过互联网从 CORS 服务获取 RTCM 差分，地面站接收后经数传链路与飞控 CAN 送到移动端，进行 RTK 位置解算。

数据流向：**CORS 服务 → QGC → 飞控 MAVLink → CAN → 移动端**。

## 一、准备工作

* 已在CORS服务商处（如千寻位置）购买账号，并获取以下信息：
  * 服务器地址（Host address）
  * 端口（Server port）
  * 用户名（Username）
  * 密码（Password）
* 运行地面站的电脑（或带地面站功能的遥控器）需同时具备两条链路：
  * **互联网**：用于连接CORS服务器；
  * **数传链路**：用于连接飞机、转发差分数据。
* 飞机已与地面站连接，地面站能正常收到飞机上报的GPS位置（连接后页面中"GGA Source"应显示为"Vehicle GPS"）。

<div class="admonition tip">
<p>使用网络差分时不需要现场 SwiftWing 基站。移动端仍需完成基线与飞控配置，操作顺序见<a href="../快速上手/">快速上手</a>；停止不使用的手动串口 / TCP 输入，避免抢占差分来源。</p>
</div>

## 二、QGC 地面站使用

1. 点击QGC图标，打开设置选项

![1790491808365](image/网络RTK的使用/1790491808365.png)

2. 选择"NTRIP/RTK"选项

![1790491980661](image/网络RTK的使用/1790491980661.png)

3. 将事先注册的信息填入对应位置

![1790492191054](image/网络RTK的使用/1790492191054.png)

对应图中标号，各字段含义如下：

1. **Host address**：CORS服务器地址（如 `203.107.45.154`）
2. **Server port**：服务器端口（如 `8003`）
3. **Username**：账号用户名
4. **Password**：账号密码
5. **Mount Point**：挂载点，即基准站数据源。本页示例服务使用 `AUTO`，实际名称按服务商提供的挂载点填写；也可点击"Browse"查看并手动选择，不要求所有服务都支持 AUTO。

<div class="rtk-fineprint">TLS 按服务商要求设置。图片为填写位置与状态示例，服务地址、账号和挂载点以自己的授权资料为准。</div>

### 当前 SwiftWing QGC B8 的配置位置

在左侧 **Server Settings**填写服务器与账号，在 **Mountpoint Browser**填写挂载点；需要上报 GGA 的服务还需在 **GGA Position Reporting**选择有效位置来源与发送周期。

[![B8的NTRIP服务器与账号设置](../RTK通用/image/实机操作/qgc-ntrip-server.png)](../RTK通用/image/实机操作/qgc-ntrip-server.png)

[![B8的NTRIP挂载点设置](../RTK通用/image/实机操作/qgc-ntrip-mountpoint.png)](../RTK通用/image/实机操作/qgc-ntrip-mountpoint.png)

[![B8的GGA位置来源与发送周期](../RTK通用/image/实机操作/qgc-ntrip-gga.png)](../RTK通用/image/实机操作/qgc-ntrip-gga.png)

<div class="rtk-caption">以上三图是留空账号的操作入口，没有连接第三方服务。不同界面版本字段位置可能不同，按对应名称填写。</div>

### 连接与查看定位状态

4. 点击"Connect"连接

![1790492243849](image/网络RTK的使用/1790492243849.png)

5. 显示"Connected"表示已经完成连接

![1790492323335](image/网络RTK的使用/1790492323335.png)

除状态显示"Connected"外，还应确认页面中"Data Received"与"To Vehicle"两组数据流量持续增长，表示差分数据已正常接收并转发给飞机。若只显示"Connected"而流量始终为0，定位状态不会变化。

6. 查看定位状态

点击主界面右上角的GPS图标，在弹出的"Vehicle GPS Status"中查看"GPS Lock"状态。连接RTK后，状态通常会按以下过程逐步收敛：

* 刚收到差分数据时，显示"3D DGPS Lock"：

![1790492389767](image/网络RTK的使用/1790492389767.png)

* 进入RTK解算后，显示"3D RTK GPS Lock (float)"，即浮点解（分米级），此时仍在收敛中：

![1790492830712](image/网络RTK的使用/1790492830712.png)

* 最终收敛为"3D RTK GPS Lock"（无float字样），即固定解；仍需完成本机安装、航向与任务所需的检查。

各状态对应关系如下：

| GPS Lock显示 | fix_type | 含义 |
| --- | --- | --- |
| 3D Lock | 3 | 普通3D卫星定位 |
| 3D DGPS Lock | 4 | 已收到差分数据，尚未进入RTK解算 |
| 3D RTK GPS Lock (float) | 5 | RTK浮点解，分米级，收敛过程中 |
| 3D RTK GPS Lock | 6 | RTK固定解；精度与任务要求另行确认 |

7. 通过MAVLink消息确认fix_type

在"Analyze Tools → MAVLink检测"中选择"GPS_RAW_INT"消息，查看其中的"fix_type"字段。

* "fix_type"为5，说明已进入RTK浮点解；在开阔环境下继续等待，收敛到6（固定解）时定位精度最高。

![1790492601463](image/网络RTK的使用/1790492601463.png)

## 三、常见问题

* **点击Connect后无法连接**：检查互联网是否正常；确认服务器地址、端口、用户名、密码是否正确；确认账号未在其他设备上登录被挤下线。
* **显示Connected但定位状态长时间不变**：检查"Data Received"/"To Vehicle"流量是否增长；确认数传链路正常、飞机已向地面站上报GPS位置。
* **长时间停留在float，无法收敛到固定解**：将飞机移到视野开阔、无遮挡的场地；确认所选Mount Point覆盖作业地区；查看卫星数量是否充足。收敛通常需要几十秒到数分钟，状态也可能反复波动，属正常现象。

## 四、Mission Planner 地面站使用

<div class="admonition note"><p>Mission Planner 的 NTRIP 配置步骤待补充；本页先按配套 QGC 流程使用。</p></div>
