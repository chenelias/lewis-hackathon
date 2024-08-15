<script>
	import { onMount } from 'svelte';
	import Matter from 'matter-js';

	const tokenkey = import.meta.env.VITE_MUSIC_API;
	let musicData = [];
	let originalMusicData = [];
	let loading = true;
	let searchQuery = '';
	let innerWidth,
		innerHeight,
		boxWidth = 80,
		boxHeight = 80,
		resultBox;
	let boxA;
	let pin = true;
	let oldSearchQueryLength = 0;

	function renderUnfunctional() {
		var Engine = Matter.Engine,
			Render = Matter.Render,
			Runner = Matter.Runner,
			Bodies = Matter.Bodies,
			Composite = Matter.Composite;

		var engine = Engine.create();
		var render = Render.create({
			element: document.body,
			engine: engine,
			options: {
				width: innerWidth,
				height: innerHeight
			}
		});
		render.options.wireframeBackground = 'transparent';
		render.options.background = 'transparent';
		boxA = Bodies.rectangle(innerWidth / 2, innerHeight / 2, boxWidth, boxHeight, {
			restitution: 0.5,
			friction: 0.1,
			frictionAir: 0.05,
			render: {
				fillStyle: 'transparent'
			}
		});

		const leftWall = Bodies.rectangle(-1, innerHeight / 2, 1, innerHeight, { isStatic: true });
		const rightWall = Bodies.rectangle(innerWidth + 1, innerHeight / 2, 1, innerHeight, {
			isStatic: true
		});
		const topWall = Bodies.rectangle(innerWidth / 2, -1, innerWidth, 1, { isStatic: true });

		var ground = Bodies.rectangle(innerWidth / 2, innerHeight, innerWidth, 1, {
			isStatic: true
		});
		Composite.add(engine.world, [boxA, ground, leftWall, rightWall, topWall]);

		var mouse = Matter.Mouse.create(render.canvas);
		var mouseConstraint = Matter.MouseConstraint.create(engine, {
			mouse: mouse,
			constraint: {
				stiffness: 1,
				render: {
					visible: false
				}
			}
		});

		Composite.add(engine.world, mouseConstraint);

		Render.run(render);
		var runner = Runner.create();
		Runner.run(runner, engine);

		Matter.Events.on(engine, 'afterUpdate', () => {
			const boxPosition = boxA.position;
			const boxAngle = boxA.angle;
			if (!pin) {
				resultBox.style.transform = `translate(${boxPosition.x / 0.83 - boxWidth}px, ${boxPosition.y / 1.35 - boxHeight}px) rotate(${boxAngle}rad)`;
				resultBox.style.position = 'absolute';
			}
		});
	}

	onMount(async () => {
		loading = true;
		const response = await fetch(
			`https://www.googleapis.com/youtube/v3/playlistItems?part=snippet&playlistId=PLyOL_RMmwqydli4pCUvprHaskPvGKRTC9&key=${tokenkey}&maxResults=1000`
		);
		const data = await response.json();
		musicData = data.items;
		originalMusicData = data.items;
		loading = false;
	});

	$: searchQuery, search();
	$: searchQuery, overText();
	function overText() {
		console.log('overText');
		if (searchQuery.length - oldSearchQueryLength > 1) {
			console.log('overText trigger');
			pin = false;
			renderUnfunctional();
			oldSearchQueryLength = searchQuery.length;
		}
	}
	function search() {
		console.log('search');
		if (searchQuery === '') {
			musicData = originalMusicData;
		} else {
			musicData = musicData.filter((item) => {
				return (
					item.snippet.title.toLowerCase().includes(searchQuery.toLowerCase()) ||
					item.snippet.videoOwnerChannelTitle.toLowerCase().includes(searchQuery.toLowerCase())
				);
			});
		}
	}
</script>

<svelte:window bind:innerWidth bind:innerHeight />

<main>
	<div
		class="absolute left-1/2 top-1/3 -translate-x-1/2 -translate-y-1/3 {pin ? 'z-[50]' : 'z-[-1]'}"
	>
		<button on:click={() => (pin = !pin)}>{pin ? 'Unpin' : 'pin'}</button>
		<input
			bind:value={searchQuery}
			type="text"
			class="border-4 border-gray-300 bg-white focus:border-gray-500 h-10 py-6 pl-5 px-2 rounded-xl text-xl focus:outline-none w-[60vw] drop-shadow-md"
			placeholder="Try search for a song or artist..."
			readonly={!pin}
		/>

		<div
			bind:offsetWidth={boxWidth}
			bind:offsetHeight={boxHeight}
			bind:this={resultBox}
			class="my-5 p-2 w-[60vw] h-[30vh] overflow-auto absolute rounded-xl border-2 drop-shadow-lg"
		>
			<ul>
				{#if pin}
					{#if loading}
						<li
							class="text-center text-lg absolute top-[50%] left-[50%] -translate-x-1/2 -translate-y-1/2"
						>
							Loading...
						</li>
					{:else if musicData.length > 0}
						{#each musicData as item}
							<li class="p-2 my-1 pl-3 hover:bg-gray-200 block rounded-lg duration-100">
								<p class="text-lg font-bold">{item.snippet.title}</p>
								<p class="text-xs text-gray-700">
									{item.snippet.videoOwnerChannelTitle.replace('- Topic', '')}
								</p>
							</li>
						{/each}
					{:else}
						<li
							class="text-center text-lg absolute top-[50%] left-[50%] -translate-x-1/2 -translate-y-1/2"
						>
							Nothing found
						</li>
					{/if}
				{:else}
					<li
						class="text-center text-lg absolute top-[50%] left-[50%] -translate-x-1/2 -translate-y-1/2"
					>
						😢 Pull me up, search bar just abandoned me
					</li>
				{/if}
			</ul>
		</div>
	</div>
</main>
