<script lang="ts">
	import Bar from '$lib/components/Bar.svelte';
	import { onMount, onDestroy } from 'svelte';
	import { flip } from 'svelte/animate';

	const BASE_HEIGHT = 25;
	const DEFAULT_KEYS = ['1', '2', '3', '4', '5', '6', '7', '8', '9', '0'];

	type GameState = 'idle' | 'playing' | 'finished';

	let gameState = $state<GameState>('idle');
	let heights = $state<{ id: number; val: number }[]>([]);
	let initialHeights = $state<{ id: number; val: number }[]>([]);
	let selectedIndex = $state<number | null>(null);
	let elapsedTime = $state(0);
	let userSwaps = $state(0);
	let timerInterval: any;
	let copied = $state(false);
	let canvas: HTMLCanvasElement;

	// Theoretical Swaps
	let theoreticalSwaps = $state<{ name: string, count: number }[]>([]);

	// Custom Controls State
	let selectionKeys = $state<string[]>(DEFAULT_KEYS);
	let showModal = $state(false);
	let editingIndex = $state<number | null>(null);
	let tempKeys = $state<string[]>([...DEFAULT_KEYS]);
	let modalError = $state('');

	// Tutorial State
	let showTutorial = $state(false);
	let tutorialStep = $state(0);
	let tutorialHeights = $state<{ id: number; val: number }[]>([
		{ id: 1, val: 3 * BASE_HEIGHT },
		{ id: 2, val: 1 * BASE_HEIGHT },
		{ id: 3, val: 5 * BASE_HEIGHT },
		{ id: 4, val: 2 * BASE_HEIGHT },
		{ id: 5, val: 4 * BASE_HEIGHT },
		{ id: 6, val: 8 * BASE_HEIGHT },
		{ id: 7, val: 6 * BASE_HEIGHT },
		{ id: 8, val: 10 * BASE_HEIGHT },
		{ id: 9, val: 7 * BASE_HEIGHT },
		{ id: 10, val: 9 * BASE_HEIGHT }
	]);
	let tutorialSelectedIndex = $state<number | null>(null);
	let tutorialActiveKey = $state<number | null>(null);
	let tutorialInterval: any;

	onMount(() => {
		const saved = localStorage.getItem('keyboard-sort-keys');
		if (saved) {
			try {
				selectionKeys = JSON.parse(saved);
				tempKeys = [...selectionKeys];
			} catch (e) {
				console.error('Failed to parse saved keys', e);
			}
		}

		if (!localStorage.getItem('keyboard-sort-tutorial-seen')) {
			showTutorial = true;
			startTutorialAnimation();
		}
	});

	function startTutorialAnimation() {
		tutorialInterval = setInterval(() => {
			tutorialStep = (tutorialStep + 1) % 4;
			
			if (tutorialStep === 0) {
				// Reset
				tutorialHeights = [
					{ id: 1, val: 3 * BASE_HEIGHT },
					{ id: 2, val: 1 * BASE_HEIGHT },
					{ id: 3, val: 5 * BASE_HEIGHT },
					{ id: 4, val: 2 * BASE_HEIGHT },
					{ id: 5, val: 4 * BASE_HEIGHT },
					{ id: 6, val: 8 * BASE_HEIGHT },
					{ id: 7, val: 6 * BASE_HEIGHT },
					{ id: 8, val: 10 * BASE_HEIGHT },
					{ id: 9, val: 7 * BASE_HEIGHT },
					{ id: 10, val: 9 * BASE_HEIGHT }
				];
				tutorialSelectedIndex = null;
				tutorialActiveKey = null;
			} else if (tutorialStep === 1) {
				// Select first bar (Pos 1 -> Key '1')
				tutorialActiveKey = 0;
				setTimeout(() => {
					tutorialSelectedIndex = 0;
					tutorialActiveKey = null;
				}, 400);
			} else if (tutorialStep === 2) {
				// Swap with fourth bar (Pos 4 -> Key '4')
				tutorialActiveKey = 3;
				setTimeout(() => {
					const temp = tutorialHeights[0];
					tutorialHeights[0] = tutorialHeights[3];
					tutorialHeights[3] = temp;
					tutorialSelectedIndex = null;
					tutorialActiveKey = null;
				}, 400);
			}
		}, 1500);
	}

	function closeTutorial() {
		clearInterval(tutorialInterval);
		showTutorial = false;
		localStorage.setItem('keyboard-sort-tutorial-seen', 'true');
	}

	function generateRandomHeights() {
		const h = Array.from({ length: 10 }, (_, i) => ({ id: i, val: (i + 1) * BASE_HEIGHT }));
		for (let i = h.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[h[i], h[j]] = [h[j], h[i]];
		}
		if (checkIfSorted(h.map(x => x.val))) return generateRandomHeights();
		return h;
	}

	function checkIfSorted(arr: number[]) {
		for (let i = 0; i < arr.length - 1; i++) {
			if (arr[i] > arr[i + 1]) return false;
		}
		return true;
	}

	function startGame() {
		clearInterval(timerInterval);
		heights = generateRandomHeights();
		initialHeights = [...heights];
		selectedIndex = null;
		elapsedTime = 0;
		userSwaps = 0;
		gameState = 'playing';
		copied = false;
		const start = Date.now();
		timerInterval = setInterval(() => {
			elapsedTime = (Date.now() - start) / 1000;
		}, 10);
	}

	function finishGame() {
		clearInterval(timerInterval);
		theoreticalSwaps = calculateAllTheoreticalSwaps(initialHeights.map(x => x.val));
		gameState = 'finished';
	}

	function calculateAllTheoreticalSwaps(arr: number[]) {
		return [
			{ name: 'bubble sort', count: bubbleSortSwaps([...arr]) },
			{ name: 'selection sort', count: selectionSortSwaps([...arr]) },
			{ name: 'insertion sort', count: insertionSortSwaps([...arr]) },
			{ name: 'quicksort', count: quickSortSwaps([...arr]) }
		];
	}

	function bubbleSortSwaps(arr: number[]) {
		let count = 0;
		let n = arr.length;
		for (let i = 0; i < n; i++) {
			for (let j = 0; j < n - i - 1; j++) {
				if (arr[j] > arr[j + 1]) {
					[arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
					count++;
				}
			}
		}
		return count;
	}

	function selectionSortSwaps(arr: number[]) {
		let count = 0;
		for (let i = 0; i < arr.length; i++) {
			let minIdx = i;
			for (let j = i + 1; j < arr.length; j++) {
				if (arr[j] < arr[minIdx]) minIdx = j;
			}
			if (minIdx !== i) {
				[arr[i], arr[minIdx]] = [arr[minIdx], arr[i]];
				count++;
			}
		}
		return count;
	}

	function insertionSortSwaps(arr: number[]) {
		let count = 0;
		for (let i = 1; i < arr.length; i++) {
			let j = i;
			while (j > 0 && arr[j - 1] > arr[j]) {
				[arr[j], arr[j - 1]] = [arr[j - 1], arr[j]];
				count++;
				j--;
			}
		}
		return count;
	}

	function quickSortSwaps(arr: number[]) {
		let count = 0;
		function partition(low: number, high: number) {
			let pivot = arr[high];
			let i = low - 1;
			for (let j = low; j < high; j++) {
				if (arr[j] < pivot) {
					i++;
					[arr[i], arr[j]] = [arr[j], arr[i]];
					count++;
				}
			}
			[arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
			count++;
			return i + 1;
		}
		function sort(low: number, high: number) {
			if (low < high) {
				let pi = partition(low, high);
				sort(low, pi - 1);
				sort(pi + 1, high);
			}
		}
		sort(0, arr.length - 1);
		return count;
	}

	function formatKey(event: KeyboardEvent): string {
		let parts = [];
		if (event.ctrlKey) parts.push('Ctrl');
		if (event.altKey) parts.push('Alt');
		if (event.shiftKey) parts.push('Shift');
		if (event.metaKey) parts.push('Meta');
		const key = event.key;
		if (!['Control', 'Alt', 'Shift', 'Meta'].includes(key)) parts.push(key.length === 1 ? key.toUpperCase() : key);
		return parts.join('+');
	}

	function getEventKeyString(event: KeyboardEvent): string {
		let parts = [];
		if (event.ctrlKey) parts.push('ctrl');
		if (event.altKey) parts.push('alt');
		if (event.shiftKey) parts.push('shift');
		if (event.metaKey) parts.push('meta');
		const key = event.key.toLowerCase();
		if (!['control', 'alt', 'shift', 'meta'].includes(key)) parts.push(key);
		return parts.join('+');
	}

	function getMappedKeyString(keyStr: string): string {
		return keyStr.toLowerCase().replace(/\s\+\s/g, '+');
	}

	function handleKeydown(event: KeyboardEvent) {
		if (showModal || showTutorial) return;
		const pressedKey = getEventKeyString(event);
		if (pressedKey === ' ' || pressedKey === 'space') {
			event.preventDefault();
			startGame();
			return;
		}
		if (gameState !== 'playing') return;
		const keyIdx = selectionKeys.findIndex(k => getMappedKeyString(k) === pressedKey);
		if (keyIdx === -1) return;
		event.preventDefault();
		if (selectedIndex === keyIdx) {
			selectedIndex = null;
		} else if (selectedIndex === null) {
			selectedIndex = keyIdx;
		} else {
			const temp = heights[selectedIndex];
			heights[selectedIndex] = heights[keyIdx];
			heights[keyIdx] = temp;
			userSwaps++;
			selectedIndex = null;
			if (checkIfSorted(heights.map(x => x.val))) finishGame();
		}
	}

	function openCustomization() {
		tempKeys = [...selectionKeys];
		modalError = '';
		showModal = true;
	}

	function saveCustomization() {
		selectionKeys = [...tempKeys];
		localStorage.setItem('keyboard-sort-keys', JSON.stringify(selectionKeys));
		showModal = false;
	}

	function handleModalKeydown(event: KeyboardEvent, index: number) {
		event.preventDefault();
		const formatted = formatKey(event);
		if (event.key === 'Control' || event.key === 'Alt' || event.key === 'Shift' || event.key === 'Meta') return;
		if (event.key === ' ') {
			modalError = 'Space bar is reserved for Start/Restart.';
			return;
		}
		if (tempKeys.some((k, i) => i !== index && k === formatted)) {
			modalError = `Key "${formatted}" is already assigned!`;
			return;
		}
		tempKeys[index] = formatted;
		modalError = '';
	}

	async function shareScore() {
		const scoreStr = elapsedTime.toFixed(2);
		const shareUrl = `${window.location.origin}/share/${scoreStr}`;
		const text = `🎹 Keyboard Sortcuts\nI sorted 10 bars in ${scoreStr}s! ⚡️\nTry to beat me: ${shareUrl}`;
		const ctx = canvas.getContext('2d');
		if (ctx) {
			ctx.fillStyle = '#323437'; ctx.fillRect(0, 0, canvas.width, canvas.height);
			ctx.fillStyle = '#e2b714'; ctx.font = 'bold 40px sans-serif'; ctx.textAlign = 'center'; ctx.fillText('Keyboard Sortcuts', canvas.width / 2, 80);
			ctx.font = 'bold 120px sans-serif'; ctx.fillText(`${scoreStr}s`, canvas.width / 2, 240);
			ctx.fillStyle = '#d1d0c5'; ctx.font = '600 30px sans-serif'; ctx.fillText('10 BARS SORTED', canvas.width / 2, 310);
			ctx.font = '400 20px sans-serif'; ctx.globalAlpha = 0.5; ctx.fillText('keyboard-sortcuts.app', canvas.width / 2, 360); ctx.globalAlpha = 1.0;
		}
		if (navigator.share) {
			try {
				const blob = await new Promise<Blob | null>((resolve) => canvas.toBlob(resolve, 'image/png'));
				const files = blob ? [new File([blob], 'score.png', { type: 'image/png' })] : [];
				const shareData: ShareData = { title: 'Keyboard Sortcuts', text, url: shareUrl };
				if (files.length > 0 && navigator.canShare && navigator.canShare({ files })) shareData.files = files;
				await navigator.share(shareData); return;
			} catch (err) { console.error('Share failed', err); }
		}
		try { await navigator.clipboard.writeText(text); copied = true; setTimeout(() => (copied = false), 2000); } catch (err) { alert('Could not copy to clipboard.'); }
	}

	onDestroy(() => {
		clearInterval(timerInterval);
		clearInterval(tutorialInterval);
	});
</script>

<svelte:window onkeydown={handleKeydown} />

<canvas bind:this={canvas} width="600" height="400" style="display: none;"></canvas>

{#if showTutorial}
	<div class="modal-backdrop">
		<div class="modal tutorial-modal">
			<h2>how to play</h2>
			<div class="tutorial-animation">
				<div class="visualizer mini-visualizer">
					{#each tutorialHeights as h, i (h.id)}
						<div animate:flip={{ duration: 400 }}>
							<Bar height={h.val} selected={tutorialSelectedIndex === i} />
						</div>
					{/each}
				</div>
				<div class="indicators">
					{#each selectionKeys as key, i}
						<div class="indicator">
							<span class="key-label keycap" class:pressed={tutorialActiveKey === i}>{key}</span>
						</div>
					{/each}
				</div>
			</div>
			
			<div class="tutorial-steps">
				<p class:active={tutorialStep === 1}>1. select a bar using its key</p>
				<p class:active={tutorialStep === 2}>2. press another key to swap</p>
			</div>

			<div class="modal-actions">
				<button class="save-btn" onclick={closeTutorial}>start sorting</button>
			</div>
		</div>
	</div>
{/if}

{#if showModal}
	<div class="modal-backdrop">
		<div class="modal">
			<h2>Customize Controls</h2>
			<p class="modal-instructions">Click a box and press a key or combination (e.g., Shift + K).</p>
			<div class="key-grid">
				{#each tempKeys as key, i}
					<div class="key-input-group">
						<label>{i + 1}</label>
						<button class="key-capture-btn" class:editing={editingIndex === i} onclick={() => editingIndex = i} onkeydown={(e) => handleModalKeydown(e, i)}>{key}</button>
					</div>
				{/each}
			</div>
			{#if modalError} <p class="error">{modalError}</p> {/if}
			<div class="modal-actions">
				<button class="cancel-btn" onclick={() => showModal = false}>Cancel</button>
				<button class="save-btn" onclick={saveCustomization}>Save Changes</button>
			</div>
		</div>
	</div>
{/if}

<main>
	<div class="header">
		<h1>keyboard sortcuts</h1>
		{#if gameState !== 'idle'}
			<div class="timer" class:finished={gameState === 'finished'}>
				{elapsedTime.toFixed(2)}
			</div>
		{/if}
	</div>

	<div class="game-container">
		{#if gameState === 'idle'}
			<div class="overlay">
				<h2>ready?</h2>
				<p>press <strong>space</strong> to start</p>
			</div>
		{/if}

		{#if gameState === 'finished'}
			<div class="overlay finished-overlay">
				<div class="finished-content">
					<div class="main-stats">
						<h2>sorted!</h2>
						<div class="final-time">{elapsedTime.toFixed(2)}s</div>
						<div class="user-swaps-count">{userSwaps} swaps</div>
						<button class="share-button" onclick={shareScore}>
							{copied ? 'copied!' : 'share score'}
						</button>
						<p class="restart-hint">press <strong>space</strong> to restart</p>
					</div>
					<div class="comparison-panel">
						<h3>algorithm comparison</h3>
						<div class="comparison-grid">
							<div class="comparison-row user-row">
								<span class="algo-name">you</span>
								<span class="algo-count">{userSwaps}</span>
							</div>
							{#each theoreticalSwaps as algo}
								<div class="comparison-row">
									<span class="algo-name">{algo.name}</span>
									<span class="algo-count">{algo.count}</span>
								</div>
							{/each}
						</div>
					</div>
				</div>
			</div>
		{/if}

		<div class="visualizer-container" class:blurred={gameState !== 'playing'}>
			<div class="visualizer">
				{#each heights as h, i (h.id)}
					<div animate:flip={{ duration: 400 }}>
						<Bar height={h.val} selected={selectedIndex === i} />
					</div>
				{/each}
			</div>
			<div class="indicators">
				{#each selectionKeys as key}
					<div class="indicator">
						<span class="key-label">{key}</span>
					</div>
				{/each}
			</div>
		</div>
	</div>

	<div class="controls">
		<div class="instructions">
			<p>use your custom keys to select and swap bars.</p>
			<p>press <strong>space</strong> to restart at any time.</p>
		</div>
		<button class="customize-btn" onclick={openCustomization}>
			customize controls
		</button>
	</div>
</main>

<style>
	:global(body) {
		margin: 0;
		background-color: #323437;
		color: #d1d0c5;
		font-family: 'Lexend', 'JetBrains Mono', monospace;
		overflow: hidden;
	}

	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		min-height: 100vh;
		gap: 3rem;
	}

	.header {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1rem;
	}

	h1 {
		margin: 0;
		font-size: 2.5rem;
		font-weight: 800;
		color: #e2b714;
		letter-spacing: -1px;
	}

	.timer {
		font-family: 'JetBrains Mono', monospace;
		font-size: 2rem;
		font-weight: 700;
		color: #646669;
	}

	.timer.finished {
		color: #e2b714;
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
		background: rgba(50, 52, 55, 0.7);
		backdrop-filter: blur(4px);
		border-radius: 24px;
		text-align: center;
		padding: 2rem;
	}

	.overlay h2 {
		font-size: 2.5rem;
		margin-bottom: 0.5rem;
		color: #e2b714;
		font-weight: 800;
	}

	.finished-overlay h2 {
		color: #e2b714;
		margin-bottom: 0;
	}

	.finished-content {
		display: flex;
		align-items: center;
		gap: 4rem;
		text-align: left;
	}

	@media (max-width: 800px) {
		.finished-content {
			flex-direction: column;
			gap: 2rem;
		}
	}

	.main-stats {
		display: flex;
		flex-direction: column;
		align-items: center;
		text-align: center;
	}

	.user-swaps-count {
		font-size: 1.2rem;
		color: #646669;
		margin-bottom: 1rem;
		text-transform: lowercase;
	}

	.comparison-panel {
		background: rgba(44, 46, 49, 0.5);
		padding: 1.5rem 2rem;
		border-radius: 16px;
		border: 1px solid #3e4144;
		min-width: 250px;
	}

	.comparison-panel h3 {
		margin: 0 0 1rem 0;
		font-size: 0.9rem;
		color: #646669;
		text-transform: lowercase;
		letter-spacing: 1px;
	}

	.comparison-grid {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
	}

	.comparison-row {
		display: flex;
		justify-content: space-between;
		font-size: 0.85rem;
		color: #d1d0c5;
		text-transform: lowercase;
	}

	.user-row {
		color: #e2b714;
		font-weight: 700;
		border-bottom: 1px solid #3e4144;
		padding-bottom: 0.5rem;
		margin-bottom: 0.25rem;
	}

	.algo-name {
		opacity: 0.8;
	}

	.algo-count {
		font-family: 'JetBrains Mono', monospace;
	}

	.final-time {
		font-size: 4rem;
		font-weight: 800;
		color: #d1d0c5;
		margin: 0;
	}

	.share-button {
		margin: 1rem 0;
		padding: 0.75rem 1.5rem;
		background-color: #2c2e31;
		color: #d1d0c5;
		border: 1px solid #3e4144;
		border-radius: 8px;
		font-family: inherit;
		font-weight: 600;
		font-size: 0.9rem;
		cursor: pointer;
		transition: all 0.2s;
		text-transform: lowercase;
	}

	.share-button:hover {
		background-color: #3e4144;
		border-color: #e2b714;
		color: #e2b714;
	}

	.restart-hint {
		margin-top: 1rem;
		color: #646669;
		font-size: 0.9rem;
	}

	.visualizer-container {
		padding: 1rem 2rem;
		border-radius: 16px;
		transition: filter 0.3s;
	}

	.blurred {
		filter: blur(10px);
		pointer-events: none;
	}

	.visualizer {
		display: flex;
		align-items: flex-end;
		gap: 12px;
		height: 250px;
		min-width: 508px;
		padding-bottom: 12px;
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
		overflow: visible;
	}

	.key-label {
		font-size: 0.75rem;
		font-weight: 600;
		color: #d1d0c5;
		text-transform: lowercase;
		background-color: #2c2e31;
		border: 1px solid #3e4144;
		border-radius: 6px;
		padding: 4px 8px;
		min-width: 24px;
		text-align: center;
		white-space: nowrap;
		position: relative;
		z-index: 5;
		transition: all 0.2s;
	}

	.keycap.pressed {
		background-color: #e2b714;
		color: #323437;
		border-color: #e2b714;
		transform: translateY(2px);
		box-shadow: 0 0 10px rgba(226, 183, 20, 0.4);
	}

	.controls {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 2rem;
	}

	.instructions {
		text-align: center;
		color: #646669;
		line-height: 1.6;
		font-size: 0.9rem;
	}

	strong {
		color: #d1d0c5;
		font-weight: 700;
	}

	.customize-btn {
		background-color: #2c2e31;
		border: 1px solid #3e4144;
		color: #646669;
		padding: 0.6rem 1.2rem;
		border-radius: 8px;
		font-family: inherit;
		font-size: 0.8rem;
		cursor: pointer;
		transition: all 0.2s;
		text-transform: lowercase;
	}

	.customize-btn:hover {
		background-color: #3e4144;
		border-color: #646669;
		color: #d1d0c5;
	}

	/* Modal Styles */
	.modal-backdrop {
		position: fixed;
		inset: 0;
		background: rgba(32, 34, 37, 0.9);
		backdrop-filter: blur(8px);
		z-index: 100;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.modal {
		background: #323437;
		padding: 3rem;
		border-radius: 24px;
		width: 95%;
		max-width: 1000px;
		color: #d1d0c5;
	}

	.modal h2 { margin: 0 0 0.5rem 0; text-align: center; color: #e2b714; text-transform: lowercase; }
	.modal-instructions { color: #646669; margin-bottom: 2.5rem; text-align: center; text-transform: lowercase; }

	.tutorial-modal {
		max-width: 700px;
	}

	.tutorial-animation {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1.5rem;
		margin: 2rem 0;
		padding: 2rem;
		background: #2c2e31;
		border-radius: 16px;
	}

	.mini-visualizer {
		min-width: unset;
		gap: 8px;
	}

	.tutorial-steps {
		text-align: center;
		margin-bottom: 2rem;
	}

	.tutorial-steps p {
		margin: 0.5rem 0;
		color: #646669;
		transition: color 0.3s;
	}

	.tutorial-steps p.active {
		color: #e2b714;
		font-weight: 700;
	}

	.key-grid {
		display: flex;
		justify-content: center;
		gap: 12px;
		margin-bottom: 2.5rem;
		padding: 2rem;
		background: #2c2e31;
		border-radius: 12px;
	}

	.key-input-group {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 12px;
		flex: 1;
		min-width: 0;
	}

	.key-input-group label {
		font-size: 0.7rem;
		font-weight: 700;
		color: #d1d0c5;
	}

	.key-capture-btn {
		width: 100%;
		height: 64px;
		padding: 8px;
		background: #323437;
		border: 2px solid #3e4144;
		border-radius: 8px;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.75rem;
		font-weight: 700;
		color: #d1d0c5;
		display: flex;
		align-items: center;
		justify-content: center;
		text-align: center;
		cursor: pointer;
		transition: all 0.2s;
		word-break: break-all;
		line-height: 1.2;
		text-transform: lowercase;
	}

	.key-capture-btn:focus, .key-capture-btn.editing {
		outline: none;
		border-color: #e2b714;
		color: #e2b714;
		transform: translateY(-2px);
	}

	.error {
		color: #ca4754;
		font-size: 0.875rem;
		font-weight: 600;
		margin-bottom: 2rem;
		text-align: center;
		text-transform: lowercase;
	}

	.modal-actions {
		display: flex;
		justify-content: center;
		gap: 2rem;
	}

	.save-btn {
		padding: 0.75rem 2rem;
		background: #e2b714;
		color: #323437;
		border: none;
		border-radius: 8px;
		font-weight: 700;
		cursor: pointer;
		font-family: inherit;
		text-transform: lowercase;
		transition: opacity 0.2s;
	}

	.save-btn:hover {
		opacity: 0.9;
	}

	.cancel-btn {
		padding: 0.75rem 2rem;
		background: #2c2e31;
		color: #646669;
		border: none;
		border-radius: 8px;
		font-weight: 700;
		cursor: pointer;
		font-family: inherit;
		text-transform: lowercase;
		transition: color 0.2s;
	}

	.cancel-btn:hover {
		color: #d1d0c5;
	}
</style>
