<script setup lang="ts">
import * as THREE from 'three';

import { MTLLoader } from 'three/addons/loaders/MTLLoader.js';
import { OBJLoader } from 'three/addons/loaders/OBJLoader.js';
import { onMounted, ref } from 'vue';
import { LineMaterial, Wireframe, WireframeGeometry2 } from 'three/examples/jsm/Addons.js';
import { update } from 'three/examples/jsm/libs/tween.module.js';

const stool_ref = ref<HTMLElement | null>(null);
defineProps<{}>()


let camera: THREE.OrthographicCamera, scene: THREE.Scene, renderer: THREE.WebGLRenderer;
let rotation_speed = 0.01;
let angle = 0;
onMounted(() => {
  init();

  setInterval(() => {
    angle += rotation_speed;
    scene_objects.forEach((obj) => {
      obj.rotation.x = angle;
    });
    update();
  }, 10);
});
const scene_objects: THREE.Object3D[] = [];
async function init() {
  // aspect
  let rect = stool_ref.value?.getBoundingClientRect();
  if (!rect) return;
  let aspect = rect.width / rect.height;
  let size = 10;
  camera = new THREE.OrthographicCamera(-size / 2, size / 2, size * aspect / 2, -size * aspect / 2, -2, 200);

  camera.position.set(-1, 1, 1);
  camera.lookAt(0, 0, 0);
  // scene

  scene = new THREE.Scene();

  const ambientLight = new THREE.AmbientLight(0xffffff, 10);
  scene.add(ambientLight);

  const pointLight = new THREE.PointLight(0xffffff, 1);
  camera.add(pointLight);
  scene.add(camera);

  // model

  const mtlLoader = new MTLLoader().setPath('/models/');
  const materials = await mtlLoader.loadAsync('stool.mtl');
  materials.preload();

  const objLoader = new OBJLoader().setPath('/models/');
  objLoader.setMaterials(materials); // optional since OBJ assets can be loaded without an accompanying MTL file

  const object = await objLoader.loadAsync('stool.obj');
  let min_extent = new THREE.Vector3(Number.POSITIVE_INFINITY, Number.POSITIVE_INFINITY, Number.POSITIVE_INFINITY);
  let max_extent = new THREE.Vector3(Number.NEGATIVE_INFINITY, Number.NEGATIVE_INFINITY, Number.NEGATIVE_INFINITY);
  object.traverse((child) => {
    if ((child as THREE.Mesh).isMesh) {
      let position = (child as THREE.Mesh).geometry.attributes.position;
      if (!position) return;
      for (let i = 0; i < position.count; i++) {
        let vertex = new THREE.Vector3().fromBufferAttribute(position, i);
        min_extent.min(vertex);
        max_extent.max(vertex);
      }
    }
  });
  let avg = new THREE.Vector3(0, 0, 0);
  avg.divideScalar(2);
  let offset = avg.multiplyScalar(-1);
  object.traverse((child) => {
    if ((child as THREE.Mesh).isMesh) {

      const geometry = new WireframeGeometry2((child as THREE.Mesh).geometry);

      let matLine = new LineMaterial({

        color: 0x000000,
        linewidth: 2, // in pixels
        dashed: true

      });

      let wireframe = new Wireframe(geometry, matLine);
      wireframe.computeLineDistances();
      wireframe.scale.set(1, 1, 1);
      wireframe.position.copy(offset);
      scene.add(wireframe);
      scene_objects.push(wireframe);
    }
  });
  object.position.copy(offset);
  scene.add(object);
  scene_objects.push(object);
  //

  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(rect.width, rect.height);
  renderer.setAnimationLoop(animate);
  renderer.setClearColor(0x000000ff, 0);
  const container = stool_ref.value;
  if (container) {
    container.appendChild(renderer.domElement);
  }

  //


  //

  window.addEventListener('resize', onWindowResize);

}

function onWindowResize() {
  let rect = stool_ref.value?.getBoundingClientRect();
  if (!rect) return;
  camera.updateProjectionMatrix();

  renderer.setSize(rect.width, rect.height);
  const container = stool_ref.value;
  if (container) {
    container.appendChild(renderer.domElement);
  }
}

function animate() {

  // controls.update();

  renderer.render(scene, camera);

}

</script>

<template>
  <div id="stool-container" ref="stool_ref">


  </div>
</template>

<style scoped>
#stool-container {
  width: 80vh;
  height: 80vh;
  overflow: hidden;
}
</style>
