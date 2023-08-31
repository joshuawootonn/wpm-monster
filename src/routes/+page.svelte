<script lang="ts">
	let myInput: HTMLInputElement;

	type Press = {
		data: string | null;
		inputType: 'insertText' | 'deleteContentBackward';
	};

	let keystrokes: Press[] = [];

	$: position = keystrokes.reduce(
		(acc, keystroke) => {
			if (keystroke.inputType === 'insertText' && keystroke.data === ' ') {
				return [...acc, []];
			} else if (keystroke.inputType === 'insertText' && keystroke.data) {
				return [...acc.slice(0, -1), (acc.at(-1) ?? []).concat(keystroke.data)];
			} else if (keystroke.inputType === 'deleteContentBackward') {
				if (acc.at(-1)?.length === 0) {
					return acc.slice(0, -1);
				}

				return [...acc.slice(0, -1), (acc.at(-1) ?? []).slice(0, -1)];
			}
			return [...acc];
		},
		[[]] as string[][]
	);

	function handleInput(e: any) {
		if (e.inputType === 'insertText' || e.inputType === 'deleteContentBackward')
			keystrokes = keystrokes.concat({ inputType: e.inputType, data: e.data });
	}

	function isEqual(a: string[], b: string[]) {
		return a.length === b.length && a.every((A, i) => A === b.at(i));
	}

	let words: string[][] = `Lorem ipsum dolor sit amet, consectetur adipiscing elit.`
		.split(' ')
		.map((word) => word.split(''));
</script>

<div class="w-[700px] mx-auto flex flex-col items-start justify-center flex-grow">
	<input
		type="text"
		class="peer opacity-0"
		bind:this={myInput}
		on:input={handleInput}
		tabindex="0"
		id="myInput"
	/>
	{position}
	<!-- svelte-ignore a11y-no-static-element-interactions -->
	<div
		class="flex flex-wrap select-none border-transparent peer-focus:border-slate-300 p-4 border-2 text-xl"
		on:click={(e) => myInput.focus()}
		on:keydown={(e) => myInput.focus()}
	>
		{#each words as word, i}
			{@const typedWord = position.at(i)}
			{@const extras = typedWord?.slice(word.length)}
			{@const isPreviouslyTyped = position.at(i + 1)}
			<div class:word class:error={typedWord && isPreviouslyTyped && !isEqual(word, typedWord)}>
				{#if extras?.length && typedWord}
					{#each typedWord as letter, j}
						{@const correctLetter = word?.at(j)}
						<span
							class:correct={letter === correctLetter}
							class:incorrect={correctLetter && letter !== correctLetter}
							class:extra={j > word.length - 1}>{correctLetter ?? letter}</span
						>
					{/each}
				{:else}
					{#each word as letter, j}
						{@const typedLetter = typedWord?.at(j)}
						<span
							class:correct={letter === typedLetter}
							class:incorrect={typedLetter && letter !== typedLetter}>{letter}</span
						>
					{/each}
				{/if}
			</div>
		{/each}
	</div>
</div>

<style>
	.correct {
		@apply text-green-500;
	}

	.incorrect {
		@apply text-red-500;
	}

	.extra {
		@apply text-red-800;
	}

	.word {
		@apply block m-1;
	}
	.error {
		@apply underline decoration-red-500;
	}
</style>
