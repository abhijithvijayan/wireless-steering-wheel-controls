# Local patches

## kicadStepUpMod-honor-model-scale.patch

KiCad lets a footprint carry a per-model `(scale (xyz sx sy sz))` on its 3D model.
The receiver board uses this on `ESP32-C3_SUPERMINI_TH.step` (1 1 0.8) and
`4051 v12.step` (1 0.65 1). Upstream KiCadStepUp (13.1.7) only warns
`wrong scale!!! set scale to (1 1 1)` and imports the model unscaled.

The patch makes StepUp honour the scale: a scaled footprint instance is placed as an
`App::Link` whose `ScaleVector` carries the KiCad scale (applied in model coordinates,
before the model rotate/offset, same order as KiCad's 3D viewer and STEP export).
The unscaled source model is kept hidden in a `Scaled_Model_Sources_<sfx>` container
under `Step_Models`. Requires the StepUp preference **Links** (`asm3_links`) to be on,
which it is on this machine. Set `apply_model_scale = False` in `kicadStepUptools.py`
to get the old behaviour back.

Where the addon lives: `~/Library/Application Support/FreeCAD/Mod/kicadStepUpMod`
(git clone of https://github.com/easyw/kicadStepUpMod). The patch is committed there on
the local branch `honor-model-scale`; `master` tracks upstream untouched.

### Re-applying after an upstream update

```sh
cd ~/Library/Application\ Support/FreeCAD/Mod/kicadStepUpMod
git checkout master && git pull
git checkout honor-model-scale && git rebase master     # or, from scratch:
# git checkout -b honor-model-scale master && git am <repo>/patches/kicadStepUpMod-honor-model-scale.patch
```

If the rebase conflicts, the patch touches only `kicadStepUptools.py`: the module
defaults near `use_Links=False`, four helper functions before `Display_info`, five
hunks inside `Load_models`, and the three `wrong scale!!!` warning sites.

Do not let the FreeCAD Addon Manager "update" this addon while on the branch; it
expects `master`. Update with git as above instead.

### Upstream

Submitted as https://github.com/easyw/kicadStepUpMod/pull/280 from the fork
`abhijithvijayan/kicadStepUpMod`, branch `honor-model-scale` (uploaded via the GitHub web UI,
so the fork branch has the same content as the local branch but a different commit hash).
