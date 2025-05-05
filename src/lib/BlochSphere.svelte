<script lang="ts">
  import './BlockSphere.css';
  import { onMount } from 'svelte';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

  let container: HTMLDivElement;
  let scene: THREE.Scene;
  let camera: THREE.PerspectiveCamera;
  let renderer: THREE.WebGLRenderer;
  let controls: OrbitControls;
  let arrow: THREE.ArrowHelper;

  // TRACK CURRENT DIRECTION
  let currentDirection = new THREE.Vector3(1, 0, 0); // initial along x-axis

  let theta = Math.PI / 2; // polar angle
  let phi = 0;             // azimuthal angle
  $: thetaDeg = (theta * 180) / Math.PI;
  $: phiDeg = (phi * 180) / Math.PI;

  const updateArrow = (animated = true) => {
  const targetDir = new THREE.Vector3(
    Math.sin(theta) * Math.cos(phi),
    Math.sin(theta) * Math.sin(phi),
    Math.cos(theta)
  ).normalize();

  if (animated) {
    const startDir = currentDirection.clone();
    let progress = 0;
    const duration = 500;

    const animateArrow = () => {
      progress += 16 / duration;
      const intermediateDir = startDir.clone().lerp(targetDir, Math.min(progress, 1)).normalize();
      arrow.setDirection(intermediateDir);

      if (progress < 1) {
        requestAnimationFrame(animateArrow);
      } else {
        currentDirection.copy(targetDir);
      }
    };

    requestAnimationFrame(animateArrow);
  } else {
    arrow.setDirection(targetDir);
    currentDirection.copy(targetDir);
  }
};


  const applyGate = (axis: THREE.Vector3, angleDegrees: number) => {
    const angle = THREE.MathUtils.degToRad(angleDegrees);
    const q = new THREE.Quaternion().setFromAxisAngle(axis, angle);

    currentDirection.applyQuaternion(q).normalize();

    // Update theta and phi from currentDirection
    theta = Math.acos(currentDirection.z);
    phi = Math.atan2(currentDirection.y, currentDirection.x);
    if (phi < 0) phi += 2 * Math.PI;

    thetaDeg = THREE.MathUtils.radToDeg(theta);
    phiDeg = THREE.MathUtils.radToDeg(phi);

    updateArrow(true);
  };

  const applyHadamard = () => {
    const axis = new THREE.Vector3(1, 0, 1).normalize(); // (X+Z)/sqrt(2)
    applyGate(axis, 180);
  };

  onMount(() => {
    scene = new THREE.Scene();

    camera = new THREE.PerspectiveCamera(75, container.clientWidth / container.clientHeight, 0.1, 1000);
    camera.position.z = 3;

    renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(container.clientWidth, container.clientHeight);
    container.appendChild(renderer.domElement);

    controls = new OrbitControls(camera, renderer.domElement);

    const sphereGeometry = new THREE.SphereGeometry(1, 32, 32);
    const wireframeMaterial = new THREE.MeshBasicMaterial({
      color: 0x00ffff,
      wireframe: true
    });
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
    <div class="slider-group">
      <label for="thetaSlider">Polar Angle θ (0°–180°): {thetaDeg.toFixed(1)}°</label>
      <input id="thetaSlider" type="range" min="0" max="180" bind:value={thetaDeg}
        on:input={() => { theta = THREE.MathUtils.degToRad(thetaDeg); updateArrow(true); }} />
    </div>

    <div class="slider-group">
      <label for="phiSlider">Azimuthal Angle φ (0°–360°): {phiDeg.toFixed(1)}°</label>
      <input id="phiSlider" type="range" min="0" max="360" bind:value={phiDeg}
        on:input={() => { phi = THREE.MathUtils.degToRad(phiDeg); updateArrow(true); }} />
    </div>

    <fieldset>
      <legend>Apply X Gates:</legend>
      <button on:click={() => applyGate(new THREE.Vector3(1,0,0), 180)}>Half Turn X (π)</button>
      <button on:click={() => applyGate(new THREE.Vector3(1,0,0), 90)}>Quarter Turn X (π/2)</button>
      <button on:click={() => applyGate(new THREE.Vector3(1,0,0), 45)}>Eighth Turn X (π/4)</button>
    </fieldset>

    <fieldset>
      <legend>Apply Y Gates:</legend>
      <button on:click={() => applyGate(new THREE.Vector3(0,1,0), 180)}>Half Turn Y (π)</button>
      <button on:click={() => applyGate(new THREE.Vector3(0,1,0), 90)}>Quarter Turn Y (π/2)</button>
      <button on:click={() => applyGate(new THREE.Vector3(0,1,0), 45)}>Eighth Turn Y (π/4)</button>
    </fieldset>

    <fieldset>
      <legend>Apply Z Gates:</legend>
      <button on:click={() => applyGate(new THREE.Vector3(0,0,1), 180)}>Half Turn Z (π)</button>
      <button on:click={() => applyGate(new THREE.Vector3(0,0,1), 90)}>Quarter Turn Z (π/2)</button>
      <button on:click={() => applyGate(new THREE.Vector3(0,0,1), 45)}>Eighth Turn Z (π/4)</button>
    </fieldset>

    <fieldset>
      <legend>Hadamard Gate:</legend>
      <button on:click={applyHadamard}>Apply H Gate</button>
    </fieldset>
  </div>
</div>


