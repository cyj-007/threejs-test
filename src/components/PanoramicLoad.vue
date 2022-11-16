<template>
  <canvas id="fourth"></canvas>
</template>

<script setup>
import right from "../assets/space/right.jpg"; //天空盒子
import left from "../assets/space/left.jpg";
import top from "../assets/space/top.jpg";
import bottom from "../assets/space/bottom.jpg";
import front from "../assets/space/front.jpg";
import back from "../assets/space/back.jpg";
import * as THREE from "three";
import {onMounted, reactive} from "vue";
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls";
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader.js';

//场景
let scene = null
//渲染器
let renderer = null
//相机
let camera = null
//轨道控制
let mouseControls = null


function main() {
  //获取id
  const canvas = document.querySelector('#fourth');
  //渲染器
  renderer = new THREE.WebGLRenderer({
    canvas,
    alpha: true, //是否背景可以透明
    antialias: true //抗拒齿
  });
  //相机
  camera = new THREE.PerspectiveCamera(45, 75, 0.01, 1000);
  // 扩大视野范围
  // camera.position.set(10, 10, 40);
  camera.position.set(30, 30, 30);
  //场景
  scene = new THREE.Scene();
  //加载obj
  loadModel('https://threejs.org/manual/examples/resources/models/windmill/windmill.obj')
  //声明鼠标控制
  createOrbitControls()
  //生成坐标
  coordinate()
  //创建盒子背景图
  createSkyBox()
  //创建光源
  createLight()
  //地面
  theGroundLoad()
  //定时器每帧渲染
  requestAnimationFrame(render);
}
//加载obj模型
function loadModel(item) {
  //利用Promise加载模型
  let lm = new Promise((resolve, reject) => {
    const objLoader = new OBJLoader();
    objLoader.load(item, (gltf) => {
      gltf.traverse((child) => {
        if (child instanceof THREE.Mesh) {
          child.material.side = THREE.DoubleSide
          // child.scale.set(50, 50, 50)
        }
      })
      resolve(gltf) //返回加载成功的模型
      reject("model加载失败！")
    });
  });
  //加载成功
  lm.then((res) => {
    console.log("model加载成功！")
    //控制模型位置
    res.position.set(0, 0, 0)
    scene.add(res);
  });
  //加载失败
  lm.catch((err) => {
    console.log(err)
  });
}
//创建轨道控制
function createOrbitControls() {
  //没有缩放阻尼
  mouseControls = new OrbitControls(
      camera,
      renderer.domElement
  );
  //创建控件对象
  mouseControls.enablePan = false; //右键平移拖拽
  mouseControls.enableZoom = true; //鼠标缩放
  mouseControls.minDistance = 0; //相机距离原点的距离范围
  // mouseControls.maxDistance = 100;
  mouseControls.enableDamping = true; //滑动阻尼
  mouseControls.dampingFactor = 0.1; //(默认.25)
  // mouseControls.maxPolarAngle = (Math.PI / 4) * 3; //y旋转角度范围
  // mouseControls.minPolarAngle = Math.PI / 4;
  mouseControls.autoRotate = true; //自转(相机)
  mouseControls.autoRotateSpeed = 1; //自转速度
}
//创建天空盒子
function createSkyBox() {
  //加载天空盒子纹理
  scene.background = new THREE.CubeTextureLoader().load([
    right,
    left,
    top,
    bottom,
    front,
    back,
  ]); //设置场景背景
}
//场景球体全景
function createPanorama(img) {
  let geometry = new THREE.SphereGeometry(500, 100, 100);
  let material = new THREE.MeshBasicMaterial({
    map: new THREE.TextureLoader().load(img), //导入图片
    color: 0xffffff,
    side: THREE.BackSide,
  });
  let mesh = new THREE.Mesh(geometry, material);
  scene.add(mesh);
}
//坐标
function coordinate(){
  //坐标
  const axes = new THREE.AxesHelper(300); //坐标系(辅助开发)
  scene.add(axes);//辅助开发坐标
}
//创建光源
function createLight() {
  const ambient = new THREE.AmbientLight("#F08080",0.5);
  scene.add(ambient);
  // const ambientLight = new THREE.AmbientLight(0xffffff); //设置环境光
  // scene.add(ambientLight); //将环境光添加到场景中
  const pointLight = new THREE.PointLight(0xffffff, 1, 0);
  pointLight.position.set(10, 10, 40); //设置点光源位置
  scene.add(pointLight); //将点光源添加至场景
}
//地面加载
function theGroundLoad(){
  const planeSize = 100;

  const loader = new THREE.TextureLoader();
  //地面图片
  const texture = loader.load('https://threejs.org/manual/examples/resources/images/checker.png');
  const planeGeo = new THREE.PlaneGeometry(planeSize, planeSize);
  const planeMat = new THREE.MeshPhongMaterial({
    map: texture,
    side: THREE.DoubleSide,
  });
  const mesh = new THREE.Mesh(planeGeo, planeMat);
  mesh.rotation.x = Math.PI * -0.5;
  scene.add(mesh);
}
//自适应
function resizeRendererToDisplaySize(renderer) {
  const canvas = renderer.domElement;
  const width = canvas.clientWidth;
  const height = canvas.clientHeight;
  const needResize = canvas.width !== width || canvas.height !== height;
  if (needResize) {
    renderer.setSize(width, height, false);
  }
  return needResize;
}
function render() {
  if (resizeRendererToDisplaySize(renderer)) {
    const canvas = renderer.domElement;
    camera.aspect = canvas.clientWidth / canvas.clientHeight;
    camera.updateProjectionMatrix();
  }
  //实时更新轨道控制
  mouseControls.update()
  renderer.render(scene, camera);
  requestAnimationFrame(render);
}
onMounted(()=>{
  main()
})
</script>


<style scoped>
#fourth {
  width: 100%;
  height: 100%;
  /*定位解决加上面宽高刷新解决自适应问题*/
  position: fixed;
}
</style>