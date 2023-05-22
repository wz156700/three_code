<template>
  <router-view></router-view>
</template>
<script>
//银河星系
import { defineComponent, onMounted } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { Clock } from "three";
// import model from "./three/model.js";
export default defineComponent({
  setup() {
    //场景
    const scene = new THREE.Scene();

    const params = {
      count: 100,
      size: 1,
      radius: 5,
      branch: 3,
      color: "#ffffff",
    };

    let geometry = null;
    let material = null;
    let points = null;

    //生成星系的函数
    const generateGalaxy = () => {
      //生成顶点
      geometry = new THREE.BufferGeometry();
      //随机生成位置
      const positions = new Float32Array(params.count * 3);

      // 设置顶点颜色
      const colors = new Float32Array(params.count * 3);
      // 循环生成点
      for (let i = 0; i < params.count; i++) {
        //
        const current = i * 3;
        //x
        positions[current] = Math.random() * params.radius;
        //y
        positions[current + 1] = 0;
        //z
        positions[current + 2] = 0;
      }
      geometry.setAttribute(
        "position",
        new THREE.BufferAttribute(positions, 3)
      );

      //载入纹理
      const textureLoader = new THREE.TextureLoader();
      const texture = textureLoader.load("./imgs/1.png");

      // 设置点材质
      material = new THREE.PointsMaterial({
        size: params.size,
        sizeAttenuation: true,
        depthWrite: true,
        blending: THREE.AdditiveBlending,
        map: texture,
        alphaMap: texture,
        transparent: true,
        // vertexColors: true, //设置顶点颜色
      });

      points = new THREE.Points(geometry, material);
      scene.add(points);
    };

    // 调用
    generateGalaxy();

    console.log(scene);
    //辅助观察的坐标系
    const axesHelper = new THREE.AxesHelper(100);
    scene.add(axesHelper);

    //光源设置
    const ambient = new THREE.AmbientLight(0xffffff, 0.4);
    scene.add(ambient);

    //相机
    const width = window.innerWidth;
    const height = window.innerHeight;
    // const width = 1600;
    // const height = 800;
    const camera = new THREE.PerspectiveCamera(30, width / height, 0.1, 1000);
    camera.position.set(0, 0, 40);
    camera.lookAt(0, 0, 0);

    // WebGL渲染器设置
    const renderer = new THREE.WebGLRenderer({
      antialias: true, //开启优化锯齿
    });
    renderer.setPixelRatio(window.devicePixelRatio); //防止输出模糊
    renderer.setSize(width, height);
    document.body.appendChild(renderer.domElement);

    const controls = new OrbitControls(camera, renderer.domElement);
    //设置时钟
    const clock = new THREE.Clock();
    // 渲染循环
    function render() {
      //   let time = clock.getElapsedTime();

      controls.update();
      renderer.render(scene, camera);
      requestAnimationFrame(render);
    }
    render();
    // 画布跟随窗口变化
    window.onresize = function () {
      renderer.setSize(window.innerWidth, window.innerHeight);
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
    };
  },
});
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

canvas {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  margin: auto;
}
</style>
