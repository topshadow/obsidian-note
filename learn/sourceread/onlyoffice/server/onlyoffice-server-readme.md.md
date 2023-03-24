# 服务端代码分析

  

编程语言: nodejs

依赖库: coolie,socket.io,rabbitmq,underscore,json

  

目录结构:

  

- Common: 通用代码，无启动项

- DocService: 服务端鉴权,文档变更编辑，多人协作等

  - DocCoService

  - baseConnector

- FileConverter: 文件格式转换,目前有转换 xml,json 等

- Metrics: 信息暴露服务

- SpellChecker: 拼写检查