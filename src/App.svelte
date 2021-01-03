<script>
	import Background from './Background.svelte';
	import MusicPlayer from './MusicPlayer.svelte';

	const DELAY_TIME_MS = 600;
	
	let timeout;
	let fetchPromise;

	let query;
	let isPlayerLoaded = false;
	let isImageLoaded = false;
	

	const onPlayerLoad = () => isPlayerLoaded = true;
	const onImageLoad = () => isImageLoaded =true;

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
	{#if isPlayerLoaded} 
		<div id='keyword'>
			{#if !isImageLoaded} 
			<h1>Type something</h1>
			{/if }
			<input on:input={updateQuery}>
		</div>
	
	{/if}
	<div class={isPlayerLoaded ? 'music-player': 'music-player not-logged'}>
		<MusicPlayer {query} onLoad={onPlayerLoad} />
	</div>
	<Background {fetchPromise} {query} onLoad={onImageLoad} />
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