<script>
	import Block from "./lib/Block.svelte";
	import explode from "./assets/explode.mp3";
	import { Howl } from "howler";

	let fullGrid = $state([]);
	let score = $state(0);
	let gameOver = $state(null);
	let newHighscore = $state(false);
	const maxBlocks = 50;

	const explosion = new Howl({
		src: explode,
		preload: true,
		volume: 0.4,
	});

	function addToGrid(x, y, from) {
		fullGrid.push({ x, y, from });
		if (fullGrid.length + 1 === maxBlocks) finishGame();
	}

	function finishGame() {
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

	function addToScore(n) {
		if (n) {
			score += n;
		} else {
			finishGame();
		}
	}
</script>

<main>
	{#if gameOver}
		<dialog open>
			<h2>
				{fullGrid.length + 1 === maxBlocks ? "Congratulations!" : "You died!"}
			</h2>
			<p>
				{fullGrid.length + 1 === maxBlocks
					? "Your village was so amazing, it exploded."
					: "Your village has exploded."}
			</p>
			<button
				onclick={() => {
					fullGrid = [];
					score = 0;
					gameOver = null;
					newHighscore = false;
				}}>Restart</button
			>
		</dialog>
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

	dialog {
		background: rgba(0, 0, 0, 0.5);
		bottom: 20px;
		z-index: 99;

		button {
			background: rgb(171, 12, 12);
		}
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
