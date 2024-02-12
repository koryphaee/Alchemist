<script lang="ts">
	import { persisted } from 'svelte-persisted-store'

	const themes = [
		[
			'#000000', // unknown
			'#CD6F7E', // salmon
			'#A55EBB', // pink
			'#69E3AB', // lime
			'#0B25C3', // dark blue
			'#3E511C', // olive
			'#68A1DF', // light blue
			'#707070', // gray
			'#A6433F', // red
			'#613289', // purple
			'#CD8646', // orange
			'#76A835', // green
			'#FBE862' // yellow
		]
	]

	let theme = persisted('theme', 0)
	let columns = persisted('columns', 7)
	let rows = persisted('rows', 2)
	let height = persisted('height', 4)
	let selectedColor = persisted('color', 0)
	let map = persisted('map', new Array<number>())

	$: colors = themes[$theme]
	$: blockWidth = Math.floor(100 / $columns)
	$: blockHeight = Math.floor(100 / ($height * $rows))
	$: blockSize = Math.min(blockWidth, blockHeight)

	function scroll(index: number, e: WheelEvent) {
		map.update((m) => {
			const offset = Math.max(-1, Math.min(e.deltaY, 1))
			m[index] = ((m[index] ?? 0) + offset + colors.length) % colors.length
			return m
		})
	}

	function click(index: number) {
		map.update((m) => {
			m[index] = $selectedColor
			return m
		})
	}

	function reset() {
		map.set([])
	}

	function format(value: number | undefined) {
		return value ? value : '?'
	}
</script>

<style lang="postcss">
	section {
		container-type: size;
	}

	input,
	button,
	select,
	div {
		@apply border-gray-500;
	}
</style>

<main class="flex h-svh flex-col items-center justify-evenly gap-10 bg-black p-5 text-white">
	<header class="flex gap-2">
		<select bind:value={$theme} class="border bg-black px-1">
			{#each themes as theme, i}
				<option value={i}>theme {i}</option>
			{/each}
		</select>

		<input class="w-10 border bg-black px-1" type="number" bind:value={$columns} min="2" max="8" />
		x
		<input class="w-10 border bg-black px-1" type="number" bind:value={$rows} min="1" max="2" />
		x
		<input class="w-10 border bg-black px-1" type="number" bind:value={$height} min="4" max="10" />

		<button class="col-span-2 cursor-pointer border px-2" on:click={reset}>reset</button>
	</header>

	<section
		class="grid w-full flex-grow place-content-center gap-3"
		style="grid-template-columns: repeat({$columns}, auto);"
	>
		{#each { length: $columns * $rows } as _, i}
			<div class="flex flex-col">
				{#each { length: $height } as _, y}
					{@const index = i * 100 + y}
					{@const color = $map[index] ?? 0}
					{@const current = color === $selectedColor && $selectedColor != 0}
					<button
						class="grid place-content-center border-x first-of-type:border-t last-of-type:rounded-b-full last-of-type:border-b"
						style="background-color: {colors[
							color
						]}; width: {blockSize}cqmin; height: {blockSize}cqmin; z-index: {current ? 100 : 1};"
						class:outline={current}
						on:wheel|passive={(e) => scroll(index, e)}
						on:click={() => click(index)}
					>
						{format(color)}
					</button>
				{/each}
			</div>
		{/each}
	</section>

	<footer class="grid grid-cols-7 gap-2">
		{#each colors as color, i}
			<button
				class="grid h-[10cqmin] w-[10cqmin] place-content-center first-of-type:row-span-2 first-of-type:h-auto"
				style="background-color: {color};"
				class:outline={$selectedColor === i}
				on:click={() => selectedColor.set(i)}
			>
				{format(i)}
			</button>
		{/each}
	</footer>
</main>
