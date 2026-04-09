<script lang="ts">
	let { height, selected = false }: { height: number; selected?: boolean } = $props();
</script>

<div class="bar" class:selected style:height="{height}px"></div>

<style>
	.bar {
		width: var(--bar-width, 40px);
		background-color: #e2b714;
		border-radius: 4px;
		transition: background-color 0.2s, transform 0.2s, box-shadow 0.2s, width 0.2s;
	}

	.selected {
		background-color: #ca4754;
		transform: scaleX(1.1);
		box-shadow: 0 0 15px rgba(202, 71, 84, 0.4);
	}
</style>
