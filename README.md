# Stereo4D Viz — 100 Random GOOD Clips

Interactive viewer for a 100-clip sample of the Stereo4D dataset. Every clip
here survives **two** gates: a coarse motion filter
(`net_displacement > 1.5 m`) and a strict per-track coherence filter
(`max_jump < 0.3 m` between any two adjacent visible observations — catches
both single-frame tracker spikes and occlusion-gap teleports in one rule).

Each clip shows:

- **Left**: left-rectified 512×512 RGB with per-cluster 2D track overlays
- **Right**: Three.js 3D viewer of the world-frame trajectories coloured by
  cluster, with the camera frustum + trail
- **Caption**: short DAVIS-style description produced by `allenai/Molmo2-8B`

All per-clip data lives in a small `.npz` parsed directly in the browser
(JSZip + a 40-line inline `.npy` parser — no backend).

### Files

```
index.html            self-contained viewer
manifest.json         clip list (id, caption, n_objects, n_tracks, split, ...)
clips/*.npz           per-clip bundle (pts3d, vis3d, tracks2d, vis2d,
                      cam_pos, cam_fwd, object_ids, n_objects, net_disp)
videos/*.mp4          left-rectified 512×512 H.264 sources
```

### Navigation

- `← Prev` / `Next →` or `[` / `]`
- URL hash is the clip id (shareable deep-link)
- **Random** button to jump
- **Spacebar** play/pause; `WASD` + `QE` / arrows pan the 3D cam

### Clip gate

1. **Coarse**: `net_displacement > 1.5 m` → keep up to top 2000 by `net_disp`.
2. **Strict**: `max_jump < 0.3 m` over every adjacent visible-observation pair
   of each track. Clip must retain ≥ 10 tracks after this pass.
3. **Cluster**: Louvain community detection on velocity-cosine affinity, then
   greedy post-merge so each clip has **≤ 3 object clusters**.

~52 % of bundled clips pass (2), ~51 % of the raw release passes (1). This
repo hosts 100 random picks among the GOOD clips (survive both gates).

### Dataset lineage

- Source: [Stereo4D](https://stereo4d.github.io/) (Jin et al., CVPR 2025), 89,964 clips.
- Captions: Molmo2-8B, short DAVIS-style one-sentence description.
- Processing on weka at `/weka/prior-default/jianingz/home/dataset/stereo4d/processed/`; see the `processed/README.md` there for the full data format and regeneration instructions.
- Pipeline executed as 40 Beaker jobs (Molmo2 captioning + reprojection), total ~2 GPU-hours across the whole dataset.

### Regenerate the sample

On a weka machine:

```bash
python /weka/prior-default/jianingz/home/dataset/stereo4d/viz_website/strict_filter_and_pick.py \
    --n_good 100 --n_bad 0 --seed <N> --max_candidates 500
# copy bundles in, commit & push
cp /weka/.../viz_website/static/data/clips/*.npz /path/to/stereo4d-viz-public/clips/
git add -A && git commit -m "new sample" && git push
```
