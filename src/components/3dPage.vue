<template>
  <div class="containner">
    <div class="page page0">
      <h1>THREE.Point</h1>
      <h3>实现星空效果</h3>
    </div>
    <div class="page page1">
      <h1>THREE.Point</h1>
      <h3>实现臂旋星系</h3>
    </div>
    <div class="page page2">
      <h1>THREE.RayCaster</h1>
      <h3>实现3D交互</h3>
    </div>
  </div>
</template>
<script>
import { defineComponent, onMounted } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { Clock } from "three";
// typical import
import gsap from "gsap";

// 引入其他插件
import ScrollTrigger from "gsap/ScrollTrigger";
import Flip from "gsap/Flip";
import Draggable from "gsap/Draggable";

// 注册插件
gsap.registerPlugin(ScrollTrigger, Draggable, Flip);

export default defineComponent({
  setup() {
    //场景
    const scene = new THREE.Scene();
    //辅助观察的坐标系
    // const axesHelper = new THREE.AxesHelper(100);
    // scene.add(axesHelper);
    //光源设置
    // const ambient = new THREE.AmbientLight(0xffffff, 0.4);
    // scene.add(ambient);
    //相机
    const width = window.innerWidth;
    const height = window.innerHeight;
    const camera = new THREE.PerspectiveCamera(30, width / height, 0.1, 300);
    camera.position.set(0, 0, 40);
    camera.lookAt(0, 0, 0);
    // WebGL渲染器设置
    const renderer = new THREE.WebGLRenderer({
      antialias: true, //开启优化锯齿
      alpha: true, // 设置渲染器透明
    });
    renderer.setPixelRatio(window.devicePixelRatio); //防止输出模糊
    renderer.setSize(width, height);
    document.body.appendChild(renderer.domElement);

    //相机控件
    // const controls = new OrbitControls(camera, renderer.domElement);

    //创建立方体
    const geometry = new THREE.BoxGeometry(2, 2, 2);
    const material = new THREE.MeshBasicMaterial({
      wireframe: true,
    });
    const redMaterial = new THREE.MeshBasicMaterial({
      color: "#ff0000",
    });
    let cubeArr = [];
    let cubeGroup = new THREE.Group();
    for (let i = 0; i < 5; i++) {
      for (let j = 0; j < 5; j++) {
        for (let k = 0; k < 5; k++) {
          const cube = new THREE.Mesh(geometry, material);
          cube.position.set(i * 2 - 4, j * 2 - 4, k * 2 - 4);

          cubeGroup.add(cube);
          cubeArr.push(cube);
        }
      }
    }

    scene.add(cubeGroup);
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

    // 鼠标移动事件
    window.addEventListener("mousemove", function (event) {
      mouse.x = event.offsetX / window.innerWidth - 0.5;
      mouse.y = -(event.offsetY / window.innerHeight - 0.5);
    });
    //创建三角形
    let sjxGroup = new THREE.Group();
    for (let i = 0; i < 50; i++) {
      // 每一个三角形，需要3个顶点，每个顶点需要3个值
      const geometry = new THREE.BufferGeometry();
      const positionArray = new Float32Array(9);
      for (let j = 0; j < 9; j++) {
        if (j % 3 == 1) {
          positionArray[j] = Math.random() * 10 - 5;
        } else {
          positionArray[j] = Math.random() * 10 - 5;
        }
      }
      geometry.setAttribute(
        "position",
        new THREE.BufferAttribute(positionArray, 3)
      );
      let color = new THREE.Color(Math.random(), Math.random(), Math.random());
      const material = new THREE.MeshBasicMaterial({
        color: color,
        transparent: true,
        opacity: 0.5,
        side: THREE.DoubleSide,
      });
      // 根据几何体和材质创建物体
      let sjxMesh = new THREE.Mesh(geometry, material);
      sjxGroup.add(sjxMesh);
    }
    scene.add(sjxGroup);
    // 弹跳小球
    const sphereGroup = new THREE.Group();
    const sphereGeometry = new THREE.SphereBufferGeometry(1, 20, 20);
    const spherematerial = new THREE.MeshStandardMaterial({
      side: THREE.DoubleSide,
    });
    const sphere = new THREE.Mesh(sphereGeometry, spherematerial);
    // 投射阴影
    sphere.castShadow = true;
    // // 创建平面
    const planeGeometry = new THREE.PlaneBufferGeometry(20, 20);
    const plane = new THREE.Mesh(planeGeometry, spherematerial);
    plane.position.set(0, -1, 0);
    plane.rotation.x = -Math.PI / 2;
    // plane.rotation.y = -Math.PI / 2;
    // 接收阴影
    plane.receiveShadow = true;
    sphereGroup.add(plane);

    sphereGroup.add(sphere);
    sjxGroup.position.set(0, -30, 0);
    // 灯光
    // 环境光
    const light = new THREE.AmbientLight(0xffffff, 0.5); // soft white light
    sphereGroup.add(light);

    const smallBall = new THREE.Mesh(
      new THREE.SphereBufferGeometry(0.1, 20, 20),
      new THREE.MeshBasicMaterial({ color: 0xff0000 })
    );
    smallBall.position.set(2, 2, 2);
    //点光源
    const pointLight = new THREE.PointLight(0xffffff, 3);
    // pointLight.position.set(2, 2, 2);
    pointLight.castShadow = true;

    // 设置阴影贴图模糊度
    pointLight.shadow.radius = 20;
    // 设置阴影贴图的分辨率
    pointLight.shadow.mapSize.set(512, 512);

    // 设置透视相机的属性
    smallBall.add(pointLight);
    sphereGroup.add(smallBall);

    sphereGroup.position.set(0, -60, 0);
    scene.add(sphereGroup);

    let arrGroup = [cubeGroup, sjxGroup, sphereGroup];

    //设置时钟
    const clock = new THREE.Clock();

    //gsap设置动画
    gsap.to(cubeGroup.rotation, {
      x: "+=" + Math.PI,
      y: "+=" + Math.PI,
      duration: 5,
      repeat: -1,
      ease: "power1.inOut",
    });
    gsap.to(sjxGroup.rotation, {
      x: "+=" + Math.PI,
      z: "+=" + Math.PI,
      duration: 3,
      repeat: -1,
      ease: "power1.inOut",
    });

    gsap.to(smallBall.position, {
      x: -3,
      duration: 6,
      repeat: -1,
      ease: "power1.inOut",
      yoyo: true,
    });

    gsap.to(smallBall.position, {
      y: 0,
      duration: 0.5,
      repeat: -1,
      ease: "power1.inOut",
      yoyo: true,
    });

    // 渲染循环
    function render() {
      //   let time = clock.getElapsedTime(); // 总时间
      //   let deltatime = clock.getDelta(); // 时间差
      //   cubeGroup.rotation.x = time * 0.5;
      //   cubeGroup.rotation.y = time * 0.5;
      //   sjxGroup.rotation.x = time * 0.3;
      //   sjxGroup.rotation.z = time * 0.3;
      //   smallBall.position.x = Math.sin(time) * 3;
      //   smallBall.position.z = Math.cos(time) * 3;
      //   smallBall.position.y = 2 + Math.sin(time * 10) / 2;
      //   sphereGroup.rotation.z = Math.sin(time) * 0.05;
      //   sphereGroup.rotation.x = Math.sin(time) * 0.05;

      camera.position.y = -Math.round(window.scrollY / window.innerHeight) * 30;
      //   console.log(deltatime);
      camera.position.x += (mouse.x * 10 - camera.position.x) * 0.1;
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

    //设置当前页
    let currentPage = 0;
    //监听滚动事件
    window.addEventListener("scroll", (event) => {
      const scrollPage = Math.round(window.scrollY / window.innerHeight);
      if (scrollPage != currentPage) {
        currentPage = scrollPage;

        gsap.to(arrGroup[currentPage].rotation, {
          z: "+=" + Math.PI * 2,
          duration: 1,
        });

        gsap.fromTo(
          `.page${currentPage} h1`,
          { x: -300 },
          {
            x: 0,
            rotation: "+=" + 360,
            duration: 1,
          }
        );
      }
    });
  },
});
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
body {
  background-color: #a5c6cf;
  overflow-x: hidden;
}

.page {
  width: 100vw;
  height: 100vh;
  color: white;
}
.page h1,
h3 {
  text-align: center;
  font-size: 30px;
}

canvas {
  position: fixed;
  left: 0;
  top: 0;
}

::-webkit-scrollbar {
  display: none;
}
</style>
