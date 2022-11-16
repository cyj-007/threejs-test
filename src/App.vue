<template>
  <div>
    <!-- 不要放el-tabs中，模型点击时间选择bug   -->
    <test></test>
  </div>

  <el-tabs v-model="activeName" style="margin: 10px" v-if="false">
<!--  <el-tabs v-model="activeName" style="margin: 10px">-->
    <el-tab-pane label="镜头聚合" name="five">
      <aggregation-of-lens></aggregation-of-lens>
    </el-tab-pane>
    <el-tab-pane label="文字提示" name="six">
      <text-prompt></text-prompt>
    </el-tab-pane>
    <el-tab-pane label="正方形背景旋转" name="first">
      <square-of-rotation></square-of-rotation>
    </el-tab-pane>
    <el-tab-pane label="obj导入" name="two">
      <kentucky-obj-import></kentucky-obj-import>
    </el-tab-pane>
    <el-tab-pane label="地面设置" name="third">
      <the-ground-is-set></the-ground-is-set>
    </el-tab-pane>
    <el-tab-pane label="全景加载+摄像头巡回或可周期" name="fourth">
      <panoramic-load></panoramic-load>
    </el-tab-pane>
  </el-tabs>
</template>

<script setup>
import HelloWorld from './components/TheGroundIsSet.vue'
import SquareOfRotation from "./components/SquareOfRotation.vue";
import {ref} from "vue";
import KentuckyObjImport from "./components/KentuckyObjImport.vue";
import test from "./components/test.vue";
import TheGroundIsSet from "./components/TheGroundIsSet.vue";
import PanoramicLoad from "./components/PanoramicLoad.vue";
import AggregationOfLens from "./components/AggregationOfLens.vue";
import TextPrompt from "./components/TextPrompt.vue";

const activeName = ref('five')



</script>
