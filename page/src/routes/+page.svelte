<script lang="ts">
	import { Canvas, extend, T } from "@threlte/core";
	import Scene from "./canvas.svelte";

	import logo from "$lib/assets/logo.png";
	import small_logo from "$lib/assets/small_logo.png";

	let logo_element: HTMLImageElement;
	let small_logo_element: HTMLImageElement;
	const parallax_cards_raw = import.meta.glob("$lib/assets/3d_cards/*.png", {
		eager: true,
	});

	let parallax_cards = {};

	import { PlaneGeometry } from "three";
    import { onMount } from "svelte";
	extend({ PlaneGeometry });

</script>

<style>
	::-webkit-scrollbar {
		display: none;
	}

	@font-face {
		font-family: balatro;
		src: url('/fonts/balatro.otf') format('opentype');
		font-weight: normal;
	}
	
</style>

<div class="no-scrollbar">
	<div class="w-screen h-screen fixed top-0 left-0 -z-1">
		<Canvas
			dpr={2}
		>
			<Scene />
		</Canvas>
	</div>
	<div id="logo" class="flex absolute top-0 left-0 w-screen h-screen -z-1 justify-center items-center flex-col -gap-3">
		<img src={logo} alt="Logo" class="w-5/8" bind:this={logo_element}/>
		<!-- translating manually may not accomadate all screen sizes -->
		<img src={small_logo} alt="Sculio" class="w-1/6 transform-[translate(0,-70px)] " bind:this={small_logo_element}/>
	</div>
</div>

{#each Object.entries(parallax_cards_raw) as [key, value]}
	<div>{key}, {value}</div>
{/each}


