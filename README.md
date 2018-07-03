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
=

直到我们建立bluetoothSocket连接之前，都要不断操作它BluetoothAdapter里的方法很多，常用的有以下几个：

cancelDiscovery() 根据字面意思，是取消发现，也就是说当我们正在搜索设备的时候调用这个方法将不再继续搜索

disable()关闭蓝牙

enable()打开蓝牙，这个方法打开蓝牙不会弹出提示，更多的时候我们需要问下用户是否打开，一下这两行代码同样是打开蓝牙，不过会提示用户：
Intemtenabler=new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
startActivityForResult(enabler,reCode);//同startActivity(enabler);

getAddress()获取本地蓝牙地址

getDefaultAdapter()获取默认BluetoothAdapter，实际上，也只有这一种方法获取BluetoothAdapter

getName()获取本地蓝牙名称

getRemoteDevice(String address)根据蓝牙地址获取远程蓝牙设备

getState()获取本地蓝牙适配器当前状态（感觉可能调试的时候更需要）

isDiscovering()判断当前是否正在查找设备，是返回true

isEnabled()判断蓝牙是否打开，已打开返回true，否则，返回false

listenUsingRfcommWithServiceRecord(String name,UUID uuid)根据名称，UUID创建并返回BluetoothServerSocket，这是创建BluetoothSocket服务器端的第一步

startDiscovery()开始搜索，这是搜索的第一步
 使用BluetoothAdapter的startDiscovery()方法来搜索蓝牙设备
startDiscovery()方法是一个异步方法，调用后会立即返回。该方法会进行对其他蓝牙设备的搜索，该过程会持续12秒。该方法调用后，搜索过程实际上是在一个System Service中进行的，所以可以调用cancelDiscovery()方法来停止搜索（该方法可以在未执行discovery请求时调用）。
请求Discovery后，系统开始搜索蓝牙设备，在这个过程中，系统会发送以下三个广播：
ACTION_DISCOVERY_START：开始搜索
ACTION_DISCOVERY_FINISHED：搜索结束
ACTION_FOUND：找到设备，这个Intent中包含两个extra fields：EXTRA_DEVICE和EXTRA_CLASS，分别包含BluetooDevice和BluetoothClass。
我们可以自己注册相应的BroadcastReceiver来接收响应的广播，以便实现某些功能


2.BluetoothDevice 描述了一个蓝牙设备
=

createRfcommSocketToServiceRecord(UUIDuuid)根据UUID创建并返回一个BluetoothSocket

getState() 蓝牙状态这里要说一下，只有在 BluetoothAdapter.STATE_ON 状态下才可以监听，具体可以看andrid api;
这个方法也是我们获取BluetoothDevice的目的――创建BluetoothSocket
这个类其他的方法，如getAddress(),getName(),同BluetoothAdapter

3.BluetoothServerSocket
=

这个类一种只有三个方法两个重载的accept(),accept(inttimeout)两者的区别在于后面的方法指定了过时时间，需要注意的是，执行这两个方法的时候，直到接收到了客户端的请求（或是过期之后），都会阻塞线程，应该放在新线程里运行！

还有一点需要注意的是，这两个方法都返回一个BluetoothSocket，最后的连接也是服务器端与客户端的两个BluetoothSocket的连接
	
	void   close()
	Closes the object and release any system resources it holds.
	void   connect()
	Attempt to connect to a remote device.
	InputStream getInputStream()
	Get the input stream associated with this socket.
	OutputStream    getOutputStream()
	Get the output stream associated with this socket.
	BluetoothDevice getRemoteDevice()
	Get the remote device this socket is connecting, or connected, to.
	获取远程设备，该套接字连接，或连接到---。
	booleanisConnected()
	Get the connection status of this socket, ie, whether there is an active connection with remote device.
	判断当前的连接状态

4.BluetoothSocket
=

跟BluetoothServerSocket相对，是客户端一共5个方法，不出意外，都会用到
	
close(),关闭
connect()连接
getInptuStream()获取输入流
getOutputStream()获取输出流
getRemoteDevice()获取远程设备，这里指的是获取bluetoothSocket指定连接的那个远程蓝牙设备

5、BluetoothClass
=

代表一个描述了设备通用特性和功能的蓝牙类。比如，一个蓝牙类会指定皆如电话、计算机或耳机的通用设备类型，可以提供皆如音频或者电话的服务。

每个蓝牙类都是有0个或更多的服务类，以及一个设备类组成。设备类将被分解成主要和较小的设备类部分。

BluetoothClass 用作一个能粗略描述一个设备（比如关闭用户界面上一个图标的设备）的线索，但当蓝牙服务事实上是被一个设备所支撑的时候，BluetoothClass的 介绍则不那么可信任。精确的服务搜寻通过SDP请求来完成。当运用createRfcommSocketToServiceRecord(UUID) 和listenUsingRfcommWithServiceRecord(String, UUID)来创建RFCOMM端口的时候，SDP请求就会自动执行。

使用getBluetoothClass()方法来获取为远程设备所提供的类。
两个内部类：

class   BluetoothClass.Device

定义所有设备类的常量

class   BluetoothClass.Service

定义所有服务类的常量

公共方法：

public int describeContents ()

描述包含在可封装编组的表示中所有特殊对象的种类。

返回值 

一个指示被Parcelabel所排列的特殊对象类型集合的位掩码。

public boolean equals (Object o)

比较带有特定目标的常量。如果他们相等则标示出来。 为了保证其相等，o必须代表相同的对象，该对象作为这个使用类依赖比较的常量。通常约定，该比较既要可移植又需灵活。

当且仅当o是一个作为接收器（使用==操作符来做比较）的精确相同的对象是，这个对象的实现才返回true值。子类通常实现equals(Object)方法，这样它才会重视这两个对象的类型和状态。

通常约定，对于equals(Object)和hashCode() 方法，如果equals对于任意两个对象返回真值，那么hashCode()必须对这些对象返回相同的纸。这意味着对象的子类通常都覆盖或者都不覆盖这两个方法。

参数

o   需要对比常量的对象

返回值

如果特定的对象和该对象相等则返回true，否则返回false。

public int getDeviceClass ()

返回BluetoothClass中的设备类部分（主要的和较小的）

从函数中返回的值可以和在BluetoothClass.Device中的公共常量做比较，从而确定哪个设备类在这个蓝牙类中是被编码的。

返回值

设备类部分

public int getMajorDeviceClass ()

返回BluetoothClass中设备类的主要部分

从函数中返回的值可以和在BluetoothClass.Device.Major中的公共常量做比较，从而确定哪个主要类在这个蓝牙类中是被编码的。

返回值

主要设备类部分

public boolean hasService (int service)

如果该指定服务类被BluetoothClass所支持，则返回true

在BluetoothClass.Service中，合法的服务类是公共常量，比如AUDIO类。

参数

service 合法服务类

返回值

如果该服务类可被支持，则返回true

public int hashCode ()

返回这个对象的整型哈希码。按约定，任意两个在equals(Object)中返回true的对象必须返回相同的哈希码。这意味着对象的子类通常通常覆盖或者都不覆盖这两个方法。

注意：除非同等对比信息发生改变，否则哈希码不随时间改变而改变。

public String toString ()  

返回这个对象的字符串，该字符串包含精确且可读的介绍。系统鼓励子类去重写该方法，并且提供了能对该对象的类型和数据进行重视的实现方法。默认的实现方法只是简单地把类名、“@“符号和该对象hashCode()方法的16进制数连接起来（如下列所示的表达式）：

	public void writeToParcel (Parcel out, int flags)
	将类的数据写入外部提供的Parcel中
	参数
	out     对象需要被写入的Parcel
	flags  和对象需要如何被写入有关的附加标志。可能是0，或者可能是PARCELABLE_WRITE_RETURN_VALUE。

以上是蓝牙主要操作的类。

基本用法：
==

1、获取本地蓝牙适配器
	
	BluetoothAdapter mAdapter= BluetoothAdapter.getDefaultAdapter();

2、打开蓝牙
	
	if(!mAdapter.isEnabled()){
		//弹出对话框提示用户是后打开
		Intent enabler = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
		startActivityForResult(enabler, REQUEST_ENABLE);
		//不做提示，强行打开，此方法需要权限&lt;uses-permissionandroid:name="android.permission.BLUETOOTH_ADMIN" /&gt;
		// mAdapter.enable();
	}

3、搜索设备

1)mAdapter.startDiscovery() 是第一步,可能你会发现没有返回的蓝牙设备

2)定义BroadcastReceiver,代码如下
	
	// 创建一个接收ACTION_FOUND广播的BroadcastReceiver
	private final BroadcastReceiver mReceiver = new BroadcastReceiver() {
	     public void onReceive(Context context, Intent intent) {
	          String action = intent.getAction();
	          // 发现设备
	          if (BluetoothDevice.ACTION_FOUND.equals(action)) {
	               // 从Intent中获取设备对象
	               BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
	               // 将设备名称和地址放入array adapter，以便在ListView中显示
	               mArrayAdapter.add(device.getName() + "/n" + device.getAddress());
	          }else if (BluetoothAdapter.ACTION_DISCOVERY_FINISHED.equals(action)) {
	               if (mNewDevicesAdapter.getCount() == 0) {
	                    Log.v(TAG, "find over");
	               }
	         }
	    };
	// 注册BroadcastReceiver
	IntentFilter filter = new IntentFilter(BluetoothDevice.ACTION_FOUND);
	registerReceiver(mReceiver, filter); // 不要忘了之后解除绑定

4、建立连接

首先Android sdk（2.0以上版本）支持的蓝牙连接是通过BluetoothSocket建立连接，服务器端（BluetoothServerSocket）和客户端（BluetoothSocket）需指定同样的UUID，才能建立连接，因为建立连接的方法会阻塞线程，所以服务器端和客户端都应启动新线程连接
1）服务器端：
	
	//UUID格式一般是"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"可到
	//http://www.uuidgenerator.com 申请
	BluetoothServerSocket serverSocket = mAdapter. listenUsingRfcommWithServiceRecord(serverSocketName,UUID);
	serverSocket.accept();

2)客户端：

	//通过BroadcastReceiver获取了BLuetoothDevice
	BluetoothSocket clienSocket=dcvice. createRfcommSocketToServiceRecord(UUID);
	clienSocket.connect();

5、数据传递

通过以上操作，就已经建立的BluetoothSocket连接了，数据传递是通过流

1）获取流
	
	inputStream = socket.getInputStream();
	outputStream = socket.getOutputStream();

2）写出、读入（JAVA常规操作）

补充一下，使设备能够被搜索
	
	Intent enabler = new Intent(BluetoothAdapter.ACTION_REQUEST_DISCOVERABLE);
	startActivityForResult(enabler,REQUEST_DISCOVERABLE);
关于蓝牙连接：

可以直接用以下代码进行连接：

	final String SPP_UUID = "00001101-0000-1000-8000-00805F9B34FB";
	UUID uuid = UUID.fromString(SPP_UUID);
	BluetoothSocket socket;
	socket = device.createInsecureRfcommSocketToServiceRecord(uuid);
	adapter.cancelDiscovery();
	socket.connect();

1.startDiscovey有可能启动失败

一般程序中会有两步：开启蓝牙、开始寻找设备。顺序执行了开启蓝牙-寻找设备的步骤，但是由于蓝牙还没有完全打开，就开始寻找设备，导致寻找失败。

解决办法：
	
	adapter = BluetoothAdapter.getDefaultAdapter();
	if (adapter == null)      {
	// 设备不支持蓝牙
	}
	// 打开蓝牙
	if (!adapter.isEnabled()){
	    adapter.enable();
	    adapter.cancelDiscovery();
	}
	// 寻找蓝牙设备，android会将查找到的设备以广播形式发出去
	while (!adapter.startDiscovery()){
	    Log.e("BlueTooth", "尝试失败");
	    try {
	        Thread.sleep(100);
	    } catch (InterruptedException e) {
	        e.printStackTrace();
	    }
	}

2.接收数据转换

使用socket.getInputStream接收到的数据是字节流，这样的数据是没法分析的，所以很多情况需要一个byte转十六进制String的函数：

	public static String bytesToHex(byte[] bytes) {
	    char[] hexChars = new char[bytes.length * 2];
	    for ( int j = 0; j &lt; bytes.length; j++ ) {
	        int v = bytes[j] &amp; 0xFF;
	        hexChars[j * 2] = hexArray[v &gt;&gt;&gt; 4];
	        hexChars[j * 2 + 1] = hexArray[v &amp; 0x0F];
	    }
	    return new String(hexChars);
	}

