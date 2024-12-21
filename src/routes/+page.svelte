<script lang="ts">
	import Pitch from '../components/pitch.svelte';
	type pitch = {
		topic: string;
		name: string;
		mode: ('online' | 'local')[];
		editable?: boolean;
		hideEdit?: boolean;
		id: string;
	};
	let pitches: pitch[] = $state([]);

	// hack night # (for data input)
	let hackNightNum: number | undefined = $state(undefined);
	let hackNightNumEditable = $state(true);

	// common breakouts setup
	let speakerName = $state('');
	let speaker = $state(false);
	let speakerDetails: pitch[] = $derived(
		speaker
			? [
					{
						topic: 'Speaker breakout session',
						name: speakerName ?? 'speaker',
						mode: ['online', 'local'],
						hideEdit: true,
						id: crypto.randomUUID()
					}
				]
			: []
	);
	let intro = $state(false);
	let introName = $state('');
	let introDetails: pitch[] = $derived(
		intro
			? [
					{
						topic: 'Civic Tech 101',
						name: introName,
						mode: ['online', 'local'],
						hideEdit: true,
						id: crypto.randomUUID()
					}
				]
			: []
	);
	let grid = $state(false);

	let inputMode: pitch['mode'] = $state([]);
	let hackNightList: pitch[] = $derived.by(() => {
		const base: pitch[] = [];

		return base.concat(speakerDetails).concat(introDetails).concat(pitches);
	});
	let breakoutRoomList = $derived.by(() => {
		// speaker / civic tech 101 can take main zoom room
		let roomNumber = speaker || intro ? 0 : 1;
		const breakoutRoomArray: number[] = [];
		for (let index = 0; index < hackNightList.length; index++) {
			if (hackNightList[index].mode?.includes('online')) {
				breakoutRoomArray.push(roomNumber);
				roomNumber++;
			} else {
				breakoutRoomArray.push(-1);
			}
		}
		return breakoutRoomArray;
	});

	const modeParser = (mode: pitch['mode'], position: number) => {
		const resultArr: string[] = [];

		if (mode.includes('local')) {
			resultArr.push('in-person');
		}
		if (mode.includes('online')) {
			if (position === -1) {
				const nextRoom = breakoutRoomList.length > 0 ? Math.max(...breakoutRoomList) + 1 : 1;
				resultArr.push(`Zoom room ${nextRoom}`);
			} else if (breakoutRoomList[position] === 0) {
				resultArr.push('Main zoom room');
			} else resultArr.push(`Zoom room ${breakoutRoomList[position]}`);
		}
		return resultArr.join(' & ');
	};

	// new pitches form
	let inputTopic = $state('');
	let inputPerson = $state('');
	const addPitch = (event: SubmitEvent & { currentTarget: EventTarget & HTMLFormElement }) => {
		event.preventDefault();

		pitches.push({
			topic: inputTopic,
			name: inputPerson,
			mode: inputMode,
			id: crypto.randomUUID()
		});

		inputTopic = '';
		inputPerson = '';
		inputMode = [];
	};

	const editForm = (pitchInput: pitch) => {
		const pitchIndex = pitches.findIndex((pitch) => pitch.id === pitchInput.id);
		pitches[pitchIndex] = pitchInput;
	};
</script>

<div class="wrapper">
	<div class="content" role="main">
		<h1 class="title">Tonight's breakouts</h1>
		<form>
			<h3>
				Hack Night #{#if hackNightNumEditable}<input type="number" bind:value={hackNightNum} />
				{:else}
					{hackNightNum}
				{/if}
				<button
					onclick={(event) => {
						event.preventDefault();

						hackNightNumEditable = !hackNightNumEditable;
					}}
				>
					{hackNightNumEditable ? 'Save' : 'Edit'}
				</button>
			</h3>
		</form>
		<details open>
			<summary> Do we have them tonight? </summary>
			<form>
				<div id="speaker-breakout">
					<input type="checkbox" id="speaker" name="mode" bind:checked={speaker} />
					<label for="speaker">Speaker breakout session</label>
					<input
						type="text"
						id="speaker-name"
						placeholder="speaker name"
						bind:value={speakerName}
					/>
				</div>
				<div id="civic-tech-101">
					<input type="checkbox" id="intro" name="mode" bind:checked={intro} />
					<label for="intro">Civic Tech 101</label>
					<input type="text" id="speaker-name" placeholder="speaker name" bind:value={introName} />
				</div>
				<input type="checkbox" id="grid" name="mode" bind:checked={grid} />
				<label for="grid">Use grid layout</label>
			</form>
		</details>
		<div class="pitches">
			{#if grid}
				<table>
					<thead>
						<tr>
							<th scope="col">Topic</th>
							<th scope="col">Person</th>
							<th scope="col">Location</th>
						</tr>
					</thead>
					<tbody>
						{#each hackNightList as pitch, index (pitch.id)}
							<tr>
								<th scope="row">{pitch.topic}</th>
								<td>{pitch.name}</td>
								<td>{modeParser(pitch.mode, index)}</td>
							</tr>
						{/each}
					</tbody>
				</table>
			{:else}
				{#each hackNightList as pitch, index (pitch.id)}
					<Pitch {pitch} {index} {breakoutRoomList} {editForm} />
				{/each}
			{/if}
		</div>
		<form id="new-pitch" onsubmit={addPitch}>
			<h2>New pitch</h2>
			<div class="simulate-row">
				<input
					autocomplete="on"
					type="text"
					id="topic"
					name="topic"
					placeholder="topic"
					bind:value={inputTopic}
					required
				/>
				<input
					autocomplete="on"
					type="text"
					id="person"
					name="person"
					placeholder="person"
					bind:value={inputPerson}
					required
					autocapitalize="on"
				/>
				<div id="simulate-mode">
					{modeParser(inputMode, -1)}
				</div>
			</div>
			<fieldset>
				<legend>Online / in-person?</legend>

				<div>
					<input type="checkbox" id="online" name="mode" value="online" bind:group={inputMode} />
					<label for="online">Online</label>
				</div>

				<div>
					<input type="checkbox" id="local" name="mode" value="local" bind:group={inputMode} />
					<label for="local">In-person</label>
				</div>
			</fieldset>
			<button>Add</button>
		</form>
	</div>
</div>
<footer class="footer"></footer>

<style>
	:global(body) {
		font-family: 'HK Grotesk', sans-serif;
	}
	table,
	form#new-pitch {
		width: 90vw;
		line-height: 1.5;
	}
	tr {
		border-bottom: 1px solid black;
	}
	tr,
	.simulate-row {
		display: grid;
		grid-template-columns: 4fr 1fr 15rem;
		text-align: left;
	}
	input {
		font-size: 20px;
	}
	details,
	form#new-pitch {
		padding: 0.5lh 0;
	}
</style>
