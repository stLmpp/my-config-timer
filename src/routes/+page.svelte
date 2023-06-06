<script>
	import { readable } from 'svelte/store';
	import Button from '../lib/components/ui/button/Button.svelte';
	import Label from '../lib/components/ui/label/Label.svelte';
	import Input from '../lib/components/ui/input/Input.svelte';
	import Accordion from '../lib/components/ui/accordion/Accordion.svelte';
	import AccordionItem from '../lib/components/ui/accordion/AccordionItem.svelte';
	import AccordionTrigger from '../lib/components/ui/accordion/AccordionTrigger.svelte';
	import AccordionContent from '../lib/components/ui/accordion/AccordionContent.svelte';
	import { browser } from '$app/environment';

	const time = readable(0, (set) => {
		const beginning = new Date();
		const beginningTime = beginning.getTime();

		const interval = setInterval(() => {
			const current = new Date();
			const currentTime = current.getTime();
			set(currentTime - beginningTime);
		}, 19);

		return () => {
			set(0);
			clearInterval(interval);
		};
	});

	let id = 0;
	let steps = [
		{ id: id++, time: 30, name: 'Exercise' },
		{ id: id++, time: 30, name: 'Rest' }
	];
	let index = 0;

	let lastValue = 0;
	let ms = 0;
	let subscription;

	$: isSubcribed = !!subscription;

	/**
	 *
	 * @param {KeyboardEvent} event
	 */
	function onKeypress(event) {
		if (event.key === ' ') {
			toggle();
		}
	}

	function toggle() {
		if (subscription) {
			subscription?.();
			subscription = null;
			lastValue = ms;
		} else {
			subscription = time.subscribe((value) => {
				ms = lastValue + value;
			});
		}
	}

	let playedSongs = [];

	/**
	 * @param {'beep'|'start'} type
	 */
	function playSound(type) {
		if (!browser) {
			return;
		}
		switch (type) {
			case 'beep': {
				new Audio('beep.mp3').play();
				break;
			}
			case 'start': {
				new Audio('start.mp3').play();
				break;
			}
		}
	}

	let timeRemaining = steps[index].time;

	$: if (steps[index].time && Math.floor(ms / 1000) >= steps[index].time) {
		subscription?.();
		playSound('start');
		playedSongs = [];
		lastValue = 0;
		ms = 0;
		subscription = null;
		index = index + 1 > steps.length - 1 ? 0 : index + 1;
		subscription = time.subscribe((value) => {
			ms = lastValue + value;
		});
	}

	$: {
		const seconds = Math.floor(ms / 1000);
		timeRemaining = steps[index].time - seconds;
		if (steps[index].time && steps[index].time - seconds < 3 && !playedSongs[seconds]) {
			playedSongs[seconds] = true;
			playSound('beep');
		}
	}
</script>

<svelte:window on:keypress={onKeypress} />

<svelte:head>
	<title>My config timer</title>
	<meta name="description" content="Configure your stopwatch with multiple steps" />
</svelte:head>

Press <kbd>Space</kbd> to {!isSubcribed ? 'start' : 'stop'}
<hr />
<Accordion>
	<AccordionItem value="config">
		<AccordionTrigger>Config</AccordionTrigger>
		<AccordionContent>
			{#each steps as step, index}
				<div class="border-b border-gray-200 p-4 odd:bg-slate-100">
					<h3 class="scroll-m-20 text-2xl font-semibold tracking-tight">Step {index}</h3>
					<div class="flex gap-2">
						<div class="grid w-full max-w-sm items-center gap-1.5 mt-3">
							<Label for="name-{index}">Name</Label>
							<Input id="name-{index}" bind:value={step.name} disabled={isSubcribed} />
						</div>
						<div class="grid w-full max-w-sm items-center gap-1.5 mt-3">
							<Label for="time-{index}">Time</Label>
							<Input id="time-{index}" bind:value={step.time} disabled={isSubcribed} />
						</div>
					</div>
					{#if steps.length > 1}
						<div class="mt-2">
							<Button
								type="button"
								variant="destructive"
								on:click={() => {
									subscription?.();
									playedSongs = [];
									lastValue = 0;
									ms = 0;
									subscription = null;
									index = 0;
									steps = steps.filter((_step) => _step.id !== step.id);
								}}
								>Delete
							</Button>
						</div>
					{/if}
				</div>
			{/each}
			<div class="p-4 mt-2">
				<Button
					type="button"
					on:click={() => {
						steps = [...steps, { name: 'new', time: 30, id: id++ }];
					}}>Add step</Button
				>
			</div>
		</AccordionContent>
	</AccordionItem>
</Accordion>

<hr />
<h2
	class="scroll-m-20 border-b pb-2 text-3xl font-semibold tracking-tight transition-colors first:mt-0"
>
	{steps[index].name}
</h2>
<hr />
<div
	class="timer font-bold md:text-9xl justify-center mt-4 mb-4 text-center"
	class:text-red-600={timeRemaining === 1}
	class:text-red-500={timeRemaining === 2}
	class:text-red-400={timeRemaining === 3}
>
	{String(Math.floor(Math.floor(ms / 1000 / 60)) % 60).padStart(2, '0')}:{String(
		Math.floor(Math.floor(ms / 1000) % 60)
	).padStart(2, '0')}:{String(Math.floor(ms % 1000)).padStart(3, '0')}
</div>

<hr />

<div class="p-4">
	<Button
		type="button"
		on:click={() => {
			toggle();
		}}>{isSubcribed ? 'Pause' : 'Start'}</Button
	>
	<Button
		type="button"
		on:click={() => {
			subscription?.();
			playedSongs = [];
			lastValue = 0;
			ms = 0;
			subscription = null;
			index = 0;
		}}>Reset</Button
	>
</div>

<style>
	:root {
		--space: 200px;
		--sep: 50px;
	}

	.timer {
		font-variant-numeric: tabular-nums;
		grid-template-columns:
			minmax(0, var(--space)) minmax(0, max-content) minmax(0, var(--space)) minmax(0, max-content)
			minmax(0, 300px);
	}

	@media (max-width: 767px) {
		.timer {
			font-size: 19.5vw;
		}
	}
</style>
