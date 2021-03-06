# 从RDS for MySQL同步至POLARDB for MySQL {#task_1580301 .task}

本小节介绍如何使用数据传输服务DTS快速创建RDS for MySQL实例同POLARDB for MySQL集群间的实时同步作业，实现RDS for MySQL到POLARDB for MySQL增量数据的实时同步。

已购买目标POLARDB for MySQL实例，详情请参见[创建POLARDB for MySQL实例](https://help.aliyun.com/document_detail/58769.html)。

## 注意事项 {#section_4oa_n0g_5o3 .section}

-   全量初始化过程中，并发INSERT会导致目标实例的表碎片，全量初始化完成后，目标实例的表空间比源集群的表空间大。
-   如果数据同步的源实例没有主键或唯一约束，且记录的全字段没有唯一性，可能会出现重复数据。

## 支持同步的SQL操作 {#section_szw_igd_cwe .section}

-   INSERT、UPDATE、DELETE、REPLACE
-   ALTER TABLE、ALTER VIEW、ALTER FUNCTION、ALTER PROCEDURE
-   CREATE DATABASE、CREATE SCHEMA、CREATE INDEX、CREATE TABLE、CREATE PROCEDURE、CREATE FUNCTION、CREATE TRIGGER、CREATE VIEW、CREATE EVENT
-   DROP FUNCTION、DROP EVENT、DROP INDEX、DROP PROCEDURE、DROP TABLE、DROP TRIGGER、DROP VIEW
-   RENAME TABLE、TRUNCATE TABLE

## 支持的同步架构 {#section_ifq_lu9_x3z .section}

-   一对一单向同步
-   一对多单向同步
-   级联单向同步
-   多对一单向同步
-   一对一双向同步

    **说明：** 关于双向同步的配置方法，请参见[RDS for MySQL实例间的双向同步](../../cn.zh-CN/用户指南/实时同步/MySQL实时同步至MySQL/RDS for MySQL实例间的双向同步.md#)。


关于各类同步架构的介绍及注意事项，请参见[数据同步拓扑介绍](../../cn.zh-CN/用户指南/实时同步/数据同步拓扑介绍.md#)。

## 功能限制 {#section_o2s_tr4_31b .section}

-   不兼容触发器

    同步对象为整个库且这个库中包含了会更新同步表内容的触发器，那么可能导致同步数据不一致。例如数据库中存在了两个表A和B。表A上有一个触发器，触发器内容为在INSERT一条数据到表A之后，在表B中插入一条数据。这种情况在同步过程中，如果源实例表A上进行了INSERT操作，则会导致表B在源实例跟目标实例数据不一致。

    此类情况须要将目标实例中的对应触发器删除掉，表B的数据由源实例同步过去，详情请参见[触发器存在情况下如何配置同步作业](https://help.aliyun.com/document_detail/26655.html) 。

-   RENAME TABLE限制

    RENAME TABLE操作可能导致同步数据不一致。例如同步对象只包含表A，如果同步过程中源实例将表A重命名为表B，那么表B将不会被同步到目标库。为避免该问题，您可以在数据同步配置时，选择同步表A和表B所在的整个数据库作为同步对象。


## 操作步骤 {#d7e56 .section}

1.  [购买数据同步作业](../../cn.zh-CN/快速入门/购买流程.md#section_39h_fto_gdl)。 

    **说明：** 购买时，选择源实例和目标实例均为**MySQL**，并选择同步拓扑为**单向同步**。

2.  登录[数据传输控制台](https://dts.console.aliyun.com/)。
3.  在左侧导航栏，单击**数据同步**。
4.  在同步作业列表页面顶部，选择数据同步实例所属地域。 

    ![选择地域](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/776198/156585059950604_zh-CN.png)

5.  定位至已购买的数据同步实例，单击**配置同步链路**。
6.  配置同步通道的源实例及目标实例信息。 

    ![配置源和目标实例信息](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/89983/156585059955043_zh-CN.png)

    |配置项目|配置选项|配置说明|
    |:---|:---|:---|
    |任务名称|-|     -   DTS为每个任务自动生成一个任务名称，任务名称没有唯一性要求。
    -   您可以根据需要修改任务名称，建议为任务配置具有业务意义的名称，便于后续的任务识别。
 |
    |源实例信息|实例类型|选择**RDS实例**。|
    |实例地区|购买数据同步实例时选择的源实例地域信息，不可变更。|
    |数据库账号|填入连接RDS实例的数据库账号。 **说明：** 

    -   该账号需具备待同步对象的SELECT、REPLICATION CLIENT、REPLICATION SLAVE权限。
    -   当源RDS实例的数据库类型为**MySQL 5.5**或**MySQL 5.6**时，无需配置**数据库账号**和**数据库密码**。
 |
    |数据库密码|填入数据库账号对应的密码。|
    |连接方式|根据需求选择**非加密连接**或**SSL安全连接**，本案例选择为**非加密连接**。 **说明：** 选择**SSL安全连接**时，需要提前开启RDS实例的SSL加密功能，详情请参见[设置SSL加密](https://help.aliyun.com/document_detail/96120.html)。

 |
    |目标实例信息|实例类型|选择为**通过专线/VPN网关/智能网关接入的自建数据库**。|
    |实例地区|购买数据同步实例时选择的目标实例地域信息，不可变更。|
    |对端专有网络|选择POLARDB实例所属的专有网络。 您可以登录[POLARDB控制台](https://polardb.console.aliyun.com/)，单击目标实例ID，进入该实例的**基本信息**页面来获取。

 ![获取VPC Id](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1227086/156585059954382_zh-CN.png)

|
    |数据库类型|选择为**MySQL**。|
    |IP地址|配置POLARDB主实例的私网IP地址，本案例中填入**172.16.20.20**。 您可以在电脑中ping目标POLARDB实例的**主地址（私网）**来获取私网IP地址。

 ![获取POLARDB的私网IP地址](../../SP_201/DNdts1898536/images/54390_zh-CN.gif)

|
    |端口|填入POLARDB实例的服务端口，默认为**3306**。|
    |数据库账号|填入连接POLARDB实例的数据库账号。 **说明：** 用于数据同步的数据库账号需具备目标同步对象的ALL权限。

 |
    |数据库密码|填入数据库账号对应的密码。|

7.  单击页面右下角的**授权白名单并进入下一步**。 

    **说明：** 此步骤会将DTS服务器的IP地址自动添加到源RDS实例和目标POLARDB实例的白名单中，用于保障DTS服务器能够正常连接源和目标实例。

8.  配置目标已存在表的处理模式和同步对象。 

    ![配置处理模式和同步对象](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/89979/156585059954325_zh-CN.png)

    |配置项目|配置说明|
    |:---|:---|
    |目标已存在表的处理模式|     -   **预检查并报错拦截**：检查目标数据库中是否有同名的表。如果目标数据库中没有同名的表，则通过该检查项目；如果目标数据库中有同名的表，则在预检查阶段提示错误，数据同步作业不会被启动。

**说明：** 如果目标库中同名的表不方便删除或重命名，您可以[设置同步对象在目标实例中的名称](../../cn.zh-CN/用户指南/实时同步/设置同步对象在目标实例中的名称.md#)来避免表名冲突。

    -   **无操作**：跳过目标数据库中是否有同名表的检查项。

**警告：** 选择为**无操作**，可能导致数据不一致，给业务带来风险，例如：

        -   表结构一致的情况下，如果在目标库遇到与源库主键的值相同的记录，在初始化阶段会保留目标库中的该条记录；在增量同步阶段则会覆盖目标库的该条记录。
        -   表结构不一致的情况下，可能会导致无法初始化数据、只能同步部分列的数据或同步失败。
 |
    |选择同步对象| 在源库对象框中单击待同步的对象，然后单击![向右小箭头](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79929/156585060040698_zh-CN.png)将其移动至已选择对象框。

 同步对象的选择粒度为库、表。

 **说明：** 

    -   如果选择整个库作为同步对象，那么该库中所有对象的结构变更操作都会同步至目标库。
    -   默认情况下，同步对象的名称保持不变。如果您需要同步对象在目标实例上名称不同，那么需要使用DTS提供的对象名映射功能，详情请参见[设置同步对象在目标实例中的名称](../../cn.zh-CN/用户指南/实时同步/设置同步对象在目标实例中的名称.md#)。
 |

9.  上述配置完成后，单击页面右下角的**下一步**。
10. 配置同步初始化的高级配置信息。 

    ![数据同步高级设置](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/17125/156585060041055_zh-CN.png)

    **说明：** 同步初始化类型细分为：结构初始化，全量数据初始化。选择**结构初始化**和**全量数据初始化**后，DTS会在增量数据同步之前，将源数据库中待同步对象的结构和存量数据，同步到目标数据库。

11. 上述配置完成后，单击页面右下角的**预检查并启动**。 

    **说明：** 

    -   在数据同步作业正式启动之前，会先进行预检查。只有预检查通过后，才能成功启动数据同步作业。
    -   如果预检查失败，单击具体检查项后的![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/17095/156585060047468_zh-CN.png)，查看失败详情。根据提示修复后，重新进行预检查。
12. 在预检查对话框中显示**预检查通过**后，关闭预检查对话框，同步作业将正式开始。
13. 等待同步作业的链路初始化完成，直至处于**同步中**状态。 

    您可以在 数据同步页面，查看数据同步作业的状态。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/17125/156585060041059_zh-CN.png)


