<template>
  <div id="three"></div>

  <!-- 可以通过样式把video的控制层放到你想放的位置  或者 通过js自己去写一些控制方法 -->
  <video id="my-video" class="video-js" controls preload="auto" autoplay width="300" loop height="260" data-setup="{}"
    crossorigin="anonymous">
    <source src="https://vod.hosvr.cn/hosvr_vod_4k_xiaozutaolun3.mp4" />
    <p class="vjs-no-js">
      To view this video please enable JavaScript, and consider upgrading
      to a web browser that
      <a href="https://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
    </p>
  </video>
  <button @click="count++">{{ count }}</button>
</template> 

<script lang="ts" setup>
import type { AppContext } from "@netless/window-manager";
import * as THREE from 'three';
//引入轨道控制器（用来通过鼠标事件控制模型旋转、缩放、移动），没有这个需求可不引入
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { computed, inject, onMounted, ref, watchEffect } from "vue";
const scene = new THREE.Scene()

//添加光源
const ambient = new THREE.AmbientLight(0xffffff, 0.5),
  light1 = new THREE.PointLight(0xffffff, 0.4),
  light2 = new THREE.PointLight(0xffffff, 0.4)

scene.add(ambient)
light1.position.set(200, 300, 400)
scene.add(light1)
light2.position.set(-200, -300, -400)
scene.add(light2)

//创建一个透视相机
const width = window.innerWidth, height = window.innerHeight,
  camera = new THREE.PerspectiveCamera(45, width / height, 1, 1000)
//设置相机位置
camera.position.set(200, 200, 200)
//设置相机方向
camera.lookAt(0, 0, 0)

//创建辅助坐标轴
const axesHelper = new THREE.AxesHelper(10)
scene.add(axesHelper)
camera.position.set(100, 100, 200)
//创建一个物体（形状）
var geometry = new THREE.SphereBufferGeometry(500, 60, 40);
// invert the geometry on the x-axis so that all of the faces point inward
geometry.scale(-1, 1, 1);
//创建材质（外观）
const material = new THREE.MeshLambertMaterial({
  color: 0xffffff,//设置材质颜色
  transparent: true,//开启透明度
  opacity: 0 //设置透明度
})
//创建一个网格模型对象
const mesh = new THREE.Mesh(geometry, material)
//把网格模型添加到三维场景
scene.add(mesh)

//创建一个WebGL渲染器
const renderer = new THREE.WebGLRenderer()
renderer.setSize(width, height)
renderer.render(scene, camera)

const controls = new OrbitControls(camera, renderer.domElement)
controls.addEventListener('change', () => {
  renderer.render(scene, camera)
})

const context = inject<AppContext>("context");
if (!context) throw new Error("must call provide('context') before mount App");

const storage = context.createStorage("counter", { count: 0 });
const real_count = ref(storage.state.count);

const count = computed<number>({
  get: () => real_count.value,
  set: (count) => storage.setState({ count }),
});

onMounted(() => {

  if (document.getElementById('my-video')) {
    document.getElementById('three')?.appendChild(renderer.domElement);
    console.log(document.querySelector("#my-video"))
    var Videotexture = new THREE.VideoTexture(document.querySelector("#my-video"));

    var Videomaterial = new THREE.MeshBasicMaterial({ map: Videotexture });

    var videomesh = new THREE.Mesh(geometry, Videomaterial);

    scene.add(videomesh);
  }
  storage.addStateChangedListener(() => {
    real_count.value = storage.state.count;
  })
}

);

watchEffect(() => {
  console.log("App.vue: count =", count.value);
});
</script>
<style scoped>
.video-js {
  position: absolute;
  left: 0;
  bottom: 0;
}
</style>