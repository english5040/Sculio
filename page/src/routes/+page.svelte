<script lang="ts">
	import { Canvas, extend } from "@threlte/core";
	import Scene from "./canvas.svelte";
	import Card_Scene from "./canvas2.svelte"; // Maybe rename canvas2.svelte to something else

	import logo from "$lib/assets/logo.png";
	import small_logo from "$lib/assets/small_logo.png";

	let logo_element: HTMLImageElement;
	let small_logo_element: HTMLImageElement;

	import { KeyframeTrack, Path, PlaneGeometry, WebGLRenderer } from "three";
	import { onMount } from "svelte";

	let card_renderer: WebGLRenderer;
	onMount(() => {
		card_renderer = new WebGLRenderer({
			antialias: true,
			alpha: true,
		});
	});

	import parallax_cards_settings from "$lib/assets/parallax_cards/parallax_cards.json";

	let parallax_cards = Object.entries(import.meta.glob(
		"$lib/assets/parallax_cards/*.png",
		{ eager: true }
	)).map(([path, module]) => {
		const cardName = path.split('/').pop() || 'default';
		return {
			name: cardName,
			path: path,
			settings: parallax_cards_settings[cardName as keyof typeof parallax_cards_settings]
		};
	});

	console.log("parallax_cards", parallax_cards);

	extend({ PlaneGeometry });
</script>

<div class="no-scrollbar">
	<div class="w-screen h-screen fixed top-0 left-0 -z-1">
		<Canvas dpr={2}>
			<Scene />
		</Canvas>
	</div>

	<div
		id="logo"
		class="flex absolute top-0 left-0 w-screen h-screen -z-1 justify-center items-center flex-col -gap-3"
	>
		<img
			src={logo}
			alt="Logo"
			class="w-5/8"
			bind:this={logo_element}
		/>
		<!-- translating manually may not accomadate all screen sizes -->
		<img
			src={small_logo}
			alt="Sculio"
			class="w-1/6 transform-[translate(0,-70px)]"
			bind:this={small_logo_element}
		/>
	</div>
	<div class="w-screen">
		<!-- <Canvas dpr={2}>
			<Card_Scene />
		</Canvas> -->

		<!-- parallax card images -->
		<div
			class="relative top-0 left-0 w-screen h-screen overflow-hidden">
			{#each parallax_cards as card}
			<!-- svelte-ignore -->
			<img
				src={card.path}
				alt={card.name}
				class="absolute"
				style="transform: translate(-50%, -50%) translateX({card.settings.x}px) translateY({card.settings.y}px) scale({card.settings.scale}) rotate({card.settings.rotation}deg);"
				/>
			{/each}
		</div>
	</div>
	<!-- add  a scroll area for the canvas to run its animation or whatever -->
	<div class="w-screen h-1/2 overflow-y-scroll overflow-x-hidden">
		<div class="h-[200vh]"></div>
	</div>
</div>

<style>
	::-webkit-scrollbar {
		display: none;
	}

	@font-face {
		font-family: balatro;
		src: url("/fonts/balatro.otf") format("opentype");
		font-weight: normal;
	}
</style>
