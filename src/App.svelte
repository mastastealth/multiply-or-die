<script>
	import Block from "./lib/Block.svelte";
	import explode from "./assets/explode.mp3";
	import { Howl } from "howler";

	let fullGrid = $state([]);
	let score = $state(0);
	let gameOver = $state(null);
	let newHighscore = $state(false);

	const explosion = new Howl({
		src: [explode],
		html5: true,
		volume: 0.5,
	});

	function addToGrid(x, y, from) {
		fullGrid.push({ x, y, from });
	}

	function addToScore(n) {
		if (n) {
			score += n;
		} else {
			gameOver = true;
			explosion.play();

			if (
				parseInt(localStorage.getItem("highscore")) < score ||
				!localStorage.getItem("highscore")
			) {
				localStorage.setItem("highscore", `${score}`);
				newHighscore = true;
			}
		}
	}
</script>

<main>
	{#if gameOver}
		<button onclick={() => location.reload()}>Restart</button>
	{/if}

	<header>
		<p>
			Score: <strong
				class={gameOver &&
					newHighscore &&
					"animate__animated animate__infinite animate__bounce"}>{score}</strong
			>
		</p>
		<p>Highscore: <strong>{localStorage.getItem("highscore") || 0}</strong></p>
	</header>

	<div class="grid">
		<Block {addToGrid} {addToScore} {gameOver} x={0} y={0} from={null} />

		{#each fullGrid as block}
			<Block
				{addToGrid}
				{addToScore}
				{gameOver}
				x={block.x}
				y={block.y}
				from={block.from}
			/>
		{/each}
	</div>
</main>

<style>
	header {
		font-size: 2em;
		position: fixed;
		top: 10px;
		left: 0;
		text-align: center;
		width: 100%;
	}

	button {
		font-size: 2em;
		position: fixed;
		top: 20px;
		right: 20px;
		z-index: 99;
	}

	.grid {
		filter: drop-shadow(1px 1px 5px black);
		position: relative;
	}

	p {
		margin-top: 0;
		margin-bottom: 10px;

		+ p {
			font-size: 0.5em;

			strong {
				color: gold;
			}
		}
	}
</style>
