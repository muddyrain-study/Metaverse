
<template></template>
<script setup>
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { Octree } from "three/examples/jsm/math/Octree"
import { Capsule } from "three/examples/jsm/math/Capsule"
import { OctreeHelper } from "three/examples/jsm/helpers/OctreeHelper"
import Stats from "three/examples/jsm/libs/stats.module"
const scene = new THREE.Scene()
scene.background = new THREE.Color(0x88ccee);
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
camera.position.set(0, 10, 10)
camera.lookAt(scene.position)

const renderer = new THREE.WebGLRenderer({
  antialias: true
})
renderer.setSize(window.innerWidth, window.innerHeight)
renderer.shadowMap.enabled = true
renderer.shadowMap.type = THREE.VSMShadowMap
// 定义渲染器的输出编码 rgbe 贴图
renderer.outputEncoding = THREE.sRGBEncoding
// 色调映射 ACESFilmicToneMapping   ACES电影色调映射
renderer.toneMapping = THREE.ACESFilmicToneMapping
document.body.appendChild(renderer.domElement)

const clock = new THREE.Clock()

const axesHelper = new THREE.AxesHelper(5)
scene.add(axesHelper)

const orbitControls = new OrbitControls(camera, renderer.domElement)
orbitControls.enableDamping = true

const stats = new Stats()
stats.dom.style.position = 'absolute'
stats.dom.style.left = '0px'
stats.dom.style.top = '0px'
document.body.appendChild(stats.dom)


const render = () => {
  const time = clock.getDelta()
  orbitControls.update()
  renderer.render(scene, camera)
  requestAnimationFrame(render)
}
render()

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

// 创建一个空间 

const worldOctree = new Octree()

// 创建一个人的碰撞体
const playerCollider = new Capsule(new THREE.Vector3(0, 0.35, 0), new THREE.Vector3(0, 1.35, 0), 0.35)

console.log(playerCollider);
console.log(worldOctree);

//  创建一个胶囊物体
const capsuleGeometry = new THREE.CapsuleGeometry(0.35, 1, 32)
const capsuleMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000, side: THREE.DoubleSide });
const capsule = new THREE.Mesh(capsuleGeometry, capsuleMaterial);
capsule.position.set(0, 0.85, 0)
capsule.castShadow = true
scene.add(capsule); capsuleMaterial
</script>

<style >
* {
  padding: 0;
  margin: 0;
}
</style>
