<script>
	import { createEventDispatcher } from "svelte";
	import MeetupItem from "./MeetupItem.svelte";
	import MeetupFilter from "./MeetupFilter.svelte";
	import Button from "../UI/Button.svelte";
	import { scale } from "svelte/transition";
	import { flip } from "svelte/animate";

	export let meetups;

	const dispatch = createEventDispatcher();

	let favsOnly = false;

	$: filteredMeetups = favsOnly ? meetups.filter((m) => m.isFavorite) : meetups;

	const setFilter = (e) => {
		favsOnly = e.detail === 1;
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

{#if editMode === 'edit'}
	<EditMeetup id={editedId} on:save={savedMeetup} on:cancel={cancelEdit} />
{/if}
{#if isLoading}
	<LoadingSpinner />
{:else}
	<MeetupGrid meetups={$meetups} on:showdetails={showDetails} on:edit={startEdit} on:add={editMeetup} />
{/if}

<section id="meetup-controls">
	<MeetupFilter on:select={setFilter} />
	<Button on:click={() => dispatch('add')}>New Meetup</Button>
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
				on:showdetails
				on:edit />
		</div>
	{/each}
</section>
