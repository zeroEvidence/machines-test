<script lang="ts">
	import { interpolateRdYlGn } from 'd3-scale-chromatic';
	import type { IMachine } from 'src/interfaces/IMachine';
	import { onDestroy, onMount } from 'svelte';

	export let machine: IMachine;
	export let clickHandler: (event: any) => void;
	export let cooldownTimer: number;
	export let isEditable: Boolean;
	export let isDeleteable: Boolean;

	let timerId: NodeJS.Timer;
	let startTime: number = machine.startTime;
	let colourNumber = machine.colourNumber;
	let now: number = Date.now() / 1000;
	let hasBeenClicked = machine.hasBeenClicked;
	let timeDiff: number = machine.timeDiff;

	function dragStart(event: any) {
		if (!isEditable) return;
		event.dataTransfer.effectAllowed = 'move';
		event.dataTransfer.dropEffect = 'move';
	}

	function dragEnd(event: any) {
		if (!isEditable) return;
		machine.position.x = event.x;
		machine.position.y = event.y;
	}

	function occupiedHander() {
		if (isDeleteable) return;
		if (!hasBeenClicked) {
			hasBeenClicked = true;
			machine.hasBeenClicked = hasBeenClicked;
		}

		startTime = Date.now() / 1000;
		machine.startTime = startTime;
		colourNumber = calcColourNumber();
		machine.colourNumber = colourNumber;
	}

	$: ((_) => {
		colourNumber = calcColourNumber(_);
		machine.colourNumber = colourNumber;
	})(cooldownTimer);

	$: ((_now, _timeDiff, _hasBeenClicked) => {
		if (!hasBeenClicked) return;
		timeDiff = now - startTime;
		machine.timeDiff = timeDiff;
	})(now, timeDiff, hasBeenClicked);

	function calcColourNumber(_?: any) {
		if (!hasBeenClicked) return 1;

		now = Date.now() / 1000;
		const cooldownTimerSeconds = cooldownTimer * 60 * 60;
		timeDiff = now - startTime;
		machine.timeDiff = timeDiff;
		return timeDiff / cooldownTimerSeconds;
	}

	onMount(() => {
		timerId = setInterval(() => {
			now = Date.now() / 1000;
			colourNumber = calcColourNumber();
			machine.colourNumber = colourNumber;
		}, 10 * 1000);
	});

	onDestroy(() => {
		clearInterval(timerId);
	});
</script>

<div
	class="machine"
	style={`left: ${machine.position.x - 15}px; top: ${machine.position.y - 15}px;`}
	draggable={true}
	on:click={(e) => {
		clickHandler(e);
		occupiedHander();
	}}
	on:dragstart={dragStart}
	on:dragend={(e) => dragEnd(e)}
>
	<div class="machine-inner" style={`background: ${interpolateRdYlGn(colourNumber)}`} />
	{#if !isNaN(timeDiff) && timeDiff}
		{#if Math.round(timeDiff / 60 / 60) >= 1}
			{Math.round(timeDiff / 60 / 60)}h<br />
		{/if}
		{#if Math.round(timeDiff / 60) >= 1}
			{Math.round(timeDiff % 60)}m
		{/if}
	{/if}
</div>

<style lang="scss">
	.machine {
		width: 30px;
		height: 30px;
		border: 2px solid black;
		position: absolute;
		display: block;

		.machine-inner {
			width: 100%;
			height: 100%;
			position: absolute;
			display: block;
			opacity: 0.7;
		}
	}
</style>
