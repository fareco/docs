# 排行榜 REST API 使用指南

阅读此文档前请确保已经阅读了[排行榜总览](leaderboard.html)，了解排行榜的核心概念及功能。

## 请求
### 请求格式
对于 POST 和 PUT 请求，请求的主体必须是 JSON 格式，而且 HTTP Header 的 Content-Type 需要设置为 `application/json`。

请求的鉴权是通过 HTTP Header 里面包含的键值对来进行的，参数如下表：

Key|Value|含义|来源
---|----|---|---
`X-LC-Id`|{{appid}}|当前应用的 `App Id`|可在控制台->设置页面查看
`X-LC-Key`| {{appkey}}|当前应用的 `App Key`|可在控制台->设置页面查看

部分管理性质的接口需要使用 `Master Key`。

### Base URL

文档中 Base URL 为绑定的 [API 自定义域名](custom-api-domain-guide.html)，
可以在「控制台 > 设置 > 应用凭证 > 服务器地址」查看。

如果暂时没有绑定域名，华北、华东节点可以使用临时的测试域名，具体域名见「控制台 > 设置 > 应用凭证 > 服务器地址」。
该域名仅供测试和原型开发阶段使用，不保证可用性，请在正式发布前为应用绑定 API 自定义域名。

LeanCloud 国际版不要求绑定自定义域名。除了使用自定义域名外，也可以使用如下共享域名：

```
appid前八位.api.lncldglobal.com
```

## 管理接口

如无特别说明，以下接口均属于管理接口，需使用 `Master Key`。
### 管理排行榜

#### 创建排行榜

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -H "Content-Type: application/json" \
  -d '{"statisticName": "world", "memberType": "_User", "order": "descending", "updateStrategy": "better", "versionChangeInterval": "month"}' \
  https://{{host}}/1.1/leaderboard/leaderboards
```

| 参数        | 约束   | 说明                                   |
| --------- | ---- | ---------------------------------------- |
| `statisticName`     | 必须   | 排行榜的名称，创建后不可修改。 |
| `memberType`      | 必须   | 排行榜的成员类型，创建后不可修改。可填写 `_Entity`、`_User` 及其他已有的 class 名称。|
| `order` | 可选   | 排行榜的排序策略，创建后不可修改。可选项有 `ascending` 或 `descending`，默认为 `descending`。 |
| `updateStrategy` | 可选   |  可选项有 `better`、`last`、`sum`，默认为 `better`。|
| `versionChangeInterval` | 可选   |  可选项有 `day`、`week`、`month`、`never`，默认为 `week`。 |

返回的主体是一个 JSON 对象，包含创建排行榜时传入的所有参数，同时包括以下信息：

* `version` 为排行榜当前版本号。
* `expiredAt` 为下次过期（重置）时间。
* `activatedAt` 当前版本的开始时间。

```json
{
  "objectId": "5b62c15a9f54540062427acc",
  "statisticName": "world",
  "memberType": "_User",
  "versionChangeInterval": "month",
  "order": "descending",
  "updateStrategy": "better",
  "version": 0,
  "createdAt": "2018-08-02T08:31:22.294Z",
  "updatedAt": "2018-08-02T08:31:22.294Z",
  "expiredAt": {
    "__type": "Date",
    "iso": "2018-08-31T16:00:00.000Z"
  },
  "activatedAt": {
    "__type": "Date",
    "iso": "2018-08-02T08:31:22.290Z"
  }
}
```

#### 获取排行榜属性

通过这个接口来查看当前排行榜的属性，例如更新策略、当前版本号等。

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  https://{{host}}/1.1/leaderboard/leaderboards/<statisticName>
```

返回的 JSON 对象包含该排行榜的所有信息：

```json
{
  "objectId": "5b0b97cf06f4fd0abc0abe35",
  "statisticName": "world",
  "memberType": "_User",
  "order": "descending",
  "updateStrategy": "better",
  "version": 5,
  "versionChangeInterval": "day",
  "expiredAt": {"__type": "Date", "iso": "2018-05-02T16:00:00.000Z"},
  "activatedAt": {"__type": "Date", "iso": "2018-05-01T16:00:00.000Z"},
  "createdAt": "2018-04-28T05:46:58.579Z",
  "updatedAt": "2018-05-01T01:00:00.000Z"
}
```

#### 修改排行榜属性

这个接口可以用来修改排行榜的 `updateStrategy` 和 `versionChangeInterval` 属性，其他属性不可修改。可以只更新某一个属性，例如只修改 versionChangeInterval：

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -H "Content-Type: application/json" \
  -d '{"versionChangeInterval": "day"}' \
  https://{{host}}/1.1/leaderboard/leaderboards/<statisticName>
```
返回的 JSON 对象包含更新的属性信息及 `updatedAt` 字段。

```json
{
  "objectId": "5b0b97cf06f4fd0abc0abe35",
  "versionChangeInterval": "day",
  "updatedAt": "2018-05-01T08:01:00.000Z"
}
```

#### 重置排行榜

无论排行榜的重置策略是什么，你都可以通过这个方法重置排行榜。重置时当前版本的数据清空，同时会归档到 csv 文件以供下载，排行榜的 `version` 会自动加一。

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  https://{{host}}/1.1/leaderboard/leaderboards/<statisticName>/incrementVersion
```

返回的 JSON 会显示重置后的当前版本号，下一次过期时间 `expiredAt`，当前版本的开始时间 `activatedAt`：

```json
{
  "objectId": "5b0b97cf06f4fd0abc0abe35",
  "version": 7,
  "expiredAt": {"__type": "Date", "iso": "2018-06-03T16:00:00.000Z"},
  "activatedAt": {"__type": "Date", "iso": "2018-05-28T06:02:56.169Z"},
  "updatedAt": "2018-05-28T06:02:56.185Z"
}
```

#### 获取历史数据归档文件

因为每个排行榜最多保存 60 个归档文件，我们建议定期使用这个接口获取归档文件后另行备份保存。

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -G \
  --data-urlencode 'limit=10' \
  https://{{host}}/1.1/leaderboard/leaderboards/<statisticName>/archives
```

返回的对象会按照 `createdAt` 降序排列。其中 `file_key` 是文件的名称，`url` 是文件的下载地址，`status` 包含以下状态：

* `scheduled`：进入归档任务队列，还未归档，这个状态通常极短。
* `inProgress`：正在归档中。
* `failed`：归档失败，出现这种情况请联系技术支持。
* `completed`：归档已完成。

```json
{
  "results": [
    {
      "objectId": "5b0b9da506f4fd0abc0abe6e",
      "statisticName": "wins",
      "version": 9,
      "status": "completed",
      "url": "https://lc-paas-files.cn-n1.lcfile.com/yK5s6YJztAwEYiWs.csv",
      "file_key": "yK5s6YJztAwEYiWs.csv",
      "activatedAt": {"__type": "Date", "iso": "2018-05-28T06:11:49.572Z"},
      "deactivatedAt": {"__type": "Date", "iso": "2018-05-30T06:11:49.951Z"},
      "createdAt": "2018-05-01T16:00.00.000Z",
      "updatedAt": "2018-05-28T06:11:50.129Z",
    }
  ]
}
```

#### 删除排行榜

**这个接口将删除排行榜的所有数据**，包括当前版本的数据及所有归档文件，删除后无法恢复，请谨慎操作。

删除时只需要指定排行榜的名称 statisticName。

```sh
curl -X DELETE \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  https://{{host}}/1.1/leaderboard/leaderboards/<statisticName>
```

删除成功时返回空 JSON 对象：

```json
{}
```

### 管理成绩

#### 更新成绩

使用 Master Key 可以更新任意成绩，但更新成绩时仍然遵循排行榜的 `updateStrategy` 属性。

更新用户成绩时需指定相应用户的 `objectId`：

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -H "Content-Type: application/json" \
  -d '[{"statisticName": "wins", "statisticValue": 5}, {"statisticName": "world", "statisticValue": 91}]' \
  https://{{host}}/1.1/leaderboard/users/<objectId>/statistics
```

返回的数据是服务端当前使用的分数：

```sh
{
  "results": [
    {
      "statisticName": "wins",
      "version": 0,
      "statisticValue": 5
    },
    {
      "statisticName": "world",
      "version": 2,
      "statisticValue": 91
    }
  ]
}
```

类似地，更新 object 成绩时需指定相应 object 的 `objectId`：

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -H "Content-Type: application/json" \
  -d '[{"statisticName": "wins", "statisticValue": 5}, {"statisticName": "weapons","statisticValue": 91}]' \
  https://{{host}}/1.1/leaderboard/objects/<objectId>/statistics
```

更新 entity 成绩时则需指定相应的字符串：

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -H "Content-Type: application/json" \
  -d '[{"statisticName": "wins", "statisticValue": 5}, {"statisticName": "cities","statisticValue": 91}]' \
  https://{{host}}/1.1/leaderboard/entities/<entityString>/statistics
```

当前用户可以更新自己的成绩，这个不属于管理接口，不需要 `Master Key`，但需要传入当前用户的 `sessionToken`（客户端 SDK 更新当前用户的成绩封装了这一接口）：

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "X-LC-Session: <sessionToken>" \
  -H "Content-Type: application/json" \
  -d '[{"statisticName": "wins", "statisticValue": 5}, {"statisticName": "world", "statisticValue": 91}]' \
  https://{{host}}/1.1/leaderboard/users/self/statistics
```


#### 强制更新成绩

附加 `overwrite=1` 会无视更新策略 better 及 sum，强制使用 last 策略更新用户的成绩。
比如，发现某个用户存在作弊行为时，可以使用这个接口强制更新用户的成绩。

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -H "Content-Type: application/json" \
  -d '[{"statisticName": "wins", "statisticValue": 10}]' \
  https://{{host}}/1.1/leaderboard/users/<uid>/statistics?overwrite=1
```

返回的数据是当前服务端使用的分数：

```json
{"results":[{"statisticName":"wins","version":0,"statisticValue":10}]}
```

类似地，附加 `overwrite=1` 可以强制更新 object 成绩和 entity 成绩。

#### 删除成绩

如果不希望某个用户出现在榜单中，可以使用该接口删除用户的成绩以及在榜单中的排名（仅删除当前排行榜的成绩，不能删除历史版本的成绩）。

```sh
curl -X DELETE \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -H "Content-Type: application/json" \
  https://{{host}}/1.1/leaderboard/users/<uid>/statistics?statistics=wins,world
```

成功删除时返回空对象：

```
{}
```

类似地，可以删除 object 的成绩：

```sh
curl -X DELETE \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  --data-urlencode 'statistics=weapons,equipments' \
  https://{{host}}/1.1/leaderboard/objects/<objectId>/statistics
```

以及 entity 的成绩：

```sh
curl -X DELETE \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  --data-urlencode 'statistics=cities' \
  https://{{host}}/1.1/leaderboard/entities/<entityString>/statistics
```

同样，当前用户可以删除自己的成绩，这个不属于管理接口，不需要 `Master Key`，但需要传入当前用户的 `sessionToken`（客户端 SDK 更新当前用户的成绩封装了这一接口）：

```sh
curl -X DELETE \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "X-LC-Session: <sessionToken>" \
  -H "Content-Type: application/json" \
  https://{{host}}/1.1/leaderboard/users/self/statistics?statistics=wins,world
```

## 查询接口

通过 REST API 可以查询成绩和排行榜，这些接口不属于管理接口，不需要 `Master Key`：
### 查询成绩

#### 查询某个成绩

指定用户的 objectId 即可获取该用户的成绩。
你可以在请求 url 中指定多个 `statistics` 来获得多个排行榜中的成绩，排行榜名称用英文逗号 `,` 隔开，如果不指定将会返回该用户参与的所有排行榜中的成绩。

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  --data-urlencode 'statistics=wins,world' \
  https://{{host}}/1.1/leaderboard/users/<objectId>/statistics
```

返回结果：

```json
{
  "results": [
    {
      "statisticName": "wins",
      "statisticValue": 5,
      "version": 0,
      "user": {
        "__type": "Pointer",
        "className": "_User",
        "objectId": "60d950629be318a249000001"
      }
    },
    {
      "statisticName": "world",
      "statisticValue": 91,
      "version": 0,
      "user": {...}
    }
  ]
}
```

类似地，指定 object 的 objectId 可以查询该 object 参与的排行榜的成绩：

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  --data-urlencode 'statistics=wins,world' \
  https://{{host}}/1.1/leaderboard/objects/<objectId>/statistics
```

返回示例：

```json
{
  "results": [
    {
      "statisticName": "wins",
      "statisticValue": 5,
      "version": 0,
      "object": {
        "__type": "Pointer",
        "className": "Weapon",
        "objectId": "60d1af149be3180684000002"
      }
    },
    {
      "statisticName": "world",
      "statisticValue": 91,
      "version": 0,
      "object": {
        "__type": "Pointer",
        "className": "Weapon",
        "objectId": "60d1af149be3180684000002"
      }
    }
  ]
}
```

获取某个 entity 成绩时则需指定该 entity 的字符串：

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  --data-urlencode 'statistics=wins,world' \
  https://{{host}}/1.1/leaderboard/entities/<entityString>/statistics
```

返回示例：

```json
{
  "results": [
    {
      "statisticName": "wins",
      "statisticValue": 5,
      "version": 0,
      "entity": "1a2b3c4d"
    },
    {
      "statisticName": "world",
      "statisticValue": 91,
      "version": 0,
      "entity": "1a2b3c4d"
    }
  ]
}
```

#### 查询一组成绩

通过这个接口可以一次性拉取多个 user 的成绩，最多不超过 200 个。在请求中，需要在 body 中传入 user 的 `objectId` 数组。

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '["60d950629be318a249000001", "60d950629be318a249000000"]'
  https://{{host}}/1.1/leaderboard/users/statistics/<statisticName>
```

返回示例：

```json
{
  "results": [
    {
      "statisticName": "wins",
      "statisticValue": 1,
      "version": 0,
      "user": {
        "__type": "Pointer",
        "className": "_User",
        "objectId": "60d950629be318a249000001"
      }
    },
    {
      "statisticName": "wins",
      "statisticValue": 2,
      "version": 0,
      "user": {
        "__type": "Pointer",
        "className": "_User",
        "objectId": "60d950629be318a249000000"
      }
    }
  ]
}
```

类似地，传入 object 的 `objectId` 数组可以一次性获取多个 object 的成绩（最多不超过 200 个）：

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '["60d950629be318a249000001", "60d950629be318a249000000"]'
  https://{{host}}/1.1/leaderboard/objects/statistics/<statisticName>
```

传入 entity 的字符串数组则可以一次性获取多个 entity 的成绩（最多不超过 200 个）：

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '["Vimur", "Fimbulthul"]'
  https://{{host}}/1.1/leaderboard/entities/statistics/<statisticName>
```

### 查询排行榜

#### 获取指定区间的排名

你可以使用这个接口来获取 Top 排名。

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'startPosition=0' \
  --data-urlencode 'maxResultsCount=20' \
  --data-urlencode 'selectKeys=username,avatar' \
  --data-urlencode 'includeKeys=avatar' \
  --data-urlencode 'includeStatistics=wins' \
  https://{{host}}/1.1/leaderboard/leaderboards/user/<statisticName>/ranks
```

| 参数        | 约束   | 说明                                   |
| --------- | ---- | ---------------------------------------- |
| startPosition  | 可选  | 排行头部起始位置，默认为 0。 |
| maxResultsCount | 可选   | 最大返回数量，默认为 20。 |
| selectKeys | 可选   |  返回用户在 `_User` 表的其他字段，支持多个字段，用英文逗号 `,` 隔开。出于安全性考虑，在非 masterKey 请求下不返回敏感字段 `email` 及 `mobilePhoneNumber`。|
| includeKeys | 可选   |  返回用户在 `_User` 表的 pointer 字段的详细信息，支持多个字段，用英文逗号 `,` 隔开。为确保安全，在非 masterKey 请求下不返回敏感字段 `email` 及 `mobilePhoneNumber`。|
| includeStatistics | 可选   |  返回该用户在其他排行榜中的成绩，如果传入了不存在的排行榜名称，将会返回错误。 |
| version | 可选   | 返回指定 version 的排行结果，默认返回当前版本的数据。|
| count  | 可选  | 值为 1 时返回该排行榜中的成员数量，默认为 0。 |

返回 JSON 对象：

```json
{
  "results": [
    {
      "statisticName": "world",
      "statisticValue": 91,
      "rank": 0,
      "user": {
        "__type": "Pointer",
        "className": "_User",
        "updatedAt": "2021-07-21T03:08:10.487Z",
        "username": "zw1stza3fy701rvgxqwiikex7",
        "createdAt": "2020-09-04T04:23:04.795Z",
        "photo": {
          "objectId": "60f78f98d9f1465d3b1da12d",
          "__type": "File",
          "url": "https://example.com/user_1.jpg",
          "updatedAt": "2021-07-21T03:08:08.692Z",
          "createdAt": "2021-07-21T03:08:08.692Z",
        },
        "objectId": "5f51c1287628f2468aa696e6"
      }
    },
    {...}
  ],
  "count": 500
}
```

查询 object 排行榜的 Top 排名的接口与之类似，只是将 `user` 替换为 `object`：

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'startPosition=0' \
  --data-urlencode 'maxResultsCount=2' \
  --data-urlencode 'selectKeys=name,avatar' \
  --data-urlencode 'includeKeys=avatar' \
  --data-urlencode 'count=1' \
  https://{{host}}/1.1/leaderboard/leaderboards/object/<statisticName>/ranks
```

同理，URL 中的 `user` 替换为 `entity` 可查询 entity 排行榜的 Top 排名：

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'startPosition=0' \
  --data-urlencode 'maxResultsCount=2' \
  --data-urlencode 'count=1' \
  https://{{host}}/1.1/leaderboard/leaderboards/entity/<statisticName>/ranks
```

#### 获取附近排名

在 URL 末端附加相应的 objectId 可获取某用户或 object 附近的排名。

获取某用户附近的排名：

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'startPosition=0' \
  --data-urlencode 'maxResultsCount=20' \
  --data-urlencode 'selectKeys=username,avatar' \
  --data-urlencode 'includeKeys=avatar' \
  https://{{host}}/1.1/leaderboard/leaderboards/user/<statisticName>/ranks/<objectId>
```

参数含义参见上面[获取指定区间的排名](#获取指定区间的排名)一节。

返回：

```json
{
  "results": [
    {
      "statisticName": "wins",
      "statisticValue": 3,
      "rank": 2,
      "user": {...}
    },
    {
      "statisticName": "wins",
      "statisticValue": 2.5,
      "rank": 3,
      "user": {
        "__type": "Pointer",
        "className": "_User",
        "username": "kate",
        "avatar": {
          "bucket": "test_files",
          "url": "https://example.com/kate.jpg",
          "objectId": "60d2faa99be3183623000000",
          "__type": "File",
          "provider": "qiniu"
        },
        "objectId": "60d2faa99be3183623000001"
      }
    },
    {
      "statisticName": "wins",
      "statisticValue": 2,
      "rank": 4,
      "user": {...}
    }
  ],
  "count": 500
}
```

获取某 object 附近的排名：

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'startPosition=0' \
  --data-urlencode 'maxResultsCount=2' \
  --data-urlencode 'selectKeys=name,avatar' \
  --data-urlencode 'includeKeys=avatar' \
  --data-urlencode 'count=1' \
  https://{{host}}/1.1/leaderboard/leaderboards/object/<statisticName>/ranks/<objectId>
```

同理，在 URL 末端附加 entity 字符串即可获取该 entity 附近的排名：

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'startPosition=0' \
  --data-urlencode 'maxResultsCount=2' \
  --data-urlencode 'count=1' \
  https://{{host}}/1.1/leaderboard/leaderboards/entity/<statisticName>/ranks/<id>
```