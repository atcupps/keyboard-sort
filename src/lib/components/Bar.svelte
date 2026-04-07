<script lang="ts">
	let { height, selected = false }: { height: number; selected?: boolean } = $props();
</script>

<div class="bar" class:selected style:height="{height}px"></div>

<style>
	.bar {
		width: 40px;
		background-color: #3b82f6;
		border-radius: 4px 4px 0 0;
		transition: background-color 0.2s, transform 0.2s;
	}

	.selected {
		background-color: #ef4444;
		transform: scaleX(1.1);
		box-shadow: 0 0 10px rgba(239, 68, 68, 0.5);
	}
</style>
