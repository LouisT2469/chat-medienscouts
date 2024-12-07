<script lang="ts">
	import { pb } from '$lib/pb'; //pocketbase Einbindung

	import { onMount, afterUpdate } from 'svelte';

	let user: string | null = null;
	let messages: any[] = [];
	let send = '';
	let inputDisabled = false; // State variable für die Deaktivierung des Inputs
	let countdownTime = 0; // Variable für die Countdown-Zeit

	// Element-Referenz für den Nachrichten-Container
	let messagesContainer: HTMLDivElement | null = null;

	// Funktion zum Scrollen nach unten
	const scrollToBottom = () => {
		if (messagesContainer) {
			messagesContainer.scrollTop = messagesContainer.scrollHeight;
		}
	};

	// Countdown-Funktion für die Zeit im Input-Feld
	const startCountdown = () => {
		countdownTime = 4; // Setze die Countdown-Zeit auf 4 Sekunden

		const countdownInterval = setInterval(() => {
			countdownTime--;
			if (countdownTime === 0) {
				clearInterval(countdownInterval); // Stoppe den Countdown, wenn die Zeit abgelaufen ist
				inputDisabled = false; // Aktiviere das Input-Feld wieder
			}
		}, 1000); // Aktualisiere den Countdown alle 1 Sekunde
	};

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
			console.log(records);
			messages = records.reverse();
			pb.collection('message').subscribe('*', (e) => {
				if (e.record.user !== user) {
					messages.push(e.record);
					messages = [...messages]; // Force re-render
					scrollToBottom(); // Scroll beim Hinzufügen einer Nachricht
				}
			});

			scrollToBottom(); // Scroll beim ersten Laden
		} catch (error) {
			console.error(error);
		}
	});

	afterUpdate(() => {
		scrollToBottom(); // Scroll bei Änderungen in der Nachrichtenliste
	});

	async function sendMessage(user: string, message: string) {
		try {
			// Neuer Regex zur Erkennung von Emojis
			const emojiRegex = /(?:\p{Emoji_Presentation}|\p{Emoji})/gu;

			let consecutiveEmojiCount = 0;

			// Überprüfe die Zeichen der Nachricht einzeln
			for (const char of message) {
				if (char.match(emojiRegex)) {
					consecutiveEmojiCount++;
					if (consecutiveEmojiCount > 3) {
						alert('Nicht mehr als drei Emojis hintereinander erlaubt!');
						return; // Bricht den Sendevorgang ab
					}
				} else {
					consecutiveEmojiCount = 0; // Zähler zurücksetzen, wenn kein Emoji
				}
			}

			pb.autoCancellation(false);
			inputDisabled = true; // Deaktiviere das Input-Feld
			startCountdown(); // Starte den Countdown

			const record = await pb.collection('message').create({
				user,
				message
			});

			messages = [...messages, record]; // Füge die Nachricht zur Liste hinzu
		} catch (error) {
			console.error(error);
		}
	}
</script>

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

	<div class="mb-2 mt-2 h-full w-full overflow-y-scroll" bind:this={messagesContainer}>
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

	<div class="static">
		<form
			on:submit|preventDefault={async () => {
				await sendMessage(user, send);
				send = '';
			}}
			class="flex w-full items-center justify-center gap-x-4 rounded-t-lg bg-black p-3"
		>
			<input
				required
				class="w-11/12 rounded-md px-2 py-1 outline-none"
				placeholder="Gib eine Nachricht ein."
				type="text"
				bind:value={send}
				disabled={inputDisabled}
			/>
			<!-- Anzeige der Countdown-Zeit -->
			<button type="submit" class="material-icons text-3xl text-white">send</button>
			<span class="text-white">{countdownTime}s</span>
		</form>
	</div>
</div>

<style>
	:root {
		background-color: #171717;
		--sb-track-color: #171717;
		--sb-thumb-color: #000000;
		--sb-size: 14px;
	}

	::-webkit-scrollbar {
		width: var(--sb-size);
	}

	::-webkit-scrollbar-track {
		background: var(--sb-track-color);
		border-radius: 10px;
	}

	::-webkit-scrollbar-thumb {
		background: var(--sb-thumb-color);
		border-radius: 10px;
	}

	@supports not selector(::-webkit-scrollbar) {
		:root {
			scrollbar-color: var(--sb-thumb-color) var(--sb-track-color);
		}
	}
</style>
