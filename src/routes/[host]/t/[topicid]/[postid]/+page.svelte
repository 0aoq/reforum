<script lang="ts">
	import Footer from "$lib/components/Footer.svelte";

	import PocketBase, { Record } from "pocketbase";
	import type { PageData } from "./$types";
	import { onMount } from "svelte";
	import Message from "$lib/components/Message.svelte";

	export let data: PageData;
	// @ts-ignore
	export const { host, topicid, postid } = data;

	// create pocketbase client
	let pb = new PocketBase(`http://${host}`);

	let topic: Record;
	let allPosts: Array<Record> = [];
	let post: Record;

	onMount(async () => {
		if (window.location.protocol === "https:") pb = new PocketBase(`https://${host}`);
		// if (!pb.authStore.model) return;

		try {
			// fetch topic
			const _topic = await pb.collection("topics").getOne(topicid);
			topic = _topic;

			// fetch post
			post = await pb.collection("posts").getOne(postid, {
				expand: "sender"
			});

			document.title = `"${post.title}" - Reforum`;

			// fetch replies
			const posts = await pb.collection("replies").getList(1, 50, {
				sort: "created",
				filter: `post = "${postid}"`,
				expand: "sender"
			});

			allPosts = posts.items;
		} catch {
			window.location.href = `/${host}`;
		}
	});

	/**
	 * @function createReply
	 * @param {any} event
	 */
	async function createReply(event: any) {
		if (!pb.authStore.model) return;

		try {
			// create record
			const record = await pb.collection("replies").create({
				content: document.getElementById("reply-content")!.innerText.trim(),
				post: postid,
				sender: pb.authStore.model.id
			});

			record.expand.sender = pb.authStore.model as any; // <- technically, this works

			// render to page
			allPosts = [...allPosts, record];

			// clear reply-content box
			document.getElementById("reply-content")!.innerHTML = "";
		} catch {
			alert("Failed to create reply to thread!");
		}
	}
</script>

<svelte:head>
	<title>Post - Reforum</title>
</svelte:head>

<body>
	<nav>
		<div class="title">
			<a
				href="/{host}/t/{topicid}"
				class="grid place-center"
				aria-label="Viewing Post: {(post || { title: '' }).title}"
				title="Click to return to topic"
			>
				{(post || { title: "" }).title}
			</a>
		</div>

		<div>
			<button
				class="primary"
				on:click={() => {
					window.location.href = `/${host}/t/${topicid}`;
				}}>View Topic</button
			>
		</div>
	</nav>

	<main>
		<section>
			<h1 style="text-align: center;">{(post || { title: "" }).title}</h1>

			<!-- main post -->
			{#if post}
				<Message
					record={post}
					{host}
					{pb}
					ownerid={post.sender}
					{topic}
					thread={post}
					type="THREAD"
				/>
			{/if}

			<!-- replies -->
			<section class="mt-2 flex" style="flex-direction: column; gap: var(--u-2);">
				<h2>Replies</h2>
				{#each allPosts as $post}
					<Message
						record={$post}
						{host}
						{pb}
						ownerid={$post.sender}
						{topic}
						thread={post}
						type="REPLY"
					/>
				{:else}
					<p>No replies...</p>
				{/each}
			</section>

			<!-- new reply -->
			{#if pb.authStore.model}
				<section id="compose" class="mt-8">
					<h2>Reply</h2>

					<form
						class="flex"
						style="justify-content: flex-end;"
						on:submit|preventDefault={createReply}
					>
						<p
							contenteditable="true"
							style="border: solid 1px var(--bg-surface-lowest);"
							id="reply-content"
						/>

						<button class="primary mt-4">Send Reply</button>
					</form>
				</section>
			{/if}
		</section>

		<Footer {host} />
	</main>
</body>

<style>
	button {
		width: max-content;
	}
</style>
