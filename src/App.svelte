<script>
	import Block from "./lib/Block.svelte";
	import explode from "./assets/explode.mp3";
	import { Howl } from "howler";
	import getRandom from "./lib/random";

	let fullGrid = $state([]);
	let score = $state(0);
	let gameOver = $state(null);
	let newHighscore = $state(false);
	let currentHighScore = $state(localStorage.getItem("highscore") || 0);
	const maxBlocks = 50;

	const explosion = new Howl({
		src: explode,
		preload: true,
		volume: 0.4,
	});

	function addToGrid(x, y, from, score) {
		fullGrid.push({ x, y, from, score });
		if (fullGrid.length + 1 === maxBlocks) finishGame();
	}

	function finishGame() {
		gameOver = true;
		explosion.play();

		// Save new highscore
		if (
			!localStorage.getItem("highscore") ||
			parseInt(localStorage.getItem("highscore")) < score
		) {
			localStorage.setItem("highscore", `${score}`);
			newHighscore = true;
			currentHighScore = score;
		}

		// Move sprites to rain down
		const bod = document.body.getBoundingClientRect();
		document.querySelectorAll("[data-mushroom]").forEach((el) => {
			el.setAttribute("data-fall", "true");
			el.style.left = `${getRandom(bod.width) - 200}px`;
			el.style.animationDuration = `${getRandom(4, 1)}.${getRandom(9, 0)}s`;
			el.style.animationDelay = `${getRandom(2, 0)}.${getRandom(9, 0)}s`;
			document.body.appendChild(el);
		});
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
					document.querySelectorAll("[data-fall]").forEach((el) => el.remove());
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
		<p>Highscore: <strong>{currentHighScore}</strong></p>
	</header>

	<div class="grid">
		{#if !gameOver}
			<Block {addToGrid} {addToScore} {gameOver} x={0} y={0} from={null} />
		{:else}
			<Block {addToGrid} {addToScore} {gameOver} x={0} y={0} from={null} />
		{/if}

		{#each fullGrid as block}
			<Block
				{addToGrid}
				{addToScore}
				{fullGrid}
				{gameOver}
				x={block.x}
				y={block.y}
				from={block.from}
				score={block.score}
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
		color: white;
		z-index: 99;

		button {
			background: rgb(171, 12, 12);
			color: white;
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

	@keyframes fall {
		from {
			translate: 0 0;
			rotate: 0deg;
		}
		to {
			translate: 0 150vh;
			rotate: 720deg;
		}
	}

	:global(img[data-fall]) {
		animation: fall 5s;
		animation-timing-function: cubic-bezier(0.43, 0.01, 1, 1);
		position: absolute;
		top: -100px;
	}
</style>
