<script>
    import favicon from '$lib/assets/favicon.svg';
    import distance from '@turf/distance';
    import { point } from '@turf/helpers';
    import { goto } from '$app/navigation';
    import { onMount } from 'svelte';
    import { PUBLIC_ANALYTICS_ID, PUBLIC_API_URL, PUBLIC_SECRET_KEY } from '$env/static/public';

    let { children } = $props();

    let isUnlocked = false;

    let userLon = 0, userLat = 0, poleLon = 0, poleLat = 0;

    const userPt = point([userLon, userLat]);
    const polePt = point([poleLon, poleLat]);

    const distanceInFeet = distance(userPt, polePt, { units: 'feet' });

    onMount(() => {
        // 1. Check if they already unlocked it earlier
        if (localStorage.getItem('hunt_unlocked') === 'true') {
            isUnlocked = true;
            return;
        }

        // 2. Look at the current URL for the ?key= parameter
        const urlParams = new URLSearchParams(window.location.search);
        const scannedKey = urlParams.get('key');

        // 3. If the key matches, unlock the game and save it to the browser
        if (scannedKey === PUBLIC_SECRET_KEY) {
            localStorage.setItem('hunt_unlocked', 'true');
            isUnlocked = true;

            // Clean up the URL so the key isn't sitting in the address bar
            window.history.replaceState({}, document.title, window.location.pathname);
            console.log("/map");
        }
    });
</script>

<svelte:head>
    <link rel="icon" href={favicon} />
</svelte:head>

{@render children()}