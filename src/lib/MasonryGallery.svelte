// src/lib/MasonryGallery.svelte
<script>
  import { onMount } from 'svelte';
  
  // Props with default values
  export let columnWidth = 250;
  export let gap = 16;
  
  let images = [];
  let loading = true;
  let lightboxImage = null;
  
  // Function to load images from the images directory
  async function loadImages() {
    try {
      // In a real app, this would be replaced with your actual image loading logic
      // You could fetch from an API, import directly, or use Vite's import.meta.glob
      const imageModules = import.meta.glob('/src/assets/images/*');
      
      // Convert the modules to an array of image URLs
      const imagePromises = Object.entries(imageModules).map(async ([path, loader]) => {
        const module = await loader();
        return {
          url: module.default,
          alt: path.split('/').pop()
        };
      });
      
      images = await Promise.all(imagePromises);
      loading = false;
    } catch (error) {
      console.error('Error loading images:', error);
      loading = false;
    }
  }
  
  function openLightbox(image) {
    lightboxImage = image;
  }
  
  function closeLightbox() {
    lightboxImage = null;
  }
  
  onMount(() => {
    loadImages();
  });
</script>

<div class="masonry-container">
  {#if loading}
    <div class="loading">Loading images...</div>
  {:else}
    <div 
      class="masonry-grid"
      style="--column-width: {columnWidth}px; --gap: {gap}px"
    >
      {#each images as image}
        <div class="masonry-item" on:click={() => openLightbox(image)}>
          <img
            src={image.url}
            alt={image.alt}
            loading="lazy"
          />
        </div>
      {/each}
    </div>
  {/if}
</div>

{#if lightboxImage}
  <div class="lightbox" on:click={closeLightbox}>
    <img src={lightboxImage.url} alt={lightboxImage.alt} />
  </div>
{/if}

<style>
  .masonry-container {
    padding: var(--gap);
  }
  
  .masonry-grid {
    columns: auto var(--column-width);
    column-gap: var(--gap);
  }
  
  .masonry-item {
    break-inside: avoid;
    margin-bottom: var(--gap);
    border-radius: 8px;
    overflow: hidden;
    cursor: pointer;
    transition: transform 0.3s ease;
  }
  
  .masonry-item:hover {
    transform: scale(1.02);
  }
  
  .masonry-item img {
    width: 100%;
    display: block;
  }
  
  .loading {
    text-align: center;
    padding: 20px;
    font-family: system-ui, -apple-system, sans-serif;
  }
  
  .lightbox {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.9);
    display: grid;
    place-items: center;
    z-index: 1000;
    cursor: pointer;
  }
  
  .lightbox img {
    max-width: 90%;
    max-height: 90vh;
    object-fit: contain;
  }
</style>