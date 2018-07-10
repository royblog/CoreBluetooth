
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

# Common Central Role Tasks
* Start up a central manager object
* Discovering Peripheral Devices That Are Advertising
* Connecting to a Peripheral Device After You’ve Discovered It
* Discovering the Services of a Peripheral That You’re Connected To
* Discovering the Characteristics of a Service
* Retrieving the Value of a Characteristic
* Writing the Value of a Characteristic

## Starting Up a Central Manager
在执行蓝牙低能耗任务之前，需要分配和初始化 Central manager，queue 为 nil，使用 main queue。创建 central manager，必须实现 centralManagerDidUpdateState: 代理方法。
```
    myCenteralManager = [[CBCentralManager alloc] initWithDelegate:self queue:nil options:nil];
```

## Discovering Peripheral Devices That Are Advertising
应用使用 scanForPeripheralsWithServices：查找附近正在 advertising 的 perpheral 设备。如果第一个参数为 nil，central manager 返回所有发现的 peripheral，无论是否支持服务。在真实的应用中，通常要指定一组 CBUUID 对象，每个对象代表着 peripheral 正在通告服务的通用唯一标志符。指定 UUID 数组，central manager 仅返回通告这些服务的 peripheral。
```
    [myCenteralManager scanForPeripheralsWithServices:nil options:nil];
```
central manager 发现 peripheral，调用 centralManager:didDiscoverPeripheral:advertisementData:RSSI: 代理方法，
```
-(void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary<NSString *,id> *)advertisementData RSSI:(NSNumber *)RSSI {
    NSLog(@"Discovered %@", peripheral.name);
}
```
如果你希望连接多个设备，可以使用数组保存发现的 peripherals，在任何情况下，找到你期望的 peripheral 设备，请停止扫描其它设备以节省电量。
```
    [myCenteralManager stopScan];
```
## Connecting to a Peripheral Device After You’ve Discovered It
发现感兴趣的服务，可以使用 connectPeripheral:options: 连接
```
    [myCenteralManager connectPeripheral:peripheral options:nil];
```
连接请求成功，调用 centralManager:didConnectPeripheral: 代理方法
```
-(void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral {
    NSLog(@"Peripheral connected");
}
```

## Discovering the Services of a Peripheral That You’re Connected To
在和 peripheral 建立连接后，可以浏览其数据。第一步是发现可用的服务，由于 peripheral 设备数据量大小限制，peripheral 的服务数量超过 advertises。可以使用 discoverServices：发现所有服务。在真实用中，通常不会传递 nil 值，因为 nil 值将返回所有的服务，浪费不必要的电量。可以指定服务的 UUIDs.
```
    [peripheral discoverServices:nil];
```
当服务被发现，调用 peripheral:didDiscoverServices: 
```
-(void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSError *)error {
    for (CBService *service in peripheral.services) {
        NSLog(@"Discovered service %@", service);
    }
}

```

## Discovering the Characteristics of a Service
找到感兴趣的服务，下一步是查找 服务的 characteristics，参数为 nil，是所有的 characteristics。在真实的应用中，第一个参数通常不传递 nil，通常指定 characteristics 的 UUIDs。
```
    [peripheral discoverCharacteristics:nil forService:interestingService];
```
当 characteristics 被发现，调用 peripheral:didDiscoverCharacteristicsForService:error: 代理
```
-(void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(NSError *)error {
    for (CBCharacteristic *characteristic in service.characteristics) {
        NSLog(@"Discovered characteristic %@", characteristic);
    }
}
```

## Retrieving the Value of a Characteristic
找到服务的 Characteristic，调用 readValueForCharacteristic:方法读取 characteristic 的值

```
    [peripheral readValueForCharacteristic:interestingCharacteristic];
```

成功读取 characteristic 的值，调用  peripheral:didUpdateValueForCharacteristic:error: 代理方法得到值。注意，并不是所有的 characteristic 值是可读的，可以检测 properties 字段中的 CBCharacteristicPropertyRead。如果你读取的值是不可读的，didUpdateValueForCharacteristic 将返回错误。

```
-(void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
    NSData *data = characteristic.value;
}
```

使用 readValueForCharacteristic: 读取一个静态值是没有问题的。获取动态时，比如心率，一直在变化，可以使用订阅的方式，当订阅的值发生变化时，可以收到通知。

```
    [peripheral setNotifyValue:YES forCharacteristic:interestingCharacteristic];
```

当订阅 characteristic 值时，调用  peripheral:didUpdateNotificationStateForCharacteristic:error:。并不是所有的值是可以订阅的，可以检测 properties 中 CBCharacteristicPropertyNotify 或 CBCharacteristicPropertyIndicate。
```
-(void)peripheral:(CBPeripheral *)peripheral didUpdateNotificationStateForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
    if (error) {
        NSLog(@"Error changing notification state: %@", [error localizedDescription]);
    }
}

```
当订阅成功，值发生改变，peripheral 设备通知到你的应用，每次值改变，调用 didUpdateNotificationStateForCharacteristic:error:。


## Writing the Value of a Characteristic
有时 characteristic 可写是有意义的。例如应用程序和蓝牙低功耗数字恒温器交互，你可能需要为恒温器设置一个房间的值。如果 characteristic 是可写的，可以通过 writeValue:forCharacteristic:type: 调用。

```
    [peripheral writeValue:dataToWrite forCharacteristic:interestingCharacteristic
```

在写入 characteristic 值，写入类型是 CBCharacteristicWriteWithResponse，让你的应用程序知道写入是否成功。characteristic 仅支持特定类型的写入，通过检查 CBCharacteristicPropertyWriteWithoutResponse 和 CBCharacteristicPropertyWrite，确定支持的写入类型。

```
-(void)peripheral:(CBPeripheral *)peripheral didWriteValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
    if (error) {
        NSLog(@"Error writing characteristic value: %@",[error description]);
    }
}
```


