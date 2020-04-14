<script>
  import { onMount } from "svelte";

  let promise = null;

  onMount(() => {
    promise = fetch("./map.svg")
      .then(response => response.text())
      .then(svg => {
        const container = document.querySelector(".svgContainer");
        container.innerHTML = svg;
      });
  });
</script>

<style>
  .st0 {
    fill-rule: evenodd;
    clip-rule: evenodd;
    fill: rgb(255, 255, 255);
    stroke: rgba(68, 68, 68, 0.5);
    stroke-width: 0.63;
    stroke-miterlimit: 10;
  }
  .svgContainer {
    width: 100%;
    height: 90vh;
  }
  .svgContainer :global(svg) {
    width: 100%;
    height: 100%;
  }
</style>

<svelte:options immutable={true} />

<div class="svgContainer">
  <small>Loading map 🗺</small>
</div>
