<script>
	import { onMount } from 'svelte';
	import { MDCRipple } from '@material/ripple';
	export let title, subhead;

	onMount(() => {
		const selector = '.mdc-button, .mdc-icon-button, .mdc-card__primary-action';
		const ripples = [].map.call(document.querySelectorAll(selector), function (el) {
			return new MDCRipple(el);
		});
	});
</script>

<div class="mdc-card my-card">
	<div class="my-card-title">{title}</div>
	<div class="my-card-subhead">{subhead}</div>
	<slot />
</div>
