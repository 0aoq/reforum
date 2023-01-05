<script lang="ts">
	import type PocketBase from "pocketbase";
	import type { Record } from "pocketbase";
	import { onMount } from "svelte";

	export let record: Record;
	export let host: string;

	export let pb: PocketBase;
	export let ownerid: string;

	export let topic: Record;
	export let thread: Record; // this is actually just a post! original post of the thread...

	export let type: "THREAD" | "REPLY"; // thread: main post, reply: reply to main post

	let protocol = "http:";
	let sender: any = { username: "", id: "", avatar: "" };

	let postOwner = false;
	let topicOwner = false;
	let threadOwner = false; // owner of the original thread

	onMount(() => {
		protocol = window.location.protocol;
		sender = (record as any).expand.sender;

		// evaluate owner roles
		if (!pb.authStore.model) return;
		postOwner = ownerid === pb.authStore.model.id;
		topicOwner = topic.creator === pb.authStore.model.id;
		threadOwner = thread.sender === pb.authStore.model.id;
	});

	// functions
	let component: HTMLElement;

	/**
	 * @function deleteThisMessage
	 * @description Delete the current message
	 */
	async function deleteThisMessage() {
		// run confirmation
		if (!confirm("Are you sure you want to do this?")) return;

		// delete
		try {
			await pb.collection(type === "THREAD" ? "posts" : "replies").delete(record.id);
			if (type === "THREAD") window.location.href = `/${host}`;
			else component.remove(); // remove component
		} catch {
			alert(`Failed to delete record with type: ${type}, id: ${record.id}`);
		}
	}
</script>

<component bind:this={component}>
	<section class="flex" style="gap: var(--u-4);">
		<!-- mod actions -->
		{#if postOwner || topicOwner || threadOwner}
			<section>
				<p class="flex align-center">
					<svg
						xmlns="http://www.w3.org/2000/svg"
						width="18"
						height="18"
						viewBox="0 0 24 24"
						fill="none"
						stroke="gold"
						stroke-width="2"
						stroke-linecap="round"
						stroke-linejoin="round"
						class="feather feather-shield mr-2"
						><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z" /></svg
					>

					{postOwner === true ? "User" : "Mod"} Actions
					<sup
						style="text-decoration: underline; font-size: xx-small;"
						class="ml-2"
						title={(postOwner === true
							? "You can see this menu because you are the original sender of this post."
							: "You can see this menu because you are the original owner of either the" +
							  " topic this post is in or the thread it is in.") +
							"\n\nThis menu can be used to delete this post or run another action."}
						>&lpar;?&rpar;</sup
					>:
				</p>

				<div class="flex mt-2" style="gap: var(--u-2);">
					<button on:click={deleteThisMessage}>Delete Content</button>
				</div>
			</section>
		{/if}

		<!-- actual body -->
		<div>
			<img
				src="{protocol}//{host}/api/files/_pb_users_auth_/{sender.id}/{sender.avatar}?thumb=180x180"
				alt="{sender.username}&apos;s avatar"
				title="{sender.username}&apos;s avatar"
				class="pfp"
			/>

			<section class="mt-2 flex justify-center align-center">
				<span class="chip">{sender.rank}</span>
			</section>
		</div>

		<div>
			<p>{record.content}</p>
		</div>
	</section>
</component>

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

	button {
		width: max-content;
	}
</style>
