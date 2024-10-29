<script lang="ts">
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
	import { pb } from '$lib/pb';

	let redirects: any[] = [];
	let newRedirect = { name: '', url: '' };
	let editingRedirect: any = null;
	let visibleDel = false;

	async function loadRedirects() {
		redirects = await pb.collection('Redirects').getFullList({ sort: 'name' });
	}

	async function createRedirect() {
		await pb.collection('Redirects').create(newRedirect);
		newRedirect = { name: '', url: '' };
		await loadRedirects();
	}

	async function updateRedirect() {
		if (editingRedirect) {
			await pb.collection('Redirects').update(editingRedirect.id, editingRedirect);
			editingRedirect = null;
			await loadRedirects();
		}
	}

	async function deleteRedirect(id: string) {
		await pb.collection('Redirects').delete(id);
		await loadRedirects();
	}

	async function deleteChat() {
		const records = await pb.collection('message').getFullList({ sort: '-created' });
		for (const record of records) {
			await pb.collection('message').delete(record.id);
		}
	}

	function copyToClipboard(text: string) {
		navigator.clipboard.writeText(text).then(() => {
			alert('Copied to clipboard!');
		});
	}

	onMount(() => {
		loadRedirects();
	});
</script>

<div class="flex h-screen w-full flex-col">
	<div class="sticky top-0 z-50">
		<div class="flex w-full items-center justify-between rounded-b-lg bg-black px-4 py-4">
			<span class="text-lg font-semibold text-white">Admin-Panel</span>
			<div class="flex items-baseline gap-x-1">
				<button
					on:click={() => goto('/')}
					class="rounded-md bg-white px-2 py-1 font-bold text-black hover:scale-[1.02] focus:bg-zinc-200"
				>
					Chat-Site
				</button>
			</div>
		</div>
	</div>
	<div class="mt-8 flex h-full w-full flex-col items-center gap-y-8 px-8">
		<div class="flex h-fit w-fit flex-col rounded-md bg-[#3b3b3b] px-4 py-2">
			<span class="text-lg font-bold text-gray-200">Chat Actions:</span>
			<button
				class="mb-1 mt-2 rounded-md bg-zinc-900/90 px-2 py-1 font-bold text-zinc-300/90"
				on:click={() => goto('/admin/export')}
			>
				Export
			</button>

			<button
				class="mt-6 rounded-md bg-red-900/90 px-2 py-1 font-bold text-zinc-300/90"
				on:click={() => {
					deleteChat();
					visibleDel = true;
					setTimeout(() => (visibleDel = false), 1000);
				}}
			>
				{#if visibleDel}
					<span>Successfully deleted!</span>
				{:else}
					Delete Chat
				{/if}
			</button>
		</div>

		<div class="w-full max-w-4xl rounded-md bg-[#3b3b3b] px-4 py-4">
			<h2 class="mb-4 text-lg font-bold text-gray-200">Manage Redirects</h2>
			<table class="w-full">
				<thead>
					<tr>
						<th class="px-4 py-2 text-left text-gray-300">Name</th>
						<th class="px-4 py-2 text-left text-gray-300">URL</th>
						<th class="px-4 py-2 text-left text-gray-300">Actions</th>
					</tr>
				</thead>
				<tbody>
					{#each redirects as redirect (redirect.id)}
						<tr>
							<td class="px-4 py-2 text-gray-300">
								{#if editingRedirect && editingRedirect.id === redirect.id}
									<input
										bind:value={editingRedirect.name}
										class="w-full rounded bg-zinc-700 px-2 py-1 text-white"
									/>
								{:else}
									{redirect.name}
								{/if}
							</td>
							<td class="px-4 py-2 text-gray-300">
								{#if editingRedirect && editingRedirect.id === redirect.id}
									<input
										bind:value={editingRedirect.url}
										class="w-full rounded bg-zinc-700 px-2 py-1 text-white"
									/>
								{:else}
									{redirect.url}
								{/if}
							</td>
							<td class="flex items-center gap-x-2 px-4 py-2">
								{#if editingRedirect && editingRedirect.id === redirect.id}
									<button
										on:click={updateRedirect}
										class="material-icons rounded-md bg-zinc-900/90 px-2 py-1 text-zinc-300/90 hover:bg-zinc-800/90"
									>
										Save
									</button>
									<button
										on:click={() => (editingRedirect = null)}
										class="material-icons rounded-md bg-zinc-900/90 px-2 py-1 text-zinc-300/90 hover:bg-zinc-800/90"
									>
										Cancel
									</button>
								{:else}
									<button
										on:click={() => copyToClipboard(`${window.location.origin}/r/${redirect.name}`)}
										class="material-icons rounded-md bg-zinc-900/90 px-2 py-1 text-zinc-300/90 hover:bg-zinc-800/90"
									>
										content_copy
									</button>
									<button
										on:click={() => (editingRedirect = { ...redirect })}
										class=" material-icons rounded-md bg-zinc-900/90 px-2 py-1 text-zinc-300/90 hover:bg-zinc-800/90"
									>
										Edit
									</button>
									<button
										on:click={() => deleteRedirect(redirect.id)}
										class=" rounded-md bg-red-900/90 px-2 py-1 font-bold text-zinc-300/90 hover:bg-red-800/90"
									>
										Delete
									</button>
								{/if}
							</td>
						</tr>
					{/each}
					<tr>
						<td class="px-2 py-2">
							<input
								bind:value={newRedirect.name}
								placeholder="New redirect name"
								required
								class=" rounded bg-zinc-700 px-2 py-1 text-white"
							/>
						</td>
						<td class="px-2 py-2">
							<input
								bind:value={newRedirect.url}
								placeholder="New redirect URL"
								type="url"
								required
								class="w-full rounded bg-zinc-700 px-2 py-1 text-white"
							/>
						</td>
						<td class="px-2 py-2">
							<button
								on:click={createRedirect}
								class="rounded-md bg-zinc-900/90 px-2 py-1 font-bold text-zinc-300/90 hover:bg-zinc-800/90"
							>
								Add
							</button>
						</td>
					</tr>
				</tbody>
			</table>
		</div>
	</div>
	<div class="static">
		<div class="flex justify-center pb-1 text-[#7a7777]">
			<span>
				<a href="https://github.com/louist2469">Louist2469</a> •
				<a href="https://github.com/TSC-Home">TSC-Home</a>
			</span>
		</div>
	</div>
</div>

<style>
	:root {
		background-color: #171717;
	}
</style>
