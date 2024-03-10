<script>
    import * as THREE from 'three'
    import { onMount } from 'svelte';

    let scene, camera, renderer, sphere, container;
    let rotationInterval = null;

    function init() {
        scene = new THREE.Scene();
        camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        renderer = new THREE.WebGLRenderer();
        renderer.setSize(window.innerWidth, window.innerHeight);
        container.appendChild(renderer.domElement);

        // Create the transparent sphere
        const sphereGeometry = new THREE.SphereGeometry(1, 32, 32);
        const sphereMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff, opacity: 0.2, transparent: true, wireframe: true});
        sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
        sphere.scale.set(2, 2, 2);
        scene.add(sphere);

        // contour lines
        const contourMaterial = new THREE.LineBasicMaterial({ color: 0xffffff });
        const torusGeometry1 = new THREE.TorusGeometry(1, 0.005, 16, 100);
        const contourLine1 = new THREE.LineLoop(torusGeometry1, contourMaterial);
        contourLine1.rotation.x = Math.PI / 2;
        sphere.add(contourLine1);

        const torusGeometry2 = new THREE.TorusGeometry(1, 0.005, 16, 100);
        const contourLine2 = new THREE.LineLoop(torusGeometry2, contourMaterial);
        contourLine2.rotation.y = Math.PI / 2;
        sphere.add(contourLine2);

        const torusGeometry3 = new THREE.TorusGeometry(1, 0.005, 16, 100);
        const contourLine3 = new THREE.LineLoop(torusGeometry3, contourMaterial);
        sphere.add(contourLine3);

        // axis arrows
        const arrowHelper1 = new THREE.ArrowHelper(new THREE.Vector3(1, 0, 0), new THREE.Vector3(0, 0, 0), 1.5, 0xff0000);
        sphere.add(arrowHelper1);

        const arrowHelper2 = new THREE.ArrowHelper(new THREE.Vector3(0, 1, 0), new THREE.Vector3(0, 0, 0), 1.5, 0x00ff00);
        sphere.add(arrowHelper2);

        const arrowHelper3 = new THREE.ArrowHelper(new THREE.Vector3(0, 0, 1), new THREE.Vector3(0, 0, 0), 1.5, 0x0000ff);
        sphere.add(arrowHelper3);

        // Set up the camera position
        let pos = 5
        if (window.innerWidth < 600) {
            pos = 7;
        }

        camera.position.z = pos;
    }

    // Render loop
    function animate() {
        requestAnimationFrame(animate);
        renderer.render(scene, camera);
    }

    // Variables for mouse interaction
    let isDragging = false;
    let previousMousePosition = {
        x: 0,
        y: 0
    };

    // Mouse event handlers
    function onMouseDown(event) {
        isDragging = true;
    }

    function onMouseMove(event) {
        if (isDragging) {
            const deltaMove = {
            x: event.clientX - previousMousePosition.x,
            y: event.clientY - previousMousePosition.y
            };

            const deltaRotationQuaternion = new THREE.Quaternion()
            .setFromEuler(new THREE.Euler(
                toRadians(deltaMove.y * 0.1),
                toRadians(deltaMove.x * 0.1),
                0,
                'XYZ'
            ));

            sphere.quaternion.multiplyQuaternions(deltaRotationQuaternion, sphere.quaternion);
        }

        previousMousePosition = {
            x: event.clientX,
            y: event.clientY
        };
    }

    function onMouseUp(event) {
        isDragging = false;
    }

    function onPointerDown(event) {
        event.preventDefault();
        isDragging = true;
        const touch = event.touches ? event.touches[0] : event;
        previousMousePosition.x = touch.clientX;
        previousMousePosition.y = touch.clientY;
    }

    function onPointerMove(event) {
        event.preventDefault();
        if (isDragging) {
            const touch = event.touches ? event.touches[0] : event;
            const deltaMoveX = touch.clientX - previousMousePosition.x;
            const deltaMoveY = touch.clientY - previousMousePosition.y;
            const sensitivity = 0.004;

            sphere.rotation.y += deltaMoveX * sensitivity;
            sphere.rotation.x += deltaMoveY * sensitivity;
            previousMousePosition.x = touch.clientX;
            previousMousePosition.y = touch.clientY;
        }
    }

    function onPointerUp(event) {
        event.preventDefault();
        isDragging = false;
    }

    function toRadians(degrees) {
        return degrees * (Math.PI / 180);
    }

    onMount(() => {
        init();
        animate();

        container.addEventListener('mousedown', onMouseDown);
        container.addEventListener('mousemove', onMouseMove);
        container.addEventListener('mouseup', onMouseUp);

        container.addEventListener('touchstart', onPointerDown);
        container.addEventListener('touchmove', onPointerMove);
        container.addEventListener('touchend', onPointerUp);
        
        return () => {
            container.removeEventListener('mousedown', onMouseDown);
            container.removeEventListener('mousemove', onMouseMove);
            container.removeEventListener('mouseup', onMouseUp);

            container.removeEventListener('touchstart', onPointerDown);
            container.removeEventListener('touchmove', onPointerMove);
            container.removeEventListener('touchend', onPointerUp);
        };
    });

    // Functions for rotating the sphere with buttons
    function rotateClockwise() {
        if (rotationInterval) clearInterval(rotationInterval);
        rotationInterval = setInterval(() => rotateSphere(-0.01), 16);
    }

    function rotateCounterClockwise() {
        if (rotationInterval) clearInterval(rotationInterval);
        rotationInterval = setInterval(() => rotateSphere(0.01), 16);
    }

    function stopRotation() {
        if (rotationInterval) clearInterval(rotationInterval);
    }

    function rotateSphere(angle) {
        // Get the direction vector from the camera to the sphere
        const direction = new THREE.Vector3();
        sphere.getWorldPosition(direction);
        direction.sub(camera.position).normalize();

        // Calculate the rotation axis perpendicular to the direction vector and camera up vector
        const axis = new THREE.Vector3();
        axis.crossVectors(direction, camera.up).normalize();

        // Calculate a new rotation axis perpendicular to the current one
        const newAxis = new THREE.Vector3();
        newAxis.crossVectors(axis, camera.up).normalize();

        // Rotate the sphere around the calculated new axis
        sphere.rotateOnWorldAxis(newAxis, angle);
    }

</script>

<div bind:this={container} class="rotating-sphere">
    <button on:mousedown={rotateClockwise} on:mouseleave={stopRotation} on:mouseup={stopRotation}>Rotate Clockwise</button>
    <button on:mousedown={rotateCounterClockwise} on:mouseleave={stopRotation} on:mouseup={stopRotation}>Rotate Counterclockwise</button>
</div>

<style>
    .rotating-sphere {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        overflow: hidden;
        margin: 0;
        padding: 0;
        touch-action: none;
    }
</style>
