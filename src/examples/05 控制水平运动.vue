
<template></template>
<script setup>
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { Octree } from "three/examples/jsm/math/Octree"
import { Capsule } from "three/examples/jsm/math/Capsule"
import { OctreeHelper } from "three/examples/jsm/helpers/OctreeHelper"
import Stats from "three/examples/jsm/libs/stats.module"
import { reactive } from "vue"

const scene = new THREE.Scene()
scene.background = new THREE.Color(0x88ccee);
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
camera.position.set(0, 10, 20)
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


const orbitControls = new OrbitControls(camera, renderer.domElement)
orbitControls.enableDamping = true

const stats = new Stats()
stats.dom.style.position = 'absolute'
stats.dom.style.left = '0px'
stats.dom.style.top = '0px'
document.body.appendChild(stats.dom)

const render = () => {
  const time = clock.getDelta()
  controlsPlayer(time)
  updatePlayer(time)
  resetPlayer()
  orbitControls.update()
  renderer.render(scene, camera)
  requestAnimationFrame(render)
}


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

const group = new THREE.Group()
group.add(plane)
scene.add(group)

// 创建一个空间 
const worldOctree = new Octree()
worldOctree.fromGraphNode(group)

// 创建一个人的碰撞体
const playerCollider = new Capsule(new THREE.Vector3(0, 0.35, 0), new THREE.Vector3(0, 1.35, 0), 0.35)
console.log(playerCollider);
//  创建一个胶囊物体
const capsuleGeometry = new THREE.CapsuleGeometry(0.35, 1, 32)
const capsuleMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000, side: THREE.DoubleSide });
const capsule = new THREE.Mesh(capsuleGeometry, capsuleMaterial);
capsule.position.set(0, 8.5, 0)
capsule.castShadow = true
scene.add(capsule);

// 将相机作为胶囊的子元素 就可以实现跟随
camera.position.set(0, 2, -5)
camera.lookAt(capsule.position)
orbitControls.target = capsule.position
capsule.add(camera)
// 设置一个重力
const gravity = -9.8
// 设置初始速度
const playerVelocity = new THREE.Vector3(0, 0, 0)
// 方向向量
const playerDirection = new THREE.Vector3(0, 0, 0)
let keyStates = {
  KeyW: false,
  KeyS: false,
  KeyA: false,
  KeyD: false,
  Space: false,
  isDown: false
}
// 玩家是否在地面上
let playerOnFloor = false

function updatePlayer(deltaTime) {
  // 设置摩擦力
  const damping = -0.05
  // 计算 y 周速度重力掉落
  if (playerOnFloor) {
    playerVelocity.y = 0
    keyStates.isDown ||
      playerVelocity.addScaledVector(playerVelocity, damping);
  } else {
    playerVelocity.y += deltaTime * gravity

  }
  // 根据叉轴 计算出 每帧的移动速度
  playerCollider.translate(playerVelocity.clone().multiplyScalar(deltaTime))
  playerCollider.getCenter(capsule.position)

  playerCollisions()
}

// 碰撞检测
function playerCollisions() {
  const result = worldOctree.capsuleIntersect(playerCollider)
  playerOnFloor = false
  if (result) {
    playerOnFloor = result.normal.y > 0
    playerCollider.translate(result.normal.multiplyScalar(result.depth))
  }
}

// 重置玩家
function resetPlayer() {
  if (capsule.position.y < -20) {
    playerCollider.start.set(0, 3.5, 0)
    playerCollider.end.set(0, 4.5, 0)
    playerCollider.radius = 0.35
    playerVelocity.set(0, 0, 0)
    playerDirection.set(0, 0, 0)
  }
}
document.addEventListener('keydown', event => {
  keyStates[event.code] = true
  keyStates.isDown = true
}, false)
document.addEventListener('keyup', event => {
  keyStates[event.code] = false
  keyStates.isDown = false
}, false)


function controlsPlayer(deltaTime) {
  if (keyStates.KeyW) {
    playerDirection.z = 1
    // // 获取胶囊前方位置
    const capsuleFront = new THREE.Vector3(0, 0, 0)
    capsule.getWorldDirection(capsuleFront)
    playerVelocity.add(capsuleFront.multiplyScalar(deltaTime))
  }
  if (keyStates.KeyS) {
    playerDirection.z = 1
    // 获取胶囊前方位置
    const capsuleFront = new THREE.Vector3(0, 0, 0);
    capsule.getWorldDirection(capsuleFront);
    // 计算玩家的速度
    playerVelocity.add(capsuleFront.multiplyScalar(-deltaTime));
  }
  if (keyStates.KeyA) {
    playerDirection.x = 1
    // 获取胶囊前方位置
    const capsuleFront = new THREE.Vector3(0, 0, 0)
    capsule.getWorldDirection(capsuleFront);
    // 侧方的方向 正前面的方向和胶囊的正上方求叉积 求出侧方方向
    capsuleFront.cross(capsule.up)
    playerVelocity.add(capsuleFront.multiplyScalar(-deltaTime))
  }
  if (keyStates.KeyD) {
    playerDirection.x = 1
    // 获取胶囊前方位置
    const capsuleFront = new THREE.Vector3(0, 0, 0)
    capsule.getWorldDirection(capsuleFront);
    // 侧方的方向 正前面的方向和胶囊的正上方求叉积 求出侧方方向
    capsuleFront.cross(capsule.up)
    playerVelocity.add(capsuleFront.multiplyScalar(deltaTime))
  }
}
render()
</script>

<style >
* {
  padding: 0;
  margin: 0;
}
</style>
