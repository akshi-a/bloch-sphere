<script lang="ts">
  import { onMount } from 'svelte';
  import * as THREE from 'three';
  // @ts-ignore
  import OrbitControls from 'three/examples/jsm/controls/OrbitControls.js';

  let container: HTMLDivElement;

  onMount(() => {
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, container.clientWidth / container.clientHeight, 0.1, 1000);
    camera.position.z = 3;

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(container.clientWidth, container.clientHeight);
    container.appendChild(renderer.domElement);

    const controls = new OrbitControls(camera, renderer.domElement);

    // Bloch Sphere
    const sphereGeometry = new THREE.SphereGeometry(1, 32, 32);
    const wireframeMaterial = new THREE.MeshBasicMaterial({ color: 0x00ffcc, wireframe: true });
    const sphere = new THREE.Mesh(sphereGeometry, wireframeMaterial);
    scene.add(sphere);

    // Axes
    const axesHelper = new THREE.AxesHelper(2);
    scene.add(axesHelper);

    // State Vector (Arrow)
    const arrowDir = new THREE.Vector3(1, 1, 0).normalize();
    const arrow = new THREE.ArrowHelper(arrowDir, new THREE.Vector3(0, 0, 0), 1, 0xff0000);
    scene.add(arrow);

    // Handle window resizing
    window.addEventListener('resize', () => {
      camera.aspect = container.clientWidth / container.clientHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(container.clientWidth, container.clientHeight);
    });

    // Render Loop
    const animate = () => {
      requestAnimationFrame(animate);
      controls.update();
      renderer.render(scene, camera);
    };
    animate();
  });
</script>

<div bind:this={container} class="bloch-container"></div>

<style>
  .bloch-container {
    width: 100%;
    height: 500px;
    background-color: black;
  }
</style>
