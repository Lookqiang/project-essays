# mediainfo.js 的使用 及 MP4在chrome上的编码问题

### 一、 MP4 在chrome上的编码要求：

MPEG4 = 带有H.264(AVC) 视频编码 和 AAC 音频编码的MPEG4 文件

### 二、mediainfo.js 的使用
1、在vue中的使用

#####   安装
```
$ npm install  mediainfo.js -save
```
##### 项目中的实例
在 vue.config.js 配置 
```
var CopyWebpackPlugin = require('copy-webpack-plugin');

const wasmFile = path.resolve(
    __dirname,
    'node_modules',
    'mediainfo.js',
    'dist',
    'MediaInfoModule.wasm'
)

module.exports = {
    configureWebpack: {
        plugins: [
            new CopyWebpackPlugin([{
                from:wasmFile 
            }])
        ]
    },
}
```
新建 mediaInfo.js 文件

```
import MediaInfo from "mediainfo.js";

function readChunk(file) {
  return (chunkSize, offset) =>
    new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.onload = (event) => {
        if (event.target.error) {
          reject(event.target.error);
        }
        resolve(new Uint8Array(event.target.result));
      };
      reader.readAsArrayBuffer(file.slice(offset, offset + chunkSize));
    });
}
//解析file 文件 获得 mediaInfo return promise
async function analyzeMedia(file) {
    let _result = {};
    let _mediaInfo = null;
    if (!file) return _result;
    await MediaInfo({
        locateFile: (url, scriptDirectory) => {
        //指定MediaInfoModule.wasm的url
        //防止生产环境中，匹配到根目录的MediaInfoModule.wasm 的文件
        return window.document.location.origin + "/" + url;
        },
    }).then((mediaInfo) => {
        _mediaInfo = mediaInfo;
    });
    await _mediaInfo
        .analyzeData(() => file.size, readChunk(file))
        .then((result) => {
        logZYKJ(result)
        _result = result;
        });
    return _result;
}

export default analyzeMedia;
```

### 总结
应用初始化的时候会加载MediaInfoModule.wasm（size：3.2m）的文件

推荐 CDN


```
<script type="text/javascript" src="https://unpkg.com/mediainfo.js/dist/mediainfo.min.js"></script>
```

github 地址 https://github.com/buzz/mediainfo.js