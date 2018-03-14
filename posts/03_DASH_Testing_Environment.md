#### &#x1F4DA; [Posts](./)

## 搭建 DASH 视频测试环境

目标：以视频文件 test.mp4 为输入，生成 DASH 媒体内容，部署到 Tomcat 上，使用 dash.js 播放，记录 QoE。

- 将视频编码为所需的 H.264/AVC 格式（如果文件本身为该格式，可以跳过该步骤）。
  ```bash
  x264 test.mp4 --output test.264 --fps 24 --preset slow \
    --bitrate 2400 --vbv-maxrate 4800 --vbv-bufsize 9600 \
    --min-keyint 48 --keyint 48 --scenecut 0 --no-scenecut --pass 1
  ```

- 将 h264 文件添加到 MP4 容器中。
  ```bash
  MP4Box -add test.264 -fps 24 container.mp4
  ```

- 将视频转换成不同比特率（可选操作）。
  ```bash
  ffmpeg -i container.mp4 -x264opts 'keyint=48:min-keyint=48:no-scenecut' \
    -s 1280x544 -b:v 2400k -b:a 128k 1280x544.mp4

  ffmpeg -i container.mp4 -x264opts 'keyint=48:min-keyint=48:no-scenecut' \
    -s 1920x816 -b:v 4800k -b:a 128k 1920x816.mp4
  ```

- 视频分片。
  ```bash
  MP4Box -dash 4000 -frag 4000 -rap -out result.mpd -bs-switching no \
    -segment-name '$RepresentationID$-$Number$$Init=init$' \
    1280x544.mp4 1920x816.mp4
  ```

	其中，'$RepresentationID$-$Number$$Init=init$'表示，所有分片以"RepresentationID-Number"的形式命名，比如，第1个 Representation 的第3个切片就命名为"1-3.m4s"（m4s 为缺省后缀，可更改），而起始切片则以"RepresentationID-init"的形式命名，则第2个 Representation 的起始切片名为"2-init.mp4"。将 mpd 文件与分片文件一同放在 Tomcat 服务器上，则可通过 HTTP 请求访问。

- 在 js 中创建 MediaPlayer。
  ```javascript
  player = dashjs.MediaPlayer().create();
  player.getDebug().setLogToBrowserConsole(false);

  // Alternative: player.initialize( [view] [, source] [, AutoPlay])
  player.initialize();
  player.attachView(document.getElementById("videoTag"));
  player.attachSource("http://114.212.84.179:8080/video/result.mpd");
  player.setAutoPlay(false);

  player.play()
  ```

- MediaPlayer 内置了很多事件，可以针对这些事件注册回调函数。
  ```javascript
  var events = dashjs.MediaPlayer.events;

  player.on(events['CAN_PLAY'], function(){
    console.log("CAN_PLAY");
    ...
  });

  player.on(events['BUFFER_EMPTY'], function(){
    console.log("BUFFER_EMPTY");
    ...
  });
  ```

- MediaPlayer 同样提供了一些统计信息，可以直接获取。
  ```javascript
  var videoQualities =
      player.getMetricsFor('video')
            .HttpList
            .filter(http => http.type == "MediaSegment")
            .map(http => http.url)
            .map(url => url.replace(/^.*[\\\/]/, ''))
            .map(file => file.split('-')[0])
            .map(Number);
  ```

	以上代码过滤所有分片请求，将其映射为视频文件，并获取相应的 RepresentationID，以供后续分析。

- 本地 js 跨域请求服务器内容时，会出现 Cross-Origin Request Blocked 错误，临时的解决方式是在 Tomcat 的 conf 文件夹下的 web.xml 中配置 filter，临时允许访问。
  ```xml
  <filter>
    <filter-name>CorsFilter</filter-name>
    <filter-class>org.apache.catalina.filters.CorsFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>CorsFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  ```

- js 中如果需要修改已有方法，可以定义新方法，将旧方法作为参数，对旧方法的参数进行修改后，再交由旧方法处理。比如需要拦截 GET 请求，并对 URL 做处理，可以如下实现。
  ```javascript
  (function(open) {
    XMLHttpRequest.prototype.open = function() {
      if(arguments[0] == 'GET'){
        arguments[1] = ...;
      }
      open.apply(this, arguments);
    };
  })(XMLHttpRequest.prototype.open);
  ```

### References

- [MPEG-DASH Content Generation with MP4Box and x264](https://bitmovin.com/mp4box-dash-content-generation-x264/) 给出了单个视频文件生成 DASH 内容的基本步骤。

- [How to encode multi-bitrate videos in MPEG-DASH for MSE based media players](https://tdngan.wordpress.com/2016/11/17/how-to-encode-multi-bitrate-videos-in-mpeg-dash-for-mse-based-media-players/) 给出了多文件版本。

- [Working with MP4Box]() 中提到了 -bs-switching 选项（为每一个Representation都生成起始文件，后续可以通过连接起始文件和分片文件生成可播放的视频）。

- [x264](https://www.videolan.org/developers/x264.html) 为 x264 官网。

- [General Documentation](https://gpac.wp.imt.fr/mp4box/mp4box-documentation/) 和 [DASH Support in MP4Box](https://gpac.wp.imt.fr/mp4box/dash/) 是 MP4Box 官方说明。

- [FFmpeg](http://ffmpeg.org/) 是 FFmpeg 官网。

- [Events Example](http://dashif.org/reference/players/javascript/nightly/dash.js/samples/getting-started-basic-embed/listening-to-events.html) 提供了处理 dash.js 事件的样例。

- [Class: MediaPlayerEvents](http://cdn.dashjs.org/latest/jsdoc/MediaPlayerEvents.html) dash.js 官方 API 之事件。

- [Module: MediaPlayer](http://cdn.dashjs.org/latest/jsdoc/module-MediaPlayer.html) dash.js 官方 API 之播放器。

- [Apache Tomcat 8 Configuration Reference](http://tomcat.apache.org/tomcat-8.5-doc/config/filter.html) 介绍了 CORS Filter。

- [How to set the allow-file-access-from-files flag option in Google Chrome on the Windows Operating System](http://www.chrome-allow-file-access-from-file.com/windows.html) Chrome 不支持本地请求的解决方法。

#### &#x1F4DA; [Posts](./)