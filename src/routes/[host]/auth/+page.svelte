<script lang="ts">
	import Footer from "$lib/components/Footer.svelte";

	import PocketBase from "pocketbase";
	import type { PageData } from "./$types";
	import { onMount } from "svelte";


	export let data: PageData;
	// @ts-ignore
	export const { host } = data;

	// create pb client
	let pb = new PocketBase(`http://${host}`);

	// handle form submit
	let errorMessage = "";

	// login function
	async function login(e: any) {
		try {
			const authData = await pb
				.collection("users")
				.authWithPassword(e.target.username.value, e.target.password.value);
			window.location.href = `/${host}/`;
		} catch (err: any) {
			errorMessage = err;
		}
	}

	// signup function
	async function signup(e: any) {
		try {
			const data = {
				username: e.target.username.value,
				password: e.target.password.value,
				passwordConfirm: e.target.password.value,
				displayName: e.target.username.value // <- we'll add support for this later
				// TODO: that
			};
			const u = await pb.collection("users").create(data);
			await login(e);
			// create badges record now
			await pb.collection("user_badges").create({
				user: u.id,
				badges: []
			});
		} catch (err: any) {
			errorMessage = err;
		}
	}

	// base submit function
	let action = "signup";
	function baseSubmit(e: any) {
		if (action === "signup") signup(e);
		else login(e);
	}

	let hasAccount = false;
	onMount(() => {
		if (window.location.protocol === "https:") pb = new PocketBase(`https://${host}`);
		if (pb.authStore.model !== null) hasAccount = true;
	});
</script>

<svelte:head>
	<title>Auth - Reforum</title>
</svelte:head>

<app>
	<main>
		<section>
			<h1>Account</h1>

			{#if !hasAccount}
				<div class="grid place-center">
					<form
						class="flex mt-12 mb-8"
						style="flex-direction: column; width: var(--u-100);"
						on:submit|preventDefault={baseSubmit}
					>
						<label for="username" class="mb-2"><b>Username&ast;</b></label>
						<input type="text" name="username" required placeholder="Username" />

						<label for="password" class="mb-2 mt-4"><b>Password&ast;</b></label>
						<input type="password" name="password" required placeholder="Password" />

						<button class="primary mt-4">Sign Up</button>
						<button
							class="mt-2"
							on:click={() => {
								action = "login";
							}}>Login</button
						>

						<p class="mt-4" style="color: red;">{errorMessage}</p>
					</form>
				</div>
			{/if}
		</section>

		<Footer {host} />
	</main>
</app>
