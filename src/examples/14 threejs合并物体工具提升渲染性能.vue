
<template></template>
<script setup>
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { Octree } from "three/examples/jsm/math/Octree"
import { Capsule } from "three/examples/jsm/math/Capsule"
import { OctreeHelper } from "three/examples/jsm/helpers/OctreeHelper"
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader"
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
  renderer.render(scene, camera)
  requestAnimationFrame(render)
}

// 创建一个平面
const planeGeometry = new THREE.PlaneGeometry(20, 20, 1, 1)
const planeMaterial = new THREE.MeshBasicMaterial({
  color: 0xffffff,
  side: THREE.DoubleSide
})
const plane = new THREE.Mesh(planeGeometry, planeMaterial)
// 材质是否接收阴影。 默认为假。
plane.receiveShadow = true
plane.rotation.x = -Math.PI / 2

scene.add(plane)

// 形状不一样和材质一样
const material = new THREE.MeshBasicMaterial({ color: 0x22ff00 });
const geometrys = []
for (let i = 0; i < 1000; i++) {
  const geometry = new THREE.TorusKnotGeometry(
    0.5 + Math.random() * 0.5,
    0.3,
    50 + parseInt(Math.random() * 50),
    16);
  const matrix = new THREE.Matrix4()
  matrix.makeRotationX(Math.random() * Math.PI)
  matrix.makeRotationY(Math.random() * Math.PI)
  matrix.makeRotationZ(Math.random() * Math.PI)

  matrix.makeScale(
    Math.random() * 0.5 + 0.5,
    Math.random() * 0.5 + 0.5,
    Math.random() * 0.5 + 0.5,
  )
  matrix.makeTranslation(
    Math.random() * 100 - 50,
    Math.random() * 100 - 50,
    Math.random() * 100 - 50,
  )
  geometry.applyMatrix4(matrix)
  geometrys.push(geometry)
}
const mergeGeometry = BufferGeometryUtils.mergeBufferGeometries(geometrys)
let mesh = new THREE.Mesh(mergeGeometry, material)
scene.add(mesh)
render()
</script>

<style >
* {
  padding: 0;
  margin: 0;
}
</style>
