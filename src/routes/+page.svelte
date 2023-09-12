<script lang="ts">
	import { LoremIpsum } from 'lorem-ipsum';
	import { onMount } from 'svelte';
	import { spring } from 'svelte/motion';

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

	let animatedCursorRect: any = spring(
		{ top: 0, left: 0, width: 0, height: 0 },
		{
			stiffness: 0.1,
			damping: 0.5
		}
	);

	onMount(() => {
		function move() {
			const arena = document.querySelector('.arena');
			const arenaRect = arena?.getBoundingClientRect();
			if (arenaRect == null) return;

			const activeLetterRect = document
				.querySelector('.active span:not(.correct):not(.incorrect):not(.extra)')
				?.getBoundingClientRect();

			if (activeLetterRect) {
				animatedCursorRect.set(
					{
						...activeLetterRect,
						top: activeLetterRect.top - arenaRect.top,
						left: activeLetterRect.left - arenaRect.left,
						width: activeLetterRect.width,
						height: activeLetterRect.height
					}
					//{ soft: true }
				);
				return;
			}
			const activeSpaceRect = document.querySelector('.active ~ .space')?.getBoundingClientRect();

			//console.log(activeSpaceRect);
			if (activeSpaceRect) {
				animatedCursorRect.set(
					{
						...activeSpaceRect,
						top: activeSpaceRect.top - arenaRect.top,
						left: activeSpaceRect.left - arenaRect.left,
						width: activeSpaceRect.width,
						height: '100%'
					}
					//{ soft: true }
				);
				return;
			}
		}

		let frame = requestAnimationFrame(function loop(t) {
			frame = requestAnimationFrame(loop);
			move();
		});

		return () => {
			cancelAnimationFrame(frame);
		};
	});

	const lorem = new LoremIpsum({
		sentencesPerParagraph: {
			max: 8,
			min: 4
		},
		wordsPerSentence: {
			max: 16,
			min: 4
		}
	});

	let words: string[][] = lorem
		.generateSentences(2)
		.split(' ')
		.map((word) => word.split(''));
</script>

<div class="w-[800px] mx-auto flex flex-col items-start justify-center">
	<div class="arena z-0 relative mb-24">
		<input
			type="text"
			class="peer opacity-0"
			bind:this={myInput}
			on:input={handleInput}
			tabindex="0"
			id="myInput"
		/>
		<!-- svelte-ignore a11y-no-static-element-interactions -->
		<div
			class=" gap-0 flex justify-start items-start flex-wrap select-none border-transparent border-black p-4 border-2 text-xl"
			on:click={(e) => myInput.focus()}
			on:keydown={(e) => myInput.focus()}
		>
			{#each words as word, i}
				{@const typedWord = position.at(i)}
				{@const extras = typedWord?.slice(word.length)}
				{@const isPreviouslyTyped = position.at(i + 1)}
				<div
					class:word
					class:active={typedWord && isPreviouslyTyped == null}
					class:error={typedWord && isPreviouslyTyped && !isEqual(word, typedWord)}
				>
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
				<span class="space my-1.5 w-2" />
			{/each}
		</div>

		<div
			class="absolute rounded-sm bg-black/10 peer-focus:bg-black/20"
			style:top={`${$animatedCursorRect?.top}px`}
			style:color="limegreen"
			style:left={`${$animatedCursorRect?.left}px`}
			style:width={`${$animatedCursorRect?.width}px`}
			style:height={`${$animatedCursorRect?.height ?? 2}px`}
		/>
		<button
			class="border-2 border-black -translate-y-0.5 px-3 py-1"
			on:click={(e) => {
				keystrokes = [];
				words = lorem
					.generateSentences(2)

					.split(' ')
					.map((word) => word.split(''));
			}}>Restart</button
		>
	</div>
</div>

<style>
	.correct {
		@apply text-emerald-500;
	}

	.incorrect {
		@apply text-rose-700;
	}

	.extra {
		@apply text-rose-700;
	}

	.word {
		@apply block my-1;
	}
	.error {
		@apply underline decoration-rose-500;
	}
</style>
