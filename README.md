# yun
云

# 云中控说明

### 边界服务器原则
* 边界服务器的总体原则是定时从云上拉取数据及多媒体资源，其中数据给中控，多媒体资源则进入共享文件夹
* 数据拉取和多媒体资源拉取分开拉取，数据拉去采用全量拉取更新，多媒体拉取则通过MD5校验差异后拉取，一旦发现新版本，要先确保多媒体全部拉取完成后，再进行数据的拉取。
* 数据推送到中控通过websocket主动连接进行推送
* 数据也支持中控主动请求来获取，所以边界服务器需要缓存上一次的拉取结果
* 主动推送还是等待中控请求获取，需在界面上可配置
* 边界服务器从云端拉去最新数据采用定时请求，定时时间可配置，配置采用cron表达式
* 边界服务器每次请求判断版本号是否变更后，再进行新版本的拉取，所以云端WEB需要有一个发布按钮，发布后版本号更改
* 边界服务器如果在主动推送模式下拉取到新数据，则直接推送给中控服务器， 如果失败则等待下次定时请求时再推送

### 中控服务器原则
* 所有本地改变都是临时性改变，边界服务器同步后所以临时性改变消失
* 不支持反向同步， 只接受边界服务的同步请求
* 中控服务器每次获取数据后，都进行数据全量删除及新增



### 数据格式
#### 总数据格式如下：
```json
{
    "version": "1.3.2",     
    "resources":[],
    "scenes":[],
    "ipadRegions":[],
    "ipadButtons":[],
    "padBackground": "XXXXXXXX"
}
```

version代码数据版本   
resources为资源列表  
scenes为场景列表（复合动作列表）    
ipadRegions为pad区域列表    
ipadButtons为pad按钮列表
padBackground为图片的base64格式

#### 资源resource格式如下：
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "procotol": "协议名称",
    "procotolParam": "协议参数",
    "address":"控制地址",
    "type":"资源类型，例如开关、灯、矩阵、电脑等"
}
```
每个属性value值就是他的含义     
上述几个属性为基础属性，不管何种资源都需要有以上几个属性，除此之外其他的资源属性则根据资源类型自由添加  
例如当type为电脑时，出了上述几个属性外，还有url、channel、mousePosition属性  
当type为矩阵时,还有属性inChannel，outChannel

#### 场景scene格式如下：
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "actions":[]
}
```
其中actions是动作集合, 动作的具体定义见后

#### 区域ipadRegion格式如下：
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "left": 0.3333,
    "top": 0.3333,
    "iconName":"blue.png",
    "useRegion":"可见范围，三种，导览模式、管理员模式、任何模式",
    "actionGroups":[]
}
```
 其中left和top为左坐标和上坐标，及距左占整个宽度的百分比以及距上占整个高度的百分比， 百分比以小数显示

actionGroups为动作组集合，动作组的具体定义见后

#### pad场景按钮格式如下：
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "sceneId": "场景ID",
    "name": "按钮名称",
    "left": 0.3333,
    "top": 0.3333,
    "iconName":"blue.png",
    "useRegion":"可见范围，三种，导览模式、管理员模式、任何模式"
}
```
其中left和top为左坐标和上坐标，及距左占整个宽度的百分比以及距上占整个高度的百分比， 百分比以小数显示

#### actionGroups动作组格式如下：
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "useRegion":"可见范围，三种，导览模式、管理员模式、任何模式",
    "actions":[]
}
```
其中actions代码动作集合， 动作的具体定义见后

#### action动作基础格式如下：
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "type": "动作类型名称,例如ComputerShowAction、SleepAction"
}
```
##### 资源动作
动作类型较多，对于资源对应的基础动作格式如下:
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "controlType": "控制类型：例如电源开、电源关",
    "resourceId": "资源ID"
}
```
电脑显示多媒体动作格式如下(类型为ComputerShowAction)：
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "controlType": "控制类型：例如电源开、电源关",
    "resourceId": "电脑资源ID",
    "url":"文件名或者URI",
    "type":"ComputerShowAction"
}
```
电脑鼠标移动动作格式如下(类型为ComputerMouseAction)：
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "controlType": "控制类型：例如电源开、电源关",
    "resourceId": "电脑资源ID",
    "mousePosition":"123,456",
    "type":"ComputerMouseAction"
}
```
矩阵切换动作格式如下(类型为MatrixSwitchAction)：
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "controlType": "控制类型：例如电源开、电源关",
    "resourceId": "电脑资源ID",
    "inChannel":"1",
    "outChannel":"2",
    "type":"MatrixSwitchAction"
}
```

##### 其他动作
休眠动作(类型为SleepAction)
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "time": 12345,
    "type":"SleepAction"
}
```
循环动作(类型为WhileAction)
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "whileCount": 1,
    "type":"WhileAction"
}
```
场景动作(类型为SceneAction)
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "sceneId": "uuid-uuid-uuid-uuid-uuid",
    "doType": "取消执行或执行",
    "type":"SceneAction"
}
```
并发任务动作(类型为ConcurrentAction)
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "actions":[],
    "type":"ConcurrentAction"
}
```
actions为并发执行的动作集合，包含所有动作类型
中止并发任务动作(类型为ConcurrentCancelAction)
```json
{
    "id": "uuid-uuid-uuid-uuid-uuid",
    "name": "名称",
    "concurrentActionId": "并发任务ID",
    "type":"ConcurrentCancelAction"
}
```
