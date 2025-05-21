<script lang="ts">
	import { T, extend } from "@threlte/core";
	import { PlaneGeometry, MeshBasicMaterial } from "three"; // Import MeshBasicMaterial
	import { Color } from "three"; // Import Color for solid color

	extend({ PlaneGeometry }); // Still need to extend PlaneGeometry

	// We are using MeshBasicMaterial for this test, no need for ShaderMaterial here
	// No useTask, onMount, uniforms, or iTime/iResolution for this simple test.
</script>

<T.Mesh>
	<T.PlaneGeometry args={[2, 2]} />
	<T.MeshBasicMaterial color={new Color("red")} side={2} />
</T.Mesh>
