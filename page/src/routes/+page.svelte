<script lang="ts">
	import { Canvas, extend, T } from "@threlte/core";
	import Scene from "./canvas.svelte";

	import logo from "$lib/assets/logo.png";

	import { PlaneGeometry, OrthographicCamera } from "three";
	extend({ PlaneGeometry });
</script>

<div class="no-scrollbar">
	<div class="w-screen h-screen fixed top-0 left-0 -z-1">
		<Canvas
			dpr={2}
		>
			<Scene />
		</Canvas>
	</div>
	<div id="logo" class="block top-0 left-0 w-screen h-screen -z-1">
		<img src={logo} alt="Logo" class="w-full" />
	</div>
</div>

<style>
	::-webkit-scrollbar {
		display: none;
	}
</style>
