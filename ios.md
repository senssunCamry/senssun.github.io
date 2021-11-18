#  香山IOS SDK使用说明



SDK版本历史：

**版本V2.0.0 version V2.0.0**

1.整合支持云体重体脂秤，及手环 integrate the ble scale and ble band

 

**SDK 文件的引用  SDK file refercence：**

1. 选中Xcode项目的TARGETS,再选中Build Setting，在Linking->Other Linker Flags中输入-all_load.

​     Open xcode, the select TARGETS->Build Setting->Linking->Other Linker Flags, Add “-all_load”

2. 关闭bitcode

​     turn off bitcode

3. 引入sqlite3.tdb库

​     import sqlite3.tdb

4. 嵌入BLE.framework, FMDB.framework 库

​     reference BLE.framework, FMDB.framework

 

**创建蓝牙管理实例** **Create BLE Manager instance:**

**AppDelegate.h**

添加代码如下 append code:

**@property** (**nonatomic**, **retain**) SSBLEDeviceManager *bleMgr;

\-  (SSBLEDeviceManager *)getBLEMgr;

 

AppDelegate.m

**@synthesize** bleMgr = _bleMgr;

\- (SSBLEDeviceManager *)getBLEMgr {

  **if** (!_bleMgr) {

​    _bleMgr = [[SSBLEDeviceManager alloc] initWithDeviceTypes:@[@(SS_BLE_Scale_Broadcast_Fat_DC2),                               @(SS_BLE_Scale_Broadcast_Fat_AC2),                                @(SS_BLE_Scale_Broadcast_Mass2)]                           rssiMin:-127];

  }

  **return** _bleMgr;

}

 

**搜索秤体** **use phone ble to scan the ble scale:**

AppDelegate *delegate =(AppDelegate *)[UIApplication sharedApplication].delegate;

  [[delegate getBLEMgr] addDelegate:**self**];

  [delegate.bleMgr scanPeripherals];

 

接收搜索回调 **receive the scan callback**

\-  (**void**)didDiscoverPeripheral:(CBPeripheral *)peripheral

 

 

**秤体连接** **connect to the ble scale****：**

AppDelegate *delegate =(AppDelegate *)[UIApplication sharedApplication].delegate;

  [delegate.bleMgr connectWithSerialNOs:@[**self**.serialNO]];

 

秤体断开连接 disconnect to the ble scale：

 AppDelegate *delegate =(AppDelegate *)[UIApplication sharedApplication].delegate;

  [delegate.bleMgr disconnectWithSerialNOs:@[**self**.serialNO]];

 

Callback:

连接上秤体回调 ble scale connected callback

\-  (**void**)didConnectPeripheral:(CBPeripheral *)peripheral

秤体主动断开连接回调 ble scale disconnected callback

\-  (**void**)didDisconnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error

允许向秤体发送命令回调 allow to send order to ble scale callback

\-  (**void**)didAllowWriteDataToPeripheral:(CBPeripheral *)peripheral

接收到秤体数据回调 receive the ble scale callback

\-  (**void**)peripheral:(CBPeripheral *)peripheral didReceiveValue:(**id**)value

 

//秤体输出的数据 the ble scale data 
 SSBLEMassFat类，详细变量值，请查看类内部变量注释 see the class 

 

order:

*//*获取用户信息命令 get the scale users order

\- (**void**)sendGetUsers:(ReplyCallbackBlock)replyCallBack;

*//*获取历史数据命令 get the user history by pin

\- (**void**)sendGetHistory:(**int**)pin :(ReplyCallbackBlock)replyCallBack;

*//*设置用户信息命令 set the user info to the ble scale

\-  (**void**)sendSetUser:(**int**)operation :(**int**)number :(**int**)pin :(**int**)sex :(**int**)heightCM :(**int**)age :(**int**)sportMode :(NSString *)unitID :(**int**)bodyMassKG :(ReplyCallbackBlock)replyCallBack

 

The more info see demo.
