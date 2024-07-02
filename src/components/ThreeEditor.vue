<template>
  <div class="container">
    <div class="controls">
      <button @click="addCube">Add Cube</button>
      <button @click="undo">Undo</button>
      <button @click="redo">Redo</button>
    </div>
    <div class="scene-settings">
      <h2>Scene Settings</h2>
    </div>
    <div class="scene-node-properties">
      <h2>Scene Node Properties</h2>
      <div>
        <h3>Position:</h3>
        <div>
          <label>x:</label>
          <input type="number" v-model.number="sceneNodeProperties.position.x" @change="updateSceneNodeProperties">
        </div>
        <div>
          <label>y:</label>
          <input type="number" v-model.number="sceneNodeProperties.position.y" @change="updateSceneNodeProperties">
        </div>
        <div>
          <label>z:</label>
          <input type="number" v-model.number="sceneNodeProperties.position.z" @change="updateSceneNodeProperties">
        </div>
      </div>
      <div>
        <h3>Rotation:</h3>
        <div>
          <label>x:</label>
          <input type="number" v-model.number="sceneNodeProperties.rotation.x" @change="updateSceneNodeProperties">
        </div>
        <div>
          <label>y:</label>
          <input type="number" v-model.number="sceneNodeProperties.rotation.y" @change="updateSceneNodeProperties">
        </div>
        <div>
          <label>z:</label>
          <input type="number" v-model.number="sceneNodeProperties.rotation.z" @change="updateSceneNodeProperties">
        </div>
      </div>
      <div>
        <h3>Scale:</h3>
        <div>
          <label>x:</label>
          <input type="number" v-model.number="sceneNodeProperties.scale.x" @change="updateSceneNodeProperties">
        </div>
        <div>
          <label>y:</label>
          <input type="number" v-model.number="sceneNodeProperties.scale.y" @change="updateSceneNodeProperties">
        </div>
        <div>
          <label>z:</label>
          <input type="number" v-model.number="sceneNodeProperties.scale.z" @change="updateSceneNodeProperties">
        </div>
      </div>
    </div>
    <div ref="container" class="scene"></div>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { TransformControls } from 'three/examples/jsm/controls/TransformControls.js';

export default {
  name: 'ThreeEditor',
  setup() {
    const container = ref(null);
    const sceneSettings = reactive({
      width: 800,
      height: 600
    });
    const sceneNodeProperties = reactive({
      position: { x: 0, y: 0, z: 0 },
      rotation: { x: 0, y: 0, z: 0 },
      scale: { x: 1, y: 1, z: 1 }
    });
    const cubeDimensions = reactive({
      width: 1,
      height: 1,
      depth: 1
    });
    const state = reactive({
      selectedObject: null
    });

    let scene, camera, renderer, controls, transformControls;

    onMounted(() => {
      if (!container.value) {
        console.error('Container ref is not assigned');
        return;
      }

      scene = new THREE.Scene();
      camera = new THREE.PerspectiveCamera(75, sceneSettings.width / sceneSettings.height, 0.1, 1000);
      camera.position.set(0, 5, 10);
      renderer = new THREE.WebGLRenderer();
      renderer.setSize(sceneSettings.width, sceneSettings.height);
      container.value.appendChild(renderer.domElement); 

      const gridHelper = new THREE.GridHelper(50, 50);
      scene.add(gridHelper);
      const axesHelper = new THREE.AxesHelper(5);
      scene.add(axesHelper);

      controls = new OrbitControls(camera, renderer.domElement);
      controls.update();

      const light = new THREE.DirectionalLight(0xffffff, 1);
      light.position.set(5, 10, 7.5);
      light.castShadow = true;
      scene.add(light);
      renderer.shadowMap.enabled = true;

      transformControls = new TransformControls(camera, renderer.domElement);
      transformControls.addEventListener('change', () => renderer.render(scene, camera));
      transformControls.addEventListener('objectChange', () => updatePropertiesFromObject(transformControls.object));
      scene.add(transformControls);

      renderer.domElement.addEventListener('click', onMouseDown);

      const animate = () => {
        requestAnimationFrame(animate);
        controls.update();
        renderer.render(scene, camera);
      };
      animate();

      window.addEventListener('resize', () => {
        camera.aspect = sceneSettings.width / sceneSettings.height;
        camera.updateProjectionMatrix();
        renderer.setSize(sceneSettings.width, sceneSettings.height);
      });
    });

    const initialPositions = [];

    const addCube = () => {
      if (!scene) {
        console.error('Scene is not initialized');
        return;
      }
      const geometry = new THREE.BoxGeometry(cubeDimensions.width, cubeDimensions.height, cubeDimensions.depth);
      const material = new THREE.MeshBasicMaterial({ color: 0xeeffe0 });
      const cube = new THREE.Mesh(geometry, material);

      const initialPosition = new THREE.Vector3(0, cubeDimensions.height / 2, 0);
      cube.position.copy(initialPosition);
      initialPositions.push(initialPosition); 
      scene.add(cube);
      console.log('Cube added at position:', cube.position);
      state.selectedObject = cube; 
      transformControls.attach(cube);
      updatePropertiesFromObject(cube);
    };


    const updateSceneNodeProperties = () => {
      if (state.selectedObject) {
        state.selectedObject.position.set(
          sceneNodeProperties.position.x,
          sceneNodeProperties.position.y,
          sceneNodeProperties.position.z
        );
        state.selectedObject.rotation.set(
          sceneNodeProperties.rotation.x,
          sceneNodeProperties.rotation.y,
          sceneNodeProperties.rotation.z
        );
        state.selectedObject.scale.set(
          sceneNodeProperties.scale.x,
          sceneNodeProperties.scale.y,
          sceneNodeProperties.scale.z
        );
      }
    };

    const updatePropertiesFromObject = (object) => {
      if (object) {
        sceneNodeProperties.position.x = object.position.x;
        sceneNodeProperties.position.y = object.position.y;
        sceneNodeProperties.position.z = object.position.z;
        sceneNodeProperties.rotation.x = object.rotation.x;
        sceneNodeProperties.rotation.y = object.rotation.y;
        sceneNodeProperties.rotation.z = object.rotation.z;
        sceneNodeProperties.scale.x = object.scale.x;
        sceneNodeProperties.scale.y = object.scale.y;
        sceneNodeProperties.scale.z = object.scale.z;
      }
    };

    const onMouseDown = (event) => {
  
    };

    return { container, addCube, updateSceneNodeProperties, sceneSettings, sceneNodeProperties, cubeDimensions };
  },
};
</script>

<style scoped>
.container {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.controls {
  flex: 1;
  margin-right: 20px;
}

.scene-settings,
.scene-node-properties {
  flex: 1;
  background-color: #f0f0f0;
  padding: 10px;
  margin-bottom: 20px;
}

.scene {
  flex: 3;
  width: 100%;
  height: 600px; 
}
</style>
