<template>
  <div id="canvasContainer" ref="canvasContainer"></div>
</template>

<script>
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

export default {
  name: "APP",
  mounted() {
    this.init();
    this.render();
  },
  data() {
    return {
      scene: null,
      camera: null,
      scene: null,
      renderer: null,
    };
  },
  methods: {
    init() {
      let container = document.getElementById("canvasContainer");
      //初始化场景
      this.scene = new THREE.Scene();
      //   初始化相机
      this.camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
      );
      // 初始化相机位置
      this.camera.position.set(1.5, 1, 1.5);
      this.camera.aspect = window.innerWidth / window.innerHeight;
      //   更新相机投影矩阵
      this.camera.updateProjectionMatrix();

      // 加载背景纹理
      const loader = new THREE.TextureLoader();
      const bgTexture = loader.load("/assets/imgs/050.jpg");
      bgTexture.mapping = THREE.EquirectangularRefractionMapping; // 球形

      this.scene.background = bgTexture;
      this.scene.environment = bgTexture;

      // 添加环境光
      const ambient = new THREE.AmbientLight(0xffffff, 1);
      this.scene.add(ambient);

      // 加载小熊模型
      const gltfLoader = new GLTFLoader();
      gltfLoader.load("/assets/model/bear.gltf", (gltf) => {
        // const model = gltf.scene;
        // Phong反射，亮材质
        const model = gltf.scene.children[0];
        model.material = new THREE.MeshPhongMaterial({
          color: 0xffffff,
          envMap: bgTexture,  // 环境纹理
          refractionRatio: 0.7,  // 折射率
          reflectivity: 0.8,  // 反射率
        });

        this.scene.add(model);
      });

      // 初始化渲染器
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      // 设置渲染器大小
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      // 监听屏幕大小变化
      window.addEventListener("resize", () => {
        this.renderer.setSize(window.innerWidth, window.innerHeight);
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
      });
      //   将渲染器添加到页面
      container.appendChild(this.renderer.domElement);

      //   添加控制器
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      // 设置控制阻尼
      this.controls.enableDamping = true;
    },
    // 渲染函数
    render() {
      //   更新控制器
      this.controls.update();
      // 渲染场景
      this.renderer.render(this.scene, this.camera);
      // 循环渲染
      requestAnimationFrame(this.render);
    },
  },
};
</script>