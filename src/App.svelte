<script>
    import { onMount } from 'svelte';
	import axios from 'axios';
	import Background from './Background.svelte';
	import MusicPlayer from './MusicPlayer.svelte';
	import Icon from './Icon.svelte';

	const { UNSPLASH_ACCESS_KEY } = process.env;
	const DELAY_TIME_MS = 600;
	const BASE_UNSPLASH_URL =  'https://api.unsplash.com/';
	
	let request;
	let src;
	let timeout;
	let fetchPromise;

	let query;
	let photographer;
	let photographerLink;
	let photoLink;
	let isPlayerLoaded = false;

	$: { 
		fetchPromise = fetchPhoto(query);
	}

	onMount(() => {
        request = axios.create({ 
			baseURL: BASE_UNSPLASH_URL,
			headers: { Authorization: `Client-ID ${UNSPLASH_ACCESS_KEY}` }
		});
	}); 

	const onPlayerLoad = () => isPlayerLoaded = true;

	async function fetchPhoto (query)  {
		if (!query) return;
		const response = await request.get(`/photos/random/?query=${query}&orientation=landscape&featured=true`);
		const {urls, user, links} = response.data;
		src = `${urls.raw}?auto=format`
		photographer = user.name;
		photographerLink = user.links.html;
		photoLink = links.html;
		return src;
		
	} 
	
	async function updateQuery (event = {target:{}}) {
		clearTimeout(timeout);
		timeout = setTimeout(() => {
			const newQuery = event.target.value;
			if (newQuery && newQuery.length < 3) return;
			query = newQuery;
		}, DELAY_TIME_MS)
	}
	
	async function reloadPhoto () {
		fetchPromise = fetchPhoto(query)
	}

</script>

<main class={src ? 'white-text': ''}>
	{#if isPlayerLoaded} 
		<div id='keyword'>
			{#if !src} 
			<h1>Type something</h1>
			{/if }
			<input on:input={updateQuery}>
			{#if query} 
				<Icon onClick={reloadPhoto} size={'small'} name='refresh-cw' />
			{/if}
		</div>
		{#await fetchPromise}
			<p>Fetching...</p>
		{:catch error}
			<p style="color: red">{error.message}</p>
		{/await}
		{#if photographerLink}
			<div class='credits'>
				<a href={photoLink} class="by" target='_blank'>by</a> 
				<a href={photographerLink} target='_blank'>{photographer}</a> 
			</div>
		{/if}
	{/if}
	<div class={isPlayerLoaded ? 'music-player': 'music-player not-logged'}>
		<MusicPlayer {query} onLoad={onPlayerLoad} />
	</div>
	<Background {src}/>
</main>

<style>


     :global(body .white-text input) {       
			 color: white;
			 border-bottom: 1px solid white !important;
     } 

 	 :global(body .white-text p) {    
             color: white;
     } 
	
	:global(body .white-text .credits) {    
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

	.credits {
		position: absolute;
		bottom: 0px;
		left: 0;
		margin: 20px;
		display: flex;
		
	}
	.credits .by {
			margin-right: 5px;
	}

	.credits a {
		color: inherit;
		text-decoration: none;
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