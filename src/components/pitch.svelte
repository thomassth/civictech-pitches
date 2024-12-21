<script lang="ts">
	import { onMount } from 'svelte';
	import isEqual from 'lodash/isEqual';

	type pitchObj = {
		topic: string;
		name: string;
		mode: ('online' | 'local')[];
		editable?: boolean;
		hideEdit?: boolean;
	};
	let {
		pitch = $bindable(),
		index,
		breakoutRoomList = $bindable(),
		editForm
	}: { pitch: pitchObj; index: number; breakoutRoomList: number[]; editForm: Function } = $props();

	let currentPitch: pitchObj | undefined = $state(undefined);
	onMount(() => {
		currentPitch = { ...pitch };
	});
	const parsedMode = $derived.by(() => {
		const resultArr: string[] = [];

		if (currentPitch?.mode.includes('local')) {
			resultArr.push('in-person');
		}
		if (currentPitch?.mode.includes('online')) {
			if (index === -1) {
				const nextRoom = breakoutRoomList.length > 0 ? Math.max(...breakoutRoomList) + 1 : 1;
				resultArr.push(`Zoom room ${nextRoom}`);
			} else if (breakoutRoomList[index] === 0) {
				resultArr.push('Main zoom room');
			} else resultArr.push(`Zoom room ${breakoutRoomList[index]}`);
		}
		return resultArr.join(' & ');
	});

	let editing = $state(!!pitch.editable);
</script>

{#if currentPitch}
	<form
		onsubmit={(event) => {
			event.preventDefault();
			editing = !editing;
			if (!editing) {
				if (!isEqual(currentPitch, pitch)) editForm(currentPitch);
			}
		}}
	>
		<p>
			{#if editing}
				<input bind:value={currentPitch.topic} name="topic" type="text" />
				<input bind:value={currentPitch.name} name="name" type="text" />
				<span>
					<input
						type="checkbox"
						id="online"
						name="mode"
						value="online"
						bind:group={currentPitch.mode}
					/>
					<label for="online">Online</label>

					<input
						type="checkbox"
						id="local"
						name="mode"
						value="local"
						bind:group={currentPitch.mode}
					/>
					<label for="local">In-person</label>
				</span>
			{:else}
				{`${currentPitch.topic} ${currentPitch.name ? `(${currentPitch.name})` : ''} - ${parsedMode}`}
			{/if}

			{#if !currentPitch.hideEdit}
				<button>
					{editing ? 'Save' : 'Edit'}
				</button>
			{/if}
		</p>
	</form>
{/if}
