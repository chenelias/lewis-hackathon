<script>
	import { onMount } from 'svelte';
	import Matter from 'matter-js';

	let engine;
	let world;
	let render;
	let box;

	// Set canvas dimensions
	const canvasWidth = 800; // Width of the canvas
	const canvasHeight = 600; // Height of the canvas
	const groundHeight = 60; // Height of the ground

	onMount(() => {
		// Create an engine
		engine = Matter.Engine.create();
		world = engine.world;

		// Create a renderer
		render = Matter.Render.create({
			element: document.body,
			engine: engine,
			options: {
				width: canvasWidth,
				height: canvasHeight,
				wireframes: false // Set to false for filled shapes
			}
		});

		// Create a ground positioned at the bottom of the canvas
		const ground = Matter.Bodies.rectangle(
			canvasWidth / 2, // x-coordinate (centered)
			canvasHeight - groundHeight / 2, // y-coordinate (bottom of the canvas)
			canvasWidth, // Width of the ground (same as canvas)
			groundHeight, // Height of the ground
			{ isStatic: true }
		);

		// Create a box that will fall, centered horizontally
		box = Matter.Bodies.rectangle(
			canvasWidth / 2, // Center the box horizontally
			100, // Initial y-coordinate (100 pixels from the top)
			80, // Width of the box
			80, // Height of the box
			{
				restitution: 0.5, // Bounciness
				friction: 0.1, // Surface friction
				frictionAir: 0.05 // Air friction
			}
		);

		// Add bodies to the world
		Matter.World.add(world, [ground, box]);

		// Run the engine and renderer
		Matter.Engine.run(engine);
		Matter.Render.run(render);
	});
</script>

<main>
	<h1>Box with Text in Matter.js</h1>
	<div class="box-text">Hello!</div>
	<!-- Text displayed on the box -->
</main>

<style>
	canvas {
		border: 1px solid black;
		position: relative; /* Make the canvas position relative for absolute positioning of text */
	}

	.box-text {
		position: absolute;
		color: white; /* Text color */
		font-size: 16px; /* Font size */
		text-align: center;
		width: 80px; /* Match the width of the box */
		top: 80px; /* Position it above the box */
		left: 50%; /* Center horizontally */
		transform: translateX(-50%); /* Adjust for centering */
	}
</style>
