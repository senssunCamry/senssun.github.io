# **AndroidSDK**

# **1.简介：**

Android sdk目前支持android4.4及以上系统，同时请使用真机调试。

# **2.导入SDK中的SO和Jar包**

将在Android SDK 压缩包中libs目录下所有子文件拷贝至Android工程的libs目录下。有libcountjni.so和SsBleManagerSDK_110.jar ，如下图所示：

​                        

# **3.修改工程的build.gradle**

添加lib和jar

![image-20211117155205228](C:\Users\camry\AppData\Roaming\Typora\typora-user-images\image-20211117155205228.png)

```
android {
    defaultConfig {

        ndk {          
            abiFilters 'armeabi-v7a','arm64-v8a'
        }
    }

    sourceSets {
        main {
            jniLibs.srcDirs = ['libs'];
        }
    }
}
```







```
dependencies {
    implementation files('libs/SsBleManagerSDK_110.jar')
}
```





# **4.添加用户权限**

在工程 AndroidManifest.xml 文件中添加如下权限



```
<uses-permission android:name="android.permission.BLUETOOTH" />  
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/> 
```





**备注：Android6.0以上如果使用蓝牙扫描需要定位权限**



# **5.初始化**

初始化即创建SDK对象，只有初始化后才可以使用SDK的各项服务。建议将初始化放在程序入口处（如Application、Activity的onCreate方法），初始化代码如下：



```
ScaleManager.init(this);
ScaleManager.getInstance().addOnScaleListener(this);
```





侦听设备连接的状态和数据接收，Activity继承接口 ScaleManager.OnScaleListener，并实现接口方法



```
public void onScaleState(int state) {
    if (state == BluetoothProfile.STATE_CONNECTED) {
      }
}
```







# **6.启动扫描**



```
ScaleManager.getInstance().scan(1000l, new SSBleScanCallback() {
    @Override
    public void onScanResult(final BleDevice device) {
        if (lvBles != null) {
            runOnUiThread(new Runnable() {
                @Override
                public void run() {

                    if (devices.contains(device)) {
                        devices.get(devices.indexOf(device)).setRssi(device.getRssi());
                    } else {
                        devices.add(device);
                    }
                    Collections.sort(devices, BleDevice.Comparator);
                    adapter.notifyDataSetChanged();
                }
            });
        }

    }
});
```





数据会通过回调 SSBleScanCallback中的 onScanResult中BleDevice结构获取附近扫描到调备的 蓝牙信号，蓝名名称 和蓝牙的mac地址



```
bleDevice.getBluetoothDevice().getAddress() //获取蓝牙mac地址
bleDevice.getBluetoothDevice().getName() // 获取蓝牙名称
bleDevice.getRssi()//获取蓝牙信号强度值，越接近0信号越好
```





# **7.停止扫描**



```
ScaleManager.getInstance().stopScan();
```





# **8.连接设备**

使用扫描出来的设备deviceID连接，连接成功之前不能停止扫描



```
ScaleManager.getInstance().connect(final BleDevice bleDevice);
```





设备连接成功后，数据会通过如下按口函数返回值 



```
public void onWeighting(WeightBean bean, Object o)
```







```
public class WeightBean {
    @Keep
    int unit; //单位
    @Keep
    int division;//分度值
    @Keep
    int zuKang; //阻抗
    @Keep
    float weightKG; //kg的重量
    @Keep
    float weightLB; //lb的重量
    @Keep
    boolean isStable;//测量的稳定
    @Keep
    boolean isDt; 
    @Keep
    WeightBean.a dataType;
    @Keep
    String address; //秤体的mac地址
    @Keep
    SSFatBean ssFatBean;
```







# **9.断开设备连接**



```
ScaleManager.getInstance().disconnect();
ScaleManager.getInstance().removeOnScaleListener(this);
```





# **10.直流和交流秤体调用算法库方法**

**注：我司SenssunLife和MovingLife的算法是按如下时间采用了不同的算法，如客户要进行数据的对比，请根据如下时间帐号来查看。**

**建议新做的APP都直接采用新的算法使用**

\1. SenssunLife 目前版本是2021-01-01开始创建的主帐号（子帐号跟随主帐号）在直流秤上使用新的直流算法

\2. SenssunLife 目前版本是2021-01-01前创建的主帐号（子帐号跟随主帐号）在直流秤上使用旧的直流算法



**如下算法是****2021-01-01**之后调用的交流和直流算法方法**



```
如下算法是2021-01-01之后用的算法
/*********
 *AC : 0 是交流算法，1 是直流算法
* Sex  ：1 为男 ，0 为女
* Impedance:秤体上传的阻抗值
* Weight：秤体测量出来的体重值kg
* Height:用户所设定的身高值cm
* age ：用户所设定的年龄值
******/
//脂肪
public native static float CountGetFat3(float Impedance,float Weight,float Height,int Age, int Sex, int Ac) ;
//水分
public native static float CountGetMoisture3(float Impedance,float Weight,float Height,int Age, int Sex, int Ac) ;
//肌肉
public native static float CountGetMuscle3(float Impedance,float Weight,float Height,int Age, int Sex,int Ac) ;
//肌肉量
public native static float CountGetMuscleMass3(float Impedance,float Weight,float Height,int Age, int Sex,int Ac) ;
//骨骼肌
public static native float CountGetSkeletalMuscle3(float Impedance,float Weight,float Height,int Age, int Sex,int Ac) ;
//骨骼率
public static native float CountGetBoneMass3(float Impedance,float Weight,float Height,int Age, int Sex,int Ac) ;
//基础代谢率
public static native float CountGetBMR3(float Impedance,float Weight,float Height,int Age, int Sex,int Ac) ;
//活动代谢
public static native float CountGetAMR3(float Impedance,float Weight,float Height,int Age, int Sex, int activity,int Ac) ;
//内脏脂肪指数
public static native float CountGetVisceralFat3(float Impedance,float Weight,float Height,int Age, int Sex,int Ac);
//皮下脂肪
public static native float CountGetSubFat3(float Impedance,float Weight,float Height,int Age, int Sex,int Ac);
//身体年龄算法
public static native float CountGetBodyAge3(float Impedance,float Weight,float Height,int Age, int Sex, int activity, int ac);
//蛋白率
public static native float CountGetProtein3(float Impedance,float Weight,float Height,int Age, int Sex,int Ac);
//健康评分
public static native float CountGetBodyScore3(float Impedance,float Weight,float Height,int Age, int Sex,int Ac) ;
//体型判断
public static native float CountGetBodyType3(float Impedance,float Weight,float Height,int Age, int Sex,int Ac) ;

//脂肪量kg
public static native float CountGetFatWeight3(float Impedance,float Weight,float Height,int Age, int Sex,int ac) ;
//皮下脂肪量kg
public static native float CountGetSubFatWeight3(float Impedance,float Weight,float Height,int Age, int Sex,int ac);
//水分量kg
public native static float CountGetMoistureWeight3(float Impedance,float Weight,float Height,int Age, int Sex, int ac) ;
//骨量kg
public static native float CountGetBoneMassWeight3(float impedance,float weight,float height,int age,int sex,int ac);
//蛋白质kg
public static native float CountGetProteinWeight3(float Impedance,float Weight,float Height,int Age, int Sex,int ac);
//骨骼肌量kg
public static native float CountGetSkeletalMuscleWeight3(float Impedance,float Weight,float Height,int Age, int Sex,int ac) ;
//版本
public static native float CountGetVersion() ;
//Bmi
public static native float CountGetbmi(float weight,float height) ;
```







如下算法是2021-01-01之后调用的交流和直流算法方法



```
//交流算法
public native static float CountGetFat(float Impedance,float Weight,float Height,int Age, int Sex) ;
public native static float CountGetMoisture(float Impedance,float Weight,float Height,int Age, int Sex) ;
public native static float CountGetMuscle(float Impedance,float Weight,float Height,int Age, int Sex) ;
public native static float CountGetMuscleMass(float Impedance,float Weight,float Height,int Age, int Sex) ;
public static native float CountGetSkeletalMuscle(float Impedance,float Weight,float Height,int Age, int Sex) ;
public static native float CountGetBoneMass(float Impedance,float Weight,float Height,int Age, int Sex) ;
public static native float CountGetBMR(float Impedance,float Weight,float Height,int Age, int Sex) ;
public static native float CountGetAMR(float Impedance,float Weight,float Height,int Age, int Sex, int activity) ;
public static native float CountGetVisceralFat(float Impedance,float Weight,float Height,int Age, int Sex) ;
public static native float CountGetSubFat(float Impedance,float Weight,float Height,int Age, int Sex);
public static native float CountGetBodyAge(float Impedance,float Weight,float Height,int Age, int Sex,int activity);
public static native float CountGetProtein(float Impedance,float Weight,float Height,int Age, int Sex);
public static native float CountGetBodyScore(float Impedance,float Weight,float Height,int Age, int Sex) ;
//体型 ，1=隐形肥胖型 2=微胖型 3=肥胖型 4=稍瘦型 5=标准型 6=强壮型 7=过瘦型 8=活力型 9=肌肉发达型
public static native float CountGetBodyType(float Impedance,float Weight,float Height,int Age, int Sex) ;
```









```
 /*************************************************************************************************************
    * 旧的直流算法
   *************************************************************************************************************/
//脂肪率,单位%
public native static float CountGetFat2(float Impedance,float Weight,float Height,int Age, int Sex) ;
//水分率,单位%
public native static float CountGetMoisture2(float Impedance,float Weight,float Height,int Age, int Sex) ;
//肌肉率,单位%
public native static float CountGetMuscle2(float Impedance,float Weight,float Height,int Age, int Sex) ;
//肌肉量,单位千克kg
public native static float CountGetMuscleMass2(float Impedance,float Weight,float Height,int Age, int Sex) ;
//骨骼肌率,单位%
public static native float CountGetSkeletalMuscle2(float Impedance,float Weight,float Height,int Age, int Sex) ;
//骨骼量
public static native float CountGetBoneMass2(float Impedance,float Weight,float Height,int Age, int Sex) ;
//基础代谢,单位千卡kcal
public static native float CountGetBMR2(float Impedance,float Weight,float Height,int Age, int Sex) ;
//运动代谢,单位千卡kcal
public static native float CountGetAMR2(float Impedance,float Weight,float Height,int Age, int Sex, int activity) ;
//皮下脂肪率,单位%
public static native float CountGetSubFat2(float Impedance,float Weight,float Height,int Age, int Sex);
//身体年龄
public static native float CountGetBodyAge2(float Impedance,float Weight,float Height,int Age, int Sex, int activity);
//蛋白质
public static native float CountGetProtein2(float Impedance,float Weight,float Height,int Age, int Sex);
//健康评分
public static native float CountGetBodyScore2(float Impedance,float Weight,float Height,int Age, int Sex) ;
//体型 ，1=隐形肥胖型 2=微胖型 3=肥胖型 4=稍瘦型 5=标准型 6=强壮型 7=过瘦型 8=活力型 9=肌肉发达型
public static native float CountGetBodyType2(float Impedance,float Weight,float Height,int Age, int Sex) ;
```







# **11.交流秤体对秤体设定的方法调用**



```
/****************
*对秤体添加用户信息
* pin: pin码，四位数， 默认0001
* sex: 性别 男：1，女：0
* height： 身高 cm 
* age ：年龄  
* activity：活动等级 0~5
* unit：单位 0默认千克
* weightKG： 体重kg
*******************/
public void addUser(String pin, int sex, int height, int age, int activity, int unit, int weightKG)
 
/***********************************
* 同步历史数据 用户序号 1-12
* pin: pin码
* isAll： false同步当前的历史数据，
          true 同步所有历史数据
************************************/
public void sycHistoryData(String pin, boolean isAll) 

/***********************************
* 删除当前用户pin的对应的用户信息
* pin: pin码
************************************/
public void deleteUser(String pin)

/***********************************
* 查询所有用户信息
************************************/
public void queryAllUser() 
```







# **12.IF912B（透传直流秤体） 秤体调用算法库方法**



```
//脂肪 直流
public static native float  CountGetFatSingle
(float impedance,float weight,float height,int age,int sex);

//水分 直流
public static native float  CountGetMoistureSingle
(float impedance,float weight,float height,int age,int sex);

//肌肉 直流
public static native float  CountGetMuscleSingle
(float impedance,float weight,float height,int age,int sex);

//骨骼率
public static native float  CountGetBoneMassSingle
(float impedance,float weight,float height,int age,int sex);

//基础代谢率
public static native float  CountGetBMRSingle
(float impedance,float weight,float height,int age,int sex);

//活动代谢
public static native float  CountGetAMRSingle
(float impedance,float weight,float height,int age,int sex,int activity) ;


//蛋白率
//public static native float  CountGetProteinSingle
//        (float bodyMuscle,float bodyHydro);

//皮下脂肪
public static native float  CountGetSubFatSingle
(float impedance,float weight,float height,int age,int sex ,float bodyFat);

//身体年龄算法
public static native float  CountGetBodyAgeSingle
(float impedance,float weight,float height,int age,int sex ,int activity, float fat ,float bodyMuscle);

//健康评分
public static native float  CountGetBodyScoreSingle
(float impedance,float weight,float height,int age,int sex ,float bodyFat);

//体型判断
public static native int  CountGetBodyTypeSingle
(float impedance,float weight,float height,int age,int sex, float fat, float muscle);

//脂肪量，单位kg
public static native float  CountGetFatSingleW
(float bodyFat,float weight);

//皮下脂肪量，单位kg
public static native float  CountGetSubFatSingleW
(float subFat,float weight);

//水分量，单位kg
public static native float  CountGetMoistureSingleW
(float Hydro,float weight);

//肌肉量
public static native float  CountGetMuscleMassSingleW
(float bodyMuscle,float weight);

//骨量，单位kg 骨骼重量
public static native float  CountGetBoneMassSingleW
(float bodyBone,float weight);
```

