https://gitee.com/open-enjoy/enjoy-iot


## 时序数据库

https://docs.taosdata.com/

数据模型
https://docs.taosdata.com/basic/model/

[主流时序数据库深度对比：TDengine、InfluxDB与IoTDB的技术特性、性能及选型考量_51CTO博客_时序数据库 influxdb](https://blog.51cto.com/u_13270164/13806671)

数据库都是十六进制解决办法

jdbc:TAOS-WS://localhost:6041?varcharAsString=true&conmode=1
## TCP调用链

### 1. 消息发布端（生产者）

**TCP数据接收 → 消息发布流程：**

1. **VertxTcpClient** 接收TCP数据
2. **DataDecoder** 解码数据
3. **TcpComponent** 处理数据并调用 `report()` 方法
4. **ThingComponent.report()** 构建 `ThingModelMessage` 对象
5. **AbstractComponent.sendMessage()** 将消息发布到 `THING_MODEL_MESSAGE_TOPIC` 主题


```
// AbstractComponent.java 中的发布逻辑
componentServices.getProducer().publish(Constants.THING_MODEL_MESSAGE_TOPIC, message);
```

### 2. 消息订阅端（消费者）

**消息订阅 → 存储到TDengine流程：**

#### 2.1 消息消费者注册

**`RuleDeviceConsumer`** 类负责订阅消息：


```
// RuleDeviceConsumer.java 中的订阅逻辑
public RuleDeviceConsumer(MqConsumer<ThingModelMessage> consumer) {
    consumer.consume(Constants.THING_MODEL_MESSAGE_TOPIC, this);
}
```

#### 2.2 消息处理分发

当消息到达时，`RuleDeviceConsumer.handler()` 方法会被调用，它会遍历所有注册的 `DeviceMessageHandler` 实现：


```
// RuleDeviceConsumer.java 中的消息处理逻辑
public void handler(ThingModelMessage msg) {
    for (DeviceMessageHandler handler : this.handlers) {
        // 异步调用每个handler的handle方法
        handler.handle(msg);
    }
}
```

#### 2.3 存储到TDengine

**`DeviceMessageLogHandler`** 是专门负责消息存储的handler：


```
// DeviceMessageLogHandler.java 中的存储逻辑
@Override
public void handle(ThingModelMessage msg) {
    //设备消息入库
    thingModelMessageData.add(msg);
}
```

#### 2.4 TDengine存储实现

**`ThingModelMessageDataImpl`** 实现TDengine存储：


```
// ThingModelMessageDataImpl.java 中的TDengine存储逻辑
@Override
public void add(ThingModelMessage msg) {
    // 构建INSERT SQL语句，按deviceId分表并关联超级表
    String sql = "INSERT INTO thing_model_message_? USING thing_model_message TAGS(?,?,?) VALUES(?,?,?,?,?,?,?,?,?)";
    // 使用TdTemplate执行SQL
    tdTemplate.update(sql, args);
}
```

### 3. 完整的调用链路总结


```
VertxTcpClient接收数据
    ↓
DataDecoder解码
    ↓
TcpComponent.report()
    ↓
ThingComponent.report() → 构建ThingModelMessage
    ↓
AbstractComponent.sendMessage() → 发布到THING_MODEL_MESSAGE_TOPIC
    ↓
RuleDeviceConsumer订阅消息 → 调用DeviceMessageHandler列表
    ↓
DeviceMessageLogHandler.handle() → 调用thingModelMessageData.add()
    ↓
ThingModelMessageDataImpl.add() → 使用TdTemplate插入TDengine
    ↓
TDengine数据库存储
```

### 4. 关键配置信息

- **消息主题**: `THING_MODEL_MESSAGE_TOPIC = "device_thing"`
- **消费者**: `RuleDeviceConsumer` 订阅该主题
- **存储Handler**: `DeviceMessageLogHandler` 负责调用存储服务
- **TDengine实现**: `ThingModelMessageDataImpl` 使用 `TdTemplate` 执行SQL

### 5. 数据流向示意图


```
TCP数据 → 消息队列(Redis MQ) → 消费者 → 存储Handler → TDengine数据库
    ↑           ↑              ↑          ↑           ↑
 VertxTcp  消息发布端     消息订阅端   存储逻辑    持久化存储
```

这样，整个从TCP数据接收到TDengine存储的完整链路就清晰了，包括了消息的发布和订阅两个关键环节。


## eiot_component表中TCP组件的启动机制

### 1. 组件启动模块

TCP组件的启动由 `TcpStarter` 负责，具体流程如下：


```
@Component
public class TcpStarter {
    @Resource
    private TcpVerticle tcpVerticle;

    @PostConstruct
    public void init() {
        Vertx vertx = Vertx.vertx();
        vertx.deployVerticle(tcpVerticle, ar -> {
            if (ar.succeeded()) {
                log.info("start tcp component success!");
            }
        });
    }
}
```

### 2. 如何通过TCP找到对应的component实现类

通过TCP类型找到对应实现类的完整流程如下：

#### 第一步：组件发现机制

- `ComponentManager` 订阅组件发现主题 `COMPONENT_DISCOVER_TOPIC`
- 当TCP组件启动时，会发送组件发现消息到该主题

#### 第二步：数据库查询

- 通过 `ComponentServiceImpl` 的 `getComponent(String type)` 方法查询数据库：


```
@Override
public ComponentInfo getComponent(String type) {
    return ComponentConvert.INSTANCE.convertInfo(
        componentMapper.selectOne(ComponentDO::getType, type)
    );
}
```

#### 第三步：TCP组件类型标识

- `TcpComponent` 的 `getType()` 方法明确返回 `"tcp"`：


```
@Override
public String getType() {
    return "tcp";
}
```

#### 第四步：组件配置管理

- `ThingComponent` 每5秒检查一次组件配置：


```
@Scheduled(fixedRate = 5000)
public void config() {
    ComponentInfo info = componentServices.getComponentApi().getInfo(getType());
    // 根据配置状态调用stateChange方法启动/停止组件
}
```

### 3. 完整的组件启动链路


```
TcpStarter.init() → TcpVerticle部署 → TcpComponent实例化 → 
组件发现消息发送 → ComponentManager接收 → 数据库记录创建 → 
ThingComponent.config()定时检查 → 根据eiot_component表配置启动TCP服务
```

### 4. 关键配置信息

- **组件类型**：`"tcp"`（对应eiot_component表的type字段）
- **组件名称**：`"tcp协议组件"`
- **启动类**：`TcpStarter`
- **实现类**：`TcpComponent`
- **Vertx管理**：`TcpVerticle`
- **客户端管理**：`VertxTcpClient`

这样，当eiot_component表中type为"tcp"的组件配置启用时，系统会自动启动对应的TCP协议组件，并通过VertxTcpClient管理TCP连接。