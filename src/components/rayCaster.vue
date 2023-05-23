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

    //辅助观察的坐标系
    const axesHelper = new THREE.AxesHelper(100);
    scene.add(axesHelper);

    //光源设置
    const ambient = new THREE.AmbientLight(0xffffff, 0.4);
    scene.add(ambient);

    //相机
    const width = window.innerWidth;
    const height = window.innerHeight;
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

    const geometry = new THREE.BoxGeometry(1, 1, 1);
    const material = new THREE.MeshBasicMaterial({
      wireframe: true,
    });
    const redMaterial = new THREE.MeshBasicMaterial({
      color: "#ff0000",
    });
    let cubeArr = [];

    for (let i = -5; i < 5; i++) {
      for (let j = -5; j < 5; j++) {
        for (let k = -5; k < 5; k++) {
          const cube = new THREE.Mesh(geometry, material);
          cube.position.set(i, j, k);
          scene.add(cube);
          cubeArr.push(cube);
        }
      }
    }
    //创建投射光线对象
    const rayCaster = new THREE.Raycaster();

    //鼠标的位置对象
    let mouse = new THREE.Vector2();
    //监听鼠标移动事件
    window.addEventListener("click", function (event) {
      mouse.x = (event.offsetX / window.innerWidth) * 2 - 1;
      mouse.y = -((event.offsetY / window.innerHeight) * 2 - 1);
      rayCaster.setFromCamera(mouse, camera);
      let results = rayCaster.intersectObjects(cubeArr);
      //   results[0].object.material = redMaterial;
      results.forEach((item) => {
        item.object.material = redMaterial;
      });
    });
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
