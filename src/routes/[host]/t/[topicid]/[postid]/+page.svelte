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
		if (!pb.authStore.model) return;

		// fetch topic
		const _topic = await pb.collection("topics").getOne(topicid);
		topic = _topic;

		// fetch post
		post = await pb.collection("posts").getOne(postid, {
			expand: "sender"
		});

		// fetch replies
		const posts = await pb.collection("replies").getList(1, 50, {
			sort: "-created",
			filter: `post = "${postid}"`,
			expand: "sender"
		});

		allPosts = posts.items;
	});
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
                aria-label="Viewing Post: {(post || { title: "" }).title}"
                title="Click to return to topic"
            >
                {(post || { title: "" }).title}
            </a>
        </div>

        <div>
            <button class="primary" on:click={() => {
                window.location.href = `/${host}/t/${topicid}`;
            }}>View Topic</button>
        </div>
    </nav>
    
    <main>
        <section>
            <h1 style="text-align: center;">{(post || { title: "" }).title}</h1>
    
            <!-- main post -->
            {#if post}
                <Message record={post} {host} />
            {/if}
    
            <!-- replies -->
            <section class="mt-2">
                {#each allPosts as post}
                    <Message record={post} {host} />
                {:else}
                    <p>No replies...</p>
                {/each}
            </section>
        </section>
    
        <Footer {host} />
    </main>
    
</body>