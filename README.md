xposed 二次开发 SDK 参数文档说明

demo: `demo.apk`
main sdk directory: `sdk/*`

## 1. 调用说明

首先下载安装微控工具 xposed 模块，如何安装请参照 xposed 模块安装，在此不再说明。 在使用 sdk 的工程中加入 sdk 的接口动态库(libwtoolsdk.so)和接口调用 jar 包 (wtoolsdk.jar)    

引入接口调用 jar 包，然后调用相关函数即可，接口类名:com.easy.wtool.sdk.WToolSDK， 相关函数说明参见接口说明。     

示例:Android Studio 调用 SDK 示例     

## 2. 接口说明

### 1) 接口函数返回说明

所有接口函数(除编解码函数)都返回一个 json 格式的字符串，返回数据字段如下:

| 序号   | 字段名     | 类型   | 说明                                       |
| ---- | ------- | ---- | ---------------------------------------- |
| 1    | result  | 整形   | 执行结果编码，必须 <br /> 0 表示成功<br />为负表示错误代码，具体含义见errmsg |
| 2    | errmsg  | 字符串  | 执行错误描述，可选                                |
| 3    | content | 字符串  | 执行结果内容，可选，为 json 格式，具体字段见各接口说明。          |

示例:

执行成功返回:{“result”:0,”errmsg”:””}

执行失败返回:{“result”:-2000,”errmsg”:”SDK 未初始化”}

注:在接口中注明【编码】的字符串，需要调用字符串解码函数解码。

### 2) 字符串编码

函数:String encodeValue(String value) 

描述:字符串进行编码

参数:value 要编码的字符串 

返回:编码后的字符串

### 3) 字符串解码

函数:String decodeValue(String value) 

描述:对字符串进行解码(在传输过程中模块对一些字符串进行了编码，使用时要 进行解码，编码的字符串在相应的接口有说明，加有【编码】标识)

参数:value 要解码的字符串 

返回:解码后的字符串

### 4) SDK 初始化

函数:String init(String appId,String authCode) 

描述:在使用 SDK 前首先调用该接口进行初始化 

参数: 

appId 应用 ID(向接口提供者申请，每个使用 sdk 的应用都有一个唯一的 appid，用于在同一设备上多个应用同时使用 sdk)

authCode 授权码(向接口提供者获取，以下同) 

返回:

见接口函数返回说明，无 content。

### 5) SDK 释放

函数:String unload()

描述:在应用退出时需要调用该接口释放 SDK 创建的资源。

### 6) 任务接口

函数:String sendTask(String taskParams)

描述:SDK 任务接口，SDK 的所有操作都使用该接口进行，如发送文本消息等，具 体操作通过 taskParams 来指定。任务执行有些是异步的(标记为【异步】)，该接口 调用后会立即返回，结果成功只代表提交任务成功，不代表执行成功，执行成功需 要通过任务执行结果回调来查看(见:设置任务执行结果回调)，非异步任务(标记 为【同步】) 不需要回调;在调用该接口前，先设置任务执行结果回调 setOnTaskEndListener

注:所有字段名都为小写字母

参数及返回、回调:

taskParams 接口参数，json 格式字符串，根据不同内容做不同操作

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
| 1    | type         | 整形   | 操作类型，取值如下:<br /> 1:发送文本消息<br />2:发送图片消息<br />3:发送语音消息<br />4:发送视频消息<br />5:获取微信好友列表<br />6:获取微信群列表<br />12:转发消息<br />26:获取自己的用户信息(微信号昵称等)<br /> |
| 2    | taskid       | 长整形  | 任务编号，这个编号全局必须唯一，在任务执行 结果回调中使用该编号查看执行结果   |
| 3    | content Json | 对象   | 操作参数，根据不同操作类型确定，见 content 定义             |

返回见接口函数返回说明，content 根据不同操作确定，见 content 定义 

#### content 定义:

##### 1) 【同步】获取微信好友列表
action=5

参数 content:

| 序号   | 字段名       | 类型   | 说明              |
| ---- | --------- | ---- | --------------- |
| 1    | pageindex | 整形   | 页号(为 0 时获取所有)   |
| 2    | pagecount | 整形   | 每页条数(为 0 是获取所有) |

返回 content:json 数组，每条好友信息如下(不需要回调，结果同步返回):

| 序号   | 字段名      | 类型   | 说明                               |
| ---- | -------- | ---- | -------------------------------- |
| 1    | wxid     | 字符串  | 好友 wxid【编码】 注:以下发消息中的 wxid 就是取于此 |
| 2    | nickname | 字符串  | 好友昵称【编码】                         |
| 3    | wxno     | 字符串  | 好友微信号，可能为空，暂无用 处【编码】             |

##### 2) 【同步】获取微信群列表
action=6 

参数 content:

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1| pageindex| 整形| 页号（为0时获取所有）|
|2| pagecount| 整形| 每页条数（为0时获取所有）|
|3| ismembers| 整形| 是否要获取成员列表（若为是则返回 members 字段）0:否，1:是|


返回 content:json 数组，每条群信息如下(不需要回调，结果同步返回): 

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1| wxid| 字符串| 群 id,wxid【编码】 注:以下发消息中的 wxid 就是取于此|
|2| nickname| 字符串| 群昵称【编码】|
|3| wxno| 字符串| 群微信号，可能为空，暂无用处【编 码】|
|4| members Json |数组 |群成员列表，格式同接口微信好友 列表返回的微信好友列表|

##### 3) 【异步】发送文本消息
action=1 

参数 content:

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1| talker| 字符串| 接收者的 wxid|
|2| atwxids Json| 数组| @给某个或几个群成员，这里是 wxid 列 表，只有发群消息有效<br /> 例:
[“wxid_xxxxx”,”wxid_yyyyy”] |
|3| text| 字符串| 文本内容【编码】|
|4| timeout| 整形| 超时时间，秒(设为-1 回调会立即返回， 不会等发送完毕，以下同)|

返回 content:空(需要回调，发送是否完成异步返回) 回调 content:

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1| result| 整形| 执行结果编码，必须, <br> 0表示成功 <br>为负表示错误代码，<br>具体含义见errmsg|
|2| errmsg| 字符串| 执行错误描述，可选|

##### 4) 【异步】发送图片消息
action=2 

参数 content:

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1| talker| 字符串| 接收者的 wxid|
|2| imagefile| 字符串| 图片文件(本地文件，包括全路 径或图片链接)|
|3| timeout| 整形| 超时时间，秒|

返回 content:空(需要回调，发送是否完成异步返回) 回调 content:

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1| result| 整形| 执行结果编码，必须，<br>0 表示成功 <br>为负表示错误代码，<br>具体含义见 errmsg|
|2| errmsg| 字符串| 执行错误描述，可选|

##### 5) 【异步】发送语音消息
action=3

参数 content:

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1 |talker| 字符串| 接收者的 wxid|
|2| voicefile| 字符串| 语音文件(本地文件，包括全路径) (注:微信语音使用特殊的编码，与标准 amr 有区别，可以使用附带提供的(PC 版)MP3 与微信语音转换工具转换任意 MP3 为微信语音)|
|3| duration| 整形| 语音时长(单位:秒，微信目前只 允许发送时长小于 60 秒的语音)|
|4| timeout| 整形| 超时时间，秒|

返回 content:空(需要回调，发送是否完成异步返回) 回调 content:

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1| result| 整形| 执行结果编码，必须<br>0 表示成功<br> 为负表示错误代码，<br>具体含义见 errmsg|
|2| errmsg| 字符串| 执行错误描述，可选|

##### 6) 【异步】发送视频消息

action=4 

参数 content:

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1| talker| 字符串| 接收者的 wxid|
|2| videofile|字符串| 视频文件(本地文件，包括全路 径)(注:目前只发送 mp4)|
|3| videothumbfile| 字符串| 视频缩略图文件(本地文件，包 括全路径)，这个也必须，图片不能过大 `320*200` 差不多了，用于好 友收到后显示的那个图片|
|4 |duration| 整形| 视频时长(单位:秒)|
|5 |timeout| 整形| 超时时间，秒|

返回 content:空(需要回调，发送是否完成异步返回) 回调 content:

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1| result| 整形| 执行结果编码，必须<br>0 表示成功<br>为负表示错误代码，<br>具体含义见errmsg|
|2| errmsg| 字符串| 执行错误描述，可选|

##### 7) 【异步】转发消息

action=12

参数 content:

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1| talker| 字符串| 接收者的 wxid|
|2| msgtype| 整型| 消息类型，整型，(见消息监听)|
|3| msgid| 字符串 |消息 ID，字符串，(见消息监听) |
|4| timeout| 整形| 超时时间，秒|

返回 content:无 content(需要回调，发送是否完成异步返回) 回调 content:

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1| result| 整形| 执行结果编码，必须<br>0 表示成功<br> 为负表示错误代码，<br>具体含义见errmsg|
|2| errmsg| 字符串| 执行错误描述，可选|

##### 8) 【同步】获取自己的用户信息
action=26 

参数 content:无

返回 content:(不需要回调，结果同步返回): 

| 序号   | 字段名          | 类型   | 说明                                       |
| ---- | ------------ | ---- | ---------------------------------------- |
|1| wxid| 字符串| 自己的 wxid|
|2| nickname| 字符串| 自己的昵称【编码】 3 wxno 字符串 自己的微信号，可能为空|


### 7) 设置任务执行结果回调

函数:void setOnTaskEndListener(OnTaskEndListener taskEndListener) 

描述:设置任务执行结果回调，任务执行结束后通过回调检测执行结果;如发送文 本消息，发送完成后会触发该回调

参数:taskEndListener 回调接口 

返回:无

回调接口:

public interface OnTaskEndListener {

public void taskEndEvent(TaskEndEvent event); }

TaskEndEvent 属性函数如下:

getType() 操作类型，整形，由任务接口传入

getTaskId() 任务编号，长整形，由任务接口传入

getContent() 结果内容，json 字符串，根据不同任务类型返回不同结果，具

体定义见任务接口

### 8) 获取 SDK 版本号 

函数:String getVersion() 

描述:获取 SDK 版本号 

参数:无

返回:版本号，如:1.0.0.0