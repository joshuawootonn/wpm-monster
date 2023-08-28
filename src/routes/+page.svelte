<script lang="ts">
	import clsx from 'clsx';
	console.log('hit');
	let myInput: HTMLInputElement;

	let value: string = 'Lorem ipsum do';
	$: valueLetters = value?.split('') ?? [];
	let letters: string[] =
		`Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque vitae lobortis arcu. Aliquam a
		pharetra justo. Suspendisse sit amet turpis semper, euismod ante at, eleifend ex. Aenean quis
		lobortis tellus. Etiam pretium tortor et gravida commodo.`.split('');
</script>

<div class="w-[700px] mx-auto flex flex-col items-start justify-center flex-grow">
	<input
		bind:value
		type="text"
		class="peer opacity-0"
		bind:this={myInput}
		tabindex="0"
		id="myInput"
	/>
	<!-- svelte-ignore a11y-no-static-element-interactions -->
	<div
		class="select-none border-transparent peer-focus:border-slate-300 p-4 border-2 text-xl"
		on:click={(e) => myInput.focus()}
		on:keydown={(e) => myInput.focus()}
	>
		{#each letters as letter, i}
			<span
				class={clsx({
					'text-cyan-500': letter === valueLetters[i],
					'text-red-500': letter !== valueLetters[i] && valueLetters[i]
				})}>{letter}</span
			>
		{/each}
	</div>
	<!-- <div>{value ?? ''}</div> -->
</div>
