<script>
    // 1. Accept the child page as a property
    let { children } = $props();
</script>

<main class="map-layout-wrapper">
    {@render children()}
</main>