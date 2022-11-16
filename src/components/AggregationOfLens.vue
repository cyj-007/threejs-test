<template>
  <canvas id="five"></canvas>
  <div class="tag" id="tag">
    <span style="color:white;font-size: 10px;padding: 5px">测试自定义文字：</span>
    <span style="font-size: 11px;font-weight: bold">测试自定义文字</span>
    <p style="padding: 5px;margin-top: -3px;">测试自定义文字</p>
  </div>
</template>

<script setup>
import right from "../assets/space/right.jpg"; //天空盒子
import left from "../assets/space/left.jpg";
import top from "../assets/space/top.jpg";
import bottom from "../assets/space/bottom.jpg";
import front from "../assets/space/front.jpg";
import back from "../assets/space/back.jpg";
import * as THREE from "three";
import {onMounted} from "vue";
//鼠标控制器
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls";
//obj加载
import {OBJLoader} from 'three/examples/jsm/loaders/OBJLoader.js';
//自定义材质加载
import {MTLLoader} from 'three/examples/jsm/loaders/MTLLoader.js';
// 补间动画,控制镜头
import TWEEN from '@tweenjs/tween.js'
//场景
let scene = null
//渲染器
let renderer = null
//相机
let camera = null
//轨道控制
let mouseControls = null
//模型mash基本信息,用于选取
let oldChildren = []
//保留obj材质
let textures = []
// 实时获取起点位置三维轴
let positionOfCamera = {
  x:10,
  y:15,
  z:100
}
// 指定目标点位
let intendedTarget = {
  x: 0,
  y: 0,
  z: 0
}

function main() {
  //场景
  scene = new THREE.Scene();

  //相机
  camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.01, 1000)
  // 设置相机位置
  camera.position.set(positionOfCamera.x, positionOfCamera.y, positionOfCamera.z)
  //摄像机镜头指向的具体坐标位置
  camera.lookAt(scene.position);

  //获取id
  const canvas = document.querySelector('#five');
  //渲染器
  renderer = new THREE.WebGLRenderer({
    canvas,
    alpha: true, //是否背景可以透明
    antialias: true //抗拒齿
  })
  //设置阴影的类型，默认是 THREE.PCFShadowMap
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  //像素设置
  renderer.setPixelRatio(window.devicePixelRatio);
  //设置画布大小
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearAlpha(0.2);
  //生成坐标
  coordinate()
  //加载obj
  // loadModel('models/test.obj')
  loadModel('https://threejs.org/manual/examples/resources/models/windmill/windmill.obj')
  //声明鼠标控制，问题方向移动坐标系指向问题
  createOrbitControls()
  //创建盒子背景图
  createSkyBox()
  //创建光源
  createLight()
  //地面
  theGroundLoad()
  //添加窗口大小监听事件
  window.addEventListener('resize', onResize, false);
  //点击事件
  document.addEventListener('click', onDocumentMouseDown, false);
  //定时器每帧渲染
  render()
}

/**
 * 相机移动方法
 */
function action(){
  new TWEEN.Tween(positionOfCamera)
      .to(intendedTarget, 1000) // 指定目标位置和耗时.
      .easing(TWEEN.Easing.Quadratic.Out) // 指定动画效果曲线.
      .onUpdate(() => {
        console.log("111")
        // 渲染时每一帧执行：设定相机位置
        camera.position.set(positionOfCamera.x,positionOfCamera.y,positionOfCamera.z);
      })
      .onComplete(()=>{
        console.log("1111")
        // 动画结束后执行
      })
      .start() // 即刻开启动画
}
/**
 * 加载obj模型和材质
 *
 * MTLLoader.load("./材质.mtl",function(materials){
 * 	console.log(materials);  //返回一个包含材质的对象MaterialCreator
 * 	OBJLoader.setMaterials(materials);   //obj模型会和MaterialCreator包含的材质对应起来
 * 	OBJLoader.load("./几何体.obj",function(obj){
 * 		console.log(obj);
 * 		obj.scale.set(10,10,10);  //设置obj模型的大小
 * 		scene.add(obj);   //将载入的obj模型添加到场景中
 * 	})
 * })
 * */
function loadModel(item) {
  //利用Promise加载模型
  let lm = new Promise((resolve, reject) => {
    const objLoader = new OBJLoader();
    let materials = new MTLLoader()
    materials.load('models/test.mtl', function (mtlParseResult) {
      console.log(mtlParseResult)
      objLoader.setMaterials(mtlParseResult);
      objLoader.load(item, (gltf) => {
        resolve(gltf) //返回加载成功的模型
        reject("model加载失败！")
      });
    })
  });
  //加载成功
  lm.then((res) => {
    console.log("model加载成功！")
    for (let child of res.children) {
      //设置大小为原来的倍数，本文件是1
      // child.scale.set(0.01,0.01,0.01);
      //设置材质颜色
      // child.material.color.set(0xff0000);
    }
    //获取材质全局定义可变
    textures = res.children
    //到时候获取颜色全局可变

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
  mouseControls.enablePan = true; //右键平移拖拽
  mouseControls.enableZoom = true; //鼠标缩放
  mouseControls.minDistance = 10; //相机距离原点的距离范围
  mouseControls.maxDistance = 600;// 最大距离
  mouseControls.enableDamping = false; //滑动阻尼
  //动态阻尼系数 就是鼠标拖拽旋转灵敏度
  mouseControls.dampingFactor = 1; //(默认.25)
  mouseControls.maxPolarAngle = (Math.PI / 4) * 2; //y旋转角度范围，下面水平
  mouseControls.minPolarAngle = Math.PI / 4 *0.1; //上面垂直
  //开启自传，实时轮询更新，不然false
  // mouseControls.autoRotate = true; //自转(相机)
  // mouseControls.autoRotateSpeed = 1; //自转速度
  mouseControls.autoRotate = false; //自转(相机)
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
  const ambientLight = new THREE.AmbientLight(0xadadad,1); //设置环境光
  scene.add(ambientLight); //将环境光添加到场景中
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
function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
  //获取三维轴
  positionOfCamera.x = camera.position.x
  positionOfCamera.y = camera.position.y
  positionOfCamera.z = camera.position.z
}
//循环处理页面渲染问题，业务可以存放，渲染不要过多
function render() {
  //实时更新轨道控制
  mouseControls.update()
  divRender({x:3,y:4,z:6});
  renderer.render(scene, camera);
  //添加实时更新，不然TWEEN没效果
  TWEEN.update();
  requestAnimationFrame(render);
}
//计算坐标位置,传入固定值
function divRender(data) {
  //计算三维坐标对应的屏幕坐标
  const position = new THREE.Vector3(data.x,data.y,data.z);
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
  let div = document.getElementById('tag')
  div.style.display = "";
  div.style.left = left + 'px';
  div.style.top = top + 'px';
}
//点击事件
function onDocumentMouseDown(event) {
  console.log(event)
  event.preventDefault();
  //还原obj材质
  for (let child of scene.children) {
    if (child.type === 'Group') {
      for (let texture of textures) {
        // console.log(texture.material.color.getStyle())
        texture.material.color.setStyle("rgb(255,255,255)")
      }
    }
  }
  // 声明 raycaster 和 mouse 变量
  let mouse = new THREE.Vector2();
  //将鼠标点击位置的屏幕坐标转成threejs中的标准坐标,具体解释见代码释义
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
  //新建一个三维单位向量 假设z方向就是0.5
  //根据照相机，把这个向量转换到视点坐标系
  let vector = new THREE.Vector3(mouse.x, mouse.y,0.5).unproject(camera);
  //在视点坐标系中形成射线,射线的起点向量是照相机， 射线的方向向量是照相机到点击的点，这个向量应该归一标准化。
  let raycaster = new THREE.Raycaster(camera.position, vector.sub(camera.position).normalize());
  //射线和模型求交，选中一系列直线
  let intersects = raycaster.intersectObjects(textures)
  console.log(intersects)
  //校验是否与模型重叠
  if (intersects.length > 0){
    //改变模型颜色
    for ( let i = 0; i < textures.length; i++ ) {
      // 获取选中的 Mesh 对象
      // if (intersects.length !== 0 && intersects[i].object instanceof THREE.Mesh) {
      textures[i].material.color.set(0xff0000);
      // }
    }
    action()

    let selected = intersects[0];//取第一个物体
    intendedTarget.x = selected.point.x + 15
    intendedTarget.y = selected.point.y + 15
    intendedTarget.z = selected.point.z + 15
  }
}
onMounted(()=>{
  main()
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