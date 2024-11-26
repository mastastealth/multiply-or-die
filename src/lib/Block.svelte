<script>
	import Timer from "easytimer.js";
	import { Howl } from "howler";
	import getRandom from "./random";
	import boom from "../assets/boom.gif";
	import powerup from "../assets/boost.mp3";
	import powerup_sm from "../assets/boost_sm.mp3";
	import squish from "../assets/squish.mp3";
	import mushroom from "../assets/mushroom.png";
	import mushroom2 from "../assets/mushroom2.png";
	import mushroom3 from "../assets/mushroom3.png";

	const timer = new Timer();
	const {
		addToGrid,
		addToScore,
		gameOver,
		fullGrid = [],
		x,
		y,
		from,
		score = 0,
	} = $props();
	const isCenter = !from;
	const style =
		!isCenter &&
		`position: absolute; 
     top: calc(var(--width) * ${y}); 
     left: calc(var(--width) * ${x}); 
     z-index: ${100 - (Math.abs(x) > Math.abs(y) ? Math.abs(x) : Math.abs(y))}`;
	const problems = {
		n: [getRandom(12), getRandom(12)],
		e: [getRandom(12), getRandom(12)],
		s: [getRandom(12), getRandom(12)],
		w: [getRandom(12), getRandom(12)],
	};
	const boost = new Howl({
		src: powerup,
		preload: true,
		volume: 0.5,
	});
	const boost2 = new Howl({
		src: powerup_sm,
		preload: true,
		volume: 0.5,
	});
	const splurt = new Howl({
		src: squish,
		preload: true,
	});

	let selected = $state(false);
	let direction = $state(["n", "e", "s", "w"].filter((d) => d !== from));
	let speed = $state(0);
	const mushroomSpr = {
		1: mushroom,
		2: mushroom2,
		3: mushroom3,
	};

	// If this was created by another box, let's do more updates to directions
	if (from) {
		if (fullGrid.find((block) => block.x === x && block.y === y - 1))
			direction = direction.filter((d) => d !== "n");
		if (fullGrid.find((block) => block.x === x && block.y === y + 1))
			direction = direction.filter((d) => d !== "s");
		if (fullGrid.find((block) => block.y === y && block.x === x - 1))
			direction = direction.filter((d) => d !== "w");
		if (fullGrid.find((block) => block.y === y && block.x === x + 1))
			direction = direction.filter((d) => d !== "e");
	}

	function onEnter(e, dir) {
		const answer = problems[dir][0] * problems[dir][1];
		const input = e.target;

		// On Enter
		if (e.key === "Enter") {
			input.blur();
			selected = false;
			const time = timer.getTimeValues();
			timer.pause();

			if (answer === parseInt(input.value)) {
				console.log("CORRECT", time.seconds);
				direction = direction.filter((d) => d !== dir);
				let score = 100;

				if (time.seconds <= 2) {
					score = 500;
					boost.play();
					speed = 2;
				} else if (time.seconds <= 5) {
					score = 200;
					boost2.play();
					speed = 1;
				} else {
					splurt.play();
				}

				addToScore(score);

				switch (dir) {
					case "n":
						return addToGrid(x, y - 1, "s", score);
					case "e":
						return addToGrid(x + 1, y, "w", score);
					case "s":
						return addToGrid(x, y + 1, "n", score);
					case "w":
						return addToGrid(x - 1, y, "e", score);
				}
			} else {
				console.log("WRONG");
				addToScore(0);
			}
		}
	}
</script>

<div
	class="block animate__animated {speed === 1 && 'animate__pulse'} {speed ===
		2 && 'animate__tada'}"
	{style}
>
	{#if score}
		<span class="score animate__animated animate__fadeOutUp">{score}</span>
	{/if}

	<label>
		<input type="checkbox" bind:checked={selected} disabled={gameOver} />
	</label>

	{#each direction as dir (dir)}
		<div class="expand" data-dir={dir}>
			<label>
				<input
					type="text"
					maxlength="3"
					onclick={() => {
						timer.start();
					}}
					onkeydown={(e) => onEnter(e, dir)}
					placeholder="Click me"
				/>
			</label>

			<footer>
				<span data-num={problems[dir][0]}>{problems[dir][0]}</span> &times;
				<span data-num={problems[dir][1]}>{problems[dir][1]}</span>
			</footer>
		</div>
	{/each}

	<img
		class={gameOver &&
			"animate__animated animate__infinite animate__rubberBand"}
		src={mushroomSpr[getRandom(3)]}
		alt="Little Guy"
		data-show
		data-mushroom
		style={`filter: hue-rotate(${getRandom(360)}deg)`}
	/>
	<img src={boom} alt="Explosion!" data-show={gameOver} />
</div>

<style>
	.block {
		background: rgb(84, 60, 12);
		border: 1px solid black;
		z-index: 101;

		input[type="checkbox"] {
			display: none;
		}

		&:hover {
			border-color: yellow;
		}

		&:has(input:checked) {
			border-color: white;
		}
	}

	.block,
	.expand {
		--width: 80px;
		aspect-ratio: 1;
		max-height: var(--width);
		position: relative;
		width: var(--width);

		label {
			display: grid;
			height: 100%;
			place-content: center;
			position: relative;
			width: 100%;
			z-index: 1;
		}
	}

	.expand {
		background: rebeccapurple;
		display: flex;
		flex-direction: column;
		left: 0;
		opacity: 0;
		padding: 10px;
		position: absolute;
		top: 0;
		transition:
			opacity 0.5s,
			transform 0.3s;
		z-index: -1;

		input {
			text-align: center;
			width: 100%;
		}

		footer {
			background: black;
			font-weight: bold;
			opacity: 0;
			width: 100%;
		}

		&[data-dir="n"] {
			top: auto;
			bottom: 0;
		}

		&[data-dir="w"] {
			left: auto;
			right: 0;
		}
	}

	.score {
		color: yellow;
		display: block;
		font-size: 1.5em;
		font-weight: bold;
		pointer-events: none;
	}

	label:has(input:checked, input[type="text"]:focus) {
		~ [data-dir],
		~ footer {
			opacity: 1;
		}
		~ [data-dir="n"] {
			transform: translate(0, calc(-1 * var(--width)));
		}
		~ [data-dir="e"] {
			transform: translate(var(--width), 0);
		}
		~ [data-dir="s"] {
			transform: translate(0, var(--width));
		}
		~ [data-dir="w"] {
			transform: translate(calc(-1 * var(--width)), 0);
		}
	}

	img {
		display: none;
		pointer-events: none;
		position: absolute;
		top: 50%;
		left: 50%;
		translate: -50% -50%;
		z-index: 1;

		&[data-show] {
			display: block;
		}

		&[data-mushroom] {
			max-width: 80%;
		}
	}

	[data-num="1"] {
		color: red;
	}
	[data-num="2"] {
		color: orange;
	}
	[data-num="3"] {
		color: yellow;
	}
	[data-num="4"] {
		color: yellowgreen;
	}
	[data-num="5"] {
		color: green;
	}
	[data-num="6"] {
		color: teal;
	}
	[data-num="7"] {
		color: cyan;
	}
	[data-num="8"] {
		color: rgb(0, 145, 255);
	}
	[data-num="9"] {
		color: blueviolet;
	}
	[data-num="10"] {
		color: magenta;
	}
	[data-num="11"] {
		color: pink;
	}
	[data-num="12"] {
		color: white;
	}
</style>
