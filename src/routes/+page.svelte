<script>
    import { onDestroy, onMount } from 'svelte';
    import { Html5Qrcode } from 'html5-qrcode';
    import { goto } from '$app/navigation';

    let html5Qrcode;
    let isScanning = $state(false);
    const cameraId = "reader"; // The ID of our DOM element

    onMount(() => {
        html5Qrcode = new Html5Qrcode(cameraId);
    });

    async function startScanner() {
        try {
            isScanning = true;
            await html5Qrcode.start(
                { facingMode: "environment" }, // Forces the back camera
                {
                    fps: 10,                 // Frames per second to scan
                    qrbox: { width: 250, height: 250 } // On-screen targeting box
                },
                (decodedText) => {
                    // Triggered when a QR code is successfully read
                    stopScanner();
                    console.log("decode ",decodedText);
                    goto('/map', { replaceState: true });

                },
                (errorMessage) => {
                    // Verbose log for debug, can be ignored in production
                }
            );
        } catch (err) {
            console.error("Camera access failed:", err);
            isScanning = false;
        }
    }

    async function stopScanner() {
        if (html5Qrcode && html5Qrcode.isScanning) {
            await html5Qrcode.stop();
            isScanning = false;
        }
    }

    onDestroy(() => {
        // Clean up camera stream if user navigates away
        stopScanner();
    });
</script>

<div class="scanner-container">
  {#if !isScanning}
    <button onclick={startScanner} class="btn-scan">
      📷 Scan Booth QR Code
    </button>
  {:else}
    <button onclick={stopScanner} class="btn-stop">
      Cancel Scan
    </button>
  {/if}

  <div id={cameraId} class:active={isScanning}></div>
</div>

<style>
    #reader {
    width: 100%;
    max-width: 400px;
    margin: 10px auto;
    border-radius: 8px;
    overflow: hidden;
    display: none;
  }
  #reader.active {
    display: block;
    border: 2px solid #3b82f6;
  }
  .btn-scan {
    background-color: #3b82f6;
    color: white;
    padding: 12px 24px;
    border-radius: 6px;
    border: none;
    font-weight: bold;
    cursor: pointer;
  }
  .btn-stop {
    background-color: #ef4444;
    color: white;
    padding: 8px 16px;
    border-radius: 6px;
    border: none;
    cursor: pointer;
  }
</style>