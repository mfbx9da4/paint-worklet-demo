<script lang="ts">
	import '../app.css';
	import { onDestroy, onMount } from 'svelte';

	const isBrowser = typeof window !== 'undefined';

	if (isBrowser) {
		CSS.paintWorklet.addModule(`/6.js`);
	}

	interface Point {
		x: number;
		y: number;
	}

	interface Camera {
		x: number;
		y: number;
		z: number;
	}

	interface Box {
		minX: number;
		minY: number;
		maxX: number;
		maxY: number;
		width: number;
		height: number;
	}

	function screenToCanvas(point: Point, camera: Camera) {
		return {
			x: point.x / camera.z + camera.x,
			y: point.y / camera.z + camera.y
		};
	}

	function canvasToScreen(point: Point, camera: Camera) {
		return {
			x: (point.x - camera.x) * camera.z,
			y: (point.y - camera.y) * camera.z
		};
	}

	function getViewport(box: Box, camera: Camera): Box {
		const p0 = { x: box.minX, y: box.minY };
		const topLeft = screenToCanvas(p0, camera);
		const p1 = { x: box.maxX, y: box.maxY };
		const bottomRight = screenToCanvas(p1, camera);

		return {
			minX: topLeft.x,
			minY: topLeft.y,
			maxX: bottomRight.x,
			maxY: bottomRight.y,
			width: bottomRight.x - topLeft.x,
			height: bottomRight.y - topLeft.y
		};
	}

	function panCamera(camera: Camera, dx: number, dy: number): Camera {
		return {
			x: camera.x - dx / camera.z,
			y: camera.y - dy / camera.z,
			z: camera.z
		};
	}

	function zoomCamera(camera: Camera, point: Point, dz: number): Camera {
		const zoom = camera.z - dz * camera.z;

		const p1 = screenToCanvas(point, camera);

		const p2 = screenToCanvas(point, { ...camera, z: zoom });

		return {
			x: camera.x + (p2.x - p1.x),
			y: camera.y + (p2.y - p1.y),
			z: zoom
		};
	}

	let camera: Camera = { x: 0, y: 0, z: 1 };
	let viewport: Box = {
		minX: 0,
		minY: 0,
		maxX: 400,
		maxY: 400,
		width: 400,
		height: 400
	};

	if (isBrowser) {
		viewport = getViewport(
			{
				minX: 0,
				minY: 0,
				maxX: window.innerWidth,
				maxY: window.innerHeight,
				width: window.innerWidth,
				height: window.innerHeight
			},
			camera
		);
		// const center = screenToCanvas({ x: window.innerWidth / 2, y: window.innerHeight / 2 }, camera);
		// camera = { x: center.x, y: center.y, z: 1 };
		// camera = panCamera(camera, window.innerWidth, window.innerHeight);
	}

	function handleWheel(event: WheelEvent) {
		event.preventDefault();

		const { clientX, clientY, deltaX, deltaY, ctrlKey } = event;

		if (ctrlKey) {
			camera = zoomCamera(
				camera,
				{ x: clientX - viewport.width / 2, y: clientY - viewport.height / 2 },
				deltaY / 100
			);
		} else {
			camera = panCamera(camera, deltaX, deltaY);
		}
	}

	onMount(() => {
		window.addEventListener('wheel', handleWheel, { passive: false });
	});

	onDestroy(() => {
		if (isBrowser) window.removeEventListener('wheel', handleWheel);
	});
</script>

<pre
	style="position: absolute; top: 0; left: 0; background: white; padding: 20px; z-index: 2; border: 1px solid #ccc;">
	Try scrolling to pan. Hold ctrl to zoom. 
  {JSON.stringify(camera)}
</pre>

<div
	style="
  transform: scale({camera.z}) translate({camera.x}px, {camera.y}px);
  width: {viewport.width}px;
  height: {viewport.height}px;
  "
	class="demo"
/>

<style>
	:global(svg) {
		fill: orange;
	}

	@property --fleck-colors {
		syntax: '<color>+';
		initial-value: black white;
		inherits: true;
	}

	@property --fleck-seed {
		syntax: '<integer>';
		initial-value: 1;
		inherits: true;
	}

	@property --fleck-count {
		syntax: '<integer>';
		initial-value: 580;
		inherits: true;
	}

	@property --fleck-density {
		syntax: '<integer>';
		initial-value: 300;
		inherits: true;
	}

	@property --fleck-size-base {
		syntax: '<length>';
		initial-value: 4px;
		inherits: true;
	}

	.demo {
		width: 400px;
		height: 400px;
		background-color: #84dce2;
		background-image: paint(fleck);
		overflow: hidden;
		--fleck-colors: #0193c1 #74d8e6 #0bc0e0 #3ddaf5 #e2f9fe #02313f;
		--fleck-seed: 1;
	}
</style>
