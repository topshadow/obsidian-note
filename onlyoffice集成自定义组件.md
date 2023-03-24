
##  1. 追加补丁代码
1. 修改  onlyoffice 的docker镜像文件`/var/www/onlyoffice/documentserver/web-apps/apps/documenteditor/main/index.html`
2. 追加 `<script src="http://localhost:4200/assets/scripts/b.js"></script>`


其中b.js源码如下,主要是与onlyoffice的`web-apps`项目的消息体系通信,直接处理关于控件下拉的事件,利用事件定位的位置参数和被点击组件的Tag属性，将我们自定义的控件插入到对应悬浮位置
代码如下
```javascript
  

function clearCustomElement() {

    let customEl = document.getElementById('my-custom'); if (customEl) { customEl.remove(); }

}

  

setTimeout(() => {

    window.asc_docs_api.prototype.asc_registerCallback('asc_onHideContentControlsActions', () => { clearCustomElement() });

    window.asc_docs_api.prototype.asc_registerCallback('asc_onShowContentControlsActions', (arg, x, y) => {

        clearCustomElement();

        let formPr = arg.pr.get_Tag();

        let data = JSON.parse(formPr);

  

        // alert('x:' + x + ' y:' + y + ' tag:' + formPr);

        debugger;

        let str = `<div >

        <div style="width: 100px;word-wrap: break-word;height: 80px;position: absolute;left: -90px;background: #e2e2e2;    background: #fff;

        box-shadow: 3px 3px 3px 3px #e2e2e2;

        border-radius: 6px;

        padding: 5px;">组件内容:${formPr}

        ${data.componentType == 'input-email' ? '<input type="email" style="max-width:100%">' : '<input  style="max-width:100%"/>'}

        </div>

        </div>`;

        let divEl = document.createElement('div');

        divEl.innerHTML = str;

        divEl.id = "my-custom";

        divEl.style = `position: absolute; z-index: 10000; left: ${x}px; top: ${y}px;`;

  

        let editor_sdk = document.getElementById('editor_sdk');

        editor_sdk.appendChild(divEl);

        setTimeout(() => {

            editor_sdk.childNodes.forEach(n => {

                if (n.id) {

                    // alert(n.id)

                    if (n.id.indexOf('menu-container-asc-gen') != -1) {

                        n.remove();

                    }

  

                }

            }, 300);

  

        })

    });

  

}, 5000);
```
## 2. 编写插件

插件配置 config.json

```json
{

    "name": "settings test",

    "guid": "asc.{FFE1F462-1EA2-4391-990D-4CC849400058}",

    "variations": [

        {

            "description": "settings test",

            "url": "index.html",

            "icons": [

                "icon.png",

                "icon@2x.png"

            ],

            "isViewer": true,

            "EditorsSupport": [

                "word",

                "cell",

                "slide"

            ],

            "isSystem": false,

            "isVisual": true,

            "isModal": false,

            "isInsideMode": true,

            "initDataType": "none",

            "initData": "",

            "buttons": []

        }

    ]

}
```
插件的首页代码 index.html

```html
<!--

 (c) Copyright Ascensio System SIA 2020

  

 Licensed under the Apache License, Version 2.0 (the "License");

 you may not use this file except in compliance with the License.

 You may obtain a copy of the License at

  

     http://www.apache.org/licenses/LICENSE-2.0

  

 Unless required by applicable law or agreed to in writing, software

 distributed under the License is distributed on an "AS IS" BASIS,

 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

 See the License for the specific language governing permissions and

 limitations under the License.

 -->

 <!DOCTYPE html>

 <html lang="en">

     <head>

         <meta charset="UTF-8" />

         <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">

         <title>Example add content controls</title>

         <style type="text/css">

             html, body {

                 margin: 0px;

                 padding: 0px;

                 overflow: hidden;

                 width:100%;

                 height:100%;

                 display: flex;

                 flex-flow: column;

             }

             .div_text {

                 width: calc(100% - 10px);

                 height: 20%;

                 margin: 5px;

                 border: 1px solid black;

             }

             .div_main {

                 display: flex;

                 flex-flow: column;

                 width: 100%;

                 height: 100%;

             }

         </style>

         <script type="text/javascript" src="https://onlyoffice.github.io/sdkjs-plugins/v1/plugins.js"></script>

         <script type="text/javascript" src="https://onlyoffice.github.io/sdkjs-plugins/v1/plugins-ui.js"></script>

         <link rel="stylesheet" href="https://onlyoffice.github.io/sdkjs-plugins/v1/plugins.css">

         <script type="text/javascript" src="http://localhost:4200/assets/scripts/code.js"></script>

     </head>

     <style>

        .div_text>div{

            flex: 1;

    text-align: center;

    background: #e2e2e2;

    color: #929292;

    height: 40px;

    vertical-align: middle;

    display: flex;

    justify-content: center;

    align-items: center;

    border-radius: 8px;

    margin-right: 5px;

    cursor: pointer;

        }

        .div_text>div.active{

            background-color: #1248f8;

    border-color: #0740f6;

    color: #fff;

        }

ul,ul *{

    list-style-type: none;

}

.ui-group{

    list-style-type: none;

    display: grid;

    grid-template-columns: 1fr 1fr;

    grid-gap: 5px;

}

.ui-group li{

    display: inline-block;

    width: 100%;

    cursor: pointer;

}
     </style>
     <body>
         <div class="div_text" style="padding-top: 20px;display: flex;height: auto;">
           <div class="active">编辑模式</div>
           <div>预览模式</div>
         </div>
         <div>
            <div style="display:grid">
                <details>
                    <summary style="    font-size: 1.2rem;">容器组件</summary>
                    <ul class="ui-group">
                        <li id="one-two">一行两列</li>
                        <li>一行三列</li>
                    </ul>
                </details>
            </div>
            <div style="display:grid">
                <details>
                    <summary style="    font-size: 1.2rem;">数据控件</summary>
                    <ul class="ui-group">
                        <li id="checkbox">选择框</li>
                        <li>多选框</li>
                    </ul>
                </details>
            </div>
         </div>
     </body>
 </html>
```


```javascript
  

function clearCustomElement() {

    let customEl = document.getElementById('my-custom'); if (customEl) { customEl.remove(); }

}

  

function getComponentHtml(type, data) {

    let componentHtml = `<input />`

    switch (type) {

        case 'checkbox':

            componentHtml = `<div>` + data.map(li => ` <input type='checkbox' />` + li.label).join('\n') + `<br></div>`

            break;

        case 'radio':

            componentHtml = `<input type='radio' />`

            break;

        case 'dropdown':

            componentHtml = `<select >

            ${data.map(li => `<option value="${li.value}">${li.label}</option>`).join('\n')}

            </select>`

    }

    let str = `<div >

    <div style="width: 100px;word-wrap: break-word;height: 80px;position: absolute;left: -90px;background: #e2e2e2;    background: #fff;

    box-shadow: 3px 3px 3px 3px #e2e2e2;

    border-radius: 6px;

    padding: 5px;">组件内容:

  ${componentHtml}

    </div>

    </div>`;

    return str;

}

  

setTimeout(() => {

    window.asc_docs_api.prototype.asc_registerCallback('asc_onHideContentControlsActions', () => { clearCustomElement() });

    window.asc_docs_api.prototype.asc_registerCallback('asc_onShowContentControlsActions', (arg, x, y) => {

        clearCustomElement();

        let formPr = arg.pr.get_Tag();

        let data = JSON.parse(formPr);

        let datasource = data.datasource;

        let html = ``;

        if (datasource) {

            fetch(datasource).then(async res => {

                let list = await res.json();

                html = getComponentHtml(data.componentType, list);

                let divEl = document.createElement('div');

                divEl.innerHTML = html;

                divEl.id = "my-custom";

                divEl.style = `position: absolute; z-index: 10000; left: ${x}px; top: ${y}px;`;

                let editor_sdk = document.getElementById('editor_sdk');

                editor_sdk.appendChild(divEl);

  

            })

        } else {

            html = `暂无数据源`

            let divEl = document.createElement('div');

            divEl.innerHTML = html;

            divEl.id = "my-custom";

            divEl.style = `position: absolute; z-index: 10000; left: ${x}px; top: ${y}px;`;

            let editor_sdk = document.getElementById('editor_sdk');

            editor_sdk.appendChild(divEl);

  

        }

        // alert('x:' + x + ' y:' + y + ' tag:' + formPr);

        debugger;

  
  
  
  

        setTimeout(() => {

            editor_sdk.childNodes.forEach(n => {

                if (n.id) {

                    // alert(n.id)

                    if (n.id.indexOf('menu-container-asc-gen') != -1) {

                        n.remove();

                    }

  

                }

            }, 300);

  

        })

    });

  

}, 5000);
```
