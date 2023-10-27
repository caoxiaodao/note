# soar-App执行

执行方法

ActionRecord ar = AppManager.me().runActionAsync（示例id，动作id，参数，上下文）

WebSocketHandler

关于netty的使用：SoarWarroomService

javascript: $axure.utils.loadJS('https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js'); setTimeout(function(){     var dom =$('[data-label=Charts]').get(0);     var Chart = echarts.init(dom);      var option = {       xAxis: {     type: 'category',     data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']   },   yAxis: {     type: 'value'   },   series: [     {       data: [150, 230, 224, 218, 135, 147, 260],       type: 'line'     }   ] };      if (option && typeof option === "object"){        Chart.setOption(option, true);     }}, 1000); 
