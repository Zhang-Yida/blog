## SAP DataService 创建 JDBC Adapter 及在 Designer 中的应用



### Adapter 创建

> 浏览器访问 Sap DataServices Management Console，通常访问网址为 http://192.168.0.178:8080/DataServices/admin.jsp

#### 找到添加 Adapter 的入口

<img src="https://raw.githubusercontent.com/Zhang-Yida/PictureBed/main/imgs/image-20201204082755915.png" alt="image-20201204082755915" style="zoom:40%;" />

![image-20201204083023765](https://raw.githubusercontent.com/Zhang-Yida/PictureBed/main/imgs/image-20201204083023765.png)

![image-20201204083139616](https://raw.githubusercontent.com/Zhang-Yida/PictureBed/main/imgs/image-20201204083139616.png)

#### 配置 JDBCAdapter 内容

> 需上传对应的 JDBC 驱动 JAR 包到服务器，添加 JAR 包路径到 Classpath 中

![image-20201204083955111](https://raw.githubusercontent.com/Zhang-Yida/PictureBed/main/imgs/image-20201204083955111.png)

> 填写驱动相关配置，由于本例子中使用的是 Hive 的 JDBC Driver，配置如下：

![image-20201204084139577](https://raw.githubusercontent.com/Zhang-Yida/PictureBed/main/imgs/image-20201204084139577.png)

#### 提交配置

> 点击 Apply 提交新建的 Adapter

#### 启动 JDBC 服务

> 返回到 Adapter 列表，勾选新建的 JDBCAdapter，点击 Start ，即可启动该服务，Status 为 Started 即为启动成功，如果失败，请查看对应的 Log files 中的日志

![image-20201204084609626](https://raw.githubusercontent.com/Zhang-Yida/PictureBed/main/imgs/image-20201204084609626.png)

### Designer 应用 Adapter 连接数据源

![image-20201204090319468](https://raw.githubusercontent.com/Zhang-Yida/PictureBed/main/imgs/image-20201204090319468.png)