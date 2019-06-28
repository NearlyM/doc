# OBS迁移方案

Danale云录像在旧的方案中录像都是存在阿里云oss上面，该方案使用的是oss提供的sdk去签名并下载文件，这种方案局限性比较大，不利于扩展。

随着业务需求的扩展，存储不仅限于oss，目前扩展至obs，后期可能扩展更多的云平台，所以下载方案要选择一个通用的方式。

目前选择原生的下载，即使用http去下载文件，该方案不再局限于平台限制，移动端先向Danale平台获取签名文件的下载url，再使用http去下载该文件。

新方案涉及到具体的业务有两个模块：**云录像和告警录像**。

下面简单的介绍下平台提供的接口，分别是针对上面两个模块。

1. 获取云录像签名文件 

   [GetCloudSecurityTokens]: https://gitlab.dana-tech.com/backend/ictun-api-doc/ictun-app-api-doc/blob/develop/api/api_v5/cloud-service-api/GetCloudSecurityTokens_demo.md

   打开可以查看该接口的使用示例，主要有三种使用方式

2. 获取告警录像签名文件

   [GetMsgSecurityTokens]: https://gitlab.dana-tech.com/backend/ictun-api-doc/ictun-app-api-doc/blob/develop/api/api_v5/cloud-service-api/API.md

   `GetMsgSecurityTokens`接口返回告警录像的缩略图签名文件和录像签名文件。具体调用可以打开连接查看。

该方案在华为配件项目中先行实施，具体的实施细节下面将列出。涉及到的类：`CloudRecordDownloadHelper`，`CloudRecordPlayback`中的`playVideoRawByCloudRecordPlayInfo`方法

1. CloudRecordDownloadHelper

   该文件可以整体替换。主要封装的是云录像和告警录像的下载逻辑，其中注意，

   **告警录像：**

   > 告警录像根据开始时间和期望的播放时长向平台查询录像列表。将列表放入阻塞队列，下载的时候去队列中获取下载地址。该方案允许有文件丢失，丢失的时候会重新去解析文件索引。

   **云录像：**

   > 云录像是根据开始时间去向平台获取第一个下载文件（可优化，一次获取多个），下载结束后根据文件名+1获取下一个文件，继续向平台查询文件下载地址，直至平台接口返回错误停止。

2. CloudRecordPlayback

   该文件中的播放方法`playVideoRawByCloudRecordPlayInfo`，其中有从`CloudRecordPlayInfo`对象的`id`中去解析`DeviceId`，所以构建`CloudRecordPlayInfo`对象的时候要将`DeviceId`添加到id字段上。

3. CloudSDModel

   `getCloudRecordPlayInfo`方法，

   对应2中的id:

   ```java
   String cloud_id = deviceId + "_"+ cloudRecordPlayInfo.getStartTime();
   ```

可以参考华为配件的代码，如有其它疑问，可与我沟通。

作者 ningerlei@danale.com
2019 年 04月 01日 