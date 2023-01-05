<script lang="ts">
	import Footer from "$lib/components/Footer.svelte";

	import PocketBase, { Record } from "pocketbase";
	import type { PageData } from "./$types";
	import { onMount } from "svelte";
	import { error } from "@sveltejs/kit";

	export let data: PageData;
	// @ts-ignore
	export const { host, topicid } = data;

	// create pocketbase client
	let pb = new PocketBase(`http://${host}`);

	let topic: Record;
	let allPosts: Array<Record> = [];

	onMount(async () => {
		if (window.location.protocol === "https:") pb = new PocketBase(`https://${host}`);
		// if (!pb.authStore.model) return;

		// fetch topic
		const _topic = await pb.collection("topics").getOne(topicid);
		topic = _topic;

		document.title = `"${_topic.name}" posts on Reforum`;

		// fetch posts
		const posts = await pb.collection("posts").getList(1, 50, {
			sort: "-created",
			filter: `topic = "${topicid}"`
		});

		allPosts = posts.items;
	});

	// functions

	/**
	 * @function createNewPost
	 * @param {any} event
	 */
	async function createNewPost(event: any) {
		if (!pb.authStore.model) return;

		try {
			// create post
			const record = await pb.collection("posts").create({
				title: event.target.post_name_reforum.value.trim().replaceAll(" ", "_"),
				content: document.getElementById("thread-content")!.innerText.trim(),
				sender: pb.authStore.model.id,
				topic: topicid
			});

			// reload
			window.location.href = `/${host}/t/${topicid}/${record.id}`;
		} catch (err) {
			alert(
				"Failed to create post! Make sure the title passes the check below:" +
					`\n\nregex check: [^\\w+$], ${
						event.target.name.value.match(/^\w+$/) === null
							? "FAILED - Try to use special characters!"
							: "PASSED"
					}`
			);

			console.log("[REPORT]", err);
		}
	}

	let doCreateNewPost = false;
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
				aria-label="Viewing Topic: {(topic || { name: '' }).name.replaceAll('_', ' ')}"
				title="Click to return to topics view"
			>
				{(topic || { name: "" }).name.replaceAll("_", " ")}
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
			{#if !doCreateNewPost}
				<!-- main view -->
				<h1 style="text-align: center;">{(topic || { name: "" }).name.replaceAll("_", " ")}</h1>

				{#if pb.authStore.model}
					<div class="flex mb-4" style="width: 100%; place-content: flex-end;">
						<!-- toggle create -->
						<button
							on:click={() => {
								doCreateNewPost = true;
							}}>Create New</button
						>
					</div>
				{/if}

				<div class="file-browser">
					{#each allPosts as post}
						<div class="listing">
							<a href="/{host}/t/{topic.id}/{post.id}">{post.title.replaceAll("_", " ")}</a>
						</div>
					{:else}
						<p>No posts in topic!</p>
					{/each}
				</div>
			{:else}
				<!-- create post form -->
				<h1 style="text-align: center;">Create Post</h1>

				<form
					on:submit|preventDefault={createNewPost}
					class="flex"
					style="gap: var(--u-2); flex-direction: column;"
				>
					<p class="form-label">Post Name</p>
					<input type="text" placeholder="Post Name" name="post_name_reforum" />

					<p class="form-label">Post Content</p>
					<p
						id="thread-content"
						contenteditable="true"
						style="border: solid 1px var(--bg-surface-lowest);"
					/>

					<button class="primary mt-4">Create Post</button>
				</form>
			{/if}
		</section>

		<Footer {host} />
	</main>
</body>

<style>
	button {
		width: max-content;
	}

	.form-label {
		user-select: none;
	}
</style>
