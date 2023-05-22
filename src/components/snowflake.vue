<template></template>
<script>
//例子效果
import { defineComponent, onMounted } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { Clock } from "three";
// import model from "./three/model.js";
export default defineComponent({
  setup() {
    //场景
    const scene = new THREE.Scene();

    function createPoints(imgName, size) {
      // 创建自定义顶点
      const particlesGemetry = new THREE.BufferGeometry();
      const count = 2000; //顶点数量

      // 设置缓冲区数组
      //顶点位置
      const positions = new Float32Array(count * 3);
      //设置顶点颜色
      const colors = new Float32Array(count * 3);

      // 设置顶点
      for (let i = 0; i < count * 3; i++) {
        positions[i] = [Math.random() - 0.5] * 100;
        colors[i] = Math.random();
      }

      particlesGemetry.setAttribute(
        "position",
        new THREE.BufferAttribute(positions, 3)
      );

      particlesGemetry.setAttribute(
        "color",
        new THREE.BufferAttribute(colors, 3)
      );

      //设置点材质
      const pintsMaterial = new THREE.PointsMaterial({
        size: size,
        sizeAttenuation: true, // 相机深度而衰减
        depthWrite: false,
        blending: THREE.AdditiveBlending, // 采用何种混合模式
      });

      //载入纹理
      const textureLoader = new THREE.TextureLoader();
      const texture = textureLoader.load(`./imgs/${imgName}.png`);
      pintsMaterial.map = texture;
      pintsMaterial.alphaMap = texture; //设置透明纹理
      pintsMaterial.transparent = true; //开启透明度
      pintsMaterial.depthWrite = false; //是否对深度缓冲区有任何影响,分层时使用
      pintsMaterial.vertexColors = true; // 启用顶点颜色设置

      const points = new THREE.Points(particlesGemetry, pintsMaterial);
      scene.add(points);
      return points;
    }

    const points = createPoints("1", 3);
    const points2 = createPoints("3", 6);

    //辅助观察的坐标系
    const axesHelper = new THREE.AxesHelper(100);
    scene.add(axesHelper);

    //光源设置
    const ambient = new THREE.AmbientLight(0xffffff, 0.4);
    scene.add(ambient);

    //相机
    // const width = window.innerWidth;
    // const height = window.innerHeight;
    const width = 1600;
    const height = 800;
    const camera = new THREE.PerspectiveCamera(30, width / height, 0.1, 50);
    camera.position.set(0, 0, 100);
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
      let time = clock.getElapsedTime();
      points.rotation.x = time * 0.2;
      points2.rotation.x = time * 0.3;
      points2.rotation.y = time * 0.1;
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
  beforeRouteLeave(to, from, next) {
    // console.log(to, from, next);
    console.log(document.body);
    document.body.removeChild(document.getElementsByTagName("canvas")[0]);
  },
});
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

canvas {
  width: 100vw;
  height: 100vh;
}
</style>
