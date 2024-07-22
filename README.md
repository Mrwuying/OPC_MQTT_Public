该文档主要负责演示，使用该软件访问OPC UA服务器后，获取点位数据，并上传到MQTT的过程
OPC UA为免费的工业协议标准，访问效率高性能好
对于OPC UA Client而言，成功连接OPC服务器需要知道，服务器IP地址，服务器servername，节点的item id
而item ID又与device ID和group ID有关

### 0、中间件运行的前置条件
#### 前置条件1：运行需要.NET 6.0支持
下载地址:[https://download.visualstudio.microsoft.com/download/pr/23c7bf0d-e22d-4372-bcb2-292eb36a5238/11af494be409759f46b679ab22e65a58/dotnet-sdk-6.0.424-win-x64.exe](https://download.visualstudio.microsoft.com/download/pr/23c7bf0d-e22d-4372-bcb2-292eb36a5238/11af494be409759f46b679ab22e65a58/dotnet-sdk-6.0.424-win-x64.exe)
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721282035164-b17146ff-2451-41e2-b78a-43ddead5dfdf.png#averageHue=%23ecebeb&clientId=u78be664c-5078-4&from=paste&height=82&id=N94V2&originHeight=103&originWidth=656&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=6902&status=done&style=none&taskId=uc9469608-44be-470e-b652-33f60c9d1a5&title=&width=524.8)
#### 前置条件2：测试环境，本地搭建MQTT Server，使用软件MQTT FX订阅本地地址和TOPIC进行中间件json内容监视
本地MQTT服务器:Broker选择开源的EMQX
注意：经测试，MQTT客户端MQTT FX和配置的ini文件内容遇到中文可能乱码，需要保证二者都为GB2312编码
#### 前置条件3：需要OPC服务，安装KEPServerEX 6或其他OPC软件后将会存在该服务，该服务将负责OPC相关的一系列支持
### 1、通过KEPServerEX 6配置OPC UA服务器
#### ①右键连接性——新建通道
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721269162167-b80a376c-bb5a-498f-928f-80d55ddbfaad.png#averageHue=%23f8f7f6&clientId=u37ea4af9-f3a1-4&from=paste&height=332&id=u7c2fd632&originHeight=415&originWidth=759&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=49522&status=done&style=none&taskId=udef605be-c61b-461c-a984-42c07d79888&title=&width=607.2)
#### ②选择驱动配【Omron Host Link】，该驱动为欧姆龙PLC的串口协议，我们以该选项作为测试样例。点击下一步
小提示：展开选项后，按键盘上的O可以快速定位到首字母为O的选项
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721269106350-6c04338d-40bf-4d55-91d3-873a718f58dc.png#averageHue=%23f7f7f6&clientId=u37ea4af9-f3a1-4&from=paste&height=664&id=ubd099a3a&originHeight=830&originWidth=1069&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=104278&status=done&style=none&taskId=u8143d782-b675-4480-b916-0dfd82407e2&title=&width=855.2)
#### ③自定义通道命名，中英文均可，命名为【欧姆龙PLC】
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721273393376-8b1f2f7a-3ef5-4c23-9f20-33da6b2767a5.png#averageHue=%23f5f5f4&clientId=u37ea4af9-f3a1-4&from=paste&height=777&id=u161c5ed1&originHeight=971&originWidth=1238&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=100452&status=done&style=none&taskId=u7c4093d6-d8a9-4138-8b6f-14e8ef648d1&title=&width=990.4)
#### ④配置通道，默认配置即可，点击下一步
以下均为默认配置
物理媒体:COM端口
COM ID:1
波特率:9600
数据位:7
奇偶性:偶
停止位:2
流量控制:无
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721269675159-a1f12a5f-c0da-4828-87b7-7590e94150ab.png#averageHue=%23f8f8f7&clientId=u37ea4af9-f3a1-4&from=paste&height=777&id=u4f671018&originHeight=971&originWidth=1238&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=191876&status=done&style=none&taskId=u00b50220-abd2-451e-96c0-737ae47b03e&title=&width=990.4)
#### ⑤后续步骤也是默认即可，一路点击下一步，完成
#### ⑥左键单击新建设备，自定义命名中英文均可，该命名对应OPC开发中的group ID，命名为【设备】，点击下一步
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721273654775-a023bf7e-1eb7-4dc7-8c15-8bee39a27632.png#averageHue=%23f7f7f6&clientId=u37ea4af9-f3a1-4&from=paste&height=777&id=ua7560894&originHeight=971&originWidth=1238&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=98438&status=done&style=none&taskId=u3ec82817-215a-4ce6-a59c-23d5a29295d&title=&width=990.4)
#### ⑦选择型号C20H，点击下一步
该型号为PLC的型号，C20H具体对应欧姆龙的CP1系列PLC，其他选项可通过网络搜索查询得到
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721273677900-fe8c41df-15c4-4ff7-a511-67cab0fd3d0e.png#averageHue=%23f7f6f6&clientId=u37ea4af9-f3a1-4&from=paste&height=777&id=u6dc575f5&originHeight=971&originWidth=1238&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=91051&status=done&style=none&taskId=u09f8f2fc-b712-4827-9ff2-a6d7c6dca98&title=&width=990.4)
#### ⑧后续页面也是使用默认配置，一路点击下一步即可，最后点击完成
可在左侧看到新建设备
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721273749988-99b13c71-9749-4333-8c5d-08818aab5dde.png#averageHue=%23fbfbfb&clientId=u37ea4af9-f3a1-4&from=paste&height=777&id=uddf72789&originHeight=971&originWidth=1238&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=71630&status=done&style=none&taskId=u4fd602b7-9bf4-4bd1-b7fa-adb41476aab&title=&width=990.4)
#### ⑨创建完group后，下面两个选项都与item ID有关
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721273767446-2cfbc0e3-1216-4a93-9460-65522d8766f8.png#averageHue=%23fafaf9&clientId=u37ea4af9-f3a1-4&from=paste&height=527&id=hW5jf&originHeight=659&originWidth=1054&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=78314&status=done&style=none&taskId=u9805ca1a-b028-47aa-a5eb-62f01ba885b&title=&width=843.2)
新建标记组，可以理解给标记分类，例如【故障类型的点位】【计数类型的点位】【状态类型的点位】
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721281445720-7f0909ea-86f4-41d7-9f9e-6680ee1ea779.png#averageHue=%23f8f7f6&clientId=u78be664c-5078-4&from=paste&height=233&id=u0c37cba7&originHeight=291&originWidth=939&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=38577&status=done&style=none&taskId=ufc44b8c4-9949-469b-ab48-558e7076128&title=&width=751.2)
新建标记，如不需要分类，也可直接建立点位
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721281432227-401801d4-bbac-47f9-9be9-a080e0d1a222.png#averageHue=%23f7f6f5&clientId=u78be664c-5078-4&from=paste&height=252&id=ucb5835cf&originHeight=315&originWidth=836&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=40539&status=done&style=none&taskId=uebc1a373-a9f7-4c7f-a0a1-cd10b6b9f07&title=&width=668.8)
#### ⑩点击【opc quick client】按钮，打开kepware自带的OPC Client，此OPC Client可在其他渠道单独下载

### 2、获取具体节点名称
具体节点名称 = [device id].[group id].[item id]
如下图，具体节点名称 = 欧姆龙PLC.设备1.计数.D100
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721281660018-69edbc9f-dec7-4a55-a167-20da21c2de8a.png#averageHue=%23f9f9f8&clientId=u78be664c-5078-4&from=paste&height=291&id=ub9f38d69&originHeight=364&originWidth=1207&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=62771&status=done&style=none&taskId=ud425885b-65bd-4b13-a1b7-8becc64dee3&title=&width=965.6)
如下图，具体节点名称 = 欧姆龙PLC.设备1.AR0
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721281784666-a525dc60-8141-4aeb-9f07-5279ad4ffbe3.png#averageHue=%23f7f6f4&clientId=u78be664c-5078-4&from=paste&height=251&id=ud7c2bdba&originHeight=314&originWidth=821&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=40655&status=done&style=none&taskId=u69cb6313-a293-43a2-aa11-87689fcbaaf&title=&width=656.8)
### 3、使能kepware的数据模拟功能
此设置可以保证在没有实物PLC的情况下，进行中间件的测试
#### ①右键【设备1】——属性
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721283320614-16f7f6dd-c94d-416a-b5d6-e05da35f4aab.png#averageHue=%23fafaf9&clientId=u78be664c-5078-4&from=paste&height=777&id=ud776c369&originHeight=971&originWidth=1238&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=90399&status=done&style=none&taskId=ub9db36ec-94d6-4be2-b737-c1a0820fb9b&title=&width=990.4)
#### ②常规——操作模式——模拟——是——确定
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721283283220-da5da5af-092a-419c-b6b9-17dde51021a8.png#averageHue=%23f4f4f4&clientId=u78be664c-5078-4&from=paste&height=777&id=u6089ebcb&originHeight=971&originWidth=1238&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=104065&status=done&style=none&taskId=ufe4ce8db-e472-4720-abbc-081650a8e36&title=&width=990.4)
### 

### 4、中间件的使用
#### ①断开连接，设置MQTT信息，图中为本地MQTT测试的一些信息，请按需更改
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721631512591-f6c852c3-d911-4f43-99dd-173842317324.png#averageHue=%23f3f3f3&clientId=u68e6f105-ff16-4&from=paste&height=638&id=u3d2e49d9&originHeight=797&originWidth=1331&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=91647&status=done&style=none&taskId=u2aa55836-3adf-4df2-86a5-73a03100b55&title=&width=1064.8)
#### ②切换到【设备列表】，点击【编辑INI文件】，将会开启INI配置文件所在的目录
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721294828615-3a2c9d7c-bde4-49b9-93fd-3f039d5d10ea.png#averageHue=%23d0d0cf&clientId=ub85352ab-5b31-4&from=paste&height=638&id=u694de2ea&originHeight=798&originWidth=1326&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=44269&status=done&style=none&taskId=u73522434-9c91-43ee-9030-f089316d977&title=&width=1060.8)
编辑ini配置文件（批量配置点位推荐方式）
为什么使用ini文件？考虑到表格导入会有各种各样的格式问题，文本相对来说好一点，最多考虑编码问题就好
**注意：手动编辑INI文件时，windows自带的记事本并不是一个好的选择，它保存时并不会按照原有编码进行保存，而是根据你的windows电脑语言区域进行选择，比如中国-简体中文，将会以"UTF-8 sig"的编码保存**
这里推荐notepad++，sublime Text，VS Code作为文本编辑器![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721295163511-165d6395-23de-4875-8168-3e7a459d7f21.png#averageHue=%23e3e4b4&clientId=ub85352ab-5b31-4&from=paste&height=299&id=ua21c196c&originHeight=374&originWidth=1411&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=117021&status=done&style=none&taskId=uc758cb97-7bc4-4abe-bcda-326cbd3f7cc&title=&width=1128.8)
item ID主要对应的是导出表格的这部分内容
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721295377340-8867516b-52c7-44f7-b5e5-787abe6d62eb.png#averageHue=%23f4f3f2&clientId=ub85352ab-5b31-4&from=paste&height=599&id=u3632390f&originHeight=749&originWidth=1328&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=105337&status=done&style=none&taskId=u942f2a7a-6b33-4b34-b630-46b6c36eca3&title=&width=1062.4)
打开导出的CSV文件，Item ID列的内容就是INI文件中的对应Item ID内容
一个group最大添加的item id数量与item id的字节数量有关，最大65535字节
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721295402922-7594d9ed-044d-4f61-815f-a26c29eda34c.png#averageHue=%23f3f1f0&clientId=ub85352ab-5b31-4&from=paste&height=371&id=uf0ea39bd&originHeight=464&originWidth=876&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=47259&status=done&style=none&taskId=ua980d05f-4d8a-43bc-850f-7d86f4c5e3c&title=&width=700.8)
#### ③保存好ini配置文件后，点击【重启中间件】
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721295503399-797afcfb-e76c-4d52-ad0f-a256aaa0a186.png#averageHue=%23d1d0d0&clientId=ub85352ab-5b31-4&from=paste&height=638&id=u4858ca77&originHeight=798&originWidth=1326&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=45527&status=done&style=none&taskId=ua5c2f7bf-a90d-4f3b-9e36-7449c219626&title=&width=1060.8)
#### ④重启后，可在MQTT FX订阅软件中的【发布主题】
可以观察到初始化的信息上报，表示MQTT连接成功
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721294509813-210d12f4-f56c-4ea5-9ffa-24f7c3fe8f25.png#averageHue=%23cecdcd&clientId=ub85352ab-5b31-4&from=paste&height=478&id=u956f32e2&originHeight=597&originWidth=1265&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=48834&status=done&style=none&taskId=ub7ca1ba0-5e20-4ff6-a99a-2e59560be34&title=&width=1012)
之后每隔3秒，会上传OPC UA 服务器的内容上传
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721294598129-9c57e3ff-93d4-4e0f-8cb5-5a1f013ec549.png#averageHue=%23a4a3a3&clientId=ub85352ab-5b31-4&from=paste&height=690&id=ufff6df3c&originHeight=862&originWidth=1262&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=65573&status=done&style=none&taskId=u4c49d799-a26b-4206-ad7f-c0945f8a9a3&title=&width=1009.6)
#### ⑤连接成功后，可在【OPC服务器】实时预览数据
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721294637306-c3cbe96c-0aa4-490d-b1ba-7c8218e41c42.png#averageHue=%23dcdbda&clientId=ub85352ab-5b31-4&from=paste&height=638&id=u92370db2&originHeight=798&originWidth=1326&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=67226&status=done&style=none&taskId=u1f66fe51-f1a2-468a-9e54-946cf52e9d5&title=&width=1060.8)
#### ⑥使用软件配置OPC点位
除了更改配置文件达到OPC点位的配置外，程序内也可以配置点位
切换到设备列表——点击添加设备
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721631616358-abaaea36-6370-45a5-b129-3338aba58887.png#averageHue=%23d1d0d0&clientId=u68e6f105-ff16-4&from=paste&height=638&id=ue368710d&originHeight=797&originWidth=1331&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=56662&status=done&style=none&taskId=ufde82479-fd55-4ced-b1c6-b10e54ddbb4&title=&width=1064.8)
表格将会多出一行，接下来填写样例信息，我选择在KepServer中配置了一个S7-1200的PLC模拟数据
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721631668462-0396857b-ee02-4edf-a5c2-8f92d1f5f9d6.png#averageHue=%23d3d3d2&clientId=u68e6f105-ff16-4&from=paste&height=638&id=u98eaefca&originHeight=797&originWidth=1331&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=42590&status=done&style=none&taskId=u1721f7f6-cf78-4e54-bfd5-b29a2fa78ff&title=&width=1064.8)
S7-1200的PLC模拟数据如下
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721631757488-963a157f-05a2-4036-b39f-f2c1c1fabdbe.png#averageHue=%23f6f5f5&clientId=u68e6f105-ff16-4&from=paste&height=107&id=ua413812f&originHeight=134&originWidth=743&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=10430&status=done&style=none&taskId=ub91f6a38-01b4-41ee-9596-9b812081e6d&title=&width=594.4)

配置该点位至程序中，进行采集，其中设备IP指的是OPC Server的IP
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721631989528-7c08c0ab-ba98-4032-a4b6-f2695fb82b83.png#averageHue=%23d1d1d1&clientId=u68e6f105-ff16-4&from=paste&height=638&id=u6df2c29d&originHeight=797&originWidth=1331&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=241002&status=done&style=none&taskId=u3227e662-0ed1-45de-9a49-588f9119bfd&title=&width=1064.8)
保存编辑——重启中间件
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25484337/1721632061948-08919ba5-0369-4eb7-a34e-b2a8997d257c.png#averageHue=%23d4d4d4&clientId=u68e6f105-ff16-4&from=paste&height=638&id=u637211a1&originHeight=797&originWidth=1331&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=58264&status=done&style=none&taskId=uf0ff4687-9764-4d18-a36d-7c3ff0eb47e&title=&width=1064.8)
重新打开软件后

