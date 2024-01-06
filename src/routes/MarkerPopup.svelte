<script>
  export let title;
  export let titleLink = null;
  export let paragraphs;
</script>

<div class="container">
  {#if titleLink}
    <a href={titleLink} target="_blank" rel="noopener noreferrer"><b>{title}</b></a>
  {:else}
    <b>{title}</b>
  {/if}
  {#each paragraphs as p}
    <p>{p}</p>
  {/each}
</div>

<style>
  div p {
    margin-top: 0.3rem;
    margin-bottom: 0.3rem;
  }
</style>
