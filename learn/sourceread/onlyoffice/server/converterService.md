# DocService/converterService 656 行

  

相关依赖:

- nodejs

  - path

- config

- mime

  

依赖模块:

  

```plantuml
package DocService{
    [taskResult]
    [DocsCoServer]
    [CanavasService]
    [wopiClient]
    [baseConnector]
}
package Common{
    [constants]
    [commandDefines]
    [storageBase]
    [formatChecker]

    component statsdclient #Yellow [statsdclient1
    组件功能分析?]
    component storageBase [storage
    base]

    component editorDatamanager

}
Common.storageBase --o  Common.editorDatamanager


```


  

## 配置属性

  

`service.Couathoring.token.enable` 是否启用 kwt

  

## convertRequest