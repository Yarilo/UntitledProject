<script>
	import Background from './Background.svelte';
	import MusicPlayer from './MusicPlayer.svelte';

	const DELAY_TIME_MS = 600;
	
	let timeout;

	let query;
	let isPlayerInitialized = false;
	let isImageLoaded = false;
	let loadingPromise;

	const onPlayerInit = () => isPlayerInitialized = true;
	const onImageLoad = () => isImageLoaded = true;
	const onImageLoading = (promise) => loadingPromise = promise;


	async function updateQuery (event = {target:{}}) {
		clearTimeout(timeout);
		timeout = setTimeout(() => {
			const newQuery = event.target.value;
			if (newQuery && newQuery.length < 3) return;
			query = newQuery;
		}, DELAY_TIME_MS)
	}


</script>

<main class={isImageLoaded ? 'white-text': ''}>
	{#if isPlayerInitialized} 
		<div id='keyword'>
			{#if !isImageLoaded} 
			<h1>Type something</h1>
			{/if }
			<input on:input={updateQuery}>
			{#await loadingPromise}
                <p>Loading...</p>
            {:catch error}
                <p style="color: red">{error.message}</p>
            {/await}
		</div>
	
	{/if}
	<div class={isPlayerInitialized ? 'music-player': 'music-player not-logged'}>
		<MusicPlayer {query} onInitialization={onPlayerInit} />
	</div>
	<Background {query} onLoading={onImageLoading} onLoad={onImageLoad}  />
</main>

<style>


     :global(body .white-text input) {       
			 color: white;
			 border-bottom: 1px solid white !important;
     } 

 	 :global(body .white-text *) {    
             color: white;
     } 
	
	main {
		text-align: center;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		min-height: 100%;
       	min-width: 100%;
	}

	.music-player {
		position: absolute;
		bottom: 0;
		right: 0;
		width: 250px;
		margin-right: 10px;
	}

	.music-player.not-logged {
		position: relative;
	}

	#keyword {
		position: relative;
		bottom: 50px;
	}

	#keyword input {
		background: transparent;
		border: none;
		border-bottom: 1px solid black;
	}
</style>