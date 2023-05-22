<template>
  <router-view></router-view>
</template>
<script>
//银河星系
import { defineComponent, onMounted } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { Clock } from "three";
export default defineComponent({
  setup() {
    //场景
    const scene = new THREE.Scene();

    const params = {
      count: 12000,
      size: 0.3,
      radius: 5,
      branch: 6,
      color: "#ff6030",
      rotateScale: 0.6,
      endcolor: "#1b3984",
    };

    let geometry = null;
    let material = null;
    let points = null;
    //开始颜色
    const centerColor = new THREE.Color(params.color);
    // 结束颜色
    const endColor = new THREE.Color(params.endcolor);

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
        // 当前点应该在哪一条分支的角度上
        const branchAngel =
          (i % params.branch) * ((2 * Math.PI) / params.branch);

        // 当前点距离圆心的位置
        const distance =
          Math.random() * params.radius * Math.pow(Math.random(), 3);
        //当前时哪一个点
        const current = i * 3;
        //xyz上的随机值
        const randomX =
          Math.pow(Math.random() * 2 - 1, 3) * (params.radius - distance) * 0.2;
        const randomY =
          Math.pow(Math.random() * 2 - 1, 3) * (params.radius - distance) * 0.2;
        const randomZ =
          Math.pow(Math.random() * 2 - 1, 3) * (params.radius - distance) * 0.2;

        // const randomX = Math.pow(Math.random() * 2 - 1, 3) * distance * 0.2;
        // const randomY = Math.pow(Math.random() * 2 - 1, 3) * distance * 0.2;
        // const randomZ = Math.pow(Math.random() * 2 - 1, 3) * distance * 0.2;

        //当前点的x,y,z位置
        //x
        positions[current] =
          Math.cos(branchAngel + distance * params.rotateScale) * distance +
          randomX;
        //y
        positions[current + 1] = 0 + randomY;
        //z
        positions[current + 2] =
          Math.sin(branchAngel + distance * params.rotateScale) * distance +
          randomZ;

        // 混合颜色形成渐变色
        const mixColor = centerColor.clone();
        mixColor.lerp(endColor, distance / params.radius);

        colors[current] = mixColor.r;
        colors[current + 1] = mixColor.g;
        colors[current + 2] = mixColor.b;
      }
      geometry.setAttribute(
        "position",
        new THREE.BufferAttribute(positions, 3)
      );
      geometry.setAttribute("color", new THREE.BufferAttribute(colors, 3));

      //载入纹理
      const textureLoader = new THREE.TextureLoader();
      const texture = textureLoader.load("./imgs/1.png");

      // 设置点材质
      material = new THREE.PointsMaterial({
        // color: new THREE.Color(params.color),
        size: params.size,
        sizeAttenuation: true,
        depthWrite: false,
        blending: THREE.AdditiveBlending,
        map: texture,
        alphaMap: texture,
        transparent: true,
        vertexColors: true, //设置顶点颜色
      });

      points = new THREE.Points(geometry, material);
      scene.add(points);
    };

    // 调用
    generateGalaxy();
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
    camera.position.set(10, 20, 10);
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
