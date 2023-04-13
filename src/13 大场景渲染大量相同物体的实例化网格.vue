
<template></template>
<script setup>
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { Octree } from "three/examples/jsm/math/Octree"
import { Capsule } from "three/examples/jsm/math/Capsule"
import { OctreeHelper } from "three/examples/jsm/helpers/OctreeHelper"
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader"
import Stats from "three/examples/jsm/libs/stats.module.js";
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

// 形状和材质必须是相同的
const geometry = new THREE.TorusKnotGeometry(1, 0.3, 100, 16);
const material = new THREE.MeshBasicMaterial({ color: 0xffff00 });
// for (let index = 0; index < 1000; index++) {
//   const torusKnot = new THREE.Mesh(geometry, material);
//   torusKnot.position.set(
//     Math.random() * 100 - 50,
//     Math.random() * 100 - 50,
//     Math.random() * 100 - 50
//   )
//   scene.add(torusKnot);
// }
const instanceMesh = new THREE.InstancedMesh(geometry, material, 10000)
for (let i = 0; i < 1000; i++) {
  const postion = new THREE.Vector3(
    Math.random() * 100 - 50,
    Math.random() * 100 - 50,
    Math.random() * 100 - 50
  )
  const rotation = new THREE.Euler(
    Math.random() * Math.PI * 2,
    Math.random() * Math.PI * 2,
    Math.random() * Math.PI * 2
  )
  // 四元数用于表示旋转
  const quaternion = new THREE.Quaternion().setFromEuler(rotation)
  const scale = new THREE.Vector3(
    Math.random() * 0.5 + 0.5,
    Math.random() * 0.5 + 0.5,
    Math.random() * 0.5 + 0.5
  )
  let matrix = new THREE.Matrix4()
  matrix.compose(postion, quaternion, scale)
  instanceMesh.setMatrixAt(i, matrix)
}
scene.add(instanceMesh)
render()
</script>

<style >
* {
  padding: 0;
  margin: 0;
}
</style>
