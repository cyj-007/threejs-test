<template>
  <div id="WebGL-output">
    <div class="tag" id="tags">
      <span style="color:white;font-size: 10px;padding: 5px">测试自定义文字：</span>
      <span style="font-size: 11px;font-weight: bold">测试自定义文字</span>
      <p style="padding: 5px;margin-top: -3px;">测试自定义文字</p>
    </div>
  </div>
</template>

<script setup>
import * as THREE from 'three'
import {onMounted} from "vue"
//鼠标控制器
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls";

var scene;
var camera;
var renderer;
var controls2;

function initThree() {
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
  renderer = new THREE.WebGLRenderer({antialias: true, alpha: true});
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.shadowMap.enabled = true; //打开阴影
  renderer.setClearAlpha(0.2);
  var axis = new THREE.AxesHelper(200);
  scene.add(axis)
  //添加环境光
  var hemiLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 1);
  hemiLight.position.set(0, 500, 0);
  scene.add(hemiLight);
  //设置相机位置
  camera.position.set(0, 3, 300)
  camera.lookAt(scene.position);
  document.getElementById("WebGL-output").appendChild(renderer.domElement);
  //鼠标控制器
  initControls();
  animate();
  window.addEventListener('resize', onResize, false); //添加窗口大小监听事件
}

function initControls() {
  controls2 = new OrbitControls(camera, renderer.domElement);
  controls2.enableDamping = true;
  controls2.dampingFactor = 1;
  controls2.enableZoom = true;
  controls2.zoomSpeed = 2;
  controls2.autoRotate = false;

  controls2.minDistance = 0;
  controls2.maxDistance = 600;
  controls2.enablePan = true;
  controls2.keyPanSpeed = 15;
}

function animate() {
  //更新控制器
  controls2.update();
  divRender({x: 30, y: 40, z: 60});
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}

function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}

//计算坐标位置,传入固定值
function divRender(data) {
  //计算三维坐标对应的屏幕坐标
  const position = new THREE.Vector3(data.x, data.y, data.z);
  const world_vector = new THREE.Vector3(position.x, position.y, position.z);
  const vector = world_vector.project(camera);
  const halfWidth = window.innerWidth / 2,
      halfHeight = window.innerHeight / 2;
  const windowPosition = {
    x: Math.round(vector.x * halfWidth + halfWidth),
    y: Math.round(-vector.y * halfHeight + halfHeight)
  };
  const left = windowPosition.x;
  const top = windowPosition.y;
  //设置div屏幕位置
  let div = document.getElementById('tags');
  div.style.display = "";
  div.style.left = left + 'px';
  div.style.top = top + 'px';
}

//初始化函数防止画布未加载
onMounted(() => {
  initThree()
})

</script>

<style scoped>
.tag {
  position: absolute;
  background-color: MidnightBlue;
  background-color: rgba(0, 10, 40);
  border-top-left-radius: 10px;
  border-bottom-right-radius: 10px;
  opacity: 0.5;
  font-size: 4px;
  color: aqua;
  padding: 1px 1px 1px;
}

</style>