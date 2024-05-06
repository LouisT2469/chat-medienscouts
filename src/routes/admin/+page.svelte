<script lang="ts">
	import { goto } from '$app/navigation';
	import PocketBase from 'pocketbase';

	const pb = new PocketBase('https://chat-ms.app.louist2469.de');

	async function deleteChat() {
		const records = await pb.collection('message').getFullList({
			sort: '-created'
		});

		records.forEach((records) => {
			//console.log('ID:', records.id);
			pb.collection('message').delete(records.id);
		});
	}

	let visibleDel = false; //false
</script>

<div class="flex h-screen w-full flex-col">
	<div class="sticky top-0 z-50">
		<div class="flex w-full items-center justify-between rounded-b-lg bg-black px-4 py-4">
			<span class="text-lg font-semibold text-white">Admin-Pannel</span>
			<div class="flex items-baseline gap-x-1">
				<button
					on:click={() => {
						goto('/');
					}}
					class="rounded-md bg-white px-2 py-1 font-bold text-black hover:scale-[1.02] focus:bg-zinc-200"
					>Chat-Site</button
				>
			</div>
		</div>
	</div>
	<div class="mt-8 flex h-full w-full justify-center gap-x-4 px-8">
		<div class="flex h-fit w-fit flex-col rounded-md bg-[#3b3b3b] px-4 py-2">
			<span class="text-lg font-bold text-gray-200">Chat-Actions:</span>
			<span class="mb-1 mt-2 italic text-gray-300">Export the complete chat.</span>
			<button
				class=" rounded-md bg-zinc-900/90 px-2 py-1 font-bold text-zinc-300/90"
				on:click={() => {
					goto('/admin/export');
				}}>Export</button
			>

			<span class="mb-1 mt-6 italic text-gray-300">Delete the entire chat!</span>
			<button
				class=" rounded-md bg-red-900/90 px-2 py-1 font-bold text-zinc-300/90"
				on:click={() => {
					deleteChat();
					visibleDel = true;
					setTimeout(() => {
						visibleDel = false;
					}, 1000);
				}}
				>{#if visibleDel}
					<span>Successfully deleted!</span>
				{:else}
					Delete
				{/if}</button
			>
		</div>
	</div>
	<div class="static">
		<div class="flex justify-center pb-1 text-[#7a7777]">
			<span>
				<a href="https://github.com/louist2469">Louist2469</a>
				• <a href="https://github.com/TSC-Home">TSC-Home</a>
			</span>
		</div>
	</div>
</div>

<style>
	:root {
		background-color: #171717;
	}
</style>
