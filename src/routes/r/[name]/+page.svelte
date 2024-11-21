<script lang="ts">
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import { pb } from '$lib/pb';
	import { page } from '$app/stores';

	let isLoading = true;
	let redirectUrl: string | null = null;
	let error: string | null = null;

	onMount(async () => {
		const name = $page.params.name;

		try {
			const record = await pb.collection('Redirects').getFirstListItem(`name="${name}"`);
			if (record.url) {
				redirectUrl = record.url;
				window.location = record.url;
			} else {
				error = 'Ungültige Weiterleitung';
			}
		} catch (e) {
			error = 'Weiterleitung nicht gefunden';
		} finally {
			isLoading = false;
		}
	}); // Wenn Seite startet, nehme den SLUG ([name]) aus der URL und leide, wenn der SLUG in der Datenbank vorhanden, auf die neue Seite weiter
</script>

<div class="flex h-screen items-center justify-center">
	{#if isLoading}
		<p class="text-2xl text-gray-600">Lade Weiterleitung...</p>
	{:else if error}
		<p class="text-2xl text-red-600">{error}</p>
	{:else if redirectUrl}
		<p class="text-2xl text-gray-600">Weiterleitung zu {redirectUrl}...</p>
	{:else}
		<p class="text-2xl text-gray-600">Unerwarteter Fehler</p>
	{/if}
</div>
