<script lang="ts">
	import Footer from "$lib/components/Footer.svelte";

	let errorMessage = "";
	async function changeServer(e: any) {
		const server = new URL((e as any).target.serverurl.value);
		// check if server is valid
		try {
			const res = await (await fetch(server.href)).json();
			// load server page
			if (res.code === 404)
				window.location.href = `/${server.hostname}${server.port !== "" ? `:${server.port}` : ""}/`;
			else errorMessage = "Server is invalid.";
		} catch {
			errorMessage = "Server is invalid.";
		}
	}
</script>

<svelte:head>
	<title>Manage Connection</title>
</svelte:head>

<body>
	<main>
		<section>
			<h1>Manage Connection</h1>

			<p>Manage your server connection. This determines where we load content from.</p>

			<!-- Connection Form -->
			<div class="grid place-center">
				<form
					class="flex mt-8 mb-8"
					on:submit={(e) => {
						e.preventDefault();
						changeServer(e);
					}}
				>
					<label for="serverurl" class="mb-2"><b>Server URL&ast;</b></label>
					<input type="url" placeholder="Server URL" name="serverurl" required />

					<button class="primary mt-4">Change Server</button>
					<p class="mt-4" style="color: red;">{errorMessage}</p>
				</form>
			</div>
		</section>

		<Footer host="#" />
	</main>
</body>

<style>
	.flex {
		flex-direction: column;
	}
	form {
		width: var(--u-100);
	}
</style>
