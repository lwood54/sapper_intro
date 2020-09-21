<script context="module">
	// export const preload = (page) => {
	export function preload(page) {
		return this.fetch("https://svelte-meetup-c9828.firebaseio.com/meetups.json")
			.then((res) => {
				if (!res.ok) {
					throw new Error("Fetching meetups failed, try again later.");
				}
				return res.json();
			})
			.then((data) => {
				const loadedMeetups = [];
				for (const key in data) {
					loadedMeetups.push({
						...data[key],
						id: key,
					});
				}
				return { fetchedMeetups: loadedMeetups.reverse() };
				// set a timeout just to observe loading spinner
				// setTimeout(() => {
				// 	isLoading = false;
				// 	meetups.setMeetups(loadedMeetups.reverse()); // use custom functions from custom store
				// }, 1000);
			})
			.catch((err) => {
				error = err;
				isLoading = false;
				console.log(err);
				this.error(500, "Could not fetch meetups.");
			});
	}
</script>

<script>
	import { createEventDispatcher, onMount, onDestroy } from "svelte";
	import MeetupItem from "../components/Meetup/MeetupItem.svelte";
	import MeetupFilter from "../components/Meetup/MeetupFilter.svelte";
	import Button from "../components/UI/Button.svelte";
	import { scale } from "svelte/transition";
	import { flip } from "svelte/animate";
	import EditMeetup from "../components/Meetup/EditMeetup.svelte";
	import LoadingSpinner from "../components/UI/LoadingSpinner.svelte";
	import meetups from "../meetups-store";

	export let fetchedMeetups; // needs to be same variable name as was returned in the preload function

	let loadedMeetups = [];
	let editMode;
	let editedId;
	let isLoading;
	let unsubscribe;

	const dispatch = createEventDispatcher();

	let favsOnly = false;

	$: filteredMeetups = favsOnly ? loadedMeetups.filter((m) => m.isFavorite) : loadedMeetups;

	onMount(() => {
		unsubscribe = meetups.subscribe((items) => {
			loadedMeetups = items;
		});
		meetups.setMeetups(fetchedMeetups);
	});

	onDestroy(() => {
		if (unsubscribe) {
			unsubscribe();
		}
	});

	const setFilter = (e) => {
		favsOnly = e.detail === 1;
	};

	const savedMeetup = (e) => {
		editMode = null;
		editedId = null;
	};

	const editMeetup = () => {
		editMode = "edit";
	};

	const cancelEdit = () => {
		editMode = null;
		editedId = null;
	};

	const startEdit = (e) => {
		editMode = "edit";
		editedId = e.detail;
	};

	const startAdd = () => {
		editMode = "edit";
	};

	const cancelError = () => {
		error = null;
	};
</script>

<style>
	#meetups {
		width: 100%;
		display: grid;
		grid-template-columns: 1fr;
		grid-gap: 1rem;
	}

	#meetup-controls {
		margin: 1rem;
		display: flex;
		justify-content: space-between;
	}

	@media (min-width: 768px) {
		#meetups {
			grid-template-columns: repeat(2, 1fr);
		}
	}

	#no-meetups {
		margin: 1rem;
		display: flex;
		justify-content: space-between;
	}
</style>

<svelte:head>
	<title>Sapper project template</title>
</svelte:head>

{#if editMode === 'edit'}
	<EditMeetup id={editedId} on:save={savedMeetup} on:cancel={cancelEdit} />
{/if}
{#if isLoading}
	<LoadingSpinner />
{:else}
	<!-- <MeetupGrid meetups={$meetups} on:showdetails={showDetails} on:edit={startEdit} on:add={editMeetup} /> -->
	<section id="meetup-controls">
		<MeetupFilter on:select={setFilter} />
		<Button on:click={startAdd}>New Meetup</Button>
	</section>
	{#if filteredMeetups.length === 0}
		<p id="no-meetups">No meetups found, you can start adding some.</p>
	{/if}
	<section id="meetups">
		{#each filteredMeetups as meetup (meetup.id)}
			<div transition:scale animate:flip={{ duration: 300 }}>
				<MeetupItem
					id={meetup.id}
					title={meetup.title}
					subtitle={meetup.subtitle}
					description={meetup.description}
					imageUrl={meetup.imageUrl}
					address={meetup.address}
					isFav={meetup.isFavorite}
					email={meetup.contactEmail}
					on:edit={startEdit} />
			</div>
		{/each}
	</section>
{/if}
