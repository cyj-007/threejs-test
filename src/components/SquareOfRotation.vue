<template>
  <div>
    <canvas id="first"></canvas>
  </div>
</template>

<script setup>
import * as THREE from 'three'
import {onMounted} from "vue"

//工具平面
const scene = new THREE.Scene()

function initThree() {
  //获取canvas
  const canvas = document.querySelector('#first')
  //创建WebGLRenderer,渲染器
  const renderer = new THREE.WebGLRenderer({
    canvas,
    antialias: true //抗锯齿
    , alpha: true //设置透明
  })
  //告诉渲染器我们要渲染阴影
  renderer.shadowMap.enabled = true;
  //设置阴影的类型，默认是 THREE.PCFShadowMap
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  //像素设置
  renderer.setPixelRatio(window.devicePixelRatio);
  //设置画布大小
  renderer.setSize(window.innerWidth, window.innerHeight);

  /**
   * PerspectiveCamera（透视摄像机） 3D场景的渲染
   * fov — 摄像机视锥体垂直视野角度。可以理解为人类的视野广度。
   * aspect — 摄像机视锥体横纵比。渲染结果的横向尺寸和纵向尺寸的比值，这里我们使用的是 浏览器窗口的宽高比。
   * near — 摄像机视锥体近端面。一切比近面更近的事物将不被渲染。
   * far — 摄像机视锥体远端面。一切比远面更远的事物将不被渲染，但是设置过大可能会影响性能。
   * */
  const camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
  )
  camera.position.z = 10
  //调整相机方向
  // camera.position.set(30, 30, 30);
  //固定于原点
  // camera.lookAt(new THREE.Vector3(0, 0, 0));
  // // 平面
  // const planeGeometry = new THREE.PlaneGeometry(60, 30);
  // const planeMaterial = new THREE.MeshLambertMaterial({
  //   color: 0xffffff
  // })
  // const plane = new THREE.Mesh(planeGeometry, planeMaterial);
  // plane.position.set(0, -2, 0);
  // //计算横切面
  // plane.rotation.x = -0.5 * Math.PI;
  // //平面接收阴影
  // plane.receiveShadow = true;
  // scene.add(plane);

  // 平行光
  const spotLight = new THREE.DirectionalLight(0xffffff)
  spotLight.position.set(10, 10, 10)
  //允许当前光产生阴影
  spotLight.castShadow = true
  scene.add(spotLight)

  /**
   * 三元坐标系
   * xyz
   * */
  const geometry = new THREE.BoxGeometry(4, 4, 4)
  //面颜色
  const material = new THREE.MeshLambertMaterial({color: 0x44aa88})
  const cube = new THREE.Mesh(geometry, material)
  //立方体和球投射阴影
  cube.castShadow = true
  scene.add(cube)

  //后端动态
  // const loader = new THREE.TextureLoader();
  // scene.background = loader.load('https://threejs.org/manual/examples/resources/images/daikanyama.jpg');

  function animate(time) {
    // 将时间转换为秒
    time *= 0.001;
    cube.rotation.x = time;
    cube.rotation.y = time;
    //元素写入画板
    renderer.render(scene, camera)
    ////循环函数获取每一帧
    requestAnimationFrame(animate)
  }
  animate()
}

//初始化函数防止画布未加载
onMounted(()=>{
  initThree()
})

</script>

<style scoped>
#first {
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