<script lang="ts">
	import type { Record } from "pocketbase";
	import { onMount } from "svelte";

	export let record: Record;
	export let host: string;

	let protocol = "http:";
	let sender: any = { username: "", id: "", avatar: "" };

	onMount(() => {
		protocol = window.location.protocol;
		sender = (record as any).expand.sender;

		console.log(record);
	});
</script>

<section class="flex" style="gap: var(--u-4);">
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
