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
let group_obj = new THREE.Group();
onMounted(() => {
  init();

  setInterval(() => {
    angle += rotation_speed;
    group_obj.rotation.x = angle;
    update();
  }, 20);
});
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
  let center = new THREE.Vector3(0, 0, 0);
  object.traverse((child) => {
    if ((child as THREE.Mesh).isMesh) {
      let geom = (child as THREE.Mesh).geometry;
      geom.computeBoundingBox();
      let bbox = geom.boundingBox
      if (!bbox) return;
      center.copy(bbox.getCenter(new THREE.Vector3()));
      let position = geom.attributes.position;
      if (!position) return;
      for (let i = 0; i < position.count; i++) {
        let vertex = new THREE.Vector3().fromBufferAttribute(position, i);
        min_extent = min_extent.min(vertex);
        max_extent = max_extent.max(vertex);
        // let helper = new THREE.AxesHelper(0.2);
        // helper.position.copy(vertex);
        // scene.add(helper);
      }
    }
  });
  // let avg = max_extent.multiplyScalar(-1).add(min_extent).multiplyScalar(0.5);
  // avg.divideScalar(2);
  let offset = center.multiplyScalar(-1).add(new THREE.Vector3(0, 1, 0));
  console.log('min_extent', min_extent);
  console.log('max_extent', max_extent);
  console.log('offset', offset);
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
      group_obj.add(wireframe);
    }
  });
  object.position.copy(offset);
  group_obj.add(object);
  scene.add(group_obj);
  //

  renderer = new THREE.WebGLRenderer({ antialias: false, alpha: true });
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
  width: 512px;
  height: 512px;
  overflow: hidden;
}
</style>
