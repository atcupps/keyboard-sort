<script lang="ts">
	import { page } from '$app/state';
	
	const score = page.params.score;
	const shareUrl = `https://keyboard-sort.vercel.app/share/${score}`;
	const ogImageUrl = `https://placehold.co/1200x630/323437/e2b714?text=Keyboard+Sortcuts:+${score}s`;
</script>

<svelte:head>
	<title>keyboard sortcuts - score: {score}s</title>
	<meta name="description" content="I just sorted 10 bars in {score} seconds! Can you beat me?" />
	
	<!-- Open Graph / Facebook -->
	<meta property="og:type" content="website" />
	<meta property="og:url" content={shareUrl} />
	<meta property="og:title" content="keyboard sortcuts - my score: {score}s" />
	<meta property="og:description" content="I just sorted 10 bars in {score} seconds! Can you beat me?" />
	<meta property="og:image" content={ogImageUrl} />

	<!-- Twitter -->
	<meta property="twitter:card" content="summary_large_image" />
	<meta property="twitter:url" content={shareUrl} />
	<meta property="twitter:title" content="keyboard sortcuts - my score: {score}s" />
	<meta property="twitter:description" content="I just sorted 10 bars in {score} seconds! Can you beat me?" />
	<meta property="twitter:image" content={ogImageUrl} />
</svelte:head>

<main>
	<div class="card">
		<h1>keyboard sortcuts</h1>
		<div class="score-display">
			<span class="label">time to beat</span>
			<span class="score">{score}s</span>
		</div>
		<p>someone just sorted 10 bars in {score} seconds using only their keyboard!</p>
		<a href="/" class="play-button">play now</a>
	</div>
</main>

<style>
	:global(body) {
		margin: 0;
		background-color: #323437;
		color: #d1d0c5;
		font-family: 'Lexend', 'JetBrains Mono', monospace;
	}

	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		min-height: 100vh;
		padding: 2rem;
	}

	.card {
		background: #2c2e31;
		padding: 3rem;
		border-radius: 24px;
		border: 1px solid #3e4144;
		text-align: center;
		max-width: 400px;
		width: 100%;
	}

	h1 {
		margin: 0 0 1.5rem 0;
		font-size: 2rem;
		font-weight: 800;
		color: #e2b714;
		letter-spacing: -1px;
	}

	.score-display {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
		margin-bottom: 2rem;
	}

	.label {
		text-transform: lowercase;
		font-weight: 700;
		color: #646669;
		letter-spacing: 0.05em;
		font-size: 0.875rem;
	}

	.score {
		font-size: 4rem;
		font-weight: 900;
		color: #d1d0c5;
		line-height: 1;
	}

	p {
		color: #646669;
		line-height: 1.6;
		margin-bottom: 2.5rem;
		text-transform: lowercase;
	}

	.play-button {
		display: inline-block;
		padding: 1rem 2.5rem;
		background-color: #e2b714;
		color: #323437;
		text-decoration: none;
		border-radius: 12px;
		font-weight: 700;
		font-size: 1.125rem;
		transition: all 0.2s;
		text-transform: lowercase;
	}

	.play-button:hover {
		opacity: 0.9;
		transform: translateY(-2px);
	}
</style>
