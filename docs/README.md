# QYAndroidAccSdk文档

_由俊云科技移动端团队开发，对外提供加速能力!_

### 对接教程(详细的可参考Demo源码)
##### 1.依赖加速库SDK

```groovy
//将sdk的aar文件拷贝到项目lib目录

android{
  ...
  defaultConfig {
    ...
      repositories {
          flatDir {
              dirs 'libs'
          }
      }
    ...  
  }
  ...
}
... 
dependencies {
  ...
  implementation (name: 'QiyouAccSdk_x.x.x', ext: 'aar')//添加依赖aar
  ...
}
```

##### 2.包含权限(<font color=#FF0000>sdk已默认包含前面的必备权限,后面的可选权限根据需要自行添加</font>)

```xml
必备权限(sdk已包含):
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
```

##### 3.混淆配置
```kotlin
sdk已默认包含,无需额外设置
```

##### 4.android版本要求
```groovy
minSdkVersion 21
```

##### 5.代码里面调用加速操作均为单例类调用
```kotlin
QyAccelerator.getInstance().xxxx(xxx)      //xxxx方法即为下面详细介绍的方法

此处注意📢：调用均是QyAccelerator类的getInstance()方法，而不是curInstance()方法，curInstance()此为sdk内部使用，业务方不要调用
```

##### 6.反馈问题时日志获取方法
```kotlin
1).先保证sdk的初始方法QyAccelerator.getInstance().init(xxx)的setDebug值为true,保证可以输出日志
2).运行程序复现问题
3).再按文章教程过滤加速器日志发给提供商即可,文章链接:https://juejin.cn/post/7264792741950439476
```

### SDK相关数据接口
##### 1.获取游戏区服列表
```kotlin
loadQyGameZoneData(packageNameStr: String? = "", onLoadQyGameZoneDataCallBack: QyUnifiedProcessStrategy.OnLoadQyGameZoneDataCallBack?)
```
>结果回调:
>
```
object : QyUnifiedProcessStrategy.OnLoadQyGameZoneDataCallBack {
    override fun loadQyGameZoneData(isSuccess: Boolean, errCode: Int? = 0, errMsg: String? = "OK", gameZoneMap: MutableMap<String, List<QyGameInfoBean.Game.ZoneInfo?>?>?) {
      
    }
}
```
>参数packageNameStr为要获取的游戏包名,多个请用英文逗号拼接,比如:"com.tencent.mm1,com.11xx.mm"
>回调参数说明：
>返回数据 isSuccess: true代表获取成功,false代表失败, 
>errMsg: isSuccess为true时为"OK"字符串,false为具体报错信息;
>gameZoneMap: 游戏区服列表字典,字典key为传递的单个包名,value为包名对应的区服列表;
>
>当isSuccess为false时,errCode值为非0,具体报错code如下:
>
>| errCode | errMsg |
>| :--:    | :----- |
>| 900    | The SDK is not initialized, please call init in Application to initialize the SDK |
>| 910    | 开放平台用户Token信息为空 |
>| 920    | Return game zone data is empty, please try again! |
>| 930    | 具体接口报错信息 |

### SDK相关加速方法

###### 初始化
```kotlin
/**
 * 在Application内调用
**/
init(application: Application, qyAccConfig: QyAccConfig, needAccStrategy: QyAccStrategy, isForceKillExistVpn: Boolean? = true)
```
> application: app上下文
>
> qyAccConfig: 全局配置器,具体配置如下
>
> needAccStrategy: 配置的加速策略,请配置[QyAccelerator.QyAccStrategy.ApkPure]即可,配错将导致检验失败抛出异常导致崩溃
>
> isForceKillExistVpn: 直接配置true即可
>***
> > qyAccConfig目前常用配置说明如下(还有未注明的其他配置可忽略):
> >
> > > setAppId  设置开放平台appId,直接设置成"QyAccSdk"即可
> > >
> > > setDebug  是否输出debug日志,默认false,生产建议关闭,会输出很多信息,日志可通过过滤: "=QyAccelerator==Log="标识查看
> > >
> > > setAppVersion   客户端版本号
> > >
> > > setLogCanUpload  是否可以上传debug日志,启用后加速会记录日志,需要时可调用后面方法获取上传
> > >
> > > e.g:  QyAccConfig.Builder().setAppId("1111").setDebug(true).build()



###### 获取sdk版本号

```kotlin
getSdkVerCode(): String
```

> 可用于未来的版本兼容判断等,一般不需要



###### 设置用户token

```kotlin
fun setQyUserToken(userToken: String?, extraGuidStr: String? = null, extraSaltStr: String? = null, setResultCallback: ((isSuccess: Boolean, errMsg: String?) -> Unit))
```

> userToken: 配置业务调用方的token,比如传入设备id等
> extraGuidStr: 可选的定制参数,根据需要传入
> extraSaltStr: 可选的定制参数,根据需要传入
> setResultCallback: 设置时会异步判断token是否有效,isSuccess为true时表示token正确有效,为false请查看errMsg失败原因,极大概率是加速启动中和加速成功时调用了该设置方法或token用错测试环境或生产环境
> 📢注意:一般调用时机为打开app如果处于登录态时主动设置一次,登录成功设置一次,加速失败或其他方法错误码为"用户token失效"类似时需设置一次,加速启动中和加速成功时注意不要设置,否则会导致加速断开等异常

###### 删除退出用户

```kotlin
delQyUserToken(): Boolean
```

> 退出用户在奇游sdk的登录态,如果加速中会同时停止加速,如果ping测任务执行中会同时停止ping测任务
> 返回值: true代表设置成功,false代表设置失败,失败原因需过滤日志查看
> 📢注意:一般调用时机为app的用户登录失效时调用,用户主动退出登录的时候调用,加速启动中和加速成功时注意不要设置


###### 启动加速

```kotlin
/**
 * 启动加速
**/
startQyGameAccelerate(qyAccGameInfo: QyAcctGameInfo?)
```

```kotlin
qyAccGameInfo: 加速相关实体信息(下面未备注的字段就是可选的,可忽略)
实体解析:QyAcctGameInfo(val accGamePkgName: String?, val accGameZoneFlag: String?, val accSelectNode: QyAcctNodeBean.Node? = null, var accNotifyConfig: AccNotifyConfigBean? = null, val dualChannel: QyAcctDualChannel? = null, val accPreFrontFailBean: AccPreFrontFailureBean? = null, val extraParam: Any? = null)
accGamePkgName: 必传,待加速包名信息,类型:String
accGameZoneFlag: 必传,传手动选择的区服标志,类型:String,目前传获取的区服id即可
AccNotifyConfigBean.notifyGameName: 可选,设置本次加速成功后通知栏本游戏显示名字,适配到[正在为您加速 - %s]里面,便于多语言适配支持(注:新App安装后默认不显示通知栏,需要系统设置通知管理里面开启显示)
AccNotifyConfigBean.notifyGameTxtTitle: 可选,设置本次加速成功后通知栏标题文字,否则使用QyAccConfig.notifyGlobalTxtTitle,如果也为空,使用sdk定死文案(正在为您加速 - %s)
AccNotifyConfigBean.notifyGameSmallLogo: 可选,设置本次加速成功后通知栏显示小图标,否则使用QyAccConfig.notifyGlobalSmallLogo,如果也为空,将不展示图标
AccNotifyConfigBean.notifyGameLargeLogo: 可选,设置本次加速通知栏显示大图标,否则使用QyAccConfig.notifyGlobalLargeLogo,如果也为空,将不展示图标
AccNotifyConfigBean.notifyClickParam: 可选,设置本次加速成功后通知栏点击参数,否则使用QyAccConfig.notifyGlobalClickParam,如果也为空,将无参数(多参数可自定义分割符号，如仿Get请求用=和&分割,获取时请用 QyAccelerator.NotifyParam 为key从onCreate和onNewIntent里面的intent获取传递的参数，同时记得在清单文件给主Activity设置 android:launchMode="singleTask/singleTop" ，否则onNewIntent将不触发)

整体大概如下:
QyAccelerator.getInstance().startQyGameAccelerate(QyAcctGameInfo("com.test.game", "900"))//启动加速
```
> 该方法会自动判定现在加速中的情况,会根据情况自动停掉前一个加速再启动新的加速,可根据日志和绑定的监听回调查看到信息,建议请根据回调状态处理好锁定界面操作,避免重复调用开始加速



###### 停止加速

```kotlin
stopQyGameAccelerate(appLayerCallStopMsg: String? = null): Boolean
```

>appLayerCallStopMsg参数非正常点击停止处调用停止方法的备注信息,手动停止请保持为空
>如果ping测任务执行中会同时停止ping测任务(可根据方法返回值判定是否停止成功,未停止成功可查看日志具体报错)



###### 绑定加速回调(可多次绑定)

```kotlin
bindQyAccRelatedListener(onQyAccRelatedListener: OnQyAccelerateListener?)

 注:onQyAccRelatedListener里面的几个回调方法是联动的，比如加速成功onAccEventCallBack会回调eventCode为QyCode_AccFinalSuccess,同时onAccStartProgress会回调进度100,onAccCurrentStatus也会回调QyStatus.AccSuccess；可以理解为onAccStartProgress和onAccCurrentStatus是onAccEventCallBack整个流程code对应的一系列事件的分段和汇总
```

>加速状态回调
>OnQyAccelerateListener.onAccCurrentStatus(status: QyAccelerator.QyStatus, curGamePkgName: String?, curGameZoneFlag: String?, eventCode: Int, eventMsg: String?, extraParam: Any?)
>
>>status为状态码,具体见下表;curGamePkgName代表当前对应操作的包名,curGameZoneFlag代表当前对应操作的区服标识,可用于事件联合比对判断操作;extraParam在某些状态可能会有对应的附加值

>| 和status对应的常量字符串 | status | 描述             | eventCode/eventMsg             | extraParam
>| :------------------------- | :--: | :----------------------- | :----------------------- | :----------------------- |
>| QyStatus.AccNormal | 0  | 正常未加速状态或停止恢复到的正常态 | eventCode为到此事件时最后的code,eventMsg为枚举值的name属性 :QyToNormalStatusFlag.FlagForWaitAcc为刚启动置正常态,QyToNormalStatusFlag.FlagForOkStopped为正常手动停止置正常态,QyToNormalStatusFlag.FlagForErrStopped为失败加速回退到正常态 | 便于后续扩展用 |
>| QyStatus.AccStarting | 1  | 启动加速中 | 加速的节点信息,为QyAcctNodeBean.Node对象 | 便于后续扩展用 |
>| QyStatus.AccSuccess | 2 | 加速成功 | 加速的节点信息,为QyAcctNodeBean.Node对象 | 便于后续扩展用 |
>| QyStatus.AccFailure | 3 | 加速失败 | eventCode和eventMsg为到此事件时最后的Code和Message | 便于后续扩展用 |
>| QyStatus.AccOkStopping | 4  | 手动停止加速中 | eventCode和eventMsg为到此事件时最后的Code和Message | 便于后续扩展用 |
>| QyStatus.AccErrStopping | 5 | 加速失败停止中 | eventCode和eventMsg为到此事件时最后的Code和Message | 便于后续扩展用 |

> 加速进度回调
> OnQyAccelerateListener.onAccCurrentProgress(progress: Int, curGamePkgName: String?, curGameZoneFlag: String?)
>
>> 当加速成功时progress会回调100,加速失败或停止时progress会回调0,中间值还有30,60等;curGamePkgName代表当前对应操作的包名,curGameZoneFlag代表当前对应操作的区服标识,可用于事件联合比对判断操作

>加速事件回调
>OnQyAccelerateListener.onAccEventCallBack(eventCode: Int, eventMsg: String?, curGamePkgName: String?, curGameZoneFlag: String?)
>
>>eventCode为事件码，eventMsg为事件对应信息,具体见附件文档;curGamePkgName代表当前对应操作的包名,curGameZoneFlag代表当前对应操作区服标识,可用于事件联合比对判断操作;
[eventCode为事件码详细对照表文档](./README_ErrCode.md)

> 加速附加回调
> OnQyAccelerateListener.onAccExtraInfoEvent(eventFlag: QyAccelerator.QyExtra, extraInfo: Any?, curGamePkgName: String?, curGameZoneFlag: String?)
>
>> eventFlag是本次回调的标志,根据策略不同可能还有特定的值,extraInfo是本次对应的信息,可能为数组,对象或普通值,根据eventFlag来确定判断

>| 和eventFlag对应的常量字符串 | eventFlag | 描述             | extraInfo             |
>| :------------------------- | :--: | :----------------------- | :----------------------- |
>| QyAccelerator.QyExtra.Http401 | 0  | 接口401数据 | 有加速接口401时回调,为HttpErrorException对象 |
>| QyAccelerator.QyExtra.Http403 | 1 | 接口403数据 | 有加速接口403时回调,为HttpErrorException对象 |

###### 解绑加速回调


```kotlin
unbindQyAccRelatedListener(onQyAccRelatedListener: OnQyAccelerateListener?)
```

> 绑定后记得解绑,参数传递绑定时的回调实例



#### 加速延迟


###### 绑定ping检测任务回调(可多次绑定)

```kotlin
bindQyAccSpeedPingTaskListener(onQyAccSpeedPingTaskCallBack: OnQyAccSpeedPingTaskCallBack?)
```
>  > OnQyAccSpeedPingTaskCallBack.qyAccSpeedPingTaskOnceStartOrStop(isStart: Boolean)

> 延迟结果
> > OnQyAccSpeedPingTaskCallBack.qyAccSpeedPingTaskOnceResult(isSuccess: Boolean, errMsg: String? = "OK", pingCount: Long, delayCount: Float, lostCount: Long, delayAvg: Float, lostRate: Float, complexPromotion: Int)
> > 参数说明:
> >
> > isSuccess: ping测是否成功
> >
> > errMsg: isSuccess为true时为"OK",false为具体的报错信息
> >
> > pingCount: ping测次数
> >
> > delayCount: 延迟总数
> >
> > lostCount: 丢包总数
> >
> > delayAvg: 平均延迟数
> >
> > lostRate: 丢包比率
> >
> > complexPromotion: 综合提升比率




###### 解绑ping检测任务回调


```kotlin
unbindQyAccSpeedPingTaskListener(onQyAccSpeedPingTaskCallBack: OnQyAccSpeedPingTaskCallBack?)
```

> 绑定后记得解绑,参数传递绑定时的回调实例




###### 启动ping检测任务

```kotlin
startAccNodePingTask(speedPingType: QyAccSpeedTest.SpeedPingType, sendPkgNum: Int? = 4, timerSecond: Int? = 5, udPingZoneList: List<String>? = null): Boolean
```
>  延迟检测启动回调(可根据方法返回值判定是否启动成功,未启动成功可查看日志具体报错)
>  speedPingType: 使用的ping测方式(类型枚举值同上"enum class SpeedPingType")
>  sendPkgNum: 发送包的数量,默认为4个,设置为空将改成4个
>  timerSecond: 是ping测任务的周期,单位是秒,限制值在5s~10s之间,包含5s和10s
>  udPingZoneList: udp的ping测区服列表,可不传递

###### 停止ping检测任务

```kotlin
stopAccNodePingTask()
```
> 不用记得停止ping测,当调用stopQyGameAccelerate停止加速也会自动停止

#### 加速计时


###### 绑定加速计时任务回调(可多次绑定)

```kotlin
bindQyAccTimingTaskListener(onQyAccTimingTaskCallBack: OnQyAccTimingTaskCallBack?)
```
>  > onQyAccTimingTaskCallBack.qyAccTimingTaskOnceStartOrStop(isStart: Boolean)

> 延迟结果
> > onQyAccTimingTaskCallBack.qyAccTimingTaskOnceResult(curSecond: Long, formatTiming: String)
> > 参数说明:
> >
> > curSecond: 当前秒数
> >
> > formatTiming: 秒数格式化,格式化成xx:xx:xx


###### 解绑加速计时任务回调


```kotlin
unbindQyAccTimingTaskListener(onQyAccTimingTaskCallBack: OnQyAccTimingTaskCallBack?)
```

> 绑定后记得解绑,参数传递绑定时的回调实例


###### 启动加速计时任务

```kotlin
startAccTimingTask(): Boolean
```
>  启动操作(可根据方法返回值判定是否启动成功,未启动成功可查看日志具体报错)



###### 停止加速计时任务

```kotlin
stopAccTimingTask()
```
> 不用记得停止,当调用stopQyGameAccelerate停止加速也会自动停止

###### 全日志切换开关

```kotlin
switchAccLogCanUpload(isLogCanUpload: Boolean)
```
> 参数:
中途切换全日志记录功能
isLogCanUpload:true打开,false关闭

###### 全日志开关状态

```kotlin
isAccLogCanUpload(): Boolean
```
> 参数:
返回全日志当前打开或关闭状态
返回true打开,false关闭

###### 全日志记录获取

```kotlin
fun getAccLogFileStorageList(): List<File>?
```
> 参数:
返回当前全部的加速日志文件记录

###### 全日志记录清除

```kotlin
fun clearAllAccLogFile(clearSourceInfo: String? = null): Boolean
```
> 参数:
执行一次全日志记录文件清除,清除后有新的加速日志又会继续记录;
返回true清除成功,false清除失败,失败原因查看日志


###### 上传全日志
```kotlin
fun uploadAccLogFileToServer(callerSelfInfo: String? = null, isUploadLastAccLog: Boolean? = false, logFileList: List<File>? = null,
                              callBack: ((logUploadStatus: QyLogUploadStatus, logFilePath: String?) -> Unit))
```
> 参数:
执行全日志内容的上传
参数
callerSelfInfo:(可选)调用方自定义的日志文件标识信息,一般设置登录的用户名
isUploadLastAccLog:(可选)上传 false:全部或true:最后一次加速日志
logFileList:(可选)需要上传的日志文件列表,不传或传null默认获取使用getAccLogFileStorageList获取到的日志文件列表
callBack回调参数logUploadStatus为日志上传状态枚举值,具体如下,获取具体的msg请用logUploadStatus.curMsg:
>```kotlin
>enum class QyLogUploadStatus(var curMsg: String){
>    UploadContextNull("上下文异常,无法上传"), UploadSwitchOff("日志上传开关未打开,无需上传"), UploadOnGoing("正在上传日志中,请等待完成再上传"),
>    UploadLogDataEmpty("暂无日志,无需上传"), UploadExcLoading("上传日志中..."), UploadLogNotExist("单次日志不存在,无法上传"),
>    UploadLogSuccess("上传单次日志成功"), UploadLogFailure("上传单次日志失败"), UploadLogException("上传单次日志异常"), UploadAllCompleted("上传全部日志完成")
>}
>```

###### 其他查询方法
```kotlin
/**
 * 是否启动加速中 （启动加速的过程中,还未加速成功）
 * @return Boolean   true-启动加速中 false-未加速,为null代表未加速
**/
isCurAccStartIng(): Boolean
```

```kotlin
/**
 * 是否加速成功（vpn已启动完成,正在加速中）
 * @return Boolean   true-加速成功 false-未加速,为null代表未加速
**/
isCurAccSuccess(): Boolean
```

```kotlin
/**
 * 获取加速游戏奇游ID(过时,已无唯一标识性,伪加速返回id均一样,请用下面isCurAccOkForGameInfo方法)
 * @return  Int  游戏ID,为null代表未初始化sdk或未加速
**/
getCurAccOkGameId(): Int?
```

```kotlin
/**
 * 获取设置的token
 * @return  String  设置的token,为null代表未初始化sdk
**/
getQyUserToken(): String?
```

```kotlin
/**
 * 获取设备唯一标识
 * @return  String  设备唯一标识
**/
getDeviceUuid(): String?
```

```kotlin
/**
 * 根据延迟数据计算综合提升
 * @return  String  综合提升
**/
fun getCalcPromotion(delay: Int): Int
```

```kotlin
/**
 * 获取网络对时后时间
 * @return  Long  毫秒
**/
curSyncDateTimeMill(): Long
```

```kotlin
/**
 * 获取网络对时后时间格式化字符串
 * @return  String  网络对时后时间格式化字符串
**/
getSyncDateTimeStr(): String?
```

```kotlin
/**
 * 获取初始化的加速配置
 * @return  QyAccConfig  加速配置类
**/
getQyAccConfig(): QyAccConfig?
```

```kotlin
/**
 * 获取初始化的app对象
 * @return  Application  初始化的app对象
**/
getQyApplication(): Application?
```

>>注:
>- sdk里面可能会有部分未在文档中列出的方法,为sdk内部使用的方法,忽略即可!
>- 如果文档注明对接方特别注意事项请仔细注意!