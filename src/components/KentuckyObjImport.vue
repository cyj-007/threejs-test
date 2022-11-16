<template>
  <canvas id="two"></canvas>
</template>


<script setup>
import * as THREE from 'three'
import { onMounted } from "vue"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader.js';


function init(){
  //获取dom
  const dom = document.querySelector("#two");
  //场景
  const scene = new THREE.Scene();
  //相机
  const camera = new THREE.PerspectiveCamera(25,
      window.innerWidth / window.innerHeight,
      0.1,
      1000);
  //设置相机坐标
  camera.position.set(0, 0, 10); //设置相机位置
  // 扩大视野范围
  camera.position.set(30, 30, 30);
  camera.lookAt(new THREE.Vector3(0, 0, 0));
  //控制器
  const controls = new OrbitControls(camera, dom);

  const objLoader = new OBJLoader();
  objLoader.load('https://threejs.org/manual/examples/resources/models/windmill/windmill.obj', (root) => {
    root.position.set(0, 0, 0);
    scene.add(root);
  });
  //渲染器
  const renderer = new THREE.WebGLRenderer({
    canvas: dom,
    alpha: true, //是否背景可以透明
    antialias: true //抗拒齿
  });
  //像素设置
  renderer.setPixelRatio(window.devicePixelRatio);
  //设置画布大小
  renderer.setSize(window.innerWidth, window.innerHeight);
  //添加场景辅助
  //添加坐标系
  // scene.add(new THREE.AxesHelper(5));
  //添加网格
  // scene.add(new THREE.GridHelper(2, 10));

  // 平行光
  const spotLight = new THREE.DirectionalLight(0xffffff)
  spotLight.position.set(10, 20, 10)
  //允许当前光产生阴影
  spotLight.castShadow = true
  scene.add(spotLight)


  //创建渲染函数
  const render = (time) => {
    // // // 将时间转换为秒
    // time *= 0.001;
    // scene.rotation.x = time;
    // scene.rotation.y = time;
    //内置 定时器
    requestAnimationFrame(render);
    //将场景和相机添加到渲染器中执行 一般60次/s
    renderer.render(scene, camera);
  };
  render()
}

onMounted(()=>{
  init()
})

</script>

<style scoped>
#two {
  width: 100%;
  height: 100%;
  position: fixed;
  background: url(../assets/dairyman.jpg) no-repeat center center;
  background-size: cover;
  margin: 0;
  overflow: hidden; /* 溢出隐藏 */
  -webkit-background-size: cover;
}
</style>

