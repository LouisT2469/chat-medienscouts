<script lang="ts">
	import PocketBase from 'pocketbase';
	import { onMount } from 'svelte';

	const pb = new PocketBase('https://chat-ms.app.louist2469.de');

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
			const records: any = await pb.collection('message').getFullList({
				sort: '-created'
			});
			console.log(records);
			messages = records.reverse();
			messages = messages;
			pb.collection('message').subscribe('*', function (e) {
				if (e.record.user !== user) {
					messages.push(e.record);
					messages = messages;
				}
			});
		} catch (error) {
			console.error(error);
		}
	});

	async function sendMessage(user: string, message: string) {
		try {
			pb.autoCancellation(false);
			const record = await pb.collection('message').create({
				user,
				message
			});
			messages = [...messages, record];
		} catch (error) {
			console.error(error);
		}
	}
</script>

<!--https://play.tailwindcss.com/FglNLK0zSj-->
<!--https://play.tailwindcss.com/eSTOhLI0DD-->

<div class="flex h-screen w-full flex-col">
	<div class="sticky top-0 z-50">
		<div class="flex w-full items-center justify-between rounded-b-lg bg-black px-4 py-4">
			<span class="text-lg font-semibold text-white">Chat-Medienscouts</span>
			<div class="flex items-baseline gap-x-1">
				<span class="font-semibold text-white">Name:</span>
				<span class="text-white">{user}</span>
			</div>
		</div>
	</div>
	<div class="mb-2 mt-2 h-full w-full overflow-y-scroll">
		<div class="flex h-full w-full flex-col items-center gap-y-2">
			{#each messages as message}
				{#if message.user === user}
					<div class="flex w-11/12 flex-col rounded-md bg-[#3bda85] px-2 py-1">
						<span class="text-sm font-bold text-black">{message.user}</span>
						<span class="text-black">{message.message}</span>
					</div>
				{:else if message.user === 'System'}
					<div class="flex w-11/12 flex-col rounded-md bg-[#f54747] px-2 py-1">
						<span class="text-sm font-extrabold text-black">{message.user}</span>
						<span class="font-semibold text-black">{message.message}</span>
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
	<div class="static">
		<form
			on:submit|preventDefault={async () => {
				await sendMessage(user, send);
				send = '';
			}}
			class="flex w-full items-center justify-center gap-x-4 rounded-t-lg bg-black p-3"
		>
			<input
				class="w-11/12 rounded-md px-2 py-1 outline-none"
				placeholder="Gib eine Nachricht ein."
				type="text"
				name=""
				id=""
				bind:value={send}
			/>
			<button class="material-icons text-3xl text-white hover:scale-[1.1]" type="submit"
				>send</button
			>
		</form>
	</div>
</div>

<style>
	:root {
		background-color: #171717;
	}
</style>
