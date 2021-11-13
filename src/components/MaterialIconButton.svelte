<script>
	import { onMount, createEventDispatcher } from 'svelte';
	import { MDCRipple } from '@material/ripple';
	import throttle from 'just-throttle';
	export let value;

	let iconButton;
	const dispatch = createEventDispatcher();

	onMount(() => {
		const fabRipple = new MDCRipple(document.querySelector('.mdc-fab'));
	});

	function clickHandler() {
		dispatch('clickEvent', { value: value });
	}
</script>

<button class="mdc-fab" aria-label="Favorite" bind:this={iconButton} on:click={throttle(clickHandler, 500)}>
	<div class="mdc-fab__ripple" />
	<span class="mdc-fab__icon material-icons"><slot /></span>
</button>
