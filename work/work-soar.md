# soar-App

1.app分为java-app和python-app，java-app的配置文件存放在/config/soar-connector/apps下面的各json配置文件当中，python-app存放在{安装目录}/containers/app/apps下的json配置文件当中（生产模式下），非生产模式下存放位置见python_app_config.properties文件中

2.存储说明

    soar_app_order存放app是否置顶和排列顺序

    soar_app_asset app的实例相关文件

    app的动作相关是写在1中的app的配置文件当中

3.动作执行流程

    3.1动作执行分为同步和异步

    3.2先创建ActionRecord和Activity记录

    3.3执行动作：选取可用节点-

2.ActionRecord和Activity

执行方法

ActionRecord ar = AppManager.me().runActionAsync（示例id，动作id，参数，上下文）

WebSocketHandler

关于netty的使用：SoarWarroomService

javascript: $axure.utils.loadJS('https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js'); setTimeout(function(){     var dom =$('[data-label=Charts]').get(0);     var Chart = echarts.init(dom);      var option = {       xAxis: {     type: 'category',     data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']   },   yAxis: {     type: 'value'   },   series: [     {       data: [150, 230, 224, 218, 135, 147, 260],       type: 'line'     }   ] };      if (option && typeof option === "object"){        Chart.setOption(option, true);     }}, 1000); 

qaxcmd --get-dbpa

DbConfigDecoder
