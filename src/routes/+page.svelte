<script lang="ts">
	import Bar from '$lib/components/Bar.svelte';
	import { onMount, onDestroy } from 'svelte';

	const BASE_HEIGHT = 25;
	const SELECTION_KEYS = ['a', 's', 'd', 'f', 'g', 'h', 'j', 'k', 'l', ';'];

	type GameState = 'idle' | 'playing' | 'finished';

	let gameState = $state<GameState>('idle');
	let heights = $state<number[]>([]);
	let selectedIndex = $state<number | null>(null);
	let elapsedTime = $state(0);
	let timerInterval: any;

	function generateRandomHeights() {
		const h = Array.from({ length: 10 }, (_, i) => (i + 1) * BASE_HEIGHT);
		// Fisher-Yates shuffle
		for (let i = h.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[h[i], h[j]] = [h[j], h[i]];
		}
		// Ensure it's not already sorted
		if (checkIfSorted(h)) {
			return generateRandomHeights();
		}
		return h;
	}

	function checkIfSorted(arr: number[]) {
		for (let i = 0; i < arr.length - 1; i++) {
			if (arr[i] > arr[i + 1]) return false;
		}
		return true;
	}

	function startGame() {
		heights = generateRandomHeights();
		selectedIndex = null;
		elapsedTime = 0;
		gameState = 'playing';
		const start = Date.now();
		timerInterval = setInterval(() => {
			elapsedTime = (Date.now() - start) / 1000;
		}, 10);
	}

	function finishGame() {
		clearInterval(timerInterval);
		gameState = 'finished';
	}

	function handleKeydown(event: KeyboardEvent) {
		const key = event.key.toLowerCase();

		if (key === ' ') {
			event.preventDefault();
			if (gameState === 'idle' || gameState === 'finished') {
				startGame();
			}
			return;
		}

		if (gameState !== 'playing') return;

		const keyIdx = SELECTION_KEYS.indexOf(key);
		if (keyIdx === -1) return;

		if (selectedIndex === keyIdx) {
			selectedIndex = null;
		} else if (selectedIndex === null) {
			selectedIndex = keyIdx;
		} else {
			// Swap
			const temp = heights[selectedIndex];
			heights[selectedIndex] = heights[keyIdx];
			heights[keyIdx] = temp;
			selectedIndex = null;

			if (checkIfSorted(heights)) {
				finishGame();
			}
		}
	}

	onDestroy(() => {
		clearInterval(timerInterval);
	});
</script>

<svelte:window onkeydown={handleKeydown} />

<main>
	<div class="header">
		<h1>Keyboard Sortcuts</h1>
		{#if gameState !== 'idle'}
			<div class="timer" class:finished={gameState === 'finished'}>
				{elapsedTime.toFixed(2)}s
			</div>
		{/if}
	</div>

	<div class="game-container">
		{#if gameState === 'idle'}
			<div class="overlay">
				<h2>Ready?</h2>
				<p>Press <strong>SPACE</strong> to Start</p>
			</div>
		{/if}

		{#if gameState === 'finished'}
			<div class="overlay finished-overlay">
				<h2>Sorted!</h2>
				<div class="final-time">{elapsedTime.toFixed(2)}s</div>
				<p>Press <strong>SPACE</strong> to Restart</p>
			</div>
		{/if}

		<div class="visualizer-container" class:blurred={gameState !== 'playing'}>
			<div class="visualizer">
				{#each heights as height, i}
					<Bar {height} selected={selectedIndex === i} />
				{/each}
			</div>
			<div class="indicators">
				{#each SELECTION_KEYS as key}
					<div class="indicator">
						<span class="key-label">{key}</span>
					</div>
				{/each}
			</div>
		</div>
	</div>

	<div class="controls">
		<div class="instructions">
			<p>Select with <strong>A-S-D-F-G-H-J-K-L-;</strong></p>
			<p>Swap with a different key while selected.</p>
		</div>
	</div>
</main>

<style>
	:global(body) {
		margin: 0;
		background-color: #f3f4f6;
		color: #1f2937;
		font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
		overflow: hidden;
	}

	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		min-height: 100vh;
		gap: 2rem;
	}

	.header {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.5rem;
	}

	h1 {
		margin: 0;
		font-size: 2.5rem;
		font-weight: 800;
		color: #111827;
	}

	.timer {
		font-family: 'JetBrains Mono', monospace;
		font-size: 1.5rem;
		font-weight: 700;
		color: #3b82f6;
		background: white;
		padding: 0.25rem 1rem;
		border-radius: 8px;
		box-shadow: 0 1px 3px rgba(0,0,0,0.1);
	}

	.timer.finished {
		color: #10b981;
		animation: pulse 1s infinite;
	}

	@keyframes pulse {
		0%, 100% { transform: scale(1); }
		50% { transform: scale(1.05); }
	}

	.game-container {
		position: relative;
	}

	.overlay {
		position: absolute;
		inset: 0;
		z-index: 10;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		background: rgba(255, 255, 255, 0.8);
		backdrop-filter: blur(4px);
		border-radius: 16px;
		text-align: center;
	}

	.overlay h2 {
		font-size: 2rem;
		margin-bottom: 0.5rem;
		color: #111827;
	}

	.finished-overlay h2 {
		color: #10b981;
	}

	.final-time {
		font-size: 3rem;
		font-weight: 800;
		color: #111827;
		margin: 1rem 0;
	}

	.visualizer-container {
		background-color: white;
		padding: 2.5rem 2rem 1.5rem 2rem;
		border-radius: 16px;
		box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
		transition: filter 0.3s;
	}

	.blurred {
		filter: blur(8px);
		pointer-events: none;
	}

	.visualizer {
		display: flex;
		align-items: flex-end;
		gap: 12px;
		height: 250px;
		min-width: 508px; /* Based on 10 bars (40px) + 9 gaps (12px) */
		padding-bottom: 8px;
		border-bottom: 2px solid #e5e7eb;
	}

	.indicators {
		display: flex;
		gap: 12px;
		margin-top: 12px;
	}

	.indicator {
		width: 40px;
		display: flex;
		justify-content: center;
	}

	.key-label {
		font-size: 0.875rem;
		font-weight: 600;
		color: #6b7280;
		text-transform: uppercase;
		background-color: #f9fafb;
		padding: 2px 6px;
		border: 1px solid #d1d5db;
		border-radius: 4px;
		min-width: 20px;
		text-align: center;
	}

	.controls {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1.5rem;
	}

	.instructions {
		text-align: center;
		color: #4b5563;
		line-height: 1.4;
		font-size: 0.95rem;
	}

	strong {
		color: #111827;
	}
</style>
