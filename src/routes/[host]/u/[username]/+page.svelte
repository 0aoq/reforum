<script lang="ts">
	import Footer from "$lib/components/Footer.svelte";
	import PocketBase, { Record } from "pocketbase";
	import type { PageData } from "../$types";
	import { onMount } from "svelte";

	export let data: PageData;
	// @ts-ignore
	export const { host, username } = data;

	// create pb client
	let protocol = "http:";
	let pb = new PocketBase(`http://${host}`);

	let user: Record = {} as any;

	onMount(async () => {
		if (window.location.protocol === "https:") {
			pb = new PocketBase(`https://${host}`);
			protocol = "https:";
		}

		// load user
		try {
			user = await pb.collection("users").getFirstListItem(`username = "${username}"`);
		} catch {
			// return home if we fail
			// window.location.href = `/${host}/`;
		}
	});

	// handle changing profile picture
	async function changePfp() {
		// get new image...
		if (!pb.authStore.model) return;
		if (!pb.authStore.model.id === (user.id as any)) return;

		// create input
		const input = document.createElement("input");
		input.type = "file";
		input.accept = "image/jpg, image/jpeg, image/png, image/svg+xml, image/gif";
		input.click();

		// wait for change
		input.addEventListener("change", async () => {
			if (!input.files || !pb.authStore.model) return;
			let file = input.files[0];
			// upload
			const data = new FormData();
			data.set("avatar", file);
			await pb.collection("users").update(pb.authStore.model.id, data);
			window.location.reload();
		});
	}

	// handle bio editing
	let bioEditMode = false;
	let bioEditContent: HTMLElement;

	async function saveBio(e: any) {
		if (!pb.authStore.model) return;
		if (!bioEditContent) return;

		// update user bio
		await pb.collection("users").update(pb.authStore.model.id, {
			bio: bioEditContent.innerText
		});
		user.bio = bioEditContent.innerText;

		// remove bio editor
		bioEditMode = false;
	}
</script>

<svelte:head>
	<title>{username} on Reforum</title>
</svelte:head>

<app>
	<nav>
		<div class="title">
			<a
				href="/{host}"
				class="grid place-center"
				aria-label="User: {username}"
				title="Click to return to channel list"
			>
				{username}
			</a>
		</div>
	</nav>

	<main>
		<section>
			<div class="mb-8">
				<h1>{username}</h1>

				{#if pb.authStore.model && user.id === pb.authStore.model.id}
					<section class="mb-2">
						Actions:

						<div class="flex mt-2" style="gap: var(--u-2);">
							<button
								style="width: max-content;"
								class="primary"
								on:click={() => {
									// clear auth state and reload
									pb.authStore.clear();
									window.location.href = `/${host}/auth`;
								}}>Sign Out</button
							>
						</div>
					</section>
				{/if}
			</div>

			<div class="flex mb-4" style="gap: var(--u-4); flex-wrap: wrap;">
				<div>
					<img
						src="{protocol}//{host}/api/files/_pb_users_auth_/{user.id}/{user.avatar}?thumb=200x200"
						alt="{username}&apos;s avatar"
						title="{username}&apos;s avatar"
						class="pfp"
					/>

					{#if pb.authStore.model && user.id === pb.authStore.model.id}
						<button on:click={changePfp}>Change Picture</button>
					{/if}

					<section class="mt-2 flex justify-center align-center">
						<span class="chip">{user.rank}</span>
					</section>
				</div>

				<div>
					{#if !bioEditMode}
						<p style="max-width: var(--u-100); white-space: pre;">{user.bio || ""}</p>
					{:else}
						<p
							contenteditable="true"
							style="max-width: var(--u-100); white-space: pre;"
							bind:this={bioEditContent}
						>
							{user.bio || "<bio goes here>"}
						</p>

						<button class="primary mt-2" style="width: max-content;" on:click={saveBio}>Save</button
						>
					{/if}

					{#if pb.authStore.model && user.id === pb.authStore.model.id && !bioEditMode}
						<button
							class="mt-2"
							on:click={() => {
								bioEditMode = true;
							}}>Edit Bio</button
						>
					{/if}
				</div>
			</div>
		</section>

		<Footer {host} />
	</main>
</app>

<style>
	.pfp {
		border-radius: 0.4rem;
		transition: all 0.2s;
	}
    
	.pfp:hover {
		transform: scale(1.02);
		box-shadow: 0 0 8px rgba(0, 0, 0, 0.25);
	}

	.pfp:active {
		transform: scale(1.02) translateY(2px);
	}
    
	span.chip {
		color: hsl(0, 0%, 0%);
		background: hsl(208, 8%, 93%);
		border-radius: 360px;
		padding: 0.2rem 1rem;
		font-size: 0.8rem;
		cursor: pointer;
		user-select: none;
		transition: all 0.05s;
	} 

	span.chip:hover {
		background: hsl(208, 8%, 87%);
		text-decoration: none;
		color: hsl(0, 0%, 0%);
	}
</style>
