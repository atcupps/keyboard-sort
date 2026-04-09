<script lang="ts">
	import { page } from '$app/state';
	
	const score = page.params.score;
	const shareUrl = `http://keyboard-sort.vercel.app/share/${score}`;
	const ogImageUrl = `https://placehold.co/1200x630/3b82f6/white?text=Keyboard+Sortcuts:+${score}s`;
</script>

<svelte:head>
	<title>Keyboard Sortcuts - Score: {score}s</title>
	<meta name="description" content="I just sorted 10 bars in {score} seconds! Can you beat me?" />
	
	<!-- Open Graph / Facebook -->
	<meta property="og:type" content="website" />
	<meta property="og:url" content={shareUrl} />
	<meta property="og:title" content="Keyboard Sortcuts - My Score: {score}s" />
	<meta property="og:description" content="I just sorted 10 bars in {score} seconds! Can you beat me?" />
	<meta property="og:image" content={ogImageUrl} />

	<!-- Twitter -->
	<meta property="twitter:card" content="summary_large_image" />
	<meta property="twitter:url" content={shareUrl} />
	<meta property="twitter:title" content="Keyboard Sortcuts - My Score: {score}s" />
	<meta property="twitter:description" content="I just sorted 10 bars in {score} seconds! Can you beat me?" />
	<meta property="twitter:image" content={ogImageUrl} />
</svelte:head>

<main>
	<div class="card">
		<h1>Keyboard Sortcuts</h1>
		<div class="score-display">
			<span class="label">Time to Beat</span>
			<span class="score">{score}s</span>
		</div>
		<p>Someone just sorted 10 bars in {score} seconds using only their keyboard!</p>
		<a href="/" class="play-button">Play Now</a>
	</div>
</main>

<style>
	:global(body) {
		margin: 0;
		background-color: #f3f4f6;
		color: #1f2937;
		font-family: 'Lexend', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
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
		background: white;
		padding: 3rem;
		border-radius: 24px;
		box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
		text-align: center;
		max-width: 400px;
		width: 100%;
	}

	h1 {
		margin: 0 0 1.5rem 0;
		font-size: 2rem;
		font-weight: 800;
		color: #111827;
	}

	.score-display {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
		margin-bottom: 2rem;
	}

	.label {
		text-transform: uppercase;
		font-weight: 700;
		color: #6b7280;
		letter-spacing: 0.05em;
		font-size: 0.875rem;
	}

	.score {
		font-size: 4rem;
		font-weight: 900;
		color: #3b82f6;
		line-height: 1;
	}

	p {
		color: #4b5563;
		line-height: 1.6;
		margin-bottom: 2.5rem;
	}

	.play-button {
		display: inline-block;
		padding: 1rem 2.5rem;
		background-color: #3b82f6;
		color: white;
		text-decoration: none;
		border-radius: 12px;
		font-weight: 700;
		font-size: 1.125rem;
		transition: all 0.2s;
		box-shadow: 0 4px 6px -1px rgba(59, 130, 246, 0.3);
	}

	.play-button:hover {
		background-color: #2563eb;
		transform: translateY(-2px);
		box-shadow: 0 8px 12px -1px rgba(59, 130, 246, 0.4);
	}
</style>
