# Auto-Boost-Essential

Auto-Boost-Essential is the latest iteration of the Auto-Boost formula, streamlined and refined for greater convenience, speeds and boosting gains!

This encoding script is intended to be paired with my [SVT-AV1-Essential](https://github.com/nekotrix/SVT-AV1-Essential) encoder fork.  
SVT-AV1-Essential sports *excellent* quality consistency, but Auto-Boost-Essential offers *exceptional* consistency!

**Here is how it works:** the script runs a first encoder fast-pass, finds scenes based on the introduced keyframes, calculate metrics scores, automatically adjusts the CRF of scenes in order to increase quality consistency and then runs a final-pass with these adjustements.  

The quality metrics at play this time again are **SSIMULACRA2** and **XPSNR**.

### Dependencies

Before using Auto-Boost-Essential, you will need to install the following dependencies:
- [SVT-AV1-Essential](https://github.com/nekotrix/SVT-AV1-Essential/releases) (a build with quarter-step CRF is mandatory with Auto-Boost-Essential v2.0+)
- Python 3.12.x or newer
- [Vship](https://codeberg.org/Line-fr/Vship/releases) (AMD/Nvidia GPU) and/or [vs-zip](https://github.com/dnjulek/vapoursynth-zip/releases/) (CPU)
- [Vapoursynth](https://github.com/vapoursynth/vapoursynth/releases) and vstools, part of [vsjetpack](https://pypi.org/project/vsjetpack/)
- [FFMS2](https://github.com/FFMS/ffms2/releases)

Lost users can refer to this [Vapoursynth install guide](https://jaded-encoding-thaumaturgy.github.io/JET-guide/master/basics/installation/) (up until "Installing the JET Packages"). If you are still lost, join the Discord server linked below and ask for help!

Arch users can run this single command to install everything:
```bash
yay -S svt-av1-essential-git vapoursynth-plugin-vsjetpack vapoursynth-plugin-vship-cuda-git vapoursynth-plugin-vszip-git ffms2
```
*Radeon users can simply replace `vapoursynth-plugin-vship-cuda-git` with `vapoursynth-plugin-vship-amd-git `.*

### Usage

Auto-Boost-Essential can be considered a helper script, as all you need to do is provide an input video file and it will manage everything for you:
```bash
python Auto-Boost-Essential.py -i "my_video_file.mp4"
```

Even though the above command is sufficient to run the script, one may use additional parameters to tweak the experience, for instance:
| Parameter | Usage |
|-----------|-------|
| `--version` | Print script version |
| `--verbose` | Enable more verbosity [*Default: not active*] |
| `--input` | Video input filepath (original source file) |
| `--temp` | The temporary directory for the script to store files in [*Default: video input filename*] |
| `--no-boosting` | Runs the script without boosting (final encode only) [*Default: not active*] |
| `--resume` | Resume the process from the last (un)completed stage [*Default: not active*] |
| `--stage` | Select stage: 1 = fast encode, 2 = calculate metrics, 3 = generate zones, 4 = final encode [*Default: all*] |
| `--ssimu2` | Select the SSIMULACRA2 metric model: vs-zip (CPU), Vship (GPU) or auto [*Default: XPSNR metric*] |
| `--target-ssimu2` | Target final average SSIMULACRA2 score; automatically adjusts base CRF from fast-pass metrics |
| `--target-ssimu2-p15` | Minimum 15th percentile SSIMULACRA2 floor [*Default: target-ssimu2 - 8.0*] |
| `--fast-speed` | Fast encode speed (Allowed: medium, fast, faster) [*Default: faster*] |
| `--final-speed` | Final encode speed (Allowed: slower, slow, medium, fast, faster) [*Default: slow*] |
| `--quality` | Base encoder --quality (Allowed: low, medium, high) [*Default: medium*] |
| `--aggressive` | More aggressive boosting [*Default: not active*] |
| `--unshackle` | Less restrictive boosting [*Default: not active*] |
| `--fast-params` | Custom fast encoding parameters |
| `--final-params` | Custom final encoding parameters |

Yes, the script is even capable of resuming unfinished encodes like Av1an, and can also be run with boosting disabled if all you care is the convenience of a SVT-AV1-Essential wrapper.

### Fork Enhancements (`ednancosta`)
- **Auto-CRF Target Discovery (`--target-ssimu2` / `--target-ssimu2-p15`):** Estimates final SSIMULACRA2 from 100% of the fast-pass (`metric_average` and 15th percentile floor) and shifts the base CRF automatically using an asymmetric slope (`1.5` down to protect transparency, `1.25` up to save size; clamped `[-6.0, +8.0]` in `0.25` steps) before computing per-scene `--zones`, delivering `ab-av1`-style target consistency with zero extra encode passes.
- **Fast-Pass Metrics Export:** Writes `fast_metrics.txt` (`avg,p15,min,effective_crf`) in the temp directory for calibration logging without extra compute cost.
- **Visible Encoder Errors:** Captures and prints the last 25 lines of `SvtAv1EncApp` `stderr` on failures instead of silencing output.
- **Numeric `--crf` Parsing & Propagation:** Uses regex parsing for `--crf` in `--fast-params` and propagates numeric CRF into `final_pass()` when not overridden in `--final-params`.
- **Zero-Copy `YUV420P10` & `vstools` Compat:** Bypasses redundant bilinear conversion on native 10-bit 4:2:0 sources and uses modern `src.set_output()`.

### Known-issue

The encoding process can hang in the last dozen of frames for certain sources. It can be caused by some specific combination of parameters or by the input file being malformed.  
The recommended procedure as of now is: to first backup the ivf file, and to then try to resume with a limited set of parameters. When the issue is caused by the source being malformed, it is possible the encode is complete but the process isn't, in which case simply interrupt the process by hand and manually copy the ivf file wherever you want.  
Proper workarounds are actively being researched.

### Contribute

The script is open to contributions!  
Namely, I'm looking to improve the boosting logic to increase consistency further, with little to no additional performance implications.  
Code refactors for clean-up and fixes are also appreciated!

### Future

Apart from said boosting logic refactor, the following features are being considered:
- Re-introducing metrics skipping for greater speeds
- ~~Comparing SSIMULACRA2 to XPSNR (because vs-zip's implementation is crazy fast and universal anyway due to running on CPU)~~
- More robust error handling
- Making the CLI better looking

## Benchmarks:

|                    Metrics                    |                    Speed                    |
|-----------------------------------------------|---------------------------------------------|
| ![Metrics](https://i.kek.sh/2Ulmd7e7zIJ.webp) | ![Speed](https://i.kek.sh/0fehFRVGuhT.webp) |
| ![Metrics](https://i.kek.sh/WckNMr7IzRa.webp) | ![Speed](https://i.kek.sh/NHYJEEeJrhB.webp) |  

*Speed may vary depending on your hardware configuration and source resolution.*

The above results are not even best-case scenarios. The selected samples are very complex. More gains are expected on your average clip, granted it contains more than one scene at a minimum!


*Join us over at [AV1 weeb edition](https://discord.gg/83dRFDFDp7)!*



