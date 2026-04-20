# Stereo4D Viz — 100 Random Good Clips

Interactive viewer for a 100-clip sample of Stereo4D clips that survived the
strict motion filter (`max_jump < 0.3 m` on 3D tracks) and Louvain community
clustering capped at ≤ 3 object clusters per clip. Each clip shows:

- **Left**: left-rectified 512×512 RGB with per-cluster 2D track overlays.
- **Right**: Three.js 3D viewer of the world-frame trajectories coloured by
  cluster, with the camera frustum + trail.
- **Caption**: short DAVIS-style description produced by Molmo2-8B.

All per-clip data lives in a small compressed `.npz` that is parsed directly in
the browser (JSZip + a small inline `.npy` parser, no server).

### Files

```
index.html       self-contained viewer
manifest.json    clip list (id, caption, n_objects, n_tracks, ...)
clips/*.npz      per-clip bundle: pts3d, vis3d, tracks2d, vis2d, cam_pos,
                 cam_fwd, object_ids, n_objects, net_disp
videos/*.mp4     left-rectified 512x512 H.264 source videos
```

### Navigation

- `← Prev` / `Next →` or `[` / `]` keys, URL hash is the clip id (shareable).
- **Random** button to jump.
- **Spacebar** play/pause the video. `WASD` + `QE` / arrow keys pan the 3D cam.

### Dataset lineage

Source: [Stereo4D](https://stereo4d.github.io/) (Jin et al., CVPR 2025).
Starting from the 89,964 clips of the public release, the full pipeline
(filter → cluster → caption → reproject → origin-shift) runs as 40 Beaker jobs
using Molmo2-8B. Of those, ~46k survive the `net_disp > 1.5 m` gate; a second
strict `max_jump < 0.3 m` pass keeps ~52 % of those. This repo hosts 100
random GOOD clips from that set.
