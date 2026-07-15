<script>
    import { onMount } from 'svelte';
    import 'leaflet/dist/leaflet.css';

    // 1. CHOSEN FIX: Make the map handles reactive state variables
    let map = $state(null);
    let userMarker = $state(null);
    let watchId;
    
    /** @type {typeof import('leaflet')} */
    let L;

    let userLat = $state(0);
    let userLon = $state(0);
    let gpsStatus = $state('Waiting for GPS…');

    onMount(() => {
        let cancelled = false;

        (async () => {
            L = (await import('leaflet')).default;
            if (cancelled) return;

            // Initialize map
            map = L.map('treasure-map').setView([33.415, -111.831], 16);

            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                maxZoom: 19,
                attribution: '© OpenStreetMap'
            }).addTo(map);

            const compassIcon = L.divIcon({
                className: 'custom-user-icon',
                html: '<div style="font-size: 24px; text-align: center;">🧭</div>',
                iconSize: [30, 30],
                iconAnchor: [15, 15]
            });

            userMarker = L.marker([33.415, -111.831], { icon: compassIcon }).addTo(map);

            // Start watching GPS
            if ('geolocation' in navigator) {
                watchId = navigator.geolocation.watchPosition(
                    (position) => {
                        userLat = position.coords.latitude;
                        userLon = position.coords.longitude;
                        gpsStatus = `Locked (±${Math.round(position.coords.accuracy)}m)`;
                    },
                    (error) => {
                        console.error("GPS Error:", error.code, error.message);
                        gpsStatus = `GPS error: ${error.message}`;
                    },
                    {
                        enableHighAccuracy: true,
                        maximumAge: 10000,
                        timeout: 15000
                    }
                );
            } else {
                gpsStatus = 'Geolocation not supported on this device.';
            }
        })();

        return () => {
            cancelled = true;
            if (watchId) navigator.geolocation.clearWatch(watchId);
            if (map) map.remove();
        };
    });

    // 2. This effect now reliably tracks map, userMarker, userLat, and userLon!
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
<p style="font-size: 0.85em; color: #555; margin-top: 6px;">{gpsStatus}</p>