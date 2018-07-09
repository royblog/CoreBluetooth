
# Core Bluetooth Overview
Core Bluetooth 框架可以让iOS，Mac 和蓝牙低能耗设备通信，如心率检测器，数字恒温器，甚至其它 iOS 设备。该框架是基于蓝牙4.0规范的抽象，隐藏了很多底层细节，使开发者更专注于应用和蓝牙低能耗设备的交互。

在蓝牙通信中，有两个主要的角色，Centrals 和 Peripherals。Peripherals 通常有其它设备需要的数据，Centrals 通常使用 Peripherals 的数据来完成某项任务，有点类似于 client-server 架构。

<p align="center" >
  <img src="https://upload-images.jianshu.io/upload_images/2153441-51fbd5b737a6d07c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" alt="Central and peripheral devices" title="Central and peripheral devices" width=80% height=80%>
</p>

Peripherals 以 advertising packet 的形式广播拥有的数据，advertising packet 是数据较小的数据包，如 Peripherals 的名字和主要功能。例如在数字恒温器可以广播当前的室温。Centrals 可以扫描和监听


