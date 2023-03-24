# DocService/server.js 371 行

  

应用程序启动入口

依赖库:

  

- config

- express

- node

  - url

  - http

  - path

  - fs

- express

  - bodyparser

  - mime

  - apicache

  - multer

    依赖模块

  

```puml

  

package DocService {

    [docCoService]

    [canvasService]

    [converService]

    [wopiClient]

    [fileConvertService]

    [serverjs]

}

  
  

```

  

api 接口列表

  

接口

  

| 方法     | 路径                             | 功能                                                                 |
| -------- | -------------------------------- | -------------------------------------------------------------------- |
| get      | index.html                       |                                                                      |
| get/post | /coauthoring/CommandService.ashx | [docsCoServer.commandFromServer](./2_DocService.md#install)          |
| get/post | /ConvertService.ashx             | [converterService.convertXml](./4.conveterService.md#convertrequest) |