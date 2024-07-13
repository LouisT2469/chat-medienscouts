<script lang="ts">
	import PocketBase from 'pocketbase';
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';

	const pb = new PocketBase('https://chat-ms.app.louist.de');

	let user: string | null = null;
	let messages: any[] = [];
	let send = '';

	onMount(async () => {
		const storedUsername = localStorage.getItem('username');

		if (storedUsername) {
			user = storedUsername;
		} else {
			const newUsername = prompt('Bitte geben Sie Ihren Benutzernamen ein:');

			if (newUsername) {
				localStorage.setItem('username', newUsername);
				user = newUsername;
			}
		}

		try {
			const records = await pb.collection('message').getFullList({
				sort: '-created'
			});
			messages = records.reverse();
		} catch (error) {
			console.error(error);
		}

		setTimeout(() => {
			window.print();
		}, 1000);
		setTimeout(() => {
			goto('/admin');
		}, 2000);
	});
</script>

<div class="flex h-screen w-full flex-col">
	<div class="sticky top-0 z-50">
		<div class="flex w-full items-center justify-between rounded-b-lg bg-black px-4 py-4">
			<span class="text-lg font-semibold text-white">Chat-Medienscouts</span>
			<span class="text-lg font-semibold text-white">Workshop - Cybermobbing</span>
			<div class="flex items-baseline gap-x-1">
				<span class="font-semibold text-white">Name:</span>
				<span class="text-white">{user}</span>
			</div>
		</div>
	</div>

	<div class="mb-2 mt-2 h-full w-full">
		<div class="flex h-full w-full flex-col items-center gap-y-2">
			{#each messages as message}
				{#if message.user === 'System' || message.user === 'system'}
					<div class="flex w-11/12 flex-col rounded-md bg-[#f54747] px-2 py-1">
						<span class="text-sm font-extrabold text-black">{message.user}</span>
						<span class="font-semibold text-black">{message.message}</span>
					</div>
				{:else if message.user === user}
					<div class="flex w-11/12 flex-col rounded-md bg-[#3bda85] px-2 py-1">
						<span class="text-sm font-bold text-black">{message.user}</span>
						<span>{message.message}</span>
					</div>
				{:else}
					<div class="flex w-11/12 flex-col rounded-md bg-[#3b3b3b] px-2 py-1">
						<span class="text-sm font-bold text-white/80">{message.user}</span>
						<span class="text-zinc-300">{message.message}</span>
					</div>
				{/if}
			{/each}
		</div>
	</div>
</div>

<style>
	:root {
		background-color: #171717;
	}
</style>
