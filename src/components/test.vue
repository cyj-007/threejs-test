<template>
  <div>
    <canvas id="five"></canvas>
    <div class="tag" id="tagReminder" style="display: none">
      <span style="color:white;font-size: 10px">测试自定义文字：</span>
      <span style="font-size: 11px;font-weight: bold">测试自定义文字</span>
      <p>测试自定义文字</p>
      <p>测试自定义文字</p>
      <p>测试自定义文字</p>
      <p>测试自定义文字</p>

      <span class="close" @click="padlock"></span>
    </div>
    <div id="tofile" ></div>
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
// 补间动画,控制镜头
import TWEEN from '@tweenjs/tween.js'
// json数据集
import jsonData from "../datapackage/package.json"
//
import tp from "../assets/logo.png"

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
let texturesOne = []
let texturesTwo = []
//保留原始颜色，用于还原
let primaryColour = {}
// 全局镜头开始位置，固定，用于还原
let positionOfCameraFixation = {
  x:20,
  y:70,
  z:100
}
// 实时获取起点位置三维轴
let positionOfCamera = {
  x:20,
  y:70,
  z:100
}
// 指定目标点位
let intendedTarget = {
  x: 0,
  y: 0,
  z: 0
}
//文字平面显示隐藏,标识符
let styleDisplay = "none"
//点击事件与滑轮
let clickPulleyControl = true
//旋转
let cord0=new THREE.Object3D;
let cord1=new THREE.Object3D;
let cord5=new THREE.Object3D;
//全局控制旋转的大小值
let aaaaaa = 0
let llllll = 0
//旋转点位数据集合
let bbbbbb = []
//滑轮点位集
let eeeeee = [
  'Obj3d66_740686_95_239',
  'Obj3d66_740686_97_828',
  'Obj3d66_740686_98_209'
]
//吊钩
let gggggg = [
  'Obj3d66_740686_96_688'
]
//obj数据采集出来进行转换，防止数据丢失bug
let ccccccc = []
let fffffff = []
let hhhhhhh = []
//测试获取数据点位名称标识符
let ddddddd = []


var fixedBoardlist=[];

function main() {
  bbbbbb = jsonData

  //场景
  scene = new THREE.Scene();

  //相机
  camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.01, 1000)
  // 设置相机位置
  camera.position.set(positionOfCameraFixation.x, positionOfCameraFixation.y, positionOfCameraFixation.z)
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
  loadModel('models/test.obj')
  // loadModel('https://threejs.org/manual/examples/resources/models/windmill/windmill.obj')
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
  //键盘监听时间
  document.addEventListener( 'keydown', onKeyDown, false );
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
        // 渲染时每一帧执行：设定相机位置
        camera.position.set(positionOfCamera.x,positionOfCamera.y,positionOfCamera.z);
      })
      .onComplete(()=>{
        // 动画结束后执行
        //开启提示信息，相机移动是异步加载，文字提示要在动画结束执行，不然定位错误
        if (styleDisplay !== "") {
          styleDisplay = ""
          divRender({x:10,y:10,z:0})
        }
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
    // let materials = new MTLLoader()
    // materials.load('models/test.mtl', function (mtlParseResult) {
    //   console.log(mtlParseResult)
    //   objLoader.setMaterials(mtlParseResult);
      objLoader.load(item, (gltf) => {
        resolve(gltf) //返回加载成功的模型
        reject("model加载失败！")
      })
    // })
  })
  //加载成功
  lm.then((res) => {
    console.log("model加载成功！")
    let cosde = new Map()
    for (let child of res.children) {
      //设置大小为原来的倍数，本文件是1
      child.scale.set(0.01,0.01,0.01);
      //获取原色
      cosde.set(child.name,child.material.color.getStyle())
      //测试数据集
      ddddddd.push(child.name)
      //取出来拼装防止材料丢失
      if (bbbbbb.includes(child.name)) {
        ccccccc.push(child)
      }
      if (eeeeee.includes(child.name)) {
        fffffff.push(child)
      }
      if (gggggg.includes(child.name)) {
        hhhhhhh.push(child)
      }
    }
    //获取原色
    primaryColour = cosde
    //控制模型位置
    res.position.set(0, 0, 0)
    //复制分组
    //塔基
    for (let cccccccElement of ccccccc) {
      cord5.add(cccccccElement)
    }
    //滑轮
    for (let fffffffElement of fffffff) {
      cord0.add(fffffffElement)
    }
    //吊钩
    for (let hhhhhhhElement of hhhhhhh) {
      cord1.add(hhhhhhhElement)
    }
    //旋转塔基做主数组，其他自由组合，控制组合配置
    cord0.add(cord1)
    cord5.add(cord0)
    scene.add(cord5)

    //获取材质全局定义可变
    texturesOne = res.children
    texturesTwo = cord5.children

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
  mouseControls.enablePan = true //右键平移拖拽
  mouseControls.enableZoom = true; //鼠标缩放
  mouseControls.minDistance = 1; //相机距离原点的距离范围
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
  // const texture = loader.load('https://threejs.org/manual/examples/resources/images/checker.png');
  const planeGeo = new THREE.PlaneGeometry(planeSize, planeSize);
  const planeMat = new THREE.MeshPhongMaterial({
    // map: texture,
    side: THREE.DoubleSide,
    color: '#000000'
  });
  const mesh = new THREE.Mesh(planeGeo, planeMat);
  mesh.rotation.x = Math.PI * -0.5;
  scene.add(mesh);
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
  let div = document.getElementById('tagReminder')
  div.style.display = styleDisplay;
  div.style.left = left + 'px';
  div.style.top = top + 'px';
}
//点击事件
function onDocumentMouseDown(event) {
  addFixedBoard(event)
  //防止重复点击获取同一模型
  if (clickPulleyControl){
    event.preventDefault();
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
    // //射线和模型求交，选中一系列直线
    let intersects = raycaster.intersectObjects(texturesOne)
    //测试数据集
    // intersects.filter(item=>{
    //   if (ddddddd.includes(item.object.name)) {
    //     console.log(item.object.name)
    //   }
    // })

    //校验是否与模型重叠
    if (intersects.length > 0){
      //改变模型颜色
      for ( let i = 0; i < texturesOne.length; i++ ) {
        // 获取选中的 Mesh 对象
        // if (intersects.length !== 0 && intersects[i].object instanceof THREE.Mesh) {
        texturesOne[i].material.color.set(0xff0000);
        // }
      }
      if (intersects.length > 0){
        //地址位置
        let selected = intersects[0];//取第一个物体
        intendedTarget.x = selected.point.x + 25
        intendedTarget.y = selected.point.y + 25
        intendedTarget.z = selected.point.z + 25
      }
      action()
      clickPulleyControl = false
      //关闭缩放
      // mouseControls.enableZoom = clickPulleyControl
      // mouseControls.enablePan = clickPulleyControl
    }
  }
}
// 键盘监听时间
function onKeyDown(event){
  switch (event.key) {
    case 'a':
      //旋转
      cord5.rotation.y-=0.1
      console.log("顺时针")
          break
    case 'd':
      cord5.rotation.y+=0.1
      console.log("逆时针")
          break
    case 'w':
      //定位
      aaaaaa += 0.2
      cord0.position.set(aaaaaa, 0, 0)
      console.log("前进")
      break
    case 's':
      //定位
      aaaaaa -= 0.2
      cord0.position.set(aaaaaa, 0, 0)
      console.log("后退")
          break
    case 'z':
      //定位
      llllll += 0.2
      cord1.position.set(0, llllll, 0)
      console.log("上升")
      break
    case 'x':
      //定位
      llllll -= 0.2
      cord1.position.set(0, llllll, 0)
      console.log("下降")
      break
    default:
      console.log("无法识别")
  }
}
//关闭显示框
function padlock(){
  //还原obj材质
  for (let child of scene.children) {
    if (child.type === 'Group') {
      for (let texture of texturesOne) {
        let newVar = primaryColour.get(texture.name);
        if (newVar !== undefined) {
          texture.material.color.setStyle(newVar)
        }
      }
    }
  }
  //还原鼠标控制缩放
  clickPulleyControl = true
  mouseControls.enableZoom = clickPulleyControl
  mouseControls.enablePan = clickPulleyControl
  //还原视角定位
  camera.position.set(positionOfCameraFixation.x, positionOfCameraFixation.y, positionOfCameraFixation.z)
  camera.updateProjectionMatrix()
  positionOfCamera = JSON.parse(JSON.stringify(positionOfCameraFixation))
  //隐藏气泡窗
  let div = document.getElementById('tagReminder')
  styleDisplay = "none"
  div.style.display = styleDisplay;
}
//自适应
function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}
//循环处理页面渲染问题，业务可以存放，渲染不要过多
function render() {
  //实时更新轨道控制
  mouseControls.update()
  renderer.render(scene, camera)
  //添加实时更新，不然TWEEN没效果
  TWEEN.update();

  // const time = Date.now() * 0.005
  // cord0.rotation.y = Math.cos( time ) * 0.75 + 2.25
  requestAnimationFrame(render)
}
onMounted(()=>{
  main()
})


//固定位置的标签
function addFixedBoard(event) {
  var windowX = event.clientX;//鼠标单击位置横坐标
  var windowY = event.clientY;//鼠标单击位置纵坐标

  var addDivDom = document.getElementById('tofile');
  var bodyDom = document.body;
  bodyDom.insertBefore(addDivDom, bodyDom.lastChild);
  // addDivDom.id=cube.uuid+"_text";
  var img=new Image();
  img.src=tp
  img.width = 50
  img.height = 50
  addDivDom.appendChild(img);

  var x = (windowX / window.innerWidth) * 2 - 1;//标准设备横坐标
  var y = -(windowY / window.innerHeight) * 2 + 1;//标准设备纵坐标
  var standardVector = new THREE.Vector3(x, y, 0.5);//标准设备坐标
  //标准设备坐标转世界坐标
  var worldVector = standardVector.unproject(camera);
  var ray = worldVector.sub(camera.position).normalize();
  //创建射线投射器对象
  var raycaster = new THREE.Raycaster(camera.position, ray);
  //返回射线选中的对象
  var intersects = raycaster.intersectObjects(texturesOne);
  if (intersects.length > 0) {
    var point=intersects[0].point;
    let fixedBoard = {
      position:point,
      div:addDivDom,
      image:img
    }
    fixedBoardlist.push(fixedBoard);
  }
  console.log(fixedBoardlist)
}

//实时修改渲染时的图片位置
function imagePosition() {
  for(var i=0;i<fixedBoardlist.length;i++){
    var position=fixedBoardlist[i].position;
    var div=fixedBoardlist[i].div;
    var x=transPosition(position).x;
    var y=transPosition(position).y;
    var image=fixedBoardlist[i].image;
    var width=image.width;
    var height=image.height;
    div.style.top=y-height+ 'px';
    div.style.left=x-width/2+ 'px';
  }
}
//三维坐标转屏幕坐标
function transPosition (position) {
  let world_vector = new THREE.Vector3(position.x, position.y, position.z);
  let vector = world_vector.project(camera);
  let halfWidth = window.innerWidth / 2,
      halfHeight = window.innerHeight / 2;
  return {
    x: Math.round(vector.x * halfWidth + halfWidth),
    y: Math.round(-vector.y * halfHeight + halfHeight)
  };
}
</script>

<style scoped>
#five {
  width: 100%;
  height: 100%;
  /*禁止滑轮*/
  position: fixed;
}
/**弹窗设置*/
.tag {
  position: absolute;
  background-color: MidnightBlue;
  background-color: rgba(0, 10, 40);
  border-radius: 10px;
  opacity: 0.5;
  font-size: 4px;
  color: aqua;
  padding: 5px;
  -moz-user-select:none; /* Firefox私有属性 */
  -webkit-user-select:none; /* WebKit内核私有属性 */
  -ms-user-select:none; /* IE私有属性(IE10及以后) */
  -khtml-user-select:none; /* KHTML内核私有属性 */
  -o-user-select:none; /* Opera私有属性 */
  user-select:none; /* CSS3属性 */
}
/**关闭按钮设置*/
.close {
  cursor: pointer;
  background: orange;
  color: #000000;
  border-radius: 12px;
  line-height: 13px;
  text-align: center;
  height: 15px;
  width: 15px;
  font-size: 16px;
  padding: 1px;
  top: -8px;
  right: -9px;
  position: absolute;
}
/* 使用十字作为关闭按钮 */
.close::before {
  content: "\2716";
}
</style>