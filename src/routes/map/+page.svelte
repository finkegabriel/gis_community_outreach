<script>
    import { onMount } from 'svelte';
    // ⚠️ CRITICAL: Leaflet will look completely broken without its CSS!
    import 'leaflet/dist/leaflet.css';

    let map;
    let userMarker;
    let watchId;
    /** @type {typeof import('leaflet')} */
    let L;

    // Your existing state variables
    let userLat = $state(0);
    let userLon = $state(0);

    onMount(() => {
        let cancelled = false;

        // Leaflet touches `window` on import, so it must only load in the browser
        (async () => {
            L = (await import('leaflet')).default;
            if (cancelled) return;

            // 1. Initialize the map (Starting at Mesa, AZ coordinates)
            map = L.map('treasure-map').setView([33.415, -111.831], 16);

            // Add the styled map tiles (Standard OSM used here, can swap for styled ones later)
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                maxZoom: 19,
                attribution: '© OpenStreetMap'
            }).addTo(map);

            // 2. Create the "Discovery Compass" marker for the user
            const compassIcon = L.divIcon({
                className: 'custom-user-icon',
                html: '<div style="font-size: 24px; text-align: center;">🧭</div>',
                iconSize: [30, 30],
                iconAnchor: [15, 15] // Centers the icon perfectly over the coord
            });

            userMarker = L.marker([33.415, -111.831], { icon: compassIcon }).addTo(map);

            // 3. Start watching the GPS hardware
            if ('geolocation' in navigator) {
                watchId = navigator.geolocation.watchPosition(
                    (position) => {
                        // Update our Svelte state with fresh GPS data
                        userLat = position.coords.latitude;
                        userLon = position.coords.longitude;
                    },
                    (error) => {
                        console.error("GPS Error:", error.message);
                    },
                    {
                        enableHighAccuracy: true, // Forces GPS chip usage, vital for a 50ft radius hunt!
                        maximumAge: 0,            // Don't use cached locations
                        timeout: 5000
                    }
                );
            }
        })();

        // Cleanup when the user leaves the page
        return () => {
            cancelled = true;
            if (watchId) navigator.geolocation.clearWatch(watchId);
            if (map) map.remove();
        };
    });

    // 4. SVELTE 5 MAGIC: Reactively move the map
    // This block automatically runs whenever userLat or userLon changes!
    $effect(() => {
        if (map && userMarker && userLat !== 0 && userLon !== 0) {
            const newLocation = new L.LatLng(userLat, userLon);
            
            // Move the marker to the new GPS coordinate
            userMarker.setLatLng(newLocation);
            
            // Smoothly pan the camera to follow the user
            map.panTo(newLocation, { animate: true, duration: 1.0 });
        }
    });
</script>

<div id="treasure-map" style="height: 60vh; width: 100%; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);"></div>