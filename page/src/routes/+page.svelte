<script lang="ts">
	import { Canvas, extend, T } from "@threlte/core";
	import Scene from "./canvas.svelte";

	import { PlaneGeometry, OrthographicCamera } from "three";
	extend({ PlaneGeometry });
</script>

<div class="w-screen h-screen fixed top-0 left-0 -z-1">
	<Canvas orthographic camera={{}}>
		<Scene />
	</Canvas>
</div>
