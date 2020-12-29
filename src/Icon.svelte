<script>
export let name;
export let onClick;
</script>

<svg class={'feather'} on:click={onClick}>
    <use xlink:href={`feather-sprite.svg#${name}`}/>
</svg>

<style>
     .feather {    
             color: white;
             cursor: pointer; /* All icons are clickable for now */
     }
</style>