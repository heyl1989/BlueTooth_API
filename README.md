# BlueTooth_API
Android蓝牙开发全面总结
基本概念

安卓平台提供对蓝牙的通讯栈的支持，允许设别和其他的设备进行无线传输数据。应用程序层通过安卓API来调用蓝牙的相关功能，这些API使程序无线连接到蓝牙设备，并拥有P2P或者多端无线连接的特性。

 蓝牙的功能：

1、扫描其他蓝牙设备

2、为可配对的蓝牙设备查询蓝牙适配器

3、建立RFCOMM通道（其实就是尼玛的认证）

4、通过服务搜索来链接其他的设备

5、与其他的设备进行数据传输

6、管理多个连接

蓝牙建立连接必须要求：

1、打开蓝牙

2、查找附近已配对或可用设备

3、连接设备

4、设备间数据交互

首先，要操作蓝牙，先要在AndroidManifest.xml里加入权限

```
<uses-permissionandroid:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permissionandroid:name="android.permission.BLUETOOTH" />
```

Android所有关于蓝牙开发的类都在android.bluetooth包下，只有8个类

1.BluetoothAdapter 蓝牙适配器

