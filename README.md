# Asahi Linux (fairydust with DP/HDMI Audio & 120Hz Fixes)

This fork contains fixes for Apple Silicon (tested on **Mac mini M1 / `j274`**) to enable stable **4K @ 120Hz video and DP/HDMI audio** over USB-C (ATC PHYs) and the built-in HDMI port.

## Problems Solved

Upstream `appledrm.hdmi_audio=1` was experimental and suffered from several issues across standby, hotplug, and modesets:

1. **AFK Slot Leak:** `av_audiosrv_teardown()` did not clear `service->enabled = false`, exhausting the 16 AFK endpoint channel slots after repeated power cycles.
2. **WirePlumber Probe Race / Standby Profile Loss:** `dcp_pcm_open()` returned `-ENXIO` when the screen was disconnected/sleeping, causing WirePlumber to permanently drop the stereo profile on boot or restart.
3. **Standby Reconnect Recovery:** When the display woke up, the stream stalled because `open_cookie` was out of sync. Replaced `SNDRV_PCM_STATE_DISCONNECTED` with `snd_pcm_stop_xrun()` and synced cookies on `prepare()`.
4. **SIO DMA Descriptors & Period Limits:** Fixed descriptor bitmap size (`devm_bitmap_zalloc`) and capped ALSA `periods_max` at 32 to prevent `-ENOMEM` loops under PipeWire.
5. **Modeset Order:** Audio service initialization is deferred until after `iomfb_modeset()` to prevent display pipeline failures on ATC PHYs.

## Branches

* **`fairydust`**: Rebased on latest Asahi upstream (`asahi-7.1.12-1` / kernel 7.1.12).
* **`fairydust-7.1.6`**: Stable tree matching Fedora Asahi Remix 7.1.6 kernel.

## Building the Kernel Module

To compile just the `appledrm` module:

```bash
make -j$(nproc) M=drivers/gpu/drm/apple
```

## Patches

All patches and cover letters are also exported as git format-patches in `asahi-patches/`.
