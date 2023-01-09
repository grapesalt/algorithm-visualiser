<script lang="ts">
	import { onMount } from 'svelte';
	import { flip } from 'svelte/animate';
	import { sineInOut } from 'svelte/easing';

	/**
	 * Generate array of n number of random integers between min and max
	 * @param n number of items
	 * @param min Lower range of numbers (default: 5)
	 * @param max Upper range of numbers (default: 1000)
	 */
	const getRandomArray = (n: number, min = 5, max = 1000) => {
		let array = [];

		for (let i = 0; i < n; i++) {
			array.push(Math.random() * (max - min) + min);
		}

		return array;
	};

	// Number of elements
	let n = 40;
	let array = getRandomArray(n);

	// Index of the currently compared values
	let _i: number | undefined;
	let _j: number | undefined;

	// Speed of the animation in ms
	let speed = 100;

	// Algorithms

	/**
	 * Sort the array using bubble sort
	 */
	function* bubbleSort() {
		let i: number;
		for (i = 0; i < array.length; i++) {
			for (let j = 0; j < array.length - i - 1; j++) {
				if (array[j] > array[j + 1]) {
					yield swap(j, j + 1);
				}
			}
		}

		return i + 1;
	}
	/**
	 * Sort the array using bubble sortselectiv
	 */
	function* selectiveSort() {
		let i: number;
		for (i = 0; i < array.length; i++) {
			for (let j = i + 1; j < array.length; j++) {
				if (array[j] < array[i]) {
					yield swap(j, i);
				}
			}
		}

		return i + 1;
	}

	/**
	 * Animate the sorting functions
	 * @param sorter The sorting function to be used
	 */
	const animateSort = (sorter: () => Generator) => {
		let frameId: number;
		let lastTime = 0;
		function sortNext(time: number) {
			frameId = requestAnimationFrame(sortNext);
			if (time - lastTime < speed) {
				return;
			}
			lastTime = time;

			const { done } = sorter().next();
			if (done) {
				_i = _j = undefined;
				cancelAnimationFrame(frameId);
			}
		}
		frameId = requestAnimationFrame(sortNext);
		return () => cancelAnimationFrame(frameId);
	};

	/**
	 * Swap the positions of 'i' and 'j' in the array
	 * @param i
	 * @param j
	 */
	function swap(i: number, j: number) {
		_i = i;
		_j = j;
		const _t = array[i];
		array[i] = array[j];
		array[j] = _t;
	}
</script>

<!-- Generate array bars -->
<div class="flex justify-center">
	{#each array as element, index (element)}
		<div
			class={`my-0 mx-px
			${index === _i || index === _j ? 'bg-red-400' : 'bg-yellow-300'} 
			inline-block`}
			style={`height: ${element * 0.75}px; width: ${n}vw`}
			animate:flip={{ duration: speed, easing: sineInOut }}
		/>
	{/each}
</div>
<button class="w-20 h-20 bg-red-300" on:click={() => animateSort(bubbleSort)}>Bubble Sort</button>

<button class="w-20 h-20 bg-red-300" on:click={() => animateSort(selectiveSort)}>
	Selective Sort
</button>

<button class="w-20 h-20 bg-green-300" on:click={() => (array = getRandomArray(n))}>
	Generate
</button>

<input type="range" min={50} max={500} bind:value={speed} />Speed
<input type="range" min={2} max={40} bind:value={n} />Size
