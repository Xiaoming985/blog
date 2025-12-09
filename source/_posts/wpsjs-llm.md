---
title: WPS集成大模型
date: 2025-06-16 10:00:00
tags:
 - wpsjs加载项
 - LLM大语言模型集成
---
# wpsjs入门

## 环境搭建
1.全局安装wpsjs，安装要求: node.js >= 18 
```shell
npm install -g wpsjs
```
2.检查安装是否成功，可执行如下命令
```shell
wpsjs -h # 查看帮助信息
wpsjs -v # 查看版本
```
3.更新
```shell
npm update -g wpsjs
```
4.创建项目
```shell
wpsjs create <project-name>

# 执行上述命令后，控制台出现如下提示:

# 第一步，选择加载项类型，文字(wps) -> word，演示(wpp) -> ppt，电子表格(et) -> excel
? 选择 WPS 加载项类型: (Use arrow keys)
❯ 文字 
  演示 
  电子表格 

# 第二步，选择熟悉的框架即可
? 选择UI框架: (Use arrow keys)
❯ Vue(推荐) 
  React 
  无 
```
5.安装依赖
```shell
cd <project-name>
npm install
```
6.启动项目
```shell
wpsjs debug
```

## 项目文件说明
```xml
<!-- public/ribbon.xml -->
<!-- 
	button 按钮
	menu 下拉选择按钮
	separator 分割线
 -->
<customUI xmlns="http://schemas.microsoft.com/office/2006/01/customui" onLoad="ribbon.OnAddinLoad">
    <ribbon startFromScratch="false">
        <tabs>
            <tab id="wpsAddinTab" label="Word AI 助手">
                <group id="btnDemoGroup" label="group1">
                    <button id="continueWrite" label="续写" onAction="ribbon.OnAction" getImage="ribbon.GetImage" size="large"></button>
                    <menu id="polish" label="润色" getImage="ribbon.GetImage" size="large">
                        <button id="polish" label="快速润色" onAction="ribbon.OnAction"></button>
                        <menuSeparator id="menuSeparator1" />
                        <button id="polish1" label="更正式" onAction="ribbon.OnAction"></button>
                        <button id="polish2" label="党政风" onAction="ribbon.OnAction"></button>
                        <button id="polish3" label="更活泼" onAction="ribbon.OnAction"></button>
                        <button id="polish4" label="口语化" onAction="ribbon.OnAction"></button>
                        <button id="polish5" label="更学术" onAction="ribbon.OnAction"></button>
                    </menu>
                    <button id="extendWrite" label="扩写" onAction="ribbon.OnAction" getImage="ribbon.GetImage" size="large"></button>
                    <button id="shorthandWrite" label="缩写" onAction="ribbon.OnAction" getImage="ribbon.GetImage" size="large"></button>
                    <button id="rewrite" label="重写" onAction="ribbon.OnAction" getImage="ribbon.GetImage" size="large"></button>
                    <button id="correct" label="文本纠错" onAction="ribbon.OnAction" getImage="ribbon.GetImage" size="large"></button>
                    <separator id="separator1" />
                    <button id="fileQA" label="文档问答" onAction="ribbon.OnAction" getImage="ribbon.GetImage" size="large"></button>
                    <menu id="translate" label="翻译" getImage="ribbon.GetImage" size="large">
                        <button id="textTranslate" label="文本翻译" onAction="ribbon.OnAction"></button>
                        <button id="contextTranslate" label="全文翻译" onAction="ribbon.OnAction"></button>
                        <button id="docTranslate" label="文档翻译" onAction="ribbon.OnAction"></button>
                    </menu>
                    <button id="suggest" label="建议反馈" onAction="ribbon.OnAction" getImage="ribbon.GetImage" size="large"></button>
                </group>
            </tab>
        </tabs>
    </ribbon>
</customUI>
```
```js
// src/components/ribbon.js
import Util from './js/util.js'
import SystemDemo from './js/systemdemo.js'
import { 
  continueWrite, 
  polish, 
  extendWrite, 
  shorthandWrite, 
  rewrite, 
  correct, 
  textTranslate, 
  docTranslate,
  fileQA,
  suggest
} from "./js/wpstool.js" // 此处是封装好的一些跟大模型交互的方法

//这个函数在整个wps加载项中是第一个执行的
function OnAddinLoad(ribbonUI) {
  if (typeof window.Application.ribbonUI != 'object') {
    window.Application.ribbonUI = ribbonUI
  }

  if (typeof window.Application.Enum != 'object') {
    // 如果没有内置枚举值
    window.Application.Enum = Util.WPS_Enum
  }

  //这几个导出函数是给外部业务系统调用的
  window.openOfficeFileFromSystemDemo = SystemDemo.openOfficeFileFromSystemDemo
  window.InvokeFromSystemDemo = SystemDemo.InvokeFromSystemDemo

  window.Application.PluginStorage.setItem('EnableFlag', false) //往PluginStorage中设置一个标记，用于控制两个按钮的置灰
  window.Application.PluginStorage.setItem('ApiEventFlag', false) //往PluginStorage中设置一个标记，用于控制ApiEvent的按钮label
  return true
}

function OnAction(control) {
  const eleId = control.Id
  switch (eleId) {
    case 'continueWrite': // 续写
      continueWrite();
      break;
    case 'polish': // 润色
      polish();
      break;
    case 'polish1': // 润色：更正式
      polish(eleId);
      break;
    case 'polish2': // 润色：党政风
      polish(eleId);
      break;
    case 'polish3': // 润色：更活泼
      polish(eleId);
      break;
    case 'polish4': // 润色：口语化
      polish(eleId);
      break;
    case 'polish5': // 润色：更学术
      polish(eleId);
      break;
    case 'extendWrite': // 扩写
      extendWrite();
      break;
    case 'shorthandWrite': // 缩写
      shorthandWrite();
      break;
    case 'rewrite': // 重写
      rewrite();
      break;
    case 'correct': // 文字纠错 
      correct();
      break;
    case 'fileQA': // 文档问答
      fileQA();
      break;
    case "textTranslate": // 文本翻译
      textTranslate();
      break;
    case "contextTranslate": // 全文翻译
      docTranslate(eleId);
      break;
    case "docTranslate": // 文档翻译
      docTranslate(eleId);
      break;
    case "suggest": // 建议反馈
      suggest(eleId);
      break;
    default:
      break
  }
  return true
}

function GetImage(control) {
  const eleId = control.Id
  if (eleId) {
    return `images/${eleId}.svg`
  }
  return 'images/newFromTemp.svg'
}

//这些函数是给wps客户端调用的
export default {
  OnAddinLoad,
  OnAction,
  GetImage
}
```

## 打包部署
1.打包
```shell
wpsjs build

# 执行上述命令，控制台会出现如下提示：
? 选择 WPS 加载项发布类型: (Use arrow keys)
❯ 在线插件 
  离线插件
```
这一步执行成功后，会生成一个wps-addon-build的文件夹

2.发布
```shell
wpsjs publish

# 执行上述命令，控制台提示服务器地址，即插件的部署路径（安装路径）：
服务器地址示例 "http://127.0.0.1/jsplugindir/"
? 请输入发布 WPS 加载项的服务器地址: 
```
这一步执行成功后，会生成一个wps-addon-publish/publish.html，用于分发（安装/卸载）插件

3.取消发布
```shell
wpsjs unpublish
```

4.在线插件部署
- 准备一个服务器容器，用于部署网页（即wpsjs build生成的包），nginx、express、tomcat...根据个人喜好挑选即可
- 修改publish.html
```js
// publish.html LoadPublishAddons方法
function LoadPublishAddons() {
    var addonList = document.getElementById("addonList");
    const url = "https://www.aaa.com/word-tool/" // word插件部署路径
    const wppUrl = "https://www.aaa.com/ppt-tool/" // ppt插件部署路径
    var curList = [
        {"name":"Word插件","addonType":"wps","online":"true","multiUser":"false","url":url},
        {"name":"PPT插件","addonType":"wpp","online":"true","multiUser":"false","url":wppUrl}
    ]; // 已发布的插件列表
    curList.forEach(function (element) {
        var param = JSON.stringify(element).replace("\"", "\'");
        UpdateElement(element, 'enable')
    });
}
```

5.离线插件
- 将打包好的文件夹，修改命名et-tool，并压缩成et-tool.7z
- 修改publish.html
```js
function LoadPublishAddons() {
  var addonList = document.getElementById("addonList");
 
  //publish.html文件的路径位置不同，url也会跟着不同，增加这行代码
  var addonPath = location.href.match(/.+\//)[0] + 'et-tool.7z'
  var curList = [{ "name": "et-tool", "addonType": "et", "online": "false", "multiUser": "false", "url": addonPath, "version": "1.0.0" }];
  curList.forEach(function (element) {
	var param = JSON.stringify(element).replace("\"", "\'");
	UpdateElement(element, 'enable')
  });
}
```
- 将打包好的et-tool.7z和publish.xml文件复制到WPS的加载项目录，如C:\Users\用户名\AppData\Roaming\kingsoft\wps\jsaddons
- 解压压缩包
- 重启WPS即可安装离线加载项

# 集成大模型
1.假设你已经部署了大语言模型，并提供了一个问答接口：https://www.aaa.com/chat/completions
2.js网络请求大模型
```js
import { fetchEventSource } from '@microsoft/fetch-event-source'
/**
 * 生成数据
 * @param {*} type 对应getPrompt的command
 * @param {*} text 
 * @param {*} abortController 用于终止请求
 * @param {*} callback 实时响应回调方法
 * @returns 
 */
export function generateData(type, text, abortController, callBack = () => {}) {
  return new Promise((resolve, reject) => {
    let resData = "";
    fetchEventSource(`https://www.aaa.com/chat/completions`, {
      method: 'POST',
      signal: abortController.signal,
      headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer " + import.meta.env.VITE_APP_API_KEY // api key
      },
      body: JSON.stringify({
        "model": "qwen2---5-72b-goxmbugy", // 千问2.5 72b模型
        "messages": [
          { "role": "user", "content": getPrompt(type, text) }
        ],
        "stream": true
      }),
      openWhenHidden: true,
      onopen(response) {
        if (response.ok) return;
        throw new Error(`连接失败: ${response.status}`);
      },
      onmessage(msg) {
        if (msg && msg.data != '[DONE]') {
          let data = JSON.parse(msg.data);
          if (data.choices && data.choices.length > 0) {
            const choice = data.choices[0];
            if(choice.finish_reason == 'stop') { //结束
              return;
            }
            if (choice.delta.content && choice.delta.content != 'undefined') {
              resData += choice.delta.content;
              callBack(choice.delta.content, resData);
            }
          }
        }
      },
      onclose() {//正常结束的回调
        if (type == 'outline') {
          resolve(resData);
        } else {
          let jsonStr = resData.replace("```json", "").replace("```", "");
          resolve(parseData(jsonStr.trim()));
        }
      },
      onerror(err){//连接出现异常回调
        reject(err)
        throw err
      }
    })
  })
}

export function getPrompt(command, text) {
	switch (command) {
		case 'slides': // 生成幻灯片大纲
		    return `<角色设定>
		        你是一位PPT大纲生成专家，我正在准备一份关于${text}的PPT，请协助生成数组格式的PPT幻灯片大纲。
		        </角色设定>
		
		        <期望输出>
		        [
		          {
		            id: "唯一id",
		            type: "cover",
		            value: "PPT主题"
		          },
		          {
		            id: "唯一id",
		            type: "catalog",
		            value: ["章节标题", "章节标题", ...],
		          },
		          {
		            id: "唯一id",
		            type: "chapter",
		            value: "章节标题",
		          },
		          {
		            id: "唯一id",
		            parentId: "指向对应章节的id",
		            type: "content",
		            value: "正文标题",
		            writingIdea: "写作思路"
		          },
		          {
		            id: "唯一id",
		            parentId: "指向对应章节的id",
		            type: "content",
		            value: "正文标题",
		            writingIdea: "写作思路"
		          },
		          ...
		          {
		            id: "唯一id",
		            type: "ending",
		            value: "感谢聆听"
		          }
		        ]
		        </期望输出>
		
		        <其他要求>
		        1.每一个章节(chapter)可包含多个内容页(content)，请尽可能包含更多的要点。
		        2.请确保输出是符合RFC-8295规范的有效JSON。
		        3.不要返回任何其他消息。
		        </其他要求>`;
		case 'partJson': // 根据部分幻灯片大纲，转换成JSON格式的大纲
		    return `<任务描述>
		        我正在准备一份PPT，请根据给定的幻灯片大纲内容，协助生成JSON格式的大纲。
		        </任务描述>
		
		        <幻灯片大纲>
		        ${text}
		        </幻灯片大纲>
		
		        <期望输出>
		        {
		          id: <唯一id>,
		          type: <幻灯片版式>,
		          value: <内容>,
		          shapes: [
		            {
		              id: <唯一id>,
		              value: <内容要点>,
		              text: <string类型，论述当前内容要点，要求具体详细>,
		            }
		          ]
		        }
		        </期望输出>
		        
		        <JSON字段要求>
		        1.id: 维持原有id，若原有id不存在，则生成新的唯一id
				2.type: 维持原有值，表示幻灯片版式
				3.value: 维持原有值
				4.shapes: 
				  - 表示当前幻灯片包含的元素，数组形式；
				  - 每个数组元素表示一个内容要点，请根据正文的写作思路(writingIdea)展开；
				  - 随机展开1到5个内容要点，但个数严格控制在1到5个之间；
				  - 内容要点要求语言精简，用做小标题，不可过长；
				  - 禁止生成重复要点，不可为空；
				5.shapes.text: String类型，用以论述正文对应的内容要点，要求给出具体的详细内容，不可为空；
		        </JSON字段要求>
		
		        <其他要求>
		        1.请确保输出是符合RFC-8295规范的有效JSON。
		        2.不要返回任何其他消息。
		        </其他要求>`;
	}
}
```
3.应用实例
```js
import { reactive } from 'vue';

const slides = reactive([]); // 幻灯片大纲
let abortController = new AbortController();
generateData(
  'slides', 
  "赛博养生", // 以赛博养生为主题，生成幻灯片大纲
  abortController,
  (currRes, res) => {
	let str = res.replace("```json", "").replace("```", "");
	let eleStr = str.substring(str.indexOf("{"), str.lastIndexOf("}") + 1); // 切割完整的json字符串
	if (eleStr.trim() && eleStr.includes("{") && eleStr.includes("}")) {
	  let tempArr = parseData(`[${eleStr}]`);
	  let ids = slides.map(item => item.id);
	  tempArr.filter(item => !ids.includes(item.id)).forEach(item => {
		slides.push(reactive({...item}));
	  })
	}
  }
).then(data => {
  console.log(data);
}).catch(err => {
  
}).finally(() => {
  
})
```