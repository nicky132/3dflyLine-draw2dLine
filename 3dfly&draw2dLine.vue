<template>
  <div class="home">
    <div class="contentPart" id="contentPart">
      <div
        :ref="'dataSummaryDom' + (index + 1)"
        class="cell active"
        v-for="(item, index) in 24"
        @click="() => openDataSummaryPop(item)"
        :key="index + 1"
      >
        <div v-text="item"></div>
      </div>
    </div>
  </div>
</template>
<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { Line2 } from "three/examples/jsm/lines/Line2.js";
import { LineMaterial } from "three/examples/jsm/lines/LineMaterial.js";
import { LineGeometry } from "three/examples/jsm/lines/LineGeometry.js";
const positions = [];
export default {
  name: "home",
  components: {},
  methods: {
    open() {},
    close() {},
    functiongGetRect(element) {
      var rect = element.getBoundingClientRect();
      var top = document.documentElement.clientTop;
      var left = document.documentElement.clientLeft;
      return {
        top: rect.top - top,
        bottom: rect.bottom - top,
        left: rect.left - left,
        right: rect.right - left,
        rect: rect,
        width: rect.width,
        height: rect.height,
      };
    },
    getDotCoord(camera, dom, targetZ = 0) {
      const scrollLeft =
        window.pageXOffset || document.documentElement.scrollLeft;
      const scrollTop =
        window.pageYOffset || document.documentElement.scrollTop;
      const zoomLevel = window.devicePixelRatio;
      const x =
        this.functiongGetRect(dom).top +
        this.functiongGetRect(dom).width / 2 +
        scrollLeft; //坐标X
      const y =
        this.functiongGetRect(dom).left +
        this.functiongGetRect(dom).height / 2 +
        scrollTop; //坐标Y
      // 屏幕坐标转标准设备坐标
      const x1 = (x / window.innerWidth) * zoomLevel * 2 - 1;
      const y1 = -((y / window.innerHeight) * zoomLevel) * 2 + 1;
      const stdVector = new THREE.Vector3(x1, y1, 0.5);
      var pos = new THREE.Vector3();
      const worldVector = stdVector.unproject(camera);
      worldVector.sub(camera.position).normalize();

      var distance = (targetZ - camera.position.z) / worldVector.z;

      pos.copy(camera.position).add(worldVector.multiplyScalar(distance));
      return pos;
    },
    _getFrameDomCenterPos() {
      let frameList = [
        this.$refs.dataSummaryDom1[0],
        this.$refs.dataSummaryDom8[0],
      ];
      return [].map.call(frameList, (item) => {
        let rect = item.getBoundingClientRect();
        let xPos = rect.left + rect.width / 2;
        let yPos = rect.top + rect.height / 2;
        console.log("item:::", xPos, yPos);
        return {
          x: xPos,
          y: yPos,
        };
      });
    },
    _get3DPosByCanvasPos(pos, targetZ = 0) {
      const x1 = (pos.x / container.clientWidth) * 2 - 1;
      const y1 = -(pos.y / container.clientHeight) * 2 + 1;
      const stdVector = new THREE.Vector3(x1, y1, 0.5);
      var pos = new THREE.Vector3();
      const worldVector = stdVector.unproject(camera);
      worldVector.sub(camera.position).normalize();

      var distance = (targetZ - camera.position.z) / worldVector.z;

      pos.copy(camera.position).add(worldVector.multiplyScalar(distance));
      return pos;
    },
    openDataSummaryPop(item) {
      alert(item);
    },
    init() {
      container = document.getElementById("contentPart");
      width = document.getElementById("contentPart").clientWidth;
      height = document.getElementById("contentPart").clientHeight;
      // 场景
    },
    animate() {
      requestAnimationFrame(this.animate);
      this.next = 0;
      this.render();
    },
    render() {
      this.next += 0.01;
      ratio.value = this.next;
      camera.updateMatrixWorld();
      requestAnimationFrame(this.render);
      renderer.render(scene, camera);
    },
  },
  mounted() {
    this.$nextTick(() => {
      let that = this;
      let camera, scene, renderer;
      let stats = null;
      let matLine;
      const container = document.getElementById("contentPart");
      let ratio = {
        value: 0,
      };
      const aspect = container.clientWidth / container.clientHeight;
      const frustumSize = 100;
      function init() {
        camera = new THREE.PerspectiveCamera(
          45,
          container.clientWidth / container.clientHeight,
          0.1,
          10000
        );
        camera.position.set(0, 0, 260);
        camera.up.set(0, 0, 1);
        scene = new THREE.Scene();
        let helper = new THREE.AxesHelper(100, 100);
        scene.add(helper);
        // scene.background = new THREE.Color(0xeeeeee);

        var ambientLight = new THREE.AmbientLight(0xffffff, 1);
        scene.add(ambientLight);

        const geometry = new THREE.BoxGeometry(1, 1, 1);
        const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
        const cube = new THREE.Mesh(geometry, material);

        renderer = new THREE.WebGLRenderer({ alpha: true });
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        renderer.setSize(container.clientWidth, container.clientHeight);
        renderer.toneMapping = THREE.ACESFilmicToneMapping;
        renderer.toneMappingExposure = 1;
        renderer.outputEncoding = THREE.sRGBEncoding;
        container.appendChild(renderer.domElement);
        document.getElementById("contentPart").appendChild(renderer.domElement);
        document.querySelector("#contentPart canvas").style.position =
          "absolute";
        document.querySelector("#contentPart canvas").style.zIndex = 1;
        document.querySelector("#contentPart canvas").style.pointerEvents =
          "none";
        const controls = new OrbitControls(camera, renderer.domElement);

        let lineMesh = createLine();
        scene.add(lineMesh);

        flyLine();
        window.addEventListener("resize", onWindowResize, false);
      }

      const flyLine = () => {
        console.log("positionspositions=", positions);
        this.flyData = [
          {
            start: {
              //起始点位置
              x: positions[0],
              y: positions[1],
              z: positions[2],
            },
            end: {
              // 结束点位置
              x: positions[3],
              y: positions[4],
              z: positions[5],
            },
            range: 180, // 飞线长度
            height: 120, // 轨迹的高度
            color: "#7D84FF", // 颜色
            speed: 2, // 速度
            size: 10, // 飞线点点的大小
          },
        ];
        const vertexShader = `
              // 接收js传入的attribute值，会经过线性插值
              attribute float current;
  
              // 接收js传入的uniform值
              uniform float uSize;
              uniform float uTime;
              uniform vec3 uColor;
              uniform float uRange;
              uniform float uTotal;
              uniform float uSpeed;
  
              // 向片元着色器传值颜色和透明度
              varying vec3 vcolor;
              varying float vopacity;
  
              void main () {
                  float size = uSize;
                  // 根据时间确定当前飞线的位置， 以结束点为准
                  float currentEnd = uTotal * fract(uTime * uSpeed);
                  // 判断当前像素点是否在飞线范围内，如果在范围内设置尺寸和透明度
                  if (current < currentEnd && current > currentEnd - uRange) {
                      // 设置渐变的尺寸，头大尾小
                      float sizePct = (uRange - (currentEnd - current)) / uRange;
                      size *= sizePct;
                      vopacity = 1.0;
                  } else {
                      vopacity = 0.0;
                  }
                  // 将颜色传递给片元着色器
                  vcolor = uColor;
                  // 设置点的大小
                  gl_PointSize = size * 0.4;
                  gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
              }
          `;
        const fragmentShader = `
              precision mediump float;
              // 接收顶点着色器传入的值
              varying float vopacity;
              varying vec3 vcolor;
  
              void main () {
                  // 设置颜色
                  gl_FragColor = vec4(vcolor, vopacity);
              }
          `;
        // 创建飞线
        this.flyData.forEach((item) => {
          initFly(item);
        });
        function initFly(options) {
          // 接收传入的参数
          const { start, end, height, size, color, range, speed } = options;
          // 起点位置
          const _start = new THREE.Vector3(start.x, start.y, start.z);
          // 结束点位置
          const _end = new THREE.Vector3(end.x, end.y, end.z);
          // 计算中建点位置
          /* lerp方法的用法： 在该向量与传入的向量v之间的线性插值，
              alpha是沿着线的长度的百分比 —— alpha = 0 时表示的是当前向量，
              alpha = 1 时表示的是所传入的向量v。
              此处传入 0.5 返回得是中间点位置
          **/
          const _center = _start.clone().lerp(_end, 0.5);
          // 把中间点的位置沿着y轴方向向上移动 height 距离
          _center.y += height;
          // 计算起点到终点间点的个数
          const number = parseInt(
            _start.distanceTo(_center) + _end.distanceTo(_center)
          );
          // 创建一条平滑的三维 二次贝塞尔曲线， 由起点、终点和一个控制点所定义
          const curve = new THREE.QuadraticBezierCurve3(_start, _center, _end);
          // 将curve分成 number-1 段；得到number个点；存入数组points 中
          const points = curve.getPoints(number);
          // 申明变量用于存点的信息
          const positions = [];
          // 申明变量用于存点的索引信息
          const current = [];
          // 遍历点数组，将索引存入current， 将点的x,y,z展开，存入positions数组
          points.forEach((item, index) => {
            positions.push(item.x, item.y, item.z);
            current.push(index);
          });
          // 创建BufferGeometry 并把positions和current传给对应的attribute属性，供顶点着色器使用
          const geometry = new THREE.BufferGeometry();
          geometry.setAttribute(
            "position",
            new THREE.Float32BufferAttribute(positions, 3)
          );
          geometry.setAttribute(
            "current",
            new THREE.Float32BufferAttribute(current, 1)
          );
          // 创建shader材质
          const material = new THREE.ShaderMaterial({
            transparent: true,
            depthWrite: false,
            depthTest: false,
            blending: THREE.AdditiveBlending,
            uniforms: {
              uSize: {
                // 点的大小
                value: size,
              },
              uTime: ratio, // 时间
              uColor: {
                // 颜色
                value: new THREE.Color(color),
              },
              uRange: {
                // 飞线长度
                value: range,
              },
              uTotal: {
                // 轨迹总长度，（点的总个数）
                value: number,
              },
              uSpeed: {
                // 飞行速度
                value: speed,
              },
            },
            vertexShader,
            fragmentShader,
          });
          // 创建并添加到场景中
          const flyPoints = new THREE.Points(geometry, material);
          scene.add(flyPoints);
        }
      };

      function get3DPosByCanvasPos(pos, targetZ = 0) {
        const x1 = (pos.x / container.clientWidth) * 2 - 1;
        const y1 = -(pos.y / container.clientHeight) * 2 + 1;
        const stdVector = new THREE.Vector3(x1, y1, 0.5);
        var pos = new THREE.Vector3();
        const worldVector = stdVector.unproject(camera);
        worldVector.sub(camera.position).normalize();

        var distance = (targetZ - camera.position.z) / worldVector.z;

        pos.copy(camera.position).add(worldVector.multiplyScalar(distance));
        return pos;
      }

      function createLine() {
        let domList = getFrameDomCenterPos();
        const colors = [];
        domList.forEach((posItem) => {
          let pos = get3DPosByCanvasPos(posItem);
          positions.push(pos.x, pos.y, pos.z);
          colors.push(1, 0, 0);
        });
        console.log("positions:::", positions);
        // positions.push(-5, 0, 0, 5, 0, 0);
        // colors.push(1, 0, 0, 1, 0, 0);
        const geometry = new LineGeometry();
        geometry.setPositions(positions);
        geometry.setColors(colors);
        matLine = new LineMaterial({
          color: 0xffffff,
          linewidth: 5, // in world units with size attenuation, pixels otherwise
          vertexColors: true,

          //resolution:  // to be set by renderer, eventually
          dashed: false,
          alphaToCoverage: true,
        });
        let line = new Line2(geometry, matLine);
        line.computeLineDistances();
        line.scale.set(1, 1, 1);
        return line;
      }

      function getFrameDomCenterPos() {
        let frameList = [
          that.$refs.dataSummaryDom8[0],
          that.$refs.dataSummaryDom10[0],
        ];
        return [].map.call(frameList, (item) => {
          let rect = item.getBoundingClientRect();
          let xPos = rect.left + rect.width / 2;
          let yPos = rect.top + rect.height / 2;
          console.log("item:::", xPos, yPos);
          return {
            x: xPos,
            y: yPos,
          };
        });
      }
      function onWindowResize() {
        camera.aspect = container.clientWidth / container.clientHeight;
        camera.updateProjectionMatrix();

        renderer.setSize(container.clientWidth, container.clientHeight);

        // render()
      }

      //
      this.next = 0;
      const animate = () => {
        this.next += 0.01;
        ratio.value = this.next;
        requestAnimationFrame(animate);
        if (matLine) {
          matLine.resolution.set(container.clientWidth, container.clientHeight); // resolution of the inset viewport
        }
        render();
      };

      function render() {
        camera.updateMatrixWorld();
        renderer.render(scene, camera);
        // stats.update();
      }
      init();
      animate();
    });
  },
};
</script>
<style lang="less" scoped>
.home {
  position: fixed;
  top: 0;
  width: 100%;
  left: 0;
  overflow: hidden;
  height: 100%;

  .contentPart {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    padding: 0 3px;
    position: relative;

    canvas {
      left: 0;
      top: 0;
      z-index: 1;
      position: absolute !important;
      width: 100% !important;
      height: 100% !important;
    }

    #mainCanvas {
      width: 100%;
      height: 100%;
    }

    // canvas,
    // #mainCanvas {
    //   position: absolute;
    //   top: 0;
    //   left: 0;
    //   width: 100%;
    //   height: 100%;
    //   // height: 338px;
    //   z-index: 1;
    //   pointer-events: none;
    //   canvas{
    //     width: 100% !important;
    //     height: 100% !important;
    //   }
    // }
    .cell {
      display: flex;
      flex-direction: column;
      flex-grow: 0;
      flex-shrink: 0;
      margin-bottom: 1vw;
      flex-basis: calc(25% - 3px);
      font: 0.9rem Arial;
      color: #4e5464;
      font-size: 14px;
      background: #f7f8fb;
      padding-left: 12px;
      padding-right: 12px;
      font-size: 14px;
      height: 36px;
      line-height: 36px;
      border-radius: 4px 4px 4px 4px;
      padding: 3px;
    }

    .cell.active {
      background: #dfe0fd;
      color: #5056c4;
    }
  }

  #contentPart {
    canvas {
      left: 0;
      top: 0;
      z-index: 1;
      position: absolute !important;
      width: 100% !important;
      height: 100% !important;
    }
  }
}
</style>
