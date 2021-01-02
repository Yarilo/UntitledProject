<script>
       import Icon from './Icon.svelte';
       export let query;
       export let request;

	let photographer;
	let photographerLink;
	let photoLink;

       $: {
           if(query) loadPhoto(query);
       }


       async function loadPhoto (query)  {
		const response = await request.get(`/photos/random/?query=${query}&orientation=landscape&featured=true`);
		const {urls, user, links} = response.data;
		const src = `${urls.raw}?auto=format&q=10` //  https://docs.imgix.com/apis/rendering/format/q
		photographer = user.name;
		photographerLink = user.links.html;
		photoLink = links.html;
		await loadImage(src)
       }
       
       function loadImage(imageUrl) {
              let pic = new Image(); 
              pic.onload = function() {
                     document.querySelector('#image').style.backgroundImage = 'url(' + imageUrl + ')';
                     return new Promise().resolve();
              }
              pic.src = imageUrl;
       }
  
       async function reloadPhoto () {
	       await loadPhoto(query)
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
       :global(.credits) {    
		color: white;
	
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

