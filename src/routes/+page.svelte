<script lang="ts">
	import type { IMachine } from 'src/interfaces/IMachine';
	import { onMount } from 'svelte';
	import Machine from '../components/machine.svelte';

	let cooldownTimer = 6;
	let machines: Array<IMachine> = [];
	let isEditable = false;
	let isDeleteable = false;

	function clickHandler(event: MouseEvent) {
		if (!isEditable) return;

		machines.push({
			position: {
				x: event.offsetX,
				y: event.offsetY
			},
			startTime: Date.now() / 1000,
			colourNumber: 1,
			hasBeenClicked: false,
			timeDiff: 0
		});

		machines = machines;
	}

	function machineClickHandler(event: any, i: number) {
		if (isDeleteable) {
			machines.splice(i, 1);
		}

		if (isEditable) {
			return;
		}

		machines = machines;
	}

	onMount(() => {
		const machinesString = localStorage.getItem('machines');
		if (machinesString) {
			machines = JSON.parse(machinesString);
		}

		setInterval(() => {
			localStorage.setItem('machines', JSON.stringify(machines));
		}, 5 * 1000);
	});
</script>

<div id="state">
	<button
		class={`function ${isEditable ? 'on' : ''}`}
		on:click={() => {
			isEditable = !isEditable;
			isDeleteable = false;
		}}
	>
		&#9881;
	</button>
	<button
		class={`function ${isDeleteable ? 'on' : ''}`}
		on:click={() => {
			isDeleteable = !isDeleteable;
			isEditable = false;
		}}
	>
		&#128465;
	</button>
	<input type="number" bind:value={cooldownTimer} class={`function`} />
</div>

<div id="floor-plan" on:click={clickHandler}>
	{#each machines as machine, i}
		<Machine
			{machine}
			{cooldownTimer}
			{isEditable}
			{isDeleteable}
			clickHandler={(e) => {
				machineClickHandler(e, i);
			}}
		/>
		<!-- on:dragstart={dragStart}
    on:dragend={(e) => dragEnd(e, machine)} -->
	{/each}
</div>

<style lang="scss">
	#floor-plan {
		height: 100vh;
		width: 100vw;
		background: #ddd;
		display: block;
		position: relative;
	}

	#state {
		position: absolute;
		display: block;
		height: 2em;
		width: 100vw;
		z-index: 10;
	}

	.function {
		font-size: 2em;
		width: 2.5em;
		background: white;
		margin: 0.1em;
		&.on {
			background: rgb(104, 202, 104);
		}
	}

	input.function {
		width: 1.75em;
		text-align: center;
	}
</style>
