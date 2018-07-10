
# Core Bluetooth Overview
Core Bluetooth 框架可以让iOS，Mac 和蓝牙低能耗设备通信，如心率检测器，数字恒温器，甚至其它 iOS 设备。该框架是基于蓝牙4.0规范的抽象，隐藏了很多底层细节，使开发者更专注于应用和蓝牙低能耗设备的交互。

在蓝牙通信中，有两个主要的角色，Centrals 和 Peripherals。Peripherals 通常有其它设备需要的数据，Centrals 通常使用 Peripherals 的数据来完成某项任务，有点类似于 client-server 架构。

<p align="center" >
  <img src="https://upload-images.jianshu.io/upload_images/2153441-51fbd5b737a6d07c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" alt="Central and peripheral devices" title="Central and peripheral devices" width=80% height=80%>
</p>

Peripherals 以 advertising packet 的形式广播拥有的数据，advertising packet 是数据较小的数据包，如数据 Peripherals 的名字和主要功能。例如在数字恒温器可以广播当前的室温。Centrals 可以扫描和监听正在 advertising 信息的 Peripherals。

<p align="center" >
  <img src="https://upload-images.jianshu.io/upload_images/2153441-7f4c5804cf246d0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" alt="Central and peripheral devices" title="Central and peripheral devices" width=80% height=80%>
</p>

连接 Peripheral，然后进行数据交互， Peripheral 的数据是如何构造的呢？Peripheral 包含一个或者多个服务，或者提供与连接信号强度相关的信息。服务是设备功能或特性数据和相关行为的集合，例如心率检测器的服务可以是检测心率检测器暴露的数据。服务本身是由特征和服务组成，特性提供了 Peripheral 的更详细信息。

<p align="center" >
  <img src="https://upload-images.jianshu.io/upload_images/2153441-0712958934e24a85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" alt="A peripheral’s service and characteristics" title="A peripheral’s service and characteristics" width=30% height=30%>
</p>

# 常见的 Central 任务


