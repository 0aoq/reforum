<script lang="ts">
	import Footer from "$lib/components/Footer.svelte";

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
		if (!pb.authStore.model) return;

		// fetch topics
		const topics = await pb.collection("topics").getList(1, 50, {
			sort: "-created"
		});

		allTopics = topics.items;
	});
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
		<div class="file-browser">
			{#each allTopics as topic}
				<div class="listing">
					<a href="/{host}/t/{topic.id}">{topic.name.replaceAll("-", " ")}</a>
				</div>
			{:else}
				<p>No topics...</p>
			{/each}
		</div>
	</section>

	<Footer {host} />
</main>
