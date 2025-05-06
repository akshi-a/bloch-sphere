<script lang="ts">
  import { onMount } from 'svelte';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import './BlockSphere.css'; 

  let container: HTMLDivElement;
  let scene: THREE.Scene;
  let camera: THREE.PerspectiveCamera;
  let renderer: THREE.WebGLRenderer;
  let controls: OrbitControls;
  let arrow: THREE.ArrowHelper;

  // state
  let currentDirection = new THREE.Vector3(1, 0, 0).normalize();
  let prevThetaDeg = 0;
  let prevPhiDeg = 0;

  // UI sliders (start at 0 so deltas work)
  let thetaSlider = 0;
  let phiSlider = 0;

  const updateArrow = (newDir: THREE.Vector3) => {
    const startDir = currentDirection.clone();
    let progress = 0;
    const duration = 200;

    const animateStep = () => {
      progress += 16 / duration;
      const intermediateDir = startDir.clone().lerp(newDir, Math.min(progress, 1)).normalize();
      arrow.setDirection(intermediateDir);
      if (progress < 1) {
        requestAnimationFrame(animateStep);
      } else {
        currentDirection.copy(newDir);
      }
    };
    requestAnimationFrame(animateStep);
  };

  
  const rotateBy = (axis: THREE.Vector3, angleDegrees: number) => {
    const angle = THREE.MathUtils.degToRad(angleDegrees);
    const q = new THREE.Quaternion().setFromAxisAngle(axis, angle);
    currentDirection.applyQuaternion(q).normalize();
    updateArrow(currentDirection);
  };



  const handleThetaChange = () => {
    const delta = thetaSlider - prevThetaDeg;
    rotateBy(
      new THREE.Vector3(-Math.sin(phiSlider * Math.PI / 180), Math.cos(phiSlider * Math.PI / 180), 0),
      delta
    );
    prevThetaDeg = thetaSlider;
  };

  const handlePhiChange = () => {
    const delta = phiSlider - prevPhiDeg;
    rotateBy(new THREE.Vector3(0, 0, 1), delta);
    prevPhiDeg = phiSlider;
  };

  const applyGate = (axis: THREE.Vector3, angleDegrees: number) => {
    rotateBy(axis, angleDegrees);
  };

  const applyHadamard = () => {
    const axis = new THREE.Vector3(1, 0, 1).normalize();
    applyGate(axis, 180);
  };

  const resetArrow = () => {
    const initialDir = new THREE.Vector3(1, 0, 0).normalize();
    currentDirection.copy(initialDir);
    prevThetaDeg = 0;
    prevPhiDeg = 0;
    thetaSlider = 0;
    phiSlider = 0;
    updateArrow(initialDir);
  };

  onMount(() => {
    scene = new THREE.Scene();
    camera = new THREE.PerspectiveCamera(75, container.clientWidth / container.clientHeight, 0.1, 1000);
    camera.position.z = 3;

    renderer = new THREE.WebGLRenderer({ antialias: true });
    const resize = () => {
      const width = container.clientWidth;
      const height = container.clientHeight;
      renderer.setSize(width, height);
      camera.aspect = width / height;
      camera.updateProjectionMatrix();
    };
    resize();
    window.addEventListener('resize', resize);

    container.appendChild(renderer.domElement);

    controls = new OrbitControls(camera, renderer.domElement);

    const sphereGeometry = new THREE.SphereGeometry(1, 32, 32);
    const wireframeMaterial = new THREE.MeshBasicMaterial({ color: 0x00ffff, wireframe: true });
    const sphere = new THREE.Mesh(sphereGeometry, wireframeMaterial);
    scene.add(sphere);

    const axesHelper = new THREE.AxesHelper(2);
    scene.add(axesHelper);

    arrow = new THREE.ArrowHelper(currentDirection.clone(), new THREE.Vector3(0, 0, 0), 1, 0xff0000);
    scene.add(arrow);

    const animateScene = () => {
      requestAnimationFrame(animateScene);
      controls.update();
      renderer.render(scene, camera);
    };
    animateScene();
  });
</script>

<div class="container">
  <div bind:this={container} class="bloch-visual"></div>
  <div class="controls">    
    <label>Polar Angle θ Increment (deg): {thetaSlider}°</label>
    <input type="range" min="-180" max="180" bind:value={thetaSlider} on:input={handleThetaChange} />
    
    <label>Azimuthal Angle φ Increment (deg): {phiSlider}°</label>
    <input type="range" min="-180" max="180" bind:value={phiSlider} on:input={handlePhiChange} />
    
    <div class="rotation-group">
      <label>Apply X-axis Rotations:</label>
      <button on:click={() => applyGate(new THREE.Vector3(1, 0, 0), 180)}>Half Turn X (π)</button>
      <button on:click={() => applyGate(new THREE.Vector3(1, 0, 0), 90)}>Quarter Turn X (π/2)</button>
      <button on:click={() => applyGate(new THREE.Vector3(1, 0, 0), 45)}>Eighth Turn X (π/4)</button>
    </div>
    
    <div class="rotation-group">
      <label>Apply Y-axis Rotations:</label>
      <button on:click={() => applyGate(new THREE.Vector3(0, 1, 0), 180)}>Half Turn Y (π)</button>
      <button on:click={() => applyGate(new THREE.Vector3(0, 1, 0), 90)}>Quarter Turn Y (π/2)</button>
      <button on:click={() => applyGate(new THREE.Vector3(0, 1, 0), 45)}>Eighth Turn Y (π/4)</button>
    </div>
    
    <div class="rotation-group">
      <label>Apply Z-axis Rotations:</label>
      <button on:click={() => applyGate(new THREE.Vector3(0, 0, 1), 180)}>Half Turn Z (π)</button>
      <button on:click={() => applyGate(new THREE.Vector3(0, 0, 1), 90)}>Quarter Turn Z (π/2)</button>
      <button on:click={() => applyGate(new THREE.Vector3(0, 0, 1), 45)}>Eighth Turn Z (π/4)</button>
    </div>
    
    <label>Hadamard Gate:</label>
    <button on:click={applyHadamard}>Apply H Gate</button>
    
    <button class="reset" on:click={resetArrow}>Reset Bloch Sphere</button>
  </div>
</div>
