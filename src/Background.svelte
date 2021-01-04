<script>     
       import { onMount } from 'svelte';
	import axios from 'axios';
       import Icon from './Icon.svelte';
       
       export let query;
       export let onLoad;
       export let onLoading;

	let photographer;
	let photographerLink;
	let photoLink;
       let request;

       const { UNSPLASH_ACCESS_KEY } = process.env;
	const BASE_UNSPLASH_URL =  'https://api.unsplash.com/';


	onMount(() => {
        request = axios.create({ 
			baseURL: BASE_UNSPLASH_URL,
			headers: { Authorization: `Client-ID ${UNSPLASH_ACCESS_KEY}` }
		});
       })


       $: {
           if(query) onLoading(loadPhoto(query));
       }

       
       async function loadPhoto (query)  {
              const response = await request.get(`/photos/random/?query=${query}&orientation=landscape&featured=true`);
              const {urls, user, links} = response.data;
              const src = `${urls.raw}?auto=format&q=10` //  https://docs.imgix.com/apis/rendering/format/q
              photographer = user.name;
              photographerLink = user.links.html;
              photoLink = links.html;
              await loadImage(src)
              onLoad();
       }
       
       function loadImage(imageUrl) {
              return new Promise((resolve, reject) => {
                     let pic = new Image(); 
                     pic.onload = function() {
                            document.querySelector('#image').style.backgroundImage = 'url(' + imageUrl + ')';
                            resolve();
                     }
                     pic.src = imageUrl;
              })
       }
  
       async function reloadPhoto () {
	       onLoading(loadPhoto(query));
       }
       
</script>

<main>
	{#if photographerLink}
	       <div class='credits'>
                     <a href={photoLink} class="by" target='_blank'>by</a> 
                     <a href={photographerLink} target='_blank'>{photographer}</a> 
                     <div class='reload-photo'>
                            <Icon onClick={reloadPhoto} size={'small'} name='refresh-cw' />
                     </div>
              </div>
	{/if}
       <div id="image"  />		
</main>

<style>


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

	.credits .reload-photo {
		margin-top: 2.5px;
		margin-left: 10px;
	}

       #image {
              position:fixed;
              top: 0;
              left: 0;
              z-index: -1000;
              background-repeat: no-repeat;
              background-attachment: fixed;
              background-position: 50% 50%;
              background-size: cover;
              min-height: 100%;
              min-width: 100%;
       }

</style>

