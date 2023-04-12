
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
scene.fog = new THREE.Fog(0x88ccee, 0, 50);

const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.001, 1000)
camera.position.set(0, 5, 10)
const backCamera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.001, 1000)
camera.position.set(0, 5, -10);

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


// const orbitControls = new OrbitControls(camera, renderer.domElement)
// orbitControls.enableDamping = true

const stats = new Stats();
stats.domElement.style.position = "absolute";
stats.domElement.style.top = "0px";
document.body.appendChild(stats.domElement);


let activeCamera = camera
const render = () => {
  const time = clock.getDelta()
  controlsPlayer(time)
  updatePlayer(time)
  resetPlayer()
  stats.update();

  // 更新动作
  if (mixer) {
    mixer.update(time)
  }

  // orbitControls.update()
  renderer.render(scene, activeCamera)
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

// 创建立方体叠楼梯效果
for (let i = 0; i < 10; i++) {
  const boxGeometry = new THREE.BoxGeometry(1, 1, 0.15)
  const boxMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 })
  const box = new THREE.Mesh(boxGeometry, boxMaterial)
  box.position.y = 0.15 + i * 0.15
  box.position.z = i * 0.3
  plane.add(box)
}

// 创建一个octree
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

// // 创建一个平面身体
// const capsuleBodyGeometry = new THREE.PlaneGeometry(1, 0.5, 1, 1)
// const capsuleBodyMaterial = new THREE.MeshBasicMaterial({
//   color: 0x0000ff,
//   side: THREE.DoubleSide
// })
// const capsuleBody = new THREE.Mesh(capsuleBodyGeometry, capsuleBodyMaterial);
// capsuleBody.position.set(0, 0.5, 0)

// 加载机器人模型
const loader = new GLTFLoader()
let actions = {}
let mixer = null
let activeAction = null
loader.load('/models/RobotExpressive.glb', (gltf) => {
  const robot = gltf.scene
  robot.scale.set(0.5, 0.5, 0.5)
  robot.position.set(0, -0.85, 0)
  scene.add(robot)
  capsule.add(robot)
  mixer = new THREE.AnimationMixer(robot)
  for (const item of gltf.animations) {
    actions[item.name] = mixer.clipAction(item)
    if (['Idle', 'Walking', 'Runing'].includes(item.name)) {
      actions[item.name].loop = THREE.LoopRepeat
      actions[item.name].clampWhenFinished = false
    } else {
      actions[item.name].clampWhenFinished = true
      actions[item.name].loop = THREE.LoopOnce
    }
  }
  activeAction = actions['Idle']
  activeAction.play()


})

// 添加半球光源
const hemisphereLight = new THREE.HemisphereLight(0xffffff, 0x444444, 1)
scene.add(hemisphereLight)

//  创建一个胶囊物体
// const capsuleGeometry = new THREE.CapsuleGeometry(0.35, 1, 32)
// const capsuleMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000, side: THREE.DoubleSide });
const capsule = new THREE.Object3D();
capsule.position.set(0, 0.85, 0)
capsule.castShadow = true

scene.add(capsule);
// capsule.add(capsuleBody);

// 将相机作为胶囊的子元素 就可以实现跟随
camera.position.set(0, 2, -5);
camera.lookAt(capsule.position);
backCamera.position.set(0, 2, 5);
backCamera.lookAt(capsule.position);
// orbitControls.target = capsule.position
// 控制旋转上下的 空 3d 对象  
const capsuleBodyControl = new THREE.Object3D()
capsuleBodyControl.add(camera)
capsuleBodyControl.add(backCamera)
capsule.add(capsuleBodyControl)
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

  // 如果有水平运动 则设置运动动作
  const currentVelocity = Math.abs(playerVelocity.x) + Math.abs(playerVelocity.z)
  console.log(currentVelocity);
  if (currentVelocity > 0.1 && currentVelocity <= 3) {
    fadeToAction('Walking')
    // 如果有水平运动 则设置运动动作
  } else if (currentVelocity > 3) {
    fadeToAction('Running')
  } else {
    fadeToAction('Idle')
  }

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
  if (event.code === "KeyV") {
    activeCamera = activeCamera === camera ? backCamera : camera;
  }
  if (event.code === "KeyT") {
    fadeToAction('Wave')
  }
}, false)

function fadeToAction(name) {
  let prevAction = activeAction
  activeAction = actions[name]
  if (prevAction !== activeAction && prevAction) {
    prevAction.fadeOut(0.3)
    activeAction.reset().setEffectiveWeight(1).setEffectiveWeight(1).fadeIn(0.3).play()
    mixer.addEventListener('finished', function (e) {
      activeAction = prevAction
      prevAction.fadeOut(0.3)
      activeAction.reset().setEffectiveWeight(1).setEffectiveWeight(1).fadeIn(0.3).play()
    })
  }
}

function controlsPlayer(deltaTime) {
  if (keyStates["KeyW"]) {
    playerDirection.z = 1;
    //获取胶囊的正前面方向
    const capsuleFront = new THREE.Vector3(0, 0, 0);
    capsule.getWorldDirection(capsuleFront);
    // console.log(capsuleFront);
    // 计算玩家的速度
    playerVelocity.add(capsuleFront.multiplyScalar(deltaTime * 5));
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
  capsule.rotation.y -= event.movementX * 0.003;
  capsuleBodyControl.rotation.x += event.movementY * 0.003;
  if (capsuleBodyControl.rotation.x > Math.PI / 8) {
    capsuleBodyControl.rotation.x = Math.PI / 8;
  } else if (capsuleBodyControl.rotation.x < -Math.PI / 8) {
    capsuleBodyControl.rotation.x = -Math.PI / 8;
  }
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
