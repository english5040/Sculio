<script lang="ts">
	import { T, extend } from "@threlte/core";
	import { ImageMaterial } from "@threlte/extras";
	import { PlaneGeometry, DoubleSide } from "three"; // Import MeshBasicMaterial
	import { Color } from "three"; // Import Color for solid color

	extend({ PlaneGeometry }); // Still need to extend PlaneGeometry

	const parallax_cards_raw = import.meta.glob(
		"$lib/assets/parallax_cards/*.png",
		{
			eager: true,
		},
	);

	console.log("parallax_cards_raw", parallax_cards_raw);

	let cards_settings = {
		"ace.png": {
			starting_position: [0, 0, 0],
		},
		"king.png": {
			starting_position: [0, 0, 0],
		},
		"queen.png": {
			starting_position: [0, 0, 0],
		},
		"jack.png": {
			starting_position: [0, 0, 0],
		},
		"rorschach.png": {
			starting_position: [0, 0, 0],
		},
		"fail.png": {
			starting_position: [0, 0, 0],
		},
		"mad.png": {
			starting_position: [0, 0, 0],
		},
		"pyro.png": {
			starting_position: [0, 0, 0],
		},
	};
</script>

<!-- as you scroll down, the cards will come out from the center of the screen -->
{#each Object.entries(parallax_cards_raw) as [path, image]}
	<T.Mesh>
		<T.PlaneGeometry args={[2, 2]} />
		<ImageMaterial
			transparent
			side={DoubleSide}
			url={path}
			radius={0.1}
			zoom={0.5}
		/>
	</T.Mesh>
{/each}
