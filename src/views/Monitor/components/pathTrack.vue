<template>
  <div class="container">
    <div id="container" style="height:100%;width:100%"></div>
    <div class="progress-wrap" v-if="lineArr.length>0&&navg">
      <div class="pic-wrap">
        <img src="../images/start.png" v-if="progStatus==='开始'" @click="startAnimation()" alt="">
        <img src="../images/pause.png" v-if="progStatus==='暂停'" @click="pauseAnimation()" alt="">
        <img src="../images/start.png" v-if="progStatus==='继续'" @click="resumeAnimation()" alt="">
        <img src="../images/stop.png" @click="stopAnimation()" alt="">
      </div>
      <div class="progress">
        <div class="percentage" :style="{width:percentage+'%'}"></div>
      </div>
      <div class="pic-wrap" style="width:150px">
        <el-button type="primary" @click="setSpeed('加速')">加速</el-button>
        <el-button type="primary" @click="setSpeed('减速')">减速</el-button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    lineArr: {
      type: Array
    }
  },
  components: {},
  data () {
    return {
      carImg: require('../images/car.png'),
      percentage: 0,
      progStatus: '开始',
      markerSpeed: 100,
      navg: null
    }
  },
  mounted () {
    setTimeout(() => {
      this.initMap()
    }, 800)
  },
  methods: {
    // 初始化轨迹
    initroad () {
      let that = this
      this.progStatus = '开始'
      this.percentage = 0
      this.markerSpeed = 100
      AMapUI.load(['ui/misc/PathSimplifier', 'lib/$'], function (PathSimplifier, $) {
        if (!PathSimplifier.supportCanvas) {
          alert('当前环境不支持 Canvas！')
          return
        }
        if (that.pathSimplifierIns) {
          that.pathSimplifierIns.setData([])
        }
        // 创建一个巡航轨迹路线
        that.pathSimplifierIns = new PathSimplifier({
          zIndex: 100, // 地图层级，
          map: that.map, // 所属的地图实例
          // 巡航路线轨迹列表
          getPath: function (pathData, pathIndex) {
            return pathData.path
          },
          // hover每一个轨迹点，展示内容
          getHoverTitle: function (pathData, pathIndex, pointIndex) {
            if (pointIndex >= 0) {
              return pathData.name + '，点：' + pointIndex + '/' + pathData.path.length
            }
            return pathData.name + '，点数量' + pathData.path.length
          },
          // 自定义样式，可设置巡航器样式，巡航轨迹样式，巡航轨迹点击、hover等不同状态下的样式，不设置则用默认样式，详情请参考api文档
          // 绘制路线节点
          renderOptions: {
            renderAllPointsIfNumberBelow: 100 // 绘制路线节点，如不需要可设置为-1
          }
        })
        // 设置数据
        that.pathSimplifierIns.setData([{
          name: '路线',
          path: that.lineArr
        }])
        var navg = that.pathSimplifierIns.createPathNavigator(0, {
          loop: false, // 循环播放
          speed: that.markerSpeed, // 巡航速度，单位千米/小时
          pathNavigatorStyle: {
            width: 26,
            height: 45,
            // 使用图片
            content: PathSimplifier.Render.Canvas.getImageContent(that.carImg, onload, onerror),
            // strokeStyle: null,
            // fillStyle: null,
            // 经过路径的样式
            pathLinePassedStyle: {
              lineWidth: 6,
              strokeStyle: 'black',
              dirArrowStyle: {
                stepSpace: 15,
                strokeStyle: 'red'
              }
            }
          }
        })
        navg.on('move', function (a) {
          let percentage = 0
          let { idx, tail } = navg.getCursor()
          let len = a.target.path.length
          if (navg.isCursorAtPathEnd()) {
            // 收尾处理
            that.progStatus = '开始'
            that.markerSpeed = 100
            navg.stop()
            that.percentage = 100
          } else {
            percentage = ((idx + tail) / len) * 100
            that.percentage = percentage
          }
        })
        that.navg = navg
      })
    },
    initMap () {
      this.map = new AMap.Map('container', {
        expandZoomRange: true,
        resizeEnable: true, // 窗口大小调整
        center: this.firstArr, // 中心 firstArr: [116.478935, 39.997761],
        zoom: 13
      })
    },
    startAnimation () {
      this.navg.start()
      this.progStatus = '暂停'
      // this.marker.moveAlong(this.lineArr, this.markerSpeed)
    },
    pauseAnimation () {
      this.progStatus = '继续'
      this.navg.pause()
    },
    resumeAnimation () {
      this.progStatus = '暂停'
      this.navg.resume()
    },
    stopAnimation () {
      this.percentage = 0
      this.progStatus = '开始'
      this.markerSpeed = 100
      this.navg.stop()
      this.initroad()
    },
    // 调速
    setSpeed (status) {
      if (this.navg === null) return
      if (status === '加速') {
        if (this.markerSpeed >= 300) return
        this.markerSpeed += 50
      } else if (status === '减速') {
        if (this.markerSpeed === 0) return
        this.markerSpeed -= 50
      }
      this.navg.setSpeed(this.markerSpeed)
    },
    toggleSelection (rows) {
      console.log(rows)
    },
    handleClick (tab, event) {
      console.log(tab, event)
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  width: 100%;
  height: 100%;
  position: relative;
  .progress-wrap {
    position: absolute;
    bottom: 30px;
    right: -20px;
    z-index: 99999;
    width: 100%;
    padding: 10px;
    background: #fff;
    display: flex;
    align-items: center;
    .pic-wrap {
      width: 100px;
      margin: 0 5px;
    }
    .progress {
      flex: 1;
      height: 6px;
      background: #ebeef5;
      border-radius: 50px;
      .percentage {
        height: 100%;
        border-radius: 50px;
        background: #409eff;
      }
    }
  }
}
</style>
