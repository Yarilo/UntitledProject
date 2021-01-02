
<script>
    import axios from 'axios';
    import { onMount } from 'svelte';
    import Icon from './Icon.svelte';

    export let query;
    export let onLoad; 

    const { SPOTIFY_CLIENT_ID } = process.env;
    const BASE_SPOTIFY_URL='https://api.spotify.com'
    const ENDED_SONG_MS = 700;
    const SEARCH_LIMIT= 50;
    const VOLUME = 1; // 0 to 1;


    const scopes = encodeURIComponent('user-modify-playback-state streaming user-read-currently-playing');
    let spotifyUrl = `https://accounts.spotify.com/authorize?client_id=${SPOTIFY_CLIENT_ID}&redirect_uri=http://localhost:5000&response_type=token&state=123&scope=${scopes}`;
    
    const SPACE = 32;

    let spotifyAccessToken;
    let player; 
    let songInfo = '';
    let songUri;
    let isPlaying = false;
    let paused = false;

    $: {
        if (query) next();
    }
      
    const ENDPOINTS = {
        PLAY: () => `${BASE_SPOTIFY_URL}/v1/me/player/play?device_id=${player._options.id}`,
        SEARCH: (query) => `${BASE_SPOTIFY_URL}/v1/search?q=${encodeURIComponent(query)}&type=track&limit=${SEARCH_LIMIT}`
    }
    
    function loadSpotify() {
        const token = sessionStorage.getItem('spotify_access_token');
        player = new Spotify.Player({
            name: 'Web Playback SDK Quick Start Player',
            getOAuthToken: cb => { cb(token); }
        });

        // Error handling
        player.addListener('initialization_error', ({ message }) => { console.error(message); });
        player.addListener('authentication_error', ({ message }) => { console.error(message); });
        player.addListener('account_error', ({ message }) => { console.error(message); });
        player.addListener('playback_error', ({ message }) => { console.error(message); });

        // Playback status updates
        player.addListener('player_state_changed', state => {
            const  { position, duration, track_window } = state;
            paused = state.paused;
            updateSongInfo(track_window.current_track);
            const hasSongEnded = position === duration || (duration - position < ENDED_SONG_MS);
            if (hasSongEnded) next();
        });

        // Ready
        player.addListener('ready', ({ device_id }) => {
            console.log('Ready with Device ID', device_id);
            player.setVolume(VOLUME);
            onLoad();
        });

        // Not Ready
        player.addListener('not_ready', ({ device_id }) => {
            console.log('Device ID has gone offline', device_id);
        });

        player.connect();
        }

	onMount(() => {
        window.onSpotifyWebPlaybackSDKReady = () => loadSpotify()
        const { hash } = window.location;
        const token_query = 'access_token='
        spotifyAccessToken = hash.substring(hash.lastIndexOf(token_query) + token_query.length, hash.indexOf("&"));
        if (spotifyAccessToken) {
            sessionStorage.setItem('spotify_access_token', spotifyAccessToken)
        }
    })

    function getAuthHeaders() {
        return { headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${spotifyAccessToken}`
      } }
    }

    async function play (songUri) {
        await axios.put(ENDPOINTS.PLAY(), { uris: [songUri] }, getAuthHeaders());
        isPlaying = true;
    }

   async function pause () {
        await player.pause();
    }
    
    async function resume() {
        await player.resume();
    }

    async function search (query) {
        const result = await axios.get(ENDPOINTS.SEARCH(query), getAuthHeaders())
        if (!result || !result.data || !result.data.tracks) {
            console.log('Error searching a song', result);
            return;
        }
        const { items } = result.data.tracks;
        const randomIndex = Math.floor(Math.random() * (items.length - 0));
        return items[randomIndex].uri;
    }

    async function next() {
        songUri = await search(query);
        play(songUri);
    }
    
    async function updateSongInfo (currentTrack) {
        const songName = currentTrack.name;
        const artists = currentTrack.artists.map(artist => artist.name).join(',');
        songInfo = `${songName} by ${artists}`;
    }

    function handleKeydown (event) {
        if (event.keyCode === SPACE) {
            paused ? resume() : pause();
        }
    }
    
  
</script>

<svelte:window on:keydown={handleKeydown} />

<main>
            { #if !spotifyAccessToken }
                <h1>Welcome</h1>
                <a href={spotifyUrl}>Log in Spotify to begin.</a>
            { :else }
                {#if isPlaying}
                    <div class="controls">
                        <div class="main-controls">
                            {#if paused}
                                <Icon onClick={resume} name='play'/>
                            {:else }
                                <Icon onClick={pause} name='pause'/>
                            {/if}
                                <Icon onClick={next} name='skip-forward'/>
                                <Icon onClick={next} name='reload'/>
                        </div>
                        <p>{songInfo}</p>
                    </div>
                {/if}
            { /if }
</main>

<style>
    main {
        display: flex;
        flex-direction: column;
    }
    .main-controls {
        display:flex;
    }
    .controls {
        display: flex;
        flex-direction: column;
        align-items: center;
        color: black;
    }

</style>

<svelte:head>
	<script src="https://sdk.scdn.co/spotify-player.js" onload={loadSpotify}></script>   
</svelte:head>
