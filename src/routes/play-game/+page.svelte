<script lang="ts">
	import { page } from '$app/stores';
	import { Button } from 'flowbite-svelte';
	import { autoAnimate } from '@formkit/auto-animate';
	import games from '../../games.json';

	interface GameServer {
		command: string;
		workdir: string;
		port: number;
	}

	interface Game {
		id: string;
		name: string;
		cover: string;
		path: string;
		server?: GameServer;
	}

	const gamesList = games as Game[];

	let id = $derived($page.url.searchParams.get('id') || '');

	let game = $derived(gamesList.find((g) => g.id === id));

	let showBar = $state(true);
	let isFullscreen = $state(false);

	let gameUrl = $derived.by(() => {
		if (!game) return '';
		if (game.server) {
			return `http://localhost:${game.server.port}/`;
		}
		return game.path;
	});

	function toggleBar() {
		showBar = !showBar;
	}

	function handleExitFullscreen() {
		if (document.fullscreenElement) {
			document.exitFullscreen();
		}
	}

	function toggleFullscreen() {
		if (!document.fullscreenElement) {
			document.documentElement.requestFullscreen();
			isFullscreen = true;
		} else {
			document.exitFullscreen();
			isFullscreen = false;
		}
	}
</script>

{#if !id}
	<div class="flex h-screen items-center justify-center">
		<div class="text-center">
			<h1 class="mb-4 text-2xl font-bold text-white">Oyun Seçilmedi</h1>
			<p class="text-gray-400">Bir oyun yüklemek için ?id= kullan</p>
		</div>
	</div>
{:else if !game}
	<div class="flex h-screen items-center justify-center">
		<div class="text-center">
			<h1 class="mb-4 text-2xl font-bold text-white">Oyun Bulunamadı</h1>
			<p class="text-gray-400">"{id}" adlı oyun mevcut değil</p>
			<a href="/" class="mt-4 inline-block">
				<Button>Anasayfaya Dön</Button>
			</a>
		</div>
	</div>
{:else if game}
	<div id="game-container" use:autoAnimate class="fixed inset-0 bg-black">
		<iframe
			src={gameUrl}
			title={game.name}
			class="w-full border-0 transition-all duration-300 {showBar
				? 'h-[calc(100%-52px)]'
				: 'h-full'}"
			sandbox="allow-scripts allow-forms allow-top-navigation allow-same-origin allow-popups allow-popups-to-escape-sandbox"
		></iframe>

		<button
			onclick={toggleBar}
			class="fixed right-3 bottom-3 z-50 rounded-lg bg-[#1F1F23]/80 p-2 text-white/80 shadow-lg backdrop-blur-sm transition-all hover:bg-[#1F1F23] hover:text-white {showBar
				? 'pointer-events-none opacity-0'
				: 'pointer-events-auto opacity-100'}"
			title="Paneli Göster"
		>
			<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 15l7-7 7 7" />
			</svg>
		</button>

		<div
			class="fixed right-0 bottom-0 left-0 flex items-center justify-between border-t border-white/10 bg-[#0d0d0d]/95 px-4 py-2 backdrop-blur-sm transition-transform duration-300 {showBar
				? 'translate-y-0'
				: 'translate-y-full'}"
		>
			<div class="flex items-center gap-3">
				<span class="truncate text-sm font-medium text-white">{game.name}</span>
			</div>

			<div class="flex items-center gap-1">
				<button
					onclick={toggleBar}
					class="rounded p-1.5 text-gray-400 transition-colors hover:bg-white/10 hover:text-white"
					title="Paneli Gizle"
				>
					{#if showBar}
						<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M19 9l-7 7-7-7"
							/>
						</svg>
					{:else}
						<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M5 15l7-7 7 7"
							/>
						</svg>
					{/if}
				</button>

				<button
					onclick={toggleFullscreen}
					class="rounded p-1.5 text-gray-400 transition-colors hover:bg-white/10 hover:text-white"
					title="Tam Ekran"
				>
					{#if isFullscreen}
						<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M9 9V4.5M9 9H4.5M9 9L3.75 3.75M9 15v4.5M9 15H4.5M9 15l-5.25 5.25M15 9h4.5M15 9V4.5M15 9l5.25-5.25M15 15h4.5M15 15v4.5m0-4.5l5.25 5.25"
							/>
						</svg>
					{:else}
						<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M4 8V4m0 0h4M4 4l5 5m11-1V4m0 0h-4m4 0l-5 5M4 16v4m0 0h4m-4 0l5-5m11 5l-5-5m5 5v-4m0 4h-4"
							/>
						</svg>
					{/if}
				</button>

				<a
					href="/"
					onclick={handleExitFullscreen}
					class="rounded p-1.5 text-gray-400 transition-colors hover:bg-white/10 hover:text-white"
					title="Menüye Dön"
				>
					<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M10 19l-7-7m0 0l7-7m-7 7h18"
						/>
					</svg>
				</a>
			</div>
		</div>
	</div>
{/if}
