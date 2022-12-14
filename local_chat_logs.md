# Js实现db接口简要说明(待更新)
## 表结构以及需要实现的接口说明
>所有的db接口返回值，统一由errCode,errMsg,data字段转换为字符串异步返回
### 消息表(写扩散消息表)
- 表名：local_chat_logs
```sqlite
CREATE TABLE `local_chat_logs` (`client_msg_id` char(64),`server_msg_id` char(64),`send_id` char(64),`recv_id` char(64),`sender_platform_id` integer,`sender_nick_name` varchar(255),`sender_face_url` varchar(255),`session_type` integer,`msg_from` integer,`content_type` integer,`content` varchar(1000),`is_read` numeric,`status` integer,`seq` integer DEFAULT 0,`send_time` integer,`create_time` integer,`attached_info` varchar(1024),`ex` varchar(1024),PRIMARY KEY (`client_msg_id`))
```

- 表结构特别说明

  | 字段名     | 类型                                                         | 说明 |
    | --------- | ------------------------------------------------------------ | ----- |
  | is_read      | bool                                         | true已读，false 未读|



#### 接口说明：
- getMessage

| 输入参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| clientMsgID      | string                                          | 消息ID ||

| 返回参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| errCode      | number                                         | 自定义即可，0成功，非0失败 |如果获取不到消息也需要返回错误|
| errMsg     | string                                          | 详细的err信息 ||
| data      | string                                          | LocalChatLog（消息表对象数据） |对象转换成string|
**参考sql语句说明：**

```sql
SELECT * FROM `local_chat_logs` WHERE client_msg_id = "063031b86f8e503c6038efb2b835f216" LIMIT 1

```
- getMessageList

| 输入参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| sourceID                                     | string  | 关于某人的ID也可能是写扩散模式下群ID|
| sessionType | number                                     | 会话类型，单聊1、读扩散群2、大群为3      ||
| count | number | 获取消息的数量 ||
| startTime | number | 消息发送时间，毫秒 ||
| isReverse | boolean | 消息为正向拉取还是反向拉取|默认情况为false，即为正向拉取（从新消息到老消息），order by 后面的排序规则为send_time DESC 降序排列，send_time为 <;当为true的情况，即为反向拉取，order by 后面的排序规则为send_time ASC 升序排列,send_time为 >|

| 返回参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| errCode      | number                                         | 自定义即可，0成功，非0失败 |获取不到的时候返回空数组不需要返回错误|
| errMsg     | string                                          | 详细的err信息 ||
| data      | string                                          | []LocalChatLog（消息表对象数组数据） ||

**参考sql语句说明：**

```sql
-- 1、sessionType == 1 && sourceID == d.loginUserID
SELECT * FROM `local_chat_logs` WHERE send_id = "812146266" And  recv_id = "812146266" AND status <=1 And session_type = 3 And send_time < 1664357584025 ORDER BY send_time DESC LIMIT 30;
-- 注：其中status固定为3
-- 2、其他场景
SELECT * FROM `local_chat_logs` WHERE send_id = "812146266" OR  recv_id = "812146266" AND status <=1 And session_type = 3 And send_time < 1664357584025 ORDER BY send_time DESC LIMIT 30;
```
- getMessageListNoTime

| 输入参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| sourceID                                     | string  | 关于某人的ID也可能是写扩散模式下群ID|
| sessionType | number                                     | 会话类型，单聊1、读扩散群2、大群为3      ||
| count | number | 获取消息的数量 ||
| isReverse | boolean | 消息为正向拉取还是反向拉取|默认情况为false，即为正向拉取（从新消息到老消息），order by 后面的排序规则为send_time DESC 降序排列，当为true的情况，即为反向拉取，order by 后面的排序规则为send_time ASC 升序排列|

| 返回参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| errCode      | number                                         | 自定义即可，0成功，非0失败 |获取不到的时候返回空数组不需要返回错误|
| errMsg     | string                                          | 详细的err信息 ||
| data      | string                                          | []LocalChatLog（消息表对象数组数据） ||

**参考sql语句说明：**

```sql
-- 1、sessionType == 1 && sourceID == d.loginUserID
SELECT * FROM `local_chat_logs` WHERE send_id = "812146266" And  recv_id = "812146266" AND status <=3 And session_type = 1  ORDER BY send_time DESC LIMIT 30;
-- 注：其中status固定为3
-- 2、其他场景
SELECT * FROM `local_chat_logs` WHERE send_id = "812146266" OR  recv_id = "812146266" AND status <=3 And session_type = 1  ORDER BY send_time DESC LIMIT 30;
```

[comment]: <> "- setChatLogFailedStatus"
- getSendingMessageList

**无输入参数**

| 返回参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| errCode      | number                                         | 自定义即可，0成功，非0失败 |获取不到的时候返回空数组不需要返回错误|
| errMsg     | string                                          | 详细的err信息 ||
| data      | string                                          |  []LocalChatLog（消息表对象数组数据） |对象数组转换成string|

**参考sql语句说明：**

```sql
select * from local_chat_logs where status = 1;
-- 消息状态：
-- 	MsgStatusSending     = 1
-- 	MsgStatusSendSuccess = 2
-- 	MsgStatusSendFailed  = 3
-- 	MsgStatusHasDeleted  = 4
-- 	MsgStatusRevoked     = 5
-- 	MsgStatusFiltered    = 6
```

- getNormalMsgSeq


**无输入参数**

| 返回参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| errCode      | number                                         | 自定义即可，0成功，非0失败 ||
| errMsg     | string                                          | 详细的err信息 ||
| data      | number                                    | 消息表中最大seq ||

**参考sql语句说明：**

```sql
SELECT IFNULL(max(seq),0) FROM `local_chat_logs`;
```


- updateMessageTimeAndStatus

| 输入参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| clientMsgID | string                                     | 客户端消息ID       ||
| serverMsgID | string | 服务器消息ID ||
| sendTime | number | 消息发送时间，毫秒 ||
| status | number | 消息状态，参照前面 ||

| 返回参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| errCode      | number                                         | 自定义即可，0成功，非0失败 |如果没更新到任何一行消息也返回错误|
| errMsg     | string                                          | 详细的err信息 ||
| data      | string                                          | 可为"" ||

**参考sql语句说明：**

```sql
 UPDATE `local_chat_logs` SET `server_msg_id`="75dee2fbd6c4f28e7895f8410be4984f",`status`=2,`send_time`=1663658950513 WHERE client_msg_id="985261c57242cf647753839854038154" And seq=0;
```

BatchUpdateMessageList

- updateMessage

| 输入参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| clientMsgID | string                                     | 客户端消息ID       ||
| args        | object | 更新字段参数对象 |内部是kv，k为字段名，v为需要更新的字段内容Js实现db接口简要说明|

| 返回参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| errCode      | number                                         | 自定义即可，0成功，非0失败 |如果没更新到任何一行消息也返回错误|
| errMsg     | string                                          | 详细的err信息 ||
| data      | string                                          | 可为"" ||

**参考sql语句说明：**

```sql
 UPDATE `local_chat_logs` SET `server_msg_id`="8c6dc2ace8ff5706880018de43916c39",`recv_id`="2041671273",`session_type`=1,`status`=2,`seq`=14 WHERE `client_msg_id` = "6edad80249cc0cf626edb88e64f8fb6d";
```

- batchInsertMessageList

| 输入参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| messageList                                     | string  | []LocalChatLog（消息表对象数组数据）|对象数组转换成string

| 返回参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| errCode      | number                                         | 自定义即可，0成功，非0失败 ||
| errMsg     | string                                          | 详细的err信息 ||
| data      | string                                          | 可为"" ||

**参考sql语句说明：**

```sql
 INSERT INTO `local_chat_logs` (`client_msg_id`,`server_msg_id`,`send_id`,`recv_id`,`sender_platform_id`,`sender_nick_name`,`sender_face_url`,`session_type`,`msg_from`,`content_type`,`content`,`is_read`,`status`,`seq`,`send_time`,`create_time`,`attached_info`,`ex`) VALUES ("01d22adafe309482391fc54f728c96a7","5cb3ecb6092843e9376fd229a72b24e0","3045326383","openIMAdmin",0,"","",1,200,1303,"{\"detail\":\"CgozMDQ1MzI2Mzgz\",\"defaultTips\":\"remove a blocked user\",\"jsonDetail\":\"{\\"userID\\":\\"3045326383\\"}\"}",false,6,1,1663557189653,1663557189652,"",""),("3492347bc55280d3c5c32398f78eae50","2b0a01eff07f502da31751fb459966bc","3045326383","openIMAdmin",0,"","",1,200,1303,"{\"detail\":\"CgozMDQ1MzI2Mzgz\",\"defaultTips\":\"remove a blocked user\",\"jsonDetail\":\"{\\"userID\\":\\"3045326383\\"}\"}",false,6,2,1663557324330,1663557324329,"","");
```

- insertMessage

| 输入参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| message                                     | string  | LocalChatLog（消息表对象）|对象转换成string

| 返回参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| errCode      | number                                         | 自定义即可，0成功，非0失败 ||
| errMsg     | string                                          | 详细的err信息 ||
| data      | string                                          | 可为"" ||

**参考sql语句说明：**

```sql
 INSERT INTO `local_chat_logs` (`client_msg_id`,`server_msg_id`,`send_id`,`recv_id`,`sender_platform_id`,`sender_nick_name`,`sender_face_url`,`session_type`,`msg_from`,`content_type`,`content`,`is_read`,`status`,`seq`,`send_time`,`create_time`,`attached_info`,`ex`) VALUES ("6edad80249cc0cf626edb88e64f8fb6d","","3045326383","2041671273",1,"Gordon111","ic_avatar_01",1,100,101,"Single chat test3045326383:2041671273:",false,1,0,1663658716992,1663658716992,"","");
```


- getMultipleMessage

| 输入参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| msgIDList                                     | string  | 消息ID列表|对象转换成string

| 返回参数     | 类型                                                         | 说明 |备注|
| --------- | ------------------------------------------------------------ | ----- |-----------------------|
| errCode      | number                                         | 自定义即可，0成功，非0失败 |获取不到的时候返回空数组不需要返回错误|
| errMsg     | string                                          | 详细的err信息 ||
| data      | string                                          | 可为"" ||

**参考sql语句说明：**

```sql
SELECT * FROM `local_sg_chat_logs_4280368097` WHERE client_msg_id IN ("d9ef1e4e63394b856b8a781ef0234b49","00da0b5471f0bca5a41c92e09419f9dc") ORDER BY send_time DESC;
```


