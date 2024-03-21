<script lang="ts">
	import Sun from 'svelte-radix/Sun.svelte';
	import Moon from 'svelte-radix/Moon.svelte';
	import { Button } from '$lib/components/ui/button/index.js';

	import { toggleMode } from 'mode-watcher';

	import PocketBase from 'pocketbase';
	import { onMount } from 'svelte';

	const pb = new PocketBase('https://svelte1sandbox.krio.synthetix.me');

	let user: any = '' || 'anonymous' || null;

	let messages: any[] = [];

	let send = '';

	onMount(async () => {
		user = prompt('Enter your name:');

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

<div class="h-screen w-full">
	<div class="m-2 flex h-auto items-center justify-between rounded-md border p-2">
		<div class="text-lg font-semibold">Chat - Medienscouts</div>
		<div>
			<Button on:click={toggleMode} variant="outline" size="icon">
				<Sun
					class="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0"
				/>
				<Moon
					class="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100"
				/>
				<span class="sr-only">Toggle theme</span>
			</Button>
		</div>
	</div>
	<div
		class="sticky bottom-0 m-2 h-[85%] scroll-mt-10 overflow-y-auto scroll-smooth rounded-md border p-2"
	>
		{#each messages as message}
			{#if message.user === user}
				<div class="m-2 rounded-md border bg-blue-500 p-2">
					<div class="font-semibold">{message.user}</div>
					<div>{message.message}</div>
				</div>
			{:else}
				<div class="m-2 rounded-md border bg-purple-500 p-2">
					<div class="font-semibold">{message.user}</div>
					<div>{message.message}</div>
				</div>
			{/if}
		{/each}
	</div>
	<form
		on:submit|preventDefault={async () => {
			await sendMessage(user, send);
			send = '';
		}}
		class="flex"
	>
		<input
			type="text"
			class="m-2 w-[80%] rounded-md border p-2"
			placeholder="your message"
			bind:value={send}
		/>
		<button type="submit" class="m-2 w-[20%] rounded-md border p-2">Send</button>
	</form>
</div>
