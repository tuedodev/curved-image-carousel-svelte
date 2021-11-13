<script>
	import CurvedCarousel from './components/CurvedCarousel.svelte';
	import images from './json/images.json';

	const pictureSliderArray = Array.from(images.items).map((i) => {
		return {
			...i,
			alt: i.caption,
			slide: null,
		};
	});
</script>

<main>
	<CurvedCarousel imageArray={pictureSliderArray} />
</main>

<style global lang="scss">
	@use "@material/fab";
	@use "@material/card";
	@include fab.core-styles;
	@include card.core-styles;
	@import url('https://fonts.googleapis.com/css2?family=Dongle&display=swap');

	$primary: snow;
	$secondary: rgba(252, 252, 252, 0.1);

	:root {
		--primary-text-color: #{$primary};
		--primary-text-color-emphasized: #d3d3d3;
		--background-color: #{$secondary};
		--background-color-emphasized: rgba(252, 252, 252, 0.25);
		--carousel-animation: 300ms;
		--shadow: rgba(0, 0, 0, 1);
	}

	* {
		padding: 0;
		margin: 0;
	}

	body {
		width: 100vw;
		min-height: 100vh;
		background: #0f2027; /* fallback for old browsers */
		background: -webkit-linear-gradient(to right, #2c5364, #203a43, #0f2027);
		background: linear-gradient(to right, #2c5364, #203a43, #0f2027);
		font-family: 'Dongle', sans-serif;
	}

	ul {
		list-style: none;
	}

	.mdc-fab {
		--mdc-theme-on-secondary: #{$primary};
		--mdc-theme-secondary: #{$secondary};
	}

	.my-card {
		--mdc-theme-surface: rgba(252, 252, 252, 0.25);
		color: var(--primary-text-color);
		padding: 16px;
		cursor: auto;
		border-radius: 0.5rem;

		& .my-card-title {
			font-size: clamp(1.8rem, 1.5rem + 2.6vw, 3.5rem);
		}

		& .my-card-subhead {
			font-size: clamp(1.2rem, 1rem + 2vw, 2.2rem);
			line-height: 1;
		}

		& .my-card-image-container {
			display: flex;
			flex-direction: row;
			flex-wrap: wrap;
			margin-top: 1.4rem;

			& .my-card-image-holder {
				flex: 0 0 auto;
				max-width: 100%;

				& img {
					max-height: 120px;
					margin: 0;
					padding: 0;
					border-radius: 0.5rem;
					max-width: 100%;
				}

				&:not(:last-child) {
					& img {
						margin-right: 8px;
					}
				}
			}
		}
	}

	.text {
		font-size: clamp(1rem, 1rem + 2.2vw, 2.6rem);
		line-height: 0.85;
	}

	.notransition {
		-webkit-transition: none !important;
		-moz-transition: none !important;
		-o-transition: none !important;
		transition: none !important;
	}

	// Media Queries
	@media (max-width: 768px) {
		.my-card {
			& .my-card-image-container {
				flex-direction: column;
				align-items: center;

				& .my-card-image-holder {
					& img {
						max-width: 180px;
						margin: 0;
						padding: 0;
					}

					&:not(:last-child) {
						& img {
							margin-right: 0;
							margin-bottom: 8px;
						}
					}
				}
			}
		}
	}
</style>
