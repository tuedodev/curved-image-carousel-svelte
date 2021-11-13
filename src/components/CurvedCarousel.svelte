<script>
	// @ts-nocheck
	import { onMount, onDestroy } from 'svelte';
	import { fly } from 'svelte/transition';
	import MaterialIconButton from './MaterialIconButton.svelte';
	import MaterialCard from './MaterialCard.svelte';
	export let imageArray;

	let vw, vh, divider, resizeObserver, captionHeight, currentActive, oldSlideCenter;
	// Mouse and Touch events
	let currentIndex = 0,
		slideWidth = 0,
		movedBy = 0,
		grabbedIndex = 0,
		startPos = 0,
		wheelCounter = 0,
		isDragging = false;
	const MINIMUM_AMOUNT_OF_IMAGES = 5;
	const numberOfSlides = imageArray.length;

	// Functions for graph
	const graph = (x) => -((Math.pow(x, 2) - 36) / 18) - 2;
	const derivation = (x) => -x / 9;
	const angle = (x) => (180 / Math.PI) * Math.atan(derivation(x));

	function setSliderPosition() {
		let newOrderArray = currentIndex > 0 ? Array.from({ length: currentIndex }, (_, i) => i) : [];
		newOrderArray = [
			...Array.from({ length: numberOfSlides - currentIndex }, (_, i) => currentIndex + i),
			...newOrderArray,
		];
		let medium = getMediumOfXAxis(imageArray);
		let slideCenter = (medium + currentIndex) % numberOfSlides;
		if (slideCenter !== oldSlideCenter) {
			flipBothEndsWithoutTransition(newOrderArray);
		}
		newOrderArray.forEach((slidenumber, index) => {
			let slide = imageArray[slidenumber].slide;
			let currentPositionOnXAxis = index - medium + 1;
			const { xpos, ypos, rotate } = calculateSlidePosition(
				currentPositionOnXAxis + (movedBy / slideWidth) * 0.9
			);
			slide.style.transform = `translate(${xpos}px, ${ypos}px) rotate(${-rotate}deg)`;
			slide.style.transform = `translate(${xpos}px, ${ypos}px) rotate(${-rotate}deg)`;
		});
		oldSlideCenter = slideCenter;
		let nowInFocus = (currentIndex + medium - 1) % numberOfSlides;
		updateCaption(nowInFocus);
	}

	function flipBothEndsWithoutTransition(array) {
		let duration = 300;
		let firstItem = array[0];
		let lastItem = array[array.length - 1];
		let medium = getMediumOfXAxis(array);
		positionSlideWithoutTransition(imageArray[firstItem], array.length - 1);
		positionSlideWithoutTransition(imageArray[lastItem], -medium);

		function positionSlideWithoutTransition(currentSlide, newPosition) {
			currentSlide.slide.classList.add('notransition');
			currentSlide.slide.style.visibility = 'hidden';
			const { xpos, ypos, rotate } = calculateSlidePosition(newPosition);
			currentSlide.slide.style.transform = `translate(${xpos}px, ${ypos}px) rotate(${-rotate}deg)`;
			// eslint-disable-next-line
			let dummy = getComputedStyle(currentSlide.slide).transform; // Force Rendering
			setTimeout(function () {
				currentSlide.slide.classList.remove('notransition');
				currentSlide.slide.style.visibility = 'visible';
			}, duration / 2);
		}
	}
	function getMediumOfXAxis(array) {
		let len = array.length;
		let medium = len % 2 === 0 ? Math.floor(len / 2) : Math.ceil(len / 2);
		return medium;
	}

	function calculateSlidePosition(xPosition) {
		let rotate = angle(xPosition);
		let xpos = xPosition * slideWidth * 0.9; // (vw / 2) - slideWidth / 2???
		let yCurve = graph(xPosition) * (vh / 1.5);
		let ypos = -yCurve; // vh / 2 - slideHeight / 2 - yCurve;
		return { xpos, ypos, rotate };
	}

	function updateCurrentIndex(value) {
		currentIndex += value;
		currentIndex = currentIndex < 0 ? numberOfSlides - 1 : currentIndex;
		currentIndex = currentIndex > numberOfSlides - 1 ? 0 : currentIndex;
		setSliderPosition();
	}

	onMount(() => {
		if (imageArray.length >= MINIMUM_AMOUNT_OF_IMAGES) {
			resizeObserver = new ResizeObserver((entries) => {
				if (Array.isArray(entries)) {
					vw = entries[0].contentRect.width;
					vh = entries[0].contentRect.height;
					divider = vw >= 960 ? 2.3 : 1.4;
					slideWidth = vw / divider;
					setSliderPosition();
				}
			});
			resizeObserver.observe(document.documentElement);
		}
	});
	onDestroy(() => {
		resizeObserver.unobserve(document.documentElement);
	});

	// Event handlers/listeners
	function grabStartHandler(event) {
		event.preventDefault();
		if (Number.isInteger(this)) {
			grabbedIndex = this;
			isDragging = true;
			startPos = getPositionX(event);
			movedBy = 0;
		}
	}
	function grabEndHandler(event) {
		let ratio = movedBy / slideWidth;
		movedBy = 0;
		isDragging = false;
		if (Math.abs(ratio) >= 0.2) {
			let add = Math.sign(ratio) * -1;
			updateCurrentIndex(add);
		} else {
			setSliderPosition();
		}
	}

	function handleMove(event) {
		if (isDragging) {
			let currentX = getPositionX(event);
			let newMovedBy = currentX - startPos;
			let ratio = Math.abs(newMovedBy / slideWidth);
			if (newMovedBy !== movedBy && ratio < 0.75) {
				movedBy = newMovedBy * 0.85;
				setSliderPosition();
			}
		}
	}

	function getPositionX(event) {
		return event.type.includes('mouse') ? event.pageX : event.touches[0].clientX;
	}

	function wheel(event) {
		let sign = Math.sign(event.deltaY);
		if ((sign > 0 && wheelCounter < 0) || (sign < 0 && wheelCounter > 0)) wheelCounter = 0;
		wheelCounter += sign * 0.2;
		let add = 0;
		if (Math.abs(wheelCounter) > 1) {
			add = Math.sign(wheelCounter);
			wheelCounter = 0;
		}
		updateCurrentIndex(add);
	}
	function updateCaption(nowInFocus) {
		if (nowInFocus !== currentActive) {
			currentActive = nowInFocus;
		}
	}

	function handleClickEvent(event) {
		let value = event.detail.value;
		if (typeof value === 'number') {
			updateCurrentIndex(value);
		}
	}
</script>

<svelte:head>
	<link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons" />
	<title>A Curved Carousel in Svelte with Grabbing Function</title>
</svelte:head>

<svelte:body on:wheel={imageArray.length >= MINIMUM_AMOUNT_OF_IMAGES ? wheel : null} />

<div class="slides-container" bind:clientWidth={vw} bind:clientHeight={vh}>
	{#if imageArray.length >= MINIMUM_AMOUNT_OF_IMAGES}
		<div class="slides-wrapper">
			<div class="arrows previous">
				<MaterialIconButton on:clickEvent={handleClickEvent} value={-1}>arrow_back_ios</MaterialIconButton>
			</div>
			<div class="arrows next">
				<MaterialIconButton on:clickEvent={handleClickEvent} value={1}>arrow_forward_ios</MaterialIconButton>
			</div>
			{#each imageArray as image, index}
				<div
					class={index === grabbedIndex && isDragging ? 'slide grabbed' : 'slide'}
					bind:this={imageArray[index].slide}
				>
					<figure
						id="f{index}"
						on:mousedown={grabStartHandler.bind(index)}
						on:mouseup={grabEndHandler}
						on:mousemove={handleMove}
						on:mouseleave={grabEndHandler}
						on:touchstart={grabStartHandler.bind(index)}
						on:touchend={grabEndHandler}
						on:touchmove={handleMove}
					>
						<img src={image.src} alt={image.alt} style="width: {slideWidth}px;" on:dragstart|preventDefault />
					</figure>
				</div>
			{/each}
			<div class="caption-container">
				<div class="caption-wrapper">
					{#if currentActive || currentActive === 0}
						{#key currentActive}
							<a
								class="text"
								href={imageArray[currentActive].href}
								target="_blank"
								rel="noopener noreferrer"
								bind:clientHeight={captionHeight}
								in:fly={{ y: captionHeight }}
								out:fly={{ y: -captionHeight }}
							>
								{imageArray[currentActive].caption}
							</a>
						{/key}
					{/if}
				</div>
			</div>
		</div>
	{:else}
		<div class="card-holder">
			<MaterialCard title="Warning" subhead="Image Carousel only works with at least 5 pictures.">
				<ul class="my-card-image-container">
					{#each imageArray as image}
						<li class="my-card-image-holder">
							<a href={image.href} target="_blank" rel="noopener noreferrer">
								<img src={image.src} alt={image.alt} on:dragstart|preventDefault />
							</a>
						</li>
					{/each}
				</ul>
			</MaterialCard>
		</div>
	{/if}
</div>

<style lang="scss">
	.slides-container {
		position: fixed;
		top: 0;
		left: 0;
		height: 100vh;
		width: 100vw;
		display: flex;
		align-items: center;
		justify-content: center;
		overflow: hidden;
		scrollbar-width: none;
		transform: translateX(0);
		will-change: transform;
		transition: transform 0.3s ease-out;
		cursor: grab;
		z-index: 0;

		& .slides-wrapper {
			position: relative;
			display: flex;
			align-items: center;
			width: 100%;
			height: 100%;
			transform-origin: center;
			z-index: 0;

			& .arrows {
				position: absolute;
				top: 50%;
				transform: translateY(-50%) scale(1.2);
				z-index: 10000;

				&.previous {
					left: 1rem;
				}

				&.next {
					right: 1rem;
				}
			}

			& .slide {
				max-height: 85vh;
				width: 100vw;
				display: flex;
				justify-content: center;
				padding: 1rem;
				position: absolute;
				transform-origin: center;
				transition: transform 0.3s ease-in-out;
				pointer-events: none;

				& figure {
					position: relative;
					pointer-events: auto;
					transform-origin: center;
					//transition: transform var(--carousel-animation) ease-in-out;
					cursor: grab;
					user-select: none;

					& img {
						object-fit: cover;
						max-width: 100%;
						max-height: 100%;
						width: auto;
						height: auto;
						box-shadow: 5px 5px 50px -1px var(--shadow);
						border-radius: 4px;
						transition: transform var(--carousel-animation) ease-in-out,
							box-shadow var(--carousel-animation) ease-in-out;
						transform: scale(1);
					}
				}

				&.grabbed {
					& figure {
						cursor: grabbing;
					}
				}
			}

			& .caption-container {
				position: absolute;
				left: 0;
				bottom: 2rem;
				width: 100%;
				height: 120px;
				overflow: hidden;

				& .caption-wrapper {
					position: static;
					display: flex;
					align-items: center;
					justify-content: center;
					height: 100%;

					& a {
						display: inline-block;
						position: absolute;
						color: var(--primary-text-color);
						text-decoration: none;
						padding: 0.2rem 0.8rem;
						background-color: var(--background-color);
						border-radius: 0.6rem;
						text-align: center;

						&:hover,
						&:focus {
							background-color: var(--background-color-emphasized);
							color: var(--primary-text-color-emphasized);
						}
					}
				}
			}
		}

		.card-holder {
			width: 85vw;
			margin: 0 auto;
			max-width: 600px;
		}
	}
</style>
