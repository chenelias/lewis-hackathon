<script>
	import { onMount } from 'svelte';
	import { fly } from 'svelte/transition';
	import Matter from 'matter-js';

	// const tokenkey = import.meta.env.VITE_MUSIC_API;
	let musicData = [];
	let originalMusicData = [];
	let loading = true;
	let searchQuery = '';
	let searchInput;
	let innerWidth,
		innerHeight,
		boxWidth = 80,
		boxHeight = 80,
		resultBox;
	let mainBox;
	let pin = true;
	let pinTimeout;
	let oldSearchQueryLength = 0;
	let wireDiv;
	let currentVideoId = '';
	let player = false;

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
		mainBox = Bodies.rectangle(innerWidth / 2, innerHeight / 2 - 130, boxWidth, boxHeight, {
			restitution: 0.2,
			friction: 0.4,
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
		const ground = Bodies.rectangle(innerWidth / 2, innerHeight + 1, innerWidth, 1, {
			isStatic: true
		});

		// var ground = Bodies.rectangle(innerWidth / 2, innerHeight, innerWidth, 1, {
		// 	isStatic: true
		// });
		Composite.add(engine.world, [mainBox, ground, leftWall, rightWall, topWall]);

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
			const boxPosition = mainBox.position;
			const boxAngle = mainBox.angle;
			if (!pin) {
				resultBox.style.transform = `translate(${boxPosition.x / 0.83 - boxWidth}px, ${boxPosition.y / 1.36 - boxHeight}px) rotate(${boxAngle}rad)`;
				resultBox.style.position = 'absolute';
			}
			if (!pin)
				if (wireDiv && pinTimeout) {
					const resultBoxRect = resultBox.getBoundingClientRect();
					const wiredDivRect = wireDiv.getBoundingClientRect();

					if (
						resultBoxRect.left < wiredDivRect.right &&
						resultBoxRect.right > wiredDivRect.left &&
						resultBoxRect.top < wiredDivRect.bottom &&
						resultBoxRect.bottom > wiredDivRect.top
					) {
						pin = true;
					}
				} else {
					setTimeout(() => {
						pinTimeout = true;
					}, 1000);
				}
		});
	}

	onMount(async () => {
		loading = true;
		// const response = await fetch(
		// 	`https://www.googleapis.com/youtube/v3/playlistItems?part=snippet&playlistId=PLyOL_RMmwqydli4pCUvprHaskPvGKRTC9&key=${tokenkey}&maxResults=1000`
		// );

		const response = await fetch(`/api.json`);
		const data = await response.json();
		musicData = data.items;
		originalMusicData = data.items;
		loading = false;
	});

	$: searchQuery, search();
	$: searchQuery, overText();
	function overText() {
		let random = Math.floor(Math.random() * 3) + 1;
		if (searchQuery.length - oldSearchQueryLength > random) {
			pin = false;
			pinTimeout = false;
			renderUnfunctional();
			searchInput.blur();
			oldSearchQueryLength = searchQuery.length;
		}
	}
	function search() {
		musicData = originalMusicData;
		oldSearchQueryLength = 0;
		musicData = musicData.filter((item) => {
			return (
				item.snippet.title.toLowerCase().includes(searchQuery.toLowerCase()) ||
				item.snippet.videoOwnerChannelTitle.toLowerCase().includes(searchQuery.toLowerCase())
			);
		});
	}
</script>

<svelte:window bind:innerWidth bind:innerHeight />
<svetle:head>
	<title>Music Finder</title>
</svetle:head>

<main>
	<div
		class="absolute left-1/2 top-1/3 -translate-x-1/2 -translate-y-1/3 {pin ? 'z-[50]' : 'z-[-1]'}"
	>
		<input
			bind:this={searchInput}
			bind:value={searchQuery}
			type="search"
			class="border-4 bg-white focus:border-gray-500 h-10 py-6 pl-5 px-2 rounded-xl text-xl focus:outline-none w-[60vw] drop-shadow-md {!pin
				? 'border-red-500'
				: 'border-gray-300'}"
			placeholder="Try search for a song or artist..."
			readonly={!pin}
		/>
		<!-- NOTE: connectWire -->
		<div
			id="wire"
			bind:this={wireDiv}
			class="h-[70px] w-[10px] fixed left-1/2 -translate-x-1/2 z-[-1] bottom-[-50px] rounded-md drop-shadow-md {!pin
				? 'bg-red-300'
				: 'bg-gray-300'}"
		></div>
		{#key pin}
			<div
				bind:offsetWidth={boxWidth}
				bind:offsetHeight={boxHeight}
				bind:this={resultBox}
				style="box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.25);"
				class="my-5 p-2 w-[60vw] h-[30vh] overflow-auto absolute rounded-xl border-2 bg-white"
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
								<li>
									<button
										class="text-left w-full p-2 drop-shadow-md my-1 pl-3 hover:bg-gray-200 block rounded-lg duration-100"
										on:click={() => {
											currentVideoId = item.snippet.resourceId.videoId;
											player = true;
										}}
									>
										<p class="text-lg font-bold">{item.snippet.title}</p>
										<p class="text-xs text-gray-700">
											{item.snippet.videoOwnerChannelTitle.replace('- Topic', '')}
										</p>
									</button>
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
							class="text-center text-lg font-bold absolute top-[50%] left-[50%] -translate-x-1/2 -translate-y-1/2 text-red-600"
						>
							Search bar is disconnected!
						</li>
					{/if}
				</ul>
			</div>
		{/key}
	</div>
</main>
<footer
	class="fixed my-2 drop-shadow-xl top-0 left-1/2 -translate-x-1/2 font-bold text-center text-md text-gray-500 z-[1000]"
>
	<p>
		Made for <span class="text-lg font-extrabold"
			>Hackathon with <a
				class="hover:underline underline-offset-2 decoration-wavy decoration-blue-800 decoration-2"
				title="Lewis GitHub page"
				target="_blank"
				href="https://github.com/elebumm">Lewis</a
			> 2024</span
		>
		by
		<a
			href="https://eliaschen.dev"
			target="_blank"
			class="hover:underline underline-offset-2 text-lg font-extrabold decoration-wavy decoration-purple-600 decoration-2"
			title="EliasChen Homepage">EliasChen</a
		>
	</p>
	<a
		class=" hover:underline underline-offset-4 text-md font-extrabold decoration-wavy decoration-orange-600 decoration-2"
		title="GitHub Repositories of this webpage"
		href="https://github.com/chenelias/lewis-hackathon"
		target="_blank">GitHub Repositories</a
	>
</footer>
{#if player && currentVideoId}
	<div
		class="bg-white fixed h-[100%] w-[100%] z-[1000]"
		transition:fly={{ y: -200, duration: 800 }}
	>
		<div class="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 z-[10000] drop-shadow-2xl">
			<div
				class="rounded-2xl bg-gray-300"
				style={`width: ${innerWidth * 0.9}px; height: ${innerWidth * 0.9}px;max-width:600px;max-height:600px;`}
			>
				<iframe
					id="player"
					type="text/html"
					class="rounded-2xl"
					src="https://www.youtube.com/embed/{currentVideoId}"
					style={`width: ${innerWidth * 0.9}px; height: ${innerWidth * 0.9}px;max-width:600px;max-height:600px;`}
					title="YouTube video player"
					frameborder="0"
					allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
					referrerpolicy="strict-origin-when-cross-origin"
				></iframe>
			</div>
			<button
				on:click={() => (player = false)}
				class="fixed left-1/2 -translate-x-1/2 rounded-full text-xl mt-5 p-4 bg-gray-300 font-bold duration-100 hover:bg-gray-400"
				><svg xmlns="http://www.w3.org/2000/svg" width="1em" height="1em" viewBox="0 0 24 24">
					<path
						fill="currentColor"
						d="M18.3 5.71a.996.996 0 0 0-1.41 0L12 10.59L7.11 5.7A.996.996 0 1 0 5.7 7.11L10.59 12L5.7 16.89a.996.996 0 1 0 1.41 1.41L12 13.41l4.89 4.89a.996.996 0 1 0 1.41-1.41L13.41 12l4.89-4.89c.38-.38.38-1.02 0-1.4"
					/>
				</svg></button
			>
		</div>
	</div>
{/if}

<style>
	#wire {
		animation: connectWire 1.5s infinite;
	}
	@keyframes connectWire {
		0% {
			bottom: -50px;
		}
		50% {
			bottom: -65px;
		}
		100% {
			bottom: -50px;
		}
	}
</style>
