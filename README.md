# 大屏图表前端开发案例

#### 演示地址：

http://e-art.top/projects/bigScreenCharts

##### 截图

![](https://img-blog.csdnimg.cn/20190621142010866.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Rhb2tlX2xp,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20190918220813794.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Rhb2tlX2xp,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20190918222401610.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Rhb2tlX2xp,size_16,color_FFFFFF,t_70)

#### 功能和特点

1. 页面内容适应各种比例的大屏。可以设置为自动等比例缩放(contain 和 cover 两种模式)，也可以设置为拉伸(stretch 模式)来铺满窗口。
2. 图表的配置项抽取各层级公用部分，以对象的合并方式(jQuery 的 extend 方法)实现最简配置。
3. 窗口缩放时图表自动缩放无需刷新，可通过拖动浏览器窗口大小测试。
4. 图表定时刷新重绘效果，可分别指定需要和不需要刷新的图表。这里仅是前端展示用，也可配合异步加载数据后重绘图表。
5. 页面显示时钟、城市定位和天气。
6. 除了常规图表，还引入 geoJson 和百度地图的两种平面地图模式、3D 地球模式以及若干有意思的算命娱乐小图表。其中地图上有唐僧取经路线和麦哲伦环球航线等等。后期还添加了学习计划执行图表。
7. 四季（春夏秋冬）颜色主题更换。
8. 配置面板，配置内容如下：
   1. 设计图尺寸
   2. 页面缩放模式
   3. 图表自动刷新频率
   4. 天气预报更新周期
   5. 指定颜色主题

#### 软件架构

- HTML5 结构
- css 采用[sass](https://www.sass.hk/)
- 布局采用 css3 中的[flex](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/CSS_layout/Flexbox)
- 部分 js 使用了 ES6+
- 图表采用[百度 echarts](https://echarts.apache.org/zh/index.html)
- 缩放原理是通过计算页面和原始尺寸的比例，控制 html 的 fontSize 来进行所有元素的尺寸缩放，元素尺寸单位用 rem。

#### 使用说明

1. 可以先配置设计图尺寸、缩放模式、刷新频率、天气预报更新周期、主题颜色等参数(在 common.js 里如下代码)，也可以在页面设置面板中配置，但后者只能存储到本地的 localStorage
   ```
   const Cfg = {
       designW: settings.designW || 1920, //设计图宽度
       designH: settings.designH || 1080, //设计图高度
       zoomMode: settings.zoomMode || (innerWidth < 768 ? 'cover' : 'contain'),
       notebookOptim: [undefined, true].includes(settings.notebookOptim),
       getWeatherPeriod: settings.getWeatherPeriod || 5, //天气预报更新周期（分）
       chartRefreshPeriod: settings.chartRefreshPeriod || 10, // 图表刷新周期（秒）
       colors: settings.colors || 'default',
       colorData: {
           default: ['lightskyblue', 'orange', 'greenyellow', 'limegreen',
               'mediumturquoise', 'mediumpurple'],
           spring: ['#BEDC6E', '#FA8C8C', '#FAAAC8', '#FAC8C8',
               '#FFFFE6', '#6E6464'],
           summer: ['#FFAE00', '#FF5200', '#007AFF', '#00BF05',
               '#DCFFFF', '#505064'],
           autumn: ['#c1ad2f',/*'#A5912D',*/ '#782323', '#783723', '#A05027',
               '#FAE6DC', '#283C14'],
           winter: ['#F5F5FA', '#96822D', '#6E5A19', '#BECDEB',
               '#E1E1F0', '#281E1E'],
       }
   };
   ```
   <!--尺寸用62.5%的HTML字号，即1rem=10px。-->
2. 开发过程中，遇到图表先用最简单必须的代码画出来，然后再根据设计图梳理一下，哪些配置是共性，哪些是个性，共性的就在 chartsCom.js 里修改，个性的就在图表实例下修改。
   ```
   let chart = echarts.init($("#ec01_line_tiobe")[0]); //初始化图表，注意命名的规范合理
   this.charts.ec01_line_tiobe = chart; //放入charts对象方便后面的刷新缩放以及其他操作
   chart.setOption(opt_line); // 设置这个类型（折线图）图表的共性
   chart.setOption({
       xAxis: { // 本图表option的个性
           nameLocation: 'start',
           inverse: true,
           data: ['2019', '2014', '2009', '2004', '1999', '1994', '1989']
       },
       yAxis: { // 本图表option的个性
           name: '排名',
           nameLocation: 'start',
           min: 1,
           inverse: true
       },
       dataZoom: { // 本图表option的个性
           type: 'inside',
           orient: 'vertical'
       },
       series: [
           {"name": "Java", data: [1, 2, 1, 1, 12, '-', 0]},
           {"name": "C", data: [2, 1, 2, 2, 1, 1, 1]},
           {"name": "C++", data: [3, 4, 3, 3, 2, 2, 3]},
           ...
       ].map(item => {
           return $.extend(true, {}, seri_line,// 折线图图表series的共性
                { // 本图表series的个性
                   smooth: false,
                   showSymbol: false,
           }, item)
       })
   })
   ```
3. css 中尺寸单位用 rem，js 中图表配置里的长度数值都要乘以全局变量 scale。
4. chrome 浏览器中为了阅读清晰，将小于 12px 的汉字字号限制到 12px，这样可能造成字号没有随页面等比例变化，可能撑坏部分布局。解除最小字号限制的方法：浏览器设置-外观-自定义字体-最小字号(可能因版本不同路径有差别，可以在设置面板中搜索“最小字号”)，将滚动条拉到最小即可。
5. chrome 浏览器模拟移动端界面的方法：按 F12 打开控制台，点击左上角选取元素旁边的移动端图标，然后页面就被放在一个特定尺寸的容器中。可以看到设备列表中已经预置了一些手机型号，点击下面的“Edit...”，在控制台点击“add custom device”按钮添加设备并指定尺寸，在大屏上一般将默认的类型 Mobile 改为 Desktop。注意，如果模拟电脑屏幕正常访问页面的情况，则高度要除去标签栏、地址栏、收藏夹等，具体尺寸可在控制台 console 下输入 window.innerHeight 即可看到。
6. 直接用浏览器打开却看不到顶部的天气、定位以及左侧设置面板，这是由于这些内容用了 jQuery 的异步请求($.get()、.load()）方法，而这些方法要在运行在服务器上才能实现。如果本地开发，需要在编辑器里配上服务器再运行，如果部署，可以把文件直接放到 tomcat 这样的容器里即可。
7. Have a nice day! 🍵
