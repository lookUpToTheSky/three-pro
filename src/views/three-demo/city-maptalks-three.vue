<template>
  <div class="home">
    <div id="home-view" ref="views"></div>
    <div class="back">
        <el-button @click="$router.go(-1)" type="primary" size="small">返 回</el-button>
    </div>
    <!-- <ul class="road-line">
      <li v-for="(val, i) in roadLine" :key="i" @click="bySubway(val)">{{val.name}}</li>
    </ul> -->
  </div>
</template>

<script>
import * as THREE from 'three'
import * as maptalks from 'maptalks'
import 'maptalks/dist/maptalks.css'
import { ThreeLayer } from 'maptalks.three'
import { CSS3DRenderer, CSS3DObject } from 'three/examples/jsm/renderers/CSS3DRenderer.js'
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer' 
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass' 
import { SMAAPass } from 'three/examples/jsm/postprocessing/SMAAPass' 
import { SMAAEdgesShader, SMAAWeightsShader, SMAABlendShader } from 'three/examples/jsm/shaders/SMAAShader' 
import { UnrealBloomPass } from '@/utils/UnrealBloomPass' 
import TWEEN from 'tween'
import Effect from '@/utils/effect.js'
const vertexShader = `
    varying vec3 vPosition;
    varying vec2 vUv;
    void main() { 
        vUv = uv; 
        vec4 mvPosition = modelViewMatrix * vec4(position, 1.0);
        gl_Position = projectionMatrix * mvPosition;
        vPosition = position;
    }
`;
let map, threeLayer;
let render1, clock = new THREE.Clock();
var effectList = [];
export default {
  data() {
    return {
      ruleForm: {},
      views: null,
      animateId: null,
      roadLine: [],
      lineList: [
        {start: {x: 112.97743006478095, y: 28.18176594058977}, end: {x: 112.95704952246206, y: 28.199260443145675}},
        {start: {x: 112.95704952246206, y: 28.199260443145675}, end: {x: 112.95704952246206, y: 28.199260443145675}},
        {start: {x: 112.98676346381774, y: 28.198138248548986}, end: {x: 112.95704952246206, y: 28.199260443145675}},
        {start: {x: 112.97080458036419, y: 28.193877847579728}, end: {x: 112.95704952246206, y: 28.199260443145675}},
        {start: {x: 112.92659619609526, y: 28.189800731714655}, end: {x: 112.95704952246206, y: 28.199260443145675}},
        {start: {x: 112.89778440871657, y: 28.165823912185743}, end: {x: 112.95704952246206, y: 28.199260443145675}},
        {start: {x: 112.97528077816185, y: 28.23863326663448}, end: {x: 112.95704952246206, y: 28.199260443145675}},
        {start: {x: 112.9631741820981, y: 28.168197563829466}, end: {x: 112.95704952246206, y: 28.199260443145675}}
      ]
    }
  },
  methods: {
    createGeometry({geometry, position,rotation, effect, color, color1, wireframe}) {
        position = threeLayer.coordinateToVector3(position) || new THREE.Vector3(0,0,0)
        rotation = rotation || new THREE.Vector3(-Math.PI/2,0,0)
        let material = null
        if(!!effect) {
          material = new THREE.ShaderMaterial({
                uniforms: {
                    iTime: {
                        value: 0
                    },
                    time: {
                      value: 0
                    },
                    color: { value: color || new THREE.Color(0xff0000) },
                    color1: { value: color1 || new THREE.Color(0x00ffff) },
                    iResolution: {
                        value: new THREE.Vector2(800, 800)
                    },
                    map: {
                      value: new THREE.ImageUtils.loadTexture('./images/smokeparticle.png')
                    }
                },
                side: 2,
                depthWrite: true,
                transparent: true,
                depthTest: false,
                wireframe: wireframe || false,
                blending: THREE.AdditiveBlending,
                vertexShader: vertexShader,
                fragmentShader: effect()
          })
        }else{
           position.z = 5
           material = new THREE.MeshBasicMaterial({
                color: color || new THREE.Color(0x00ffff),
                side: 2,
                transparent: true,
                depthTest: false,
                wireframe: wireframe || false,
                blending: THREE.AdditiveBlending,
          })
        }
        let mesh = new THREE.Mesh(geometry, material);
        mesh.position.copy(position);
        mesh.rotation.set(rotation.x, rotation.y, rotation.z)
        threeLayer.addMesh(mesh)
        return mesh
    },
    createEffect() {
      const circle1 = this.createGeometry({
        geometry: new THREE.CircleGeometry(10, 32, 32),
        effect: Effect.effect44,
        color: new THREE.Color(0xff0000),
        position: {x: 112.97743006478095, y: 28.18176594058977},
        rotation: {x: 0, y: 0, z: 0 }
      })

      const circle2 = this.createGeometry({
        geometry: new THREE.CircleGeometry(10, 32, 32),
        effect: Effect.effect35,
        color: new THREE.Color(0xff00ff),
        position: {x: 112.95704952246206, y: 28.199260443145675},
        rotation: {x: 0, y: 0, z: 0 },
        wireframe: false
      })

      const circle3 = this.createGeometry({
        geometry: new THREE.CircleGeometry(10, 32, 32),
        effect: Effect.effect1,
        color: new THREE.Color(0xff00ff),
        position: { x: 112.97080458036419, y: 28.193877847579728 },
        rotation: {x: 0, y: 0, z: 0 },
        wireframe: false
      })

      const sphere = this.createGeometry({
        geometry: new THREE.SphereGeometry(10, 32, 32, 0, Math.PI*2, 0, Math.PI/2),
        effect: Effect.effect39,
        color: new THREE.Color(0x00ffff),
        position: {x: 112.98676346381774, y: 28.198138248548986},
        rotation: {x: Math.PI/2, y: 0, z: 0 }
      })
      
      const Cylinder = this.createGeometry({
        geometry: new THREE.CylinderBufferGeometry( 10, 10, 5, 32 , 32, true),
        effect: Effect.effect42,
        color: new THREE.Color(0xffff00),
        position: {x: 112.92659619609526, y: 28.189800731714655},
        rotation: {x: Math.PI/2, y: 0, z: 0 }
      })
      effectList = [circle1,circle2,circle3,sphere,Cylinder]
      let tweenA = new TWEEN.Tween(Cylinder.scale).to({x: 2, y: 1, z: 2}, 2000)
      tweenA.start()
      tweenA.onComplete(() => {
        Cylinder.scale.set(0.1, 0.1, 0.1)
        tweenA.start()
      })
    },
    createPointGeo() {
        const geometryList = [
          { 
            geometry: new THREE.ConeGeometry( 2, 4, 4, 1, true),
            color: new THREE.Color(0xffff00),
            coordinate: {x: 112.97743006478095, y: 28.18176594058977},
            time: 1000,
            text: '侯家塘'
          },
          { 
            geometry: new THREE.ConeGeometry( 2, 4, 4, 1, true),
            color: new THREE.Color(0xff0000),
            coordinate: {x: 112.95704952246206, y: 28.199260443145675},
            time: 1500,
            text: '橘子洲'
          },
          { 
            geometry: new THREE.ConeGeometry( 2, 4, 4, 1, true),
            color: new THREE.Color(0xff00ff),
            coordinate: {x: 112.98676346381774, y: 28.198138248548986},
            time: 1200,
            text: '五一广场'
          },
          { 
            geometry: new THREE.ConeGeometry( 2, 4, 4, 1, true),
            color: new THREE.Color(0xddff00),
            coordinate: {x: 112.97080458036419, y: 28.193877847579728},
            time: 1000,
            text: '黄兴广场'
          },
          { 
            geometry: new THREE.ConeGeometry( 2, 4, 4, 1, true),
            color: new THREE.Color(0x00ff00),
            coordinate: {x: 112.92659619609526, y: 28.189800731714655},
            time: 800,
            text: '岳麓山'
          }
        ]
        geometryList.forEach(item => {
          const { geometry, color, coordinate, time, wireframe, rotation, text} = item
          // 八面体
          const eightGeo = this.createGeometry({
            geometry,
            color,
            position: coordinate,
            wireframe,
            rotation
          })
          this.createText(text, coordinate)
          let T1 = new TWEEN.Tween(eightGeo.position).to({ z: 6 }, time)
          let T2 = new TWEEN.Tween(eightGeo.position).to({ z: 5 }, time)
          T1.start()
          T1.chain(T2)
          T2.chain(T1)
          
          let T3 = new TWEEN.Tween(eightGeo.rotation).to({ y: Math.PI*2 }, time*5)
          let T4 = new TWEEN.Tween(eightGeo.rotation).to({ y: 0 }, time*5)
          T3.start()
          T3.chain(T4)
          T4.chain(T3)
        })
        
    },
    // 文本
    createText(text, position) {
        position = threeLayer.coordinateToVector3(position)
        let element = document.createElement('div')
        element.className = 'info-wrap'
        element.innerText = text
        let obj = new CSS3DObject(element)
        position.z = 10
        obj.position.copy(position);
        obj.rotateX(Math.PI/2)
        obj.scale.set(0.1, 0.1, 0.1)
        threeLayer.addMesh(obj)
    },
    createLineFly() {
      // 飞线
      this.lineList.forEach( (val, index) => {
        let start = threeLayer.coordinateToVector3(val.start)
        let end = threeLayer.coordinateToVector3(val.end)
        start = new THREE.Vector3(start.x, start.y, start.z )
        end = new THREE.Vector3(end.x, end.y, end.z )
        const { curve, mesh } = this.addLines(start, end)
        this.flyLine(curve) 
        threeLayer.addMesh(mesh)
      })
    },
    // 曲线
    addLines( v0, v3 ) {
        var path = new THREE.LineCurve3(v0, v3)
        var p = path.getPoints(120)
        var d = v0.distanceTo( v3 )/3
        var v1 = new THREE.Vector3( p[40].x, p[40].y, p[40].z  + d)
        var v2 = new THREE.Vector3( p[80].x, p[80].y, p[80].z  + d)

        // 绘制三维三次贝赛尔曲线
        var curve = new THREE.CubicBezierCurve3( v0, v1, v2, v3 );
        var geometry = new THREE.BufferGeometry();
        var points = curve.getPoints( 150 );
        var positions = [];
        var colors = [], index = [];
        var color = new THREE.Color();
        /**
         * HSL中使用渐变
         * h — hue value between 0.0 and 1.0
         * s — 饱和度 between 0.0 and 1.0
         * l — 亮度 between 0.0 and 1.0
         */
        for (var j = 0; j < points.length; j ++) {
            // color.setHSL( .31666+j*0.005,0.7, 0.7); //绿色
            color.setHSL( .81666+j*0.0025, 0.88, 0.88); //粉色
            colors.push( color.r, color.g, color.b );
            positions.push( points[j].x, points[j].y, points[j].z );
            index.push(j/(points.length - 1))
        }
        geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))
        geometry.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3))
        geometry.setAttribute('index', new THREE.Float32BufferAttribute(index, 1))
        var matLine = new THREE.LineBasicMaterial( {
            vertexColors: true,
            transparent: true,
            dashed: false
        } );
        return {
          curve,
          mesh: new THREE.Line( geometry, matLine )
        }
    },
    // 路径飞线
    flyLine(curve) {
      var positions = [], index = [], current = [];
      let points = curve.getPoints(5000)
      const geometry = new THREE.BufferGeometry().setFromPoints(points)

      for (var j = 0; j < points.length; j++) {
          positions.push( points[j].x, points[j].y, points[j].z );
          index.push(j/(points.length - 1))
          current.push(j)
      }
      geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))
      geometry.setAttribute('index', new THREE.Float32BufferAttribute(index, 1))
      geometry.setAttribute('current', new THREE.Float32BufferAttribute(current, 1))
      const material = new THREE.ShaderMaterial({
        uniforms: {
          iTime: { value: 0 },
          size: { value: 200 },
          uRange: { value: 400 },
          uTotal: { value: points.length},
          uColor: { value: new THREE.Color(0x00ffff) },
          uColor1: { value: new THREE.Color(0xff00ff) }
        },
        transparent: true,
        depthTest: true,
        side: 2,
        vertexShader: `
        attribute float index;
        attribute float current;
        uniform float size;
        uniform vec3 uColor;
        uniform vec3 uColor1;
        uniform float uRange;
        uniform float uTotal;
        uniform float iTime;
        varying vec3 vColor;
        varying float vOpacity;
        void main() {
           // 需要当前显示的索引  
            float showNumber = uTotal * mod(iTime/2.0, 1.0);
            float s = size;
            if (showNumber > current && showNumber < current + uRange) {
                vOpacity = 1.0;
                s *= (current + uRange - showNumber) / uRange;
            } else {
                vOpacity = 0.0;
                s = 0.0;
            }
            // 顶点着色器计算后的Position
            // mix 混淆颜色
            vColor = mix(uColor, uColor1, index);
            vec4 mvPosition = modelViewMatrix * vec4(position, 1.0);
            gl_Position = projectionMatrix * mvPosition; 
            // 大小
            gl_PointSize = s* 5.0 / (-mvPosition.z);
        }`,
        fragmentShader: `
        uniform vec3 uColor;
        uniform vec3 uColor1;
        varying vec3 vColor; 
        varying float vOpacity; 
        void main() {
            gl_FragColor = vec4(vColor, vOpacity);
        }`
      })
      const mesh = new THREE.Points(geometry, material)
      effectList.push(mesh)
      threeLayer.addMesh(mesh)
    },
    // 地铁线
    cityRoadLine() {
      fetch('./geojson/cs_dt.json').then(res => {
        return res.json()
      }).then( geojson => {
        const textList = []
        this.roadLine = []
        geojson.l.forEach((item, i) => {
          let temp = [], list = []
          item.st.forEach(val => {
            let location = val.sl.split(',')
            let p = threeLayer.coordinateToVector3({x: Number(location[0]), y: Number(location[1])})
            temp.push([Number(location[0]),Number(location[1])])
            list.push(p)
            var text = new maptalks.Marker(location,
            {
                'properties' : {
                  'name' : val.n
                },
                'symbol' : {
                  'textName' : '{name}',  
                  'textSize': 10,
                  'textFill': '#ffffff',
                  'textHaloFill': '#000000',
                  'textHaloRadius': 2,
                }
            })
            textList.push(text)
          })
          this.roadLine.push({ name: item.kn, line: temp })
          // var line = new maptalks.LineString(temp,
          //   {
          //     arrowStyle : null, // arrow-style : now we only have classic
          //     arrowPlacement : 'vertex-last', // arrow's placement: vertex-first, vertex-last, vertex-firstlast, point
          //     visible : true,
          //     editable : true,
          //     cursor : null,
          //     shadowBlur : 0,
          //     shadowColor : 'black',
          //     draggable : false,
          //     dragShadow : false, // display a shadow during dragging
          //     drawOnAxis : null,  // force dragging stick on a axis, can be: x, y
          //     symbol: {
          //       'lineColor' : '#' + item.cl,
          //       'lineWidth' : 2,
          //       'textPlacement' : 'vertex'
          //     }
          // });
          // let layer = new maptalks.VectorLayer(item.kn, line).addTo(map);
          // let layer = new maptalks.VectorLayer(item.kn).addTo(map);
          // layer.setZIndex(i)
          // textList.forEach(ele => {
          //   ele.addTo(layer)
          // })
          this.threeLine(list, '#' + item.cl)
        })
        let layer = new maptalks.VectorLayer('地铁').addTo(map);
        layer.setZIndex(2)
        textList.forEach(ele => {
          ele.addTo(layer)
        })
      })
    },
    threeLine(list, color) {
      const curve3 = new THREE.CatmullRomCurve3(list)
      var geometry = new THREE.TubeGeometry( curve3, 256, 0.2, 4, false );
      var material = new THREE.ShaderMaterial({
            uniforms: {
                iTime: {
                    value: 0
                },
                color: { value: new THREE.Color(color) },
                iResolution: {
                    value: new THREE.Vector2(800, 800)
                }
            },
            side: 2,
            depthWrite: true,
            transparent: true,
            depthTest: false,
            blending: THREE.AdditiveBlending,
            vertexShader: vertexShader,
            fragmentShader: `
            uniform float iTime;
            uniform vec3 color;
            varying vec2 vUv;
            void main() {
                
                float d = mod(vUv.x * 1.0 - iTime/2.0, 1.0);

                d = pow(d, 10.0);

                vec4 lightColor = vec4(mix(color*d, color, 0.2), 1.0);

                gl_FragColor = lightColor;
            }`
      })
      var mesh = new THREE.Mesh( geometry, material );
      threeLayer.addMesh( mesh );
      effectList.push(mesh)
    },
    // 加载地图
    loadMap() {
        this.views = this.$refs.views
        map = new maptalks.Map(this.views, {
            center : [112.982279, 28.19409],
            zoom : 14,
            pitch: 60,
            minZoom: 12,
            maxZoom: 19,
            // bearing: 180,
            centerCross: false,
            doubleClickZoom: false,
            attribution: false,
            layerSwitcherControl: {
              'position'  : 'top-right',
              'baseTitle' : '主题样式',
              'overlayTitle' : '',
              'excludeLayers' : ['模型', '地铁'],
              'containerClass' : 'maptalks-layer-switcher'
            },
            baseLayer: new maptalks.GroupTileLayer('tile', [
              new maptalks.TileLayer('白底地图',{
                'visible' : false,
                'urlTemplate': 'https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png',
                'subdomains'  : ['a','b','c','d'],
              }),
              new maptalks.TileLayer('黑底地图',{
                'urlTemplate': 'https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}.png',
                'subdomains'  : ['a','b','c','d']
              }),
              new maptalks.TileLayer('卫星地图',{
                'visible' : false,
                'urlTemplate': 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}.jpg',
                'subdomains'  : ['a','b','c','d']
              })
            ])
        }) 
        threeLayer = new ThreeLayer('模型', {
            forceRenderOnMoving: true,
            forceRenderOnRotating: true,
            animation: true
        });
        // threeLayer.setZIndex(10)
        threeLayer.addTo(map);
        threeLayer.prepareToDraw = this.initScene
        map.on('click', (params) => {
          let point = threeLayer.coordinateToVector3(params.coordinate)
          map.animateTo({
            center: params.coordinate,
            zoom: 16,
            pitch: 60
          }, {
            duration: 3000
          });
        })

        render1 = new CSS3DRenderer({alpha: true}) //antialias: true, 
        render1.setSize(this.views.clientWidth, this.views.clientHeight)
        this.views.appendChild(render1.domElement)
        render1.domElement.style.position = 'absolute'
        render1.domElement.style.top = 0
        render1.domElement.style.zIndex = 2
        render1.domElement.style.pointerEvents = 'none'

        // 动画循环
        ThreeLayer.prototype.setRendererRenderScene = function () {
            this.getRenderer().renderScene = function () {
                const layer = this.layer;
                layer._callbackBaseObjectAnimation();
                this._syncCamera();

                const renderer = this.context, camera = this.camera, scene = this.scene;
                TWEEN.update()
                const elapsedTime = clock.getElapsedTime()
                // Update passes
                effectList.forEach(object => {
                    if(object.material && object.material.uniforms){
                        object.material.uniforms.iTime.value = elapsedTime
                    }
                })
                if(layer.bloomEnable && layer.composer && layer.composer.passes.length > 1) {
                  this.layer.composer.render(0)
                  renderer.clearDepth()
                  camera.layers.set(0)
                  renderer.render(scene, camera);
                }
                render1.render(scene, camera)
                this.completeRender();
            }
        }
        threeLayer.setRendererRenderScene()
        this.cityRoadLine()
    },
    initScene(gl, scene, camera) {
        let AmbientLight = new THREE.AmbientLight( 0x404040);
        scene.add( AmbientLight );
        var DirectionalLight = new THREE.DirectionalLight( 0xffffff, 2);
        DirectionalLight.position.set(0, -20, 50).normalize();
        scene.add( DirectionalLight );
        this.initComposer()
        this.createPointGeo()
        this.createEffect()
        this.getGeoJson()
        this.createLineFly()
    },
    initComposer() {
      const renderer = threeLayer.getThreeRenderer()
      threeLayer.composer = new EffectComposer(renderer)
      let size = threeLayer.getMap().getSize()
      threeLayer.composer.setSize(size.width, size.height)

      const scene = threeLayer.getScene()
      const camera = threeLayer.getCamera()
      threeLayer.renderPass = new RenderPass(scene, camera)
      threeLayer.composer.addPass(threeLayer.renderPass)
      // 抗锯齿
      const smaaPass = new SMAAPass(SMAAEdgesShader)
      threeLayer.composer.addPass(smaaPass)
      //辉光
      const bloomPass = threeLayer.bloomPass = new UnrealBloomPass(new THREE.Vector2(size.width, size.height))
      threeLayer.composer.addPass(bloomPass)
      threeLayer.bloomEnable = true
      bloomPass.renderToScreen = true
      bloomPass.exposure = 1
      bloomPass.strength = 2
      bloomPass.radius = 0.5
      bloomPass.threshold = 0.1

    },
    getGeoJson() {
        const xhr = new XMLHttpRequest()
        xhr.open('get', './geojson/cs_city.json', true)
        xhr.onload = () => {
            if(xhr.status === 200) {
                const features = JSON.parse(xhr.response).features
                const material = new THREE.ShaderMaterial({
                uniforms: {
                    iTime: { value: 0 },
                    time: { value: 0 },
                    color: { value: new THREE.Color(0x00ffff) },
                    color1: { value: new THREE.Color(0x0b1060) }
                },
                side: 2,
                depthWrite: true,
                transparent: false,
                depthTest: true,
                vertexShader: vertexShader,
                fragmentShader: `
                    uniform float time;
                    uniform vec3 color;
                    uniform vec3 color1;
                    varying vec3 vPosition;
                    void main() {

                        float indexMix = vPosition.z / 5.0;
                        
                        vec3 lightColor = mix(color1, color, indexMix);

                        gl_FragColor = vec4(lightColor, 1.0);
                }`
                })
                var heightPerLevel = 2.5;
                var polygons = features.map(f => {
                    const polygon = maptalks.GeoJSON.toGeometry(f);
                    var levels = f.properties.height || 1;
                    polygon.setProperties({
                        height: heightPerLevel * levels,
                    });
                    return polygon;
                });
                const extrudePolygons = threeLayer.toExtrudePolygons(polygons, { topColor: '#fff', interactive: true }, material);
                threeLayer.addMesh(extrudePolygons)
            }
        }
        xhr.send()
    },
    // 乘坐地铁
    bySubway(way) {
      let count = way.line.length-1
       this.replay(way, count)
    },
    replay(way, count) {
      let _this = this
      map.animateTo({
        zoom: 15,
        center: way.line[count],
        pitch: 0
      }, {
        duration: 2000,

      }, function(frame) {
          if (frame.state.playState === 'finished' && count > 0) {
            count--
            _this.replay(way, count)
          }
      })
    }
  },
  mounted() {
    this.loadMap()
    map.on('resize', () => {
      render1.setSize(this.views.clientWidth, this.views.clientHeight)
    })
  },
  destroyed() {
    effectList = []
    map = null
    threeLayer.getScene().clear()
  }
}
</script>

<style lang="scss" scoped>
.home {
  width: 100vw;
  height: 100vh;
} 
#home-view {
  width: 100%;
  height: 100%;
  position: relative;
  background: url('~@/assets/img/sky.jpg') no-repeat;
}
.effect-list {
  position: absolute;
  top: 0;
  left: 0;
  color: #fff;
  overflow: auto;
  height: 100vh;
}
li > button {
  padding: 2px 5px;
  font-size: 14px;
  margin-bottom: 2px;
}
.back {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 99;
  display: flex;
}
.road-line {
  position: absolute;
  bottom: 10px;
  left: 50%;
  z-index: 99;
  display: flex;
  transform: translateX(-50%);
  li {
    color: #fff;
    border: 1px solid #ccc;
    padding: 10px;
    cursor: pointer;
    &:hover {
      background-color: cyan;
    }
  }
}
</style>

<style lang="scss">
.maptalks-layer-switcher {
  .panel {
    padding: 20px 20px 0 0 !important;
  }
}
.info-wrap {
  color:rgb(7, 209, 245);
  font-size: 12px;
  padding: 5px 10px;
  text-align: center;
  text-shadow: 0 0 2px #11e1f0;
  font-family: 'kaiti';
  border: 1px solid #fff;
  background-image: linear-gradient(to bottom,  rgba(255, 6, 234, 0.6), rgba(126, 65, 126, 0.527));
}
</style>
