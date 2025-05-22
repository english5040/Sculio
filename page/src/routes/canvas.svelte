<script lang="ts">
	import { T, extend, useTask, useThrelte } from "@threlte/core";
	import { onMount, onDestroy } from "svelte";
	import {
		ShaderMaterial,
		Clock,
		PlaneGeometry,
	} from "three";
	import { Vector2, Vector3, Vector4 } from "three";

	extend({ ShaderMaterial, PlaneGeometry });

	const { size } = useThrelte();

	let iResolution = new Vector2($size.width, $size.height);
	let iTime = 0;

	$: {
		console.log("iResolution", iResolution);
	}
	
	// --- CHANGE THESE VALUES TO CHANGE THE EFFECT ---
	let SPIN_ROTATION = 0.0;
	let SPIN_SPEED = 7.0;
	let OFFSET = new Vector2(0.4, 0.25);
	// SCULIO COLOUR PALETTE
	// wavy cream: 214, 191, 140
	// swampy green: 64, 80, 83
	// border brown: 72, 36, 36
	// The values below are divided by 255 to normalize them to [0, 1], which is how the shader expects them.
	let COLOUR_1 = new Vector4(72 / 255, 36 / 255, 36 / 255, 1.0);
	let COLOUR_2 = new Vector4(214/ 255, 191 / 255, 140 / 255, 1.0);
	let COLOUR_3 = new Vector4(64 / 255, 80 / 255, 83 / 255, 1.0);
	let CONTRAST = 2.5; // default: 3.5
	let LIGHTING = 0.4;
	let AMOUNT = 0.20; // default: 0.25
	let PIXEL_FILTER = 845.0;
	let SPIN_EASE = 1.0;
	const PI = 3.14159265359;
	let IS_ROTATE = false;

	const uniforms = {
		iResolution: { value: iResolution },
		iTime: { value: iTime },

		SPIN_ROTATION: { value: SPIN_ROTATION },
		SPIN_SPEED: { value: SPIN_SPEED },
		OFFSET: { value: OFFSET },
		COLOUR_1: {
			value: COLOUR_1,
		},
		COLOUR_2: { value: COLOUR_2 },
		COLOUR_3: {
			value: COLOUR_3,
		},
		CONTRAST: { value: CONTRAST },
		LIGHTING: { value: LIGHTING },
		SPIN_AMOUNT: { value: AMOUNT },
		PIXEL_FILTER: { value: PIXEL_FILTER },
		SPIN_EASE: { value: SPIN_EASE },
		PI: { value: PI },
		IS_ROTATE: { value: IS_ROTATE },
	};

	let backgroundMaterial: ShaderMaterial;

	let clock = new Clock();
	useTask((ctx) => {
		iTime = clock.getElapsedTime();
	});
	onMount(() => {
		backgroundMaterial = new ShaderMaterial({
			uniforms: uniforms,
			vertexShader: vertexShader,
			fragmentShader: fragmentShader,
			side: 2,
		});
		iResolution.set($size.width, $size.height); //FIX: This might not be working? Double check if the size is being set correctly on mount.
		backgroundMaterial.uniforms.iResolution.value = iResolution;
	});

	onDestroy(() => {
		if (backgroundMaterial) {
			backgroundMaterial.dispose();
		}
	});

	const fragmentShader = `
		uniform vec2      iResolution;           // viewport resolution (in pixels)
		uniform float     iTime;                 // shader playback time (in seconds)
		
		// Original by localthunk (https://www.playbalatro.com)

		// Configuration (modify these values to change the effect)
		uniform float SPIN_ROTATION;
		uniform float SPIN_SPEED;
		uniform vec2 OFFSET;
		uniform vec4 COLOUR_1;
		uniform vec4 COLOUR_2;
		uniform vec4 COLOUR_3;
		uniform float CONTRAST;
		uniform float LIGHTING;
		uniform float SPIN_AMOUNT;
		uniform float PIXEL_FILTER;
		uniform float SPIN_EASE;
		#define PI 3.14159265359;
		uniform bool IS_ROTATE;

		vec4 effect(vec2 screenSize, vec2 screen_coords) {
		    float pixel_size = length(screenSize.xy) / PIXEL_FILTER;
		    vec2 uv = (floor(screen_coords.xy*(1./pixel_size))*pixel_size - 0.5*screenSize.xy)/length(screenSize.xy) - OFFSET;
		    float uv_len = length(uv);
		    
		    float speed = (SPIN_ROTATION*SPIN_EASE*0.2);
		    if(IS_ROTATE){
		       speed = iTime * speed;
		    }
		    speed += 302.2;
		    float new_pixel_angle = atan(uv.y, uv.x) + speed - SPIN_EASE*20.*(1.*SPIN_AMOUNT*uv_len + (1. - 1.*SPIN_AMOUNT));
		    vec2 mid = (screenSize.xy/length(screenSize.xy))/2.;
		    uv = (vec2((uv_len * cos(new_pixel_angle) + mid.x), (uv_len * sin(new_pixel_angle) + mid.y)) - mid);
		    
		    uv *= 30.;
		    speed = iTime*(SPIN_SPEED);
		    vec2 uv2 = vec2(uv.x+uv.y);
		    
		    for(int i=0; i < 5; i++) {
			uv2 += sin(max(uv.x, uv.y)) + uv;
			uv  += 0.5*vec2(cos(5.1123314 + 0.353*uv2.y + speed*0.131121),sin(uv2.x - 0.113*speed));
			uv  -= 1.0*cos(uv.x + uv.y) - 1.0*sin(uv.x*0.711 - uv.y);
		    }
		    
		    float contrast_mod = (0.25*CONTRAST + 0.5*SPIN_AMOUNT + 1.2);
		    float paint_res = min(2., max(0.,length(uv)*(0.035)*contrast_mod));
		    float c1p = max(0.,1. - contrast_mod*abs(1.-paint_res));
		    float c2p = max(0.,1. - contrast_mod*abs(paint_res));
		    float c3p = 1. - min(1., c1p + c2p);
		    float light = (LIGHTING - 0.2)*max(c1p*5. - 4., 0.) + LIGHTING*max(c2p*5. - 4., 0.);
		    return (0.3/CONTRAST)*COLOUR_1 + (1. - 0.3/CONTRAST)*(COLOUR_1*c1p + COLOUR_2*c2p + vec4(c3p*COLOUR_3.rgb, c3p*COLOUR_1.a)) + light;
		}

		void main() {
		    vec2 uv = gl_FragCoord.xy /iResolution.xy;

		    gl_FragColor = effect(iResolution.xy, uv * iResolution.xy);
		}	
		`;

	const vertexShader = `
    			void main() {
		      		gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
		    	}
		`;

	$: if (backgroundMaterial && backgroundMaterial.uniforms) {
		backgroundMaterial.uniforms.iResolution.value = iResolution;
		backgroundMaterial.uniforms.iTime.value = iTime;
		backgroundMaterial.uniforms.IS_ROTATE.value = IS_ROTATE;

		// Update other dynamic uniform values
		backgroundMaterial.uniforms.SPIN_ROTATION.value = SPIN_ROTATION;
		backgroundMaterial.uniforms.SPIN_SPEED.value = SPIN_SPEED;
		backgroundMaterial.uniforms.OFFSET.value = OFFSET;
		backgroundMaterial.uniforms.COLOUR_1.value = COLOUR_1;
		backgroundMaterial.uniforms.COLOUR_2.value = COLOUR_2;
		backgroundMaterial.uniforms.COLOUR_3.value = COLOUR_3;
		backgroundMaterial.uniforms.CONTRAST.value = CONTRAST;
		backgroundMaterial.uniforms.LIGHTING.value = LIGHTING;
		backgroundMaterial.uniforms.SPIN_AMOUNT.value = AMOUNT;
		backgroundMaterial.uniforms.PIXEL_FILTER.value = PIXEL_FILTER;
		backgroundMaterial.uniforms.SPIN_EASE.value = SPIN_EASE;
	}

	// Update iResolution when the size changes
	// $: {
	// 	if (
	// 		$size.width !== iResolution.x ||
	// 		$size.height !== iResolution.y
	// 	) {
	// 		iResolution.set($size.width, $size.height);
	// 		// This is only needed if backgroundMaterial exists already
	// 		if (backgroundMaterial) {
	// 			backgroundMaterial.uniforms.iResolution.value =
	// 				iResolution;
	// 		}
	// 	}
	// }

	$ : {
		if(
		$size.height !== iResolution.y || $size.width !== iResolution.x){
			iResolution.set($size.width, $size.height);
			// This is only needed if backgroundMaterial exists already
			if (backgroundMaterial) {
				backgroundMaterial.uniforms.iResolution.value =
					iResolution;
			}
		}
	}
</script>

<T.Mesh
	material={backgroundMaterial}
	position.x={0}
	position.y={0}
	position.z={0}
>
	<!-- setting size to [100, 100] is a brute force way to increase the size of the mesh! Maybe find an alternative? -->
	<T.PlaneGeometry args={[100, 100]}></T.PlaneGeometry>
</T.Mesh>
