# editor<br/>
vue + wangeditor富文本编辑器<br/>
1、在vue的项目中引用的 wangeditor ^3.1.1 富文本编辑器 需要先 npm install <br/>
2、用的时候直接js引入 import Editor from 'Editor.vue' from存放的地址 <br/>
3、模版里 “<editor v-model="detail" class="editor" :menus="menus"/>” 【detail为编辑器输入的值，menus为菜单配置】<br/>
   工具参考 https://www.kancloud.cn/wangfupeng/wangeditor3/332599 使用手册<br/>
    
4、遇到难点：<br/>
   &nbsp;图片在进行本地上传的时候需要对图片的宽高进行限制，但由于该工具是简洁版并为针对这块进行设置，只能获取到图片大小size<br/>
   &nbsp;在提供的自定义上传图片事件中引入了ES2017的新特性 async function 异步函数 通过 await 处理异步 async 计算的结果和错误 
   &nbsp;await（只允许在 async(异步) 函数内部使用）等待其操作对象 Promise 返回：<br/>
   &nbsp;如果 Promise 是完成状态，await 的结果是完成态的值。
   &nbsp;如果 Promise 是拒绝状态，await 会抛出拒绝值。<br/>
   &nbsp;该插件只支持图片单张上传 后期功能要求改为多图片上传 利用回调、并对签名进行了重置、为防止同名覆盖问题，对命名进行了字符串与时间戳的拼接、XML发送数据进行请求有个时间响应故而加了个定时器进行延迟执行第二张……类推后面的图片调用
5、未做处理：<br/>
   网络链接图片地址无法进行限制，仅能获取到宽高，没想到好的解决方案，只能在插入前进行限制，唯一想到的就是需要对源码进行修改，后面需求被砍了就没细做了。<br/>
6、知识点：<br/>
   &nbsp;XMLHttpRequest (XHR)对象可以与服务器交互。您可以从URL获取数据，而无需让整个的页面刷新。这使得Web页面可以只更新页面的局部，而不影响用户的操作   
