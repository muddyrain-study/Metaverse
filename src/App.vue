
<template>
  <div id="container" ref="container"></div>
</template>
<script setup>
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { Octree } from "three/examples/jsm/math/Octree"
import { Capsule } from "three/examples/jsm/math/Capsule"
import { OctreeHelper } from "three/examples/jsm/helpers/OctreeHelper"
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader"
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader"
import Stats from "three/examples/jsm/libs/stats.module.js";
import * as BufferGeometryUtils from "three/examples/jsm/utils/BufferGeometryUtils"
import { Reflector } from "three/examples/jsm/objects/Reflector"
import { onMounted, ref } from "vue"

const scene = new THREE.Scene()
const container = ref(null)
scene.background = new THREE.Color(0x88ccee);
// scene.fog = new THREE.Fog(0x88ccee, 0, 50);

onMounted(() => {
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.001, 1000)
  camera.position.set(0, 5, 10)
  const renderer = new THREE.WebGLRenderer({
    antialias: true
  })
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.VSMShadowMap
  // 定义渲染器的输出编码 rgbe 贴图
  renderer.outputEncoding = THREE.sRGBEncoding
  // 色调映射 ACESFilmicToneMapping   ACES电影色调映射
  renderer.toneMapping = THREE.ACESFilmicToneMapping

  container.value.appendChild(renderer.domElement)

  const clock = new THREE.Clock()


  const orbitControls = new OrbitControls(camera, renderer.domElement)
  orbitControls.enableDamping = true

  const stats = new Stats();
  stats.domElement.style.position = "absolute";
  stats.domElement.style.top = "0px";
  document.body.appendChild(stats.domElement);

  console.log(renderer.info);
  const render = () => {
    const time = clock.getDelta()
    stats.update();
    orbitControls.update()
    renderer.render(scene, camera)
    requestAnimationFrame(render)
  }


  // 创建canvas 对象 
  const canvas = document.createElement('canvas')
  canvas.width = 1080
  canvas.height = 1080
  canvas.style.position = 'absolute'
  canvas.style.left = '0'
  canvas.style.top = '0'
  canvas.style.zIndex = 1000000
  canvas.style.transformOrigin = `0 0`
  canvas.style.transform = `scale(0.1)`
  const context = canvas.getContext('2d')

  // 添加视频纹理
  const video = document.createElement("video")
  video.src = "./video/keji1.mp4"
  video.muted = true
  video.loop = true
  video.play()
  const texture = new THREE.VideoTexture(video);
  // 创建一个平面
  const planeGeometry = new THREE.PlaneGeometry(2, 2, 1, 1);
  const planeMaterial = new THREE.MeshBasicMaterial({
    color: 0xffffff,
    side: THREE.DoubleSide,
    map: texture,
    alphaMap: texture,
    transparent: true,
    blending: THREE.AdditiveBlending,
    depthWrite: false,
  });
  const plane = new THREE.Mesh(planeGeometry, planeMaterial);
  scene.add(plane);

  function drawVideoText() {
    context.clearRect(0, 0, canvas.width, canvas.height);
    context.drawImage(video, 0, 0, canvas.width, canvas.height);

    context.textAlign = "center";
    context.textBaseline = "middle";
    context.font = "bold 100px Arial";
    context.fillStyle = "rgba(255,255,255,1)";
    context.fillText("Hello World", canvas.width / 2, canvas.height / 2);

    texture.needsUpdate = true;
    planeMaterial.needsUpdate = true;
  }
  document.body.appendChild(canvas);

  render()
})
</script>

<style >
* {
  padding: 0;
  margin: 0;
}

#container {
  width: 100vw;
  height: 100vh;
}
</style>
