
<template></template>
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
import { reactive } from "vue"

const scene = new THREE.Scene()
scene.background = new THREE.Color(0x88ccee);
// scene.fog = new THREE.Fog(0x88ccee, 0, 50);

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

document.body.appendChild(renderer.domElement)

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
  cubeCamera.update(renderer, scene)
  renderer.render(scene, camera)
  requestAnimationFrame(render)
}


const hdrLoader = new RGBELoader()
hdrLoader.load(
  "./hdr/023.hdr", texture => {
    texture.mapping = THREE.EquirectangularReflectionMapping
    texture.format = THREE.RGBAFormat
    scene.background = texture
    scene.environment = texture
    sphereMaterial.envMap = cubeRenderTarget.texture
  }


)
// 创建球
const sphereGeometry = new THREE.SphereGeometry(1, 32, 32)
const sphereMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xffffff,
  // transmission: 1, // 透明度
  transparent: true,
  roughness: 0, // 粗糙度
  metalness: 1, // 金属度
  // reflectivity: 1, // 反射率
  // ior: 2.33, // 强度
  // iridescenceIOR: 1.33 // 虹彩IOR强度
})
// 彩虹色
sphereMaterial._iridescence = 1
const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial)
sphere.position.set(0, 0, 0)
scene.add(sphere)

const boxGeometry = new THREE.BoxGeometry(1, 1, 1)
const boxMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xffffff
})
const box = new THREE.Mesh(boxGeometry, boxMaterial)
box.position.set(5, 0, 0)
scene.add(box)

// 创建 cubeTarget
const cubeRenderTarget = new THREE.WebGLCubeRenderTarget(512)
const cubeCamera = new THREE.CubeCamera(0.1, 1000, cubeRenderTarget)

render()

</script>

<style >
* {
  padding: 0;
  margin: 0;
}
</style>
