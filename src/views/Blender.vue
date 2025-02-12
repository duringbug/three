<template>
    <div class="canvas-container">
        <!-- Vue 普通元素，可以放在 canvas 后面 -->
        <div class="overlay-elements">
            <button @click="changeBackgroundColor">Change Background</button>
            <p>This is a Vue element over the canvas!</p>
        </div>

        <canvas id="webgl" ref="canvas"></canvas>
    </div>
</template>

  
<script lang="ts" setup>
import {
    onMounted,
    ref
} from 'vue';
import * as THREE from 'three';
import {
    GLTFLoader
} from 'three/examples/jsm/loaders/GLTFLoader.js';
import {
    DRACOLoader
} from 'three/examples/jsm/loaders/DRACOLoader.js';
import {
    OrbitControls
} from 'three/examples/jsm/controls/OrbitControls.js';
import * as dat from 'dat.gui'

const changeBackgroundColor = () => {
    if (scene) {
        scene.background = new THREE.Color(Math.random(), Math.random(), Math.random()); // 随机颜色
    }
};

const scene = new THREE.Scene();
// 创建 canvas 引用
const canvas = ref < HTMLCanvasElement | null > (null);

// 在组件挂载后初始化 Three.js 场景
onMounted(() => {
    if (!canvas.value) return;

    const loader = new GLTFLoader();
    const dracoLoader = new DRACOLoader();
    dracoLoader.setDecoderPath('/draco/gltf/');

    loader.setDRACOLoader(dracoLoader);

    // 设置大小
    const sizes = {
        width: window.innerWidth,
        height: window.innerHeight,
    };

    // 场景
    scene.background = new THREE.Color('#ffffff');

    // 加载模型
    loader.load('/glb/arcade-room-texturized.glb', (gltf) => {
        gltf.scene.traverse((child) => {
            if (child instanceof THREE.Mesh) { // 使用 instanceof 检查
                child.receiveShadow = true;
                child.castShadow = true;
                // 修改材质的漫反射颜色
                const material = child.material as THREE.MeshStandardMaterial;
                if (material) {
                    // 增加漫反射效果，可以根据需求修改颜色
                    material.color = new THREE.Color(0xffffff); // 设置漫反射颜色为白色
                    material.roughness = 0.5; // 设置粗糙度，影响表面的光泽度
                }
            }
        });
        scene.add(gltf.scene);
    });

    // 灯光设置

    const pointLight1 = new THREE.PointLight(0xFF5A00, 1.4);
    pointLight1.position.set(1.4441, 1.9779, 0.70389);
    pointLight1.intensity = 10
    scene.add(pointLight1);

    const pointLight2 = new THREE.PointLight(0xFF5A00, 1.4);
    pointLight2.position.set(-2.2229, 3.765, -0.37849);
    pointLight2.intensity = 10
    scene.add(pointLight2);

    const pointLight3 = new THREE.PointLight(0x00E5FF, 0.7);
    pointLight3.position.set(-0.90519, 0.80477, 0.3215);
    pointLight3.intensity = 10
    scene.add(pointLight3);

    const directionalLight = new THREE.DirectionalLight(0x203574, 1);
    directionalLight.castShadow = true;
    directionalLight.shadow.bias = -0.0005;
    directionalLight.position.set(-4, 8, -4);
    directionalLight.lookAt(0, 0, 0);
    directionalLight.intensity = 10
    scene.add(directionalLight);

    const shadowLight = new THREE.DirectionalLight(0x203574, 0.8);
    shadowLight.castShadow = true;
    shadowLight.shadow.bias = -0.0005;
    shadowLight.position.set(4, 8, 4);
    shadowLight.lookAt(0, 0, 0);
    shadowLight.intensity = 5
    scene.add(shadowLight);

    // 相机设置
    const camera = new THREE.PerspectiveCamera(45, sizes.width / sizes.height, 0.1, 100);
    camera.position.set(30, 30, 30);
    camera.lookAt(0, 0, 0);
    scene.add(camera);

    // Orbit 控制器
    const controls = new OrbitControls(camera, canvas.value);
    controls.autoRotate = false;
    controls.autoRotateSpeed = 0.4;
    controls.enableDamping = true;
    controls.maxPolarAngle = Math.PI * 0.4;

    // 渲染器设置
    const renderer = new THREE.WebGLRenderer({
        canvas: canvas.value,
        antialias: true,
        alpha: true,
    });
    renderer.setPixelRatio(Math.min(2, window.devicePixelRatio));
    renderer.setSize(sizes.width, sizes.height);
    renderer.toneMapping = THREE.CineonToneMapping;
    renderer.toneMappingExposure = 1.75;
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    renderer.setClearColor('#ffffff');
    renderer.render(scene, camera);

    // 创建 GUI 控制面板的方法
    function createGui() {
        const gui = new dat.GUI();

        const lightControls = {
            pointLight1Color: pointLight1.color.getHex(),
            pointLight2Color: pointLight2.color.getHex(),
            pointLight3Color: pointLight3.color.getHex(),
            directionalLightColor: directionalLight.color.getHex(),
            shadowLightColor: shadowLight.color.getHex(),
            // 添加 pointLight1 位置控制
            pointLight1Position: {
                x: pointLight1.position.x,
                y: pointLight1.position.y,
                z: pointLight1.position.z
            },
            cameraPosition: {
                x: camera.position.x,
                y: camera.position.y,
                z: camera.position.z
            }
        };

        // 添加颜色控制面板
        gui.addColor(lightControls, 'pointLight1Color').onChange((value) => {
            pointLight1.color.set(value);
        }).name('Point Light 1');

        gui.addColor(lightControls, 'pointLight2Color').onChange((value) => {
            pointLight2.color.set(value);
        }).name('Point Light 2');

        gui.addColor(lightControls, 'pointLight3Color').onChange((value) => {
            pointLight3.color.set(value);
        }).name('Point Light 3');

        gui.addColor(lightControls, 'directionalLightColor').onChange((value) => {
            directionalLight.color.set(value);
        }).name('Directional Light');

        gui.addColor(lightControls, 'shadowLightColor').onChange((value) => {
            shadowLight.color.set(value);
        }).name('Shadow Light');

        // 添加位置控制面板并折叠
        const pointLightFolder = gui.addFolder('Point Light 1 Position');
        pointLightFolder.add(lightControls.pointLight1Position, 'x', -10, 10).onChange((value) => {
            pointLight1.position.x = value;
        }).name('X');
        pointLightFolder.add(lightControls.pointLight1Position, 'y', -10, 10).onChange((value) => {
            pointLight1.position.y = value;
        }).name('Y');
        pointLightFolder.add(lightControls.pointLight1Position, 'z', -10, 10).onChange((value) => {
            pointLight1.position.z = value;
        }).name('Z');

        const cameraFolder = gui.addFolder('Camera Position');
        cameraFolder.add(lightControls.cameraPosition, 'x', -30, 30).onChange((value) => {
            camera.position.x = value;
        }).name('X');
        cameraFolder.add(lightControls.cameraPosition, 'y', -30, 30).onChange((value) => {
            camera.position.x = value;
        }).name('Y');
        cameraFolder.add(lightControls.cameraPosition, 'z', -30, 30).onChange((value) => {
            camera.position.x = value;
        }).name('Z');
    }

    // 创建 GUI 控制面板
    createGui();

    // 动画循环
    const tick = () => {
        controls.update();
        renderer.render(scene, camera);
        window.requestAnimationFrame(tick);
    };

    tick();
});
</script>

  
<style scoped>
/* 样式 */
.canvas-container {
  display: flex;
  justify-content: center;  /* 水平居中 */
  align-items: center;      /* 垂直居中 */
  height: 100vh;            /* 占满屏幕高度 */
}
.overlay-elements {
    position: absolute;
    top: 20px;
    left: 20px;
    color: white;
    z-index: 10;
    font-size: 18px;
}

button {
    padding: 10px 20px;
    margin: 10px 0;
    cursor: pointer;
}
</style>
