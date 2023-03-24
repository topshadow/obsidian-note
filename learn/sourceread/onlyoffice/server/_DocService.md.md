[Toc]

  

# 项目 DocService

  

文件 `source/DocService/Sources/DocCoService.js`

行数: 3922 行

  

依赖:

  

- socket.io

- underscore

- cluser

- crypto

- jsonwebtoken

- jwt

- ms  `将特定事件表达式转为millsecond` ,详情参看脚注[[_DocService.md#ms库使用说明]]

- deep_equal

  

## 引用的模块

  

- Common:

  - storage_done

  - logger

  - constans

  - utils

  - commondefines

  - statsdclient

  - license

- DocService

  - baseConnector

  - canvasService

  - connectorService

  - taskResult

  - gc

  - shutdown

  - pubsubRabbitmq

  - wopiClient

  

# 配置的对象

  

文件 `/Common/config/default.json`

  

# 功能:

  

- 动态插件功能: 将插件复制到指定插件目录下,会动态监测插件，自动强制推送插件更新事件

  

# 流程分析

  

1. 构建 Editor 配置对象

  

```plantuml
@startjson
#highlight "services" / "CoAuthoring" / "sql"
#highlight "services" / "CoAuthoring" / "editor"

{
    "services": {
        "CoAuthoring": {
            "server": {
                "port": 8000,
                "workerpercpu": 1,
                "mode": "development",
                "limits_tempfile_upload": 104857600,
                "limits_image_size": 26214400,
                "limits_image_download_timeout": {
                    "connectionAndInactivity": "2m",
                    "wholeCycle": "2m"
                },
                "callbackRequestTimeout": {
                    "connectionAndInactivity": "10m",
                    "wholeCycle": "10m"
                },
                "healthcheckfilepath": "../public/healthcheck.docx",
                "savetimeoutdelay": 5000,
                "edit_singleton": false,
                "forgottenfiles": "forgotten",
                "forgottenfilesname": "output",
                "maxRequestChanges": 20000,
                "openProtectedFile": true,
                "editorDataStorage": "editorDataMemory",
                "assemblyFormatAsOrigin": true,
                "newFileTemplate": "../../document-templates/new",
                "downloadFileAllowExt": [
                    "pdf",
                    "xlsx"
                ],
                "tokenRequiredParams": true
            },
            "sql": {
                "type": "postgres",
                "tableChanges": "doc_changes",
                "tableResult": "task_result",
                "dbHost": "localhost",
                "dbPort": 5432,
                "dbName": "onlyoffice",
                "dbUser": "onlyoffice",
                "dbPass": "onlyoffice",
                "charset": "utf8",
                "connectionlimit": 10,
                "max_allowed_packet": 1048575,
                "pgPoolExtraOptions": {}
            },
            "redis": {
                "name": "redis",
                "prefix": "ds:",
                "host": "localhost",
                "port": 6379,
                "options": {}
            },
            "pubsub": {
                "maxChanges": 0
            },
            "expire": {
                "saveLock": 60,
                "presence": 300,
                "locks": 604800,
                "changeindex": 86400,
                "lockDoc": 30,
                "message": 86400,
                "lastsave": 604800,
                "forcesave": 604800,
                "saved": 3600,
                "documentsCron": "0 */2 * * * *",
                "files": 86400,
                "filesCron": "00 00 */1 * * *",
                "filesremovedatonce": 100,
                "sessionidle": "1h",
                "sessionabsolute": "30d",
                "sessionclosecommand": "2m",
                "pemStdTTL": "1h",
                "pemCheckPeriod": "10m",
                "updateVersionStatus": "5m",
                "monthUniqueUsers": "1y"
            },
            "secret": {
                "inbox": {
                    "string": "secret",
                    "file": ""
                },
                "outbox": {
                    "string": "secret",
                    "file": ""
                },
                "session": {
                    "string": "secret",
                    "file": ""
                }
            },
            "token": {
                "enable": {
                    "browser": false,
                    "request": {
                        "inbox": false,
                        "outbox": false
                    }
                },
                "inbox": {
                    "header": "Authorization",
                    "prefix": "Bearer ",
                    "inBody": false
                },
                "outbox": {
                    "header": "Authorization",
                    "prefix": "Bearer ",
                    "algorithm": "HS256",
                    "expires": "5m",
                    "inBody": false,
                    "urlExclusionRegex": ""
                }
            },
            "editor": {
                "spellcheckerUrl": "",
                "reconnection": {
                    "attempts": 50,
                    "delay": "2s"
                },
                "binaryChanges": false,
                "websocketMaxPayloadSize": "1.5MB",
                "maxChangesSize": "0mb"
            },
            "sockjs": {
                "sockjs_url": "",
                "disable_cors": true,
                "websocket": true
            },
            "socketio": {
                "connection": {
                    "path": "/doc/",
                    "serveClient": false,
                    "pingTimeout": 20000,
                    "pingInterval": 25000,
                    "maxHttpBufferSize": 1e8
                }
            }
        },
        "license": {
            "license_file": "",
            "warning_limit_percents": 70,
            "packageType": 0
        }
    }
}
@endjson
```

  

2. 模块暴露方法

  

```puml

@startuml

interface ServiceCoServer{

    install(server,callback);// 主服务加载,2268行

    commandFromServer(req, res)

}

@enduml

  

```

  

2.1 install 方法流程

  

```puml

@startuml

title 总体流程

创建socketServer ->鉴权连接

鉴权连接 --> connection

connection --> message

message --> disconnection

@enduml

```

  

对象图

  

```puml

@startuml
object tenantManager{

   getDefaultTenant(); //获取默认租户

   getTenant(ctx,domain); // 调用getDefaultTenant()

   isMultitenantMode();// 根据config中的tenants.baseDir属性判断是否多租户,默认不是多租户;

   getTenantByConnection();// 默认固定租户，获取默认租户

  

}

object socketio

object operationContext{

   tenant; //当前租户  从tenantManager中获取

   initDefault(); //初始化多租户

   initFromConnection();// 获取当前文档id,用户id

  

}

operationContext --o socketio
@enduml

```

  

## 事件处理

  

### install

  

```puml

@startuml

start

:初始化 server;

if (jwt 认证) then (yes)

:connection;

else (no)

:发送认证结果和错误码;

stop

endif

:创建 operationContext;

:处理事件 onmessage;

switch (data.type)

case (auth)

:auth;

if (auth) then (认证失败)

:disconnection;

endif

break

case (message)

:onMessage;

:利用 socketio 派发消息;

break;

case (cursor)

:onCursor;

break;

case (getLock)

:getLock;

break;

case (saveChanges)

:saveChanges;

break;

case (isSaveLock)

:isSaveLock;

break;

case (unSaveLock)

:unSaveLock;

break;

case (getMessages)

:getMessages;

break;

case (unLockDocument)

:unLockDocument;

break;

case (close)

:closeDocument;

break;

case (versionHistory)

:versionHistory;

break;

case (openDocument)

:canvasService.openDocument;

break;

case (clientLog)

:存储日志;

break;

case (extendSession)

break;

case (forceSaveStart)

break;

case (rpc)

:startRpc;

break;

case (authChangesAck)

:authChangesAck;

  

endswitch

stop
@enduml

```

  

文件 `DocService/editorDataMemory.js`

  

```puml

@startuml
interface DocumentData{
expireAt:number;
fencingToken:string;//分布式锁
}

class EditorData{
    data:Map<string,DocumentData>
    forceSaveTimer:{}
    uniqueUser :{};
    uniqueUsersOfMonth : {};
    uniqueViewUser : {};
    uniqueViewUsersOfMonth : {};
    shutdown : {};
    stat : {};
    _getDocumentData(ctx,docId:string);
    _checkAndUnlock();
    addPresence();
    removePresence();
    lockSave();
    lockAuth();
    getDocumentPresenceExpired();
    removePresenceDocument();
    addLocks();
    removeLocks();
    getLocks();

}

EditorData -up-o DocumentData
@enduml

```

  

### commandFromServer

  

```puml
start
:登陆认证;
:参数解析;
switch (param.c)
  case (info)
    :查询文档详情;
  break;
  case (drop)
    :销毁文档;
    break;
 break;
  case (save)
    :保存文档;
  break;
  case (forcesave)
    :强制保存;
    break;
  case (version)
    :查询版本;
    break;
  case (licence)
    :返回许可证;
    break;
endswitch
stop

```

  

---

  

  
## ms库使用说明
> npmjs 官方说明 https://www.npmjs.com/package/ms
   Use this package to easily convert various time formats to milliseconds.
```javascript
ms('2 days') // 172800000
ms('1d') // 86400000
ms('10h') // 36000000
ms('2.5 hrs') // 9000000
ms('2h') // 7200000
ms('1m') // 60000
ms('5s') // 5000
ms('1y') // 31557600000
ms('100') // 100
ms('-3 days') // -259200000
ms('-1h') // -3600000
ms('-200') // -200
```
 