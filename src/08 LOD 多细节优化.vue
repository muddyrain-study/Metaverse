
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
scene.fog = new THREE.Fog(0x88ccee, 0, 50);
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
camera.position.set(0, 5, 10)
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


// const orbitControls = new OrbitControls(camera, renderer.domElement)
// orbitControls.enableDamping = true

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
  // orbitControls.update()
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

// 创建立方体叠楼梯效果
for (let i = 0; i < 10; i++) {
  const boxGeometry = new THREE.BoxGeometry(1, 1, 0.15)
  const boxMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 })
  const box = new THREE.Mesh(boxGeometry, boxMaterial)
  box.position.y = 0.15 + i * 0.15
  box.position.z = i * 0.3
  plane.add(box)
}
const group = new THREE.Group()
group.add(plane)
scene.add(group)

// 创建一个空间 
const worldOctree = new Octree()
worldOctree.fromGraphNode(group)
// 创建八叉树 hleper
const octreeHelper = new OctreeHelper(worldOctree)
scene.add(octreeHelper)

// 创建一个人的碰撞体
const playerCollider = new Capsule(new THREE.Vector3(0, 0.35, 0), new THREE.Vector3(0, 1.35, 0), 0.35)

// 创建一个平面身体
const capsuleBodyGeometry = new THREE.PlaneGeometry(1, 0.5, 1, 1)
const capsuleBodyMaterial = new THREE.MeshBasicMaterial({
  color: 0x0000ff,
  side: THREE.DoubleSide
})
const capsuleBody = new THREE.Mesh(capsuleBodyGeometry, capsuleBodyMaterial);
capsuleBody.position.set(0, 0.5, 0)
//  创建一个胶囊物体
const capsuleGeometry = new THREE.CapsuleGeometry(0.35, 1, 32)
const capsuleMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000, side: THREE.DoubleSide });
const capsule = new THREE.Mesh(capsuleGeometry, capsuleMaterial);
capsule.position.set(0, 8.5, 0)
capsule.castShadow = true
capsule.add(camera)
scene.add(capsule);
capsule.add(capsuleBody);

// 将相机作为胶囊的子元素 就可以实现跟随
camera.position.set(0, 2, -5)
camera.lookAt(capsule.position)
// orbitControls.target = capsule.position
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
  console.log(event.code);
  keyStates[event.code] = true
  keyStates.isDown = true
}, false)
document.addEventListener('keyup', event => {
  keyStates[event.code] = false
  keyStates.isDown = false
}, false)


function controlsPlayer(deltaTime) {
  if (keyStates["KeyW"]) {
    playerDirection.z = 1;
    //获取胶囊的正前面方向
    const capsuleFront = new THREE.Vector3(0, 0, 0);
    capsule.getWorldDirection(capsuleFront);
    // console.log(capsuleFront);
    // 计算玩家的速度
    playerVelocity.add(capsuleFront.multiplyScalar(deltaTime));
  }
  if (keyStates["KeyS"]) {
    playerDirection.z = 1;
    //获取胶囊的正前面方向
    const capsuleFront = new THREE.Vector3(0, 0, 0);
    capsule.getWorldDirection(capsuleFront);
    // console.log(capsuleFront);
    // 计算玩家的速度
    playerVelocity.add(capsuleFront.multiplyScalar(-deltaTime));
  }
  if (keyStates["KeyA"]) {
    playerDirection.x = 1;
    //获取胶囊的正前面方向
    const capsuleFront = new THREE.Vector3(0, 0, 0);
    capsule.getWorldDirection(capsuleFront);

    // 侧方的方向，正前面的方向和胶囊的正上方求叉积，求出侧方的方向
    capsuleFront.cross(capsule.up);
    // console.log(capsuleFront);
    // 计算玩家的速度
    playerVelocity.add(capsuleFront.multiplyScalar(-deltaTime));
  }
  if (keyStates["KeyD"]) {
    playerDirection.x = 1;
    //获取胶囊的正前面方向
    const capsuleFront = new THREE.Vector3(0, 0, 0);
    capsule.getWorldDirection(capsuleFront);
    // 侧方的方向，正前面的方向和胶囊的正上方求叉积，求出侧方的方向
    capsuleFront.cross(capsule.up);
    // console.log(capsuleFront);
    // 计算玩家的速度
    playerVelocity.add(capsuleFront.multiplyScalar(deltaTime));
  }
  if (keyStates["Space"]) {
    playerVelocity.y = 5;
  }
}
window.addEventListener('mousemove', event => {
  // capsule.rotation.x -= event.movementY * 0.003;
  capsule.rotation.y -= event.movementX * 0.003;
}, false)
window.addEventListener("mousedown", event => {
  // 锁定鼠标指针
  document.body.requestPointerLock()
})

const lod = new THREE.LOD();
const material = new THREE.MeshBasicMaterial({ color: 0xff0000, wireframe: true })
for (let i = 0; i < 5; i++) {
  const geometry = new THREE.SphereGeometry(1, 22 - i * 5, 22 - i * 5)
  const mesh = new THREE.Mesh(geometry, material)
  lod.addLevel(mesh, i * 5)
}
lod.position.set(10, 0, 10)
scene.add(lod)

render()
</script>

<style >
* {
  padding: 0;
  margin: 0;
}
</style>
