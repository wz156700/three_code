<template></template>
<script>
import { defineComponent, onMounted } from "vue";
//导入three
import * as THREE from "three";
//导入相机控件
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
// 导入时钟
import { Clock } from "three";
//导入cannon 引擎
import * as CANNON from "cannon-es";

//目标：使用cannon引擎
export default defineComponent({
  setup() {
    //场景
    const scene = new THREE.Scene();

    // 球
    const sphereGeometry = new THREE.SphereGeometry(1, 20, 20);
    const material = new THREE.MeshStandardMaterial();
    const sphere = new THREE.Mesh(sphereGeometry, material);
    //开启阴影
    sphere.castShadow = true;
    scene.add(sphere);

    // 地面
    const floorGeometry = new THREE.PlaneGeometry(10, 10);
    const floorMaterial = new THREE.MeshStandardMaterial();
    const floor = new THREE.Mesh(floorGeometry, floorMaterial);
    floor.position.set(0, -5, 0);
    floor.rotation.x = -(Math.PI / 2);
    floor.receiveShadow = true;
    scene.add(floor);

    //创建物理世界
    const world = new CANNON.World({
      gravity: new CANNON.Vec3(0, -9.8, 0),
    });

    // 创建物理小球
    const spherePhysical = new CANNON.Sphere(1);
    // 设置物理小球物理世界材质
    const spherePhysicalMaterial = new CANNON.Material("shpere");

    // 创建物理世界的物体
    const sphereBody = new CANNON.Body({
      shape: spherePhysical, //形状
      position: new CANNON.Vec3(0, 0, 0), // 位置
      mass: 1, // 小球的质量
      material: spherePhysicalMaterial,
    });

    // 将物体添加至物理世界
    world.addBody(sphereBody);

    //创建物理地面
    const floorPhysical = new CANNON.Plane();
    const floorPhysicalMaterial = new CANNON.Material();
    const floorBody = new CANNON.Body({
      mass: 0, //当质量为0时，可以使得物体保持不动
      shape: floorPhysical,
      position: new CANNON.Vec3(0, -5, 0),
      material: floorPhysicalMaterial,
    });
    // 旋转地面位置
    floorBody.quaternion.setFromAxisAngle(
      new CANNON.Vec3(1, 0, 0),
      -Math.PI / 2
    );
    world.addBody(floorBody);

    // 创建击打声音
    const hitSound = new Audio("/audio/metalHit.mp3");

    // 设置两种材质碰撞的参数
    const defaultContactMaterial = new CANNON.ContactMaterial(
      spherePhysicalMaterial,
      floorPhysicalMaterial,
      {
        friction: 0.1, // 摩擦力
        restitution: 0.7, // 弹性
      }
    );

    // 将材质的关联设置添加到物理世界
    world.addContactMaterial(defaultContactMaterial);

    // 监听小球与地面碰撞事件
    sphereBody.addEventListener("collide", function (e) {
      // 获取碰撞强度
      const impactsStrength = e.contact.getImpactVelocityAlongNormal("floor");
      console.log(impactsStrength);
      if (impactsStrength > 2) {
        //重新从0进行播放
        hitSound.currentTime = 0;
        hitSound.play();
      }
    });

    //辅助观察的坐标系
    const axesHelper = new THREE.AxesHelper(100);
    scene.add(axesHelper);

    //光源设置
    const ambient = new THREE.AmbientLight(0xffffff, 0.4);
    scene.add(ambient);
    const dirLight = new THREE.DirectionalLight(0xffffff, 1);
    dirLight.position.set(100, 60, 50);

    scene.add(dirLight);
    // 开启阴影
    dirLight.castShadow = true;

    // // 可视化平行光阴影对应的正投影相机对象
    // const cameraHelper = new THREE.CameraHelper(dirLight.shadow.camera);
    // scene.add(cameraHelper);

    //相机
    const width = window.innerWidth;
    const height = window.innerHeight;
    const camera = new THREE.PerspectiveCamera(30, width / height, 0.1, 1000);
    camera.position.set(0, 0, 20);
    camera.lookAt(0, 0, 0);

    // WebGL渲染器设置
    const renderer = new THREE.WebGLRenderer({
      antialias: true, //开启优化锯齿
    });
    renderer.setPixelRatio(window.devicePixelRatio); //防止输出模糊
    renderer.setSize(width, height);
    document.body.appendChild(renderer.domElement);
    renderer.shadowMap.enabled = true; //允许渲染阴影

    const controls = new OrbitControls(camera, renderer.domElement);
    //监听鼠标、键盘事件
    controls.addEventListener("change", function () {
      // console.log(camera.position);
      renderer.render(scene, camera); //执行渲染操作
    });

    // 设置控制器阻尼，让控制器更有真实效果,必须在动画循环里调用.update()。
    controls.enableDamping = true;

    //设置时钟
    const clock = new THREE.Clock();
    // 渲染循环
    function render() {
      let time = clock.getDelta();
      // 更新物理引擎里世界的物体
      world.step(1 / 120, time);
      sphere.position.copy(sphereBody.position);

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
