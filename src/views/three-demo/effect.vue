<template>
  <div class="home">
    <div class="home-view" ref="views"></div>
    <ul class="effect-list">
      <li v-for="(item, k) in effectList" :key="k">
        <el-button :plain="activeIndex !== k" size="mini" type="success" @click="setNewEffect(item, k)">{{k}}</el-button>
      </li>
    </ul>
    <el-button @click="$router.go(-1)" class="back" type="primary">退 出</el-button>
  </div>
</template>

<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls' // 控制器
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
let scene, camera, render, controler, flyControls, trackballControls, clock = new THREE.Clock();;
export default {
  data() {
    return {
      list: [],
      image: '',
      views: null,
      effectList: Effect,
      total: 0,
      activeIndex: 'effect44'
    }
  },
  methods: {
     // 使用canvas创建纹理
    generateSprite() {
        var canvas = document.createElement('canvas');
        canvas.width = 128;
        canvas.height = 128;

        var context = canvas.getContext('2d');
        var gradient = context.createLinearGradient(canvas.width / 2, canvas.height / 2, 0, canvas.width / 2, canvas.height / 2, canvas.width / 2);
        gradient.addColorStop(0, 'rgba(11,255,255,1)');

        context.fillStyle = gradient;
        context.fillRect(0, 0, canvas.width, canvas.height);

        var texture = new THREE.Texture(canvas);
        texture.needsUpdate = true;
        texture.wrapS = THREE.RepeatWrapping;
        texture.wrapT = THREE.RepeatWrapping;
        texture.repeat.set(2, 1);
        return texture;

    },
    getMesh(fragmentShader) {
        // const geometry = new THREE.CylinderGeometry(0.2, 0.2, 100, 16, 16, true);
        // const geometry = new THREE.BoxGeometry(25, 25, 100, 16, 16, 16, true);
        // const geometry = new THREE.CircleGeometry(50, 64);
        // for(let i = 0; i < 1; i++) {
          const material = new THREE.ShaderMaterial({
              uniforms: {
                  iTime: {
                      value: 0
                  },
                  size: { value: 2 },
                  random: {
                      value: Math.random(),
                  },
                  color: { value: new THREE.Color(0xff0000) },
                  color1: { value: new THREE.Color(0xff0000) },
                  iResolution: {
                      value: new THREE.Vector2(800, 800)
                  },
                  iChannel0: {
                      value: new THREE.ImageUtils.loadTexture('./images/smokeparticle.png')
                  },
                  iChannel1: {
                      value: new THREE.ImageUtils.loadTexture('./images/smokeparticle.png')
                  },
                  map: {
                    value: new THREE.ImageUtils.loadTexture('./images/smokeparticle.png')
                  }
              },
              side: 2,
              depthWrite: false,
              transparent: true,
              vertexShader: vertexShader,
              fragmentShader: fragmentShader
          })
          // let geometry = new THREE.CylinderGeometry(10, 10, 100, 16, 16, true);
          const geometry = new THREE.CircleGeometry(50, 64);
          let Octahedron = new THREE.SphereGeometry(30, 32);
          let OctahedronMesh = new THREE.Mesh(Octahedron, material);
          OctahedronMesh.position.y = 10;
          let plane = new THREE.Mesh(geometry, material);

          // plane.position.set(Math.random()*2000*unitX, 50, Math.random()*2000*unitY)
          plane.rotation.x = -Math.PI/2;
          scene.add(plane)
        // }
        // let plane = new THREE.Mesh(geometry, material);
        // plane.rotation.x = -Math.PI / 2;
        // return plane;
    },
    setSkyBox(type) {
      var loader = new THREE.TextureLoader();
      var skyBox = new THREE.BoxGeometry(5000,5000,5000);
      var rootPath = './images/';
      var imgNameArr = ['_posx','_negx','_posy','_negy','_posz','_negz'];
      var format = '.jpg';
      var materialArr = [];
      for(let i=0; i< imgNameArr.length;i++) {
        materialArr.push(new THREE.MeshBasicMaterial({
          map:loader.load(rootPath+type+imgNameArr[i]+format),
          side: THREE.DoubleSide,
          fog: false
        }));
      }
      const sky = new THREE.Mesh(skyBox, materialArr);
      scene.add(sky);
    },
    init() {
      this.views = this.$refs.views
      scene = new THREE.Scene()
      let fog = new THREE.Fog(0xff00ff, 1, 1000)
      scene.fog = fog

      camera = new THREE.PerspectiveCamera(45, this.views.clientWidth/this.views.clientHeight, 0.1, 10000)
      camera.position.set(0, 100, 105);
      render = new THREE.WebGLRenderer({antialias: true, alpha: true})
      render.setSize(this.views.clientWidth, this.views.clientHeight)
      this.views.appendChild(render.domElement)
      controler = new OrbitControls(camera, render.domElement);
      controler.minDistance = 1;
      controler.maxDistance = 800;
      controler.enableDamping = true
      let AmbientLight = new THREE.AmbientLight( 0x888888 );
      scene.add( AmbientLight );
      var DirectionalLight = new THREE.DirectionalLight( 0xffffff);
      DirectionalLight.position.set( 0, 50, 0 );
      scene.add( DirectionalLight );


      this.getMesh(Effect[this.activeIndex]())

      let gridHelper = new THREE.GridHelper(80, 80, '#ccddaa')
      // scene.add(gridHelper)
    },
    setNewEffect(effect, k) {
     this.activeIndex = k
     scene.clear()
    this.setSkyBox('star')
     this.getMesh(effect())
    },
    // 创建粒子系统
    createPointCloud(geometry) {
        var material = new THREE.MeshPhysicalMaterial({
            color: 0xffff00,
            size: 1,
            transparent: true,
            map: this.generateSprite(),
            // blending: THREE.AdditiveBlending,
            depthTest: false
        });
        var cloud = new THREE.Mesh(geometry, material);
        // cloud.sortParticles = true;
        return cloud;
    },
    animation() {
        render.render(scene,camera)
        let delta = clock.getDelta()
        controler.update(delta)
        const elapsedTime = clock.getElapsedTime()
        // Update passes
        scene.traverse((child) => {
            if (child.material){
              if(child.material.uniforms) {
                  child.material.uniforms.iTime.value = elapsedTime
              }
            }
        })
        requestAnimationFrame(this.animation);
    }
  },
  mounted() {
    this.init()
    this.animation()
    this.setSkyBox('star')
    window.onresize = () => {
      render.setSize(this.views.clientWidth, this.views.clientHeight)
      camera.aspect = this.views.clientWidth / this.views.clientHeight//相机重置可视范围
      camera.updateProjectionMatrix();
    }
  },
  destroyed() {
    scene.clear()
    render.dispose()
  }
}
</script>

<style scoped>
.home {
  width: 100vw;
  height: 100vh;
} 
.home-view {
  width: 100%;
  height: 100%;
  background: rgb(0, 0, 0.5);
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
  bottom: 10px;
  right: 50%;
  z-index: 99;
  transform: translateX(50%);
}
</style>
