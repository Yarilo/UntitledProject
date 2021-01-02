<script>
       export let src;


       $: {
              loadImage(src);
       }

       function loadImage(imageUrl) {
              let pic = new Image(); 
              pic.onload = function() {
                     document.querySelector('#image').style.backgroundImage = 'url(' + imageUrl + ')';
              }
              pic.src = imageUrl;
       }
  
</script>

<main>
       <div id="image"  />		
</main>

<style>


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

