<template>
  
</template>
<script>
import { defineComponent } from "vue";
import * as THREE from 'three'

export default defineComponent({
  setup() { 
    //获取页面宽高
    const width = window.innerWidth;
    const height = window.innerHeight;

    // 创建场景
    const scene = new THREE.Scene();
    console.log(scene);

    // 创建相机
    const camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
    console.log(camera);
  }
})
</script>

<style>
*{
  margin: 0;
  padding: 0;
}

canvas{
  width: 100vw;
  height: 100vh;
}
</style>
