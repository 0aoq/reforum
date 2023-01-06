<script lang="ts">
	import Footer from "$lib/components/Footer.svelte";
	import config from "$lib/config";

	import PocketBase, { Record } from "pocketbase";
	import type { PageData } from "./$types";
	import { onMount } from "svelte";

	export let data: PageData;
	// @ts-ignore
	export const { host } = data;

	// create pocketbase client
	let pb = new PocketBase(`http://${host}`);
	let allTopics: Array<Record> = [];

	onMount(async () => {
		if (window.location.protocol === "https:") pb = new PocketBase(`https://${host}`);
		// if (!pb.authStore.model) return;

		// fetch topics
		const topics = await pb.collection("topics").getList(1, 50, {
			sort: "-created"
		});

		allTopics = topics.items;
	});

	// functions

	/**
	 * @function createNewTopic
	 * @param event
	 */
	async function createNewTopic(event: any) {
		if (!pb.authStore.model) return;

		try {
			const newTopic = await pb.collection("topics").create({
				name: event.target.topic_name_reforum.value.trim().replaceAll(" ", "_"),
				creator: pb.authStore.model.id
			});

			window.location.href = `/${host}/t/${newTopic.id}`;
			doCreateNewTopic = false;
		} catch (err) {
			alert(
				"Failed to create topic! Make sure the title passes the check below:" +
					`\n\nregex check: [^\\w+$], ${
						event.target.topic_name_reforum.value.match(/^\w+$/) === null
							? "FAILED - Try to use special characters!"
							: "PASSED"
					}`
			);

			console.log("[REPORT]", err);
		}
	}

	let doCreateNewTopic = false;
</script>

<svelte:head>
	<title>Topics - Reforum</title>
</svelte:head>

<main>
	<section class="mb-2 flex justify-center align-center" style="gap: var(--u-2);">
		{#if !pb.authStore.model}
			<a href="/{host}/auth" class="chip">Account</a>
		{:else}
			<a href="/{host}/u/{pb.authStore.model.username}" class="chip">Account</a>
		{/if}
	</section>

	<section>
		{#if !doCreateNewTopic}
			<!-- main view -->
			{#if pb.authStore.model && config.userPermissions.canCreateTopics}
				<div class="flex mb-4" style="width: 100%; place-content: flex-end;">
					<!-- toggle create -->
					<button
						on:click={() => {
							doCreateNewTopic = true;
						}}>Create New</button
					>
				</div>
			{/if}

			<div class="file-browser">
				{#each allTopics as topic}
					<div class="listing">
						<a href="/{host}/t/{topic.id}">{topic.name.replaceAll("_", " ")}</a>
					</div>
				{:else}
					<p>No topics...</p>
				{/each}
			</div>
		{:else}
			<!-- create new topic -->
			<form
				on:submit|preventDefault={createNewTopic}
				class="flex"
				style="gap: var(--u-2); flex-direction: column;"
			>
				<p class="form-label">Topic Name</p>
				<input type="text" name="topic_name_reforum" placeholder="Topic Name" />

				<button class="mt-4">Create Topic</button>
			</form>
		{/if}
	</section>

	<Footer {host} />
</main>

<style>
	button {
		width: max-content;
	}

	.form-label {
		user-select: none;
	}
</style>
