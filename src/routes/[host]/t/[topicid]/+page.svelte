<script lang="ts">
	import Footer from "$lib/components/Footer.svelte";

	import PocketBase, { Record } from "pocketbase";
	import type { PageData } from "./$types";
	import { onMount } from "svelte";

	export let data: PageData;
	// @ts-ignore
	export const { host, topicid } = data;

	// create pocketbase client
	let pb = new PocketBase(`http://${host}`);

	let topic: Record;
	let allPosts: Array<Record> = [];

	onMount(async () => {
		if (window.location.protocol === "https:") pb = new PocketBase(`https://${host}`);
		if (!pb.authStore.model) return;

		// fetch topic
		const _topic = await pb.collection("topics").getOne(topicid);
		topic = _topic;

		// fetch posts
		const posts = await pb.collection("posts").getList(1, 50, {
			sort: "-created",
			filter: `topic = "${topicid}"`
		});

		allPosts = posts.items;
	});
</script>

<svelte:head>
	<title>Topic - Reforum</title>
</svelte:head>

<body>
	<nav>
		<div class="title">
			<a
				href="/{host}"
				class="grid place-center"
				aria-label="Viewing Post: {(topic || { name: '' }).name}"
				title="Click to return to topics view"
			>
				{(topic || { name: "" }).name}
			</a>
		</div>

		<div>
			<button
				class="primary"
				on:click={() => {
					window.location.href = `/${host}`;
				}}>View Topics</button
			>
		</div>
	</nav>

	<main>
		<section>
			<h1 style="text-align: center;">{(topic || { name: "" }).name}</h1>

			<div class="file-browser">
				{#each allPosts as post}
					<div class="listing">
						<a href="/{host}/t/{topic.id}/{post.id}">{post.title.replaceAll("-", " ")}</a>
					</div>
				{:else}
					<p>No posts in topic!</p>
				{/each}
			</div>
		</section>

		<Footer {host} />
	</main>
</body>
