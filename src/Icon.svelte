<script>
export let name;
export let onClick;
export let size;
</script>

<svg class={'feather'} class:small={size === 'small'} on:click={onClick}>
    <use xlink:href={`feather-sprite.svg#${name}`}/>
</svg>

<style>
     .feather {    
        color: white;
        cursor: pointer; /* All icons are clickable for now */
     }
     .small {
         width: 16px;
         height:16px;
     }

</style>