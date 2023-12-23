<script lang="ts">
	import { persisted } from 'svelte-persisted-store'

	const colors = [
		"#000000", // unknown
		"#CD6F7E", // salmon
		"#A55EBB", // pink
		"#69E3AB", // lime
		"#0B25C3", // dark blue
		"#3E511C", // olive
		"#68A1DF", // light blue
		"#707070", // gray
		"#A6433F", // red
		"#613289", // purple
		"#CD8646", // orange
		"#76A835", // green
		"#FBE862", // yellow
	]

	let columns = persisted("columns", 7)
	let rows = persisted("rows", 2)
	let height = persisted("height", 4)
	let selectedColor = persisted("color", 0)
	let map = persisted("map", new Array<number>())

	$: blockWidth = Math.floor(100 / $columns)
	$: blockHeight = Math.floor(100 / ($height * $rows))
	$: blockSize = Math.min(blockWidth, blockHeight)

	function scroll(index: number, e: WheelEvent) {
		map.update(m => {
			const offset = Math.max(-1, Math.min(e.deltaY, 1))
			m[index] = ((m[index] ?? 0) + offset + colors.length) % colors.length
			return m
		})
	}

	function click(index: number) {
		map.update(m => {
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

<style>
	section {
		container-type: size;
	}
</style>

<main class="flex flex-col items-center justify-evenly gap-10 bg-black p-5 h-svh  text-white border-slate-400">

	<header class="flex gap-2">
		<input class="bg-black border-2 px-1" type="number" bind:value={$columns} min="2" max="8"/>
		x
		<input class="bg-black border-2 px-1" type="number" bind:value={$rows} min="1" max="2"/>
		x
		<input class="bg-black border-2 px-1" type="number" bind:value={$height} min="4" max="10"/>

		<button class="border-2 rounded-full cursor-pointer px-2 col-span-2" on:click={reset}>reset</button>
	</header>

	<section class="grid gap-3 place-content-center flex-grow w-full" style="grid-template-columns: repeat({$columns}, auto);">
		{#each { length: $columns * $rows } as _, i}
			<div class="flex flex-col">
				{#each { length: $height } as _, y}
					{@const index = i * 100 + y}
					<button
						class="grid place-content-center border-x first-of-type:border-t last-of-type:border-b last-of-type:rounded-b-full"
						style="background-color: {colors[$map[index] ?? 0]}; width: {blockSize}cqmin; height: {blockSize}cqmin;"
						on:wheel|passive={e => scroll(index, e)}
						on:click={() => click(index)}>
						{format($map[index])}
					</button>
				{/each}
			</div>
		{/each}
	</section>

	<footer class="grid grid-cols-7 gap-2">
		{#each colors as color, i}
			<button
				class="w-[10cqmin] h-[10cqmin] grid place-content-center first-of-type:h-auto first-of-type:row-span-2"
				style="background-color: {color};"
				class:outline={$selectedColor === i}
				on:click={() => selectedColor.set(i)}>
				{format(i)}
			</button>
		{/each}
	</footer>

</main>
