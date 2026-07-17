# Radar As A Primary Modality Perception: First-Principles Notes

This repo builds a first-principles picture of radar-only perception from a practical perspective: (1) develop a mathematical model of the radar with its assumptions explicit, so that methods can be evaluated and adapted to specialized cases, and (2) evaluate the tradeoffs and limitations involved in choosing between them.

The overall goal is to be able to assess the limits and tradeoffs of an approach in advance, separating methods that will work for a specific case from those that won't, when working in a nonstandard area less covered by existing domain methodology.

Specifically:

- The calibration note analyzes imperfect calibration due to a misplaced calibration target or the target's finite angular size. This establishes how precise the placement must be, and how small the target has to be, to achieve a given maximally acceptable residual calibration error. It also serves as an illustrative example of how a calibration procedure should be chosen to reach a desired precision, and why error analysis of the procedure itself matters.
- The deformable-objects note establishes the limitations of typical phase-correction procedures when detecting or imaging objects at high angle resolution, and hence the operating regime within which such a system works correctly.
- The ML-methods note analyzes common ML approaches on radar data against the same physical limits: the data requirements of each approach, the preprocessing it presumes, what it takes to generalize across environments, the errors induced by artifacts such as multipath and by the limited intrinsic precision of the raw data, and what training targets are reasonable given these fundamental limitations.
- The overall notation and general framework are established in the core definitions note.

At the same time, the notes are self-contained. Nothing beyond a general technical background (complex numbers, Fourier transforms) is assumed, so they can equally be used by someone new to radar to develop an understanding of radar imaging from the ground up. A short primer and glossary for such readers is at the [end of this README](#a-primer-for-readers-new-to-radar).

## Motivation

Radar as the sole perception modality is an unusual constraint because of the precision available. High sensitivity and relatively high measurement uncertainty are characteristic of radar: a lidar emits narrow laser beams in every direction and reconstructs an object as a high-resolution point cloud, with each point measured directly by time of flight; a radar instead illuminates the whole scene with a millimeter-wavelength wave and receives the superposition of everything that reflects back. Spatial information is recovered only indirectly, from phase relations across antennas and across time, which makes the measurement sensitive to channel calibration, motion, and multipath. Engineering and regulatory limitations of radar design also impose typically quite high measurement uncertainty.

For most perception tasks, radar as the primary modality is therefore not justified. In typical perception pipelines radar serves as a detector providing distance, velocity, and direction of moving targets, as a supplementary modality to cameras and lidar, and a safety layer for bad weather conditions.

It becomes the primary sensor when the target is hidden behind material, since radar is the only perception sensor that sees through it. In such tasks the sensor's limitations cannot be compensated by fusion with cameras or lidar and have to be handled within the radar processing itself.

The other established case where radar is the primary modality for high-resolution imaging is satellite and airborne SAR. The methodology there is well developed, but it addresses a specific geometry: essentially a single moving antenna, with techniques that rely on single-path precision correction, the trajectory of the moving platform recovered to wavelength precision relative to an approximately static scene. These methods rely on there being a single motion to correct: one platform trajectory against a static scene. For multiple moving targets, especially deformable ones in a cluttered environment, there is no single trajectory whose correction restores coherence for the whole scene, and the methods do not transfer directly.

First-principles analysis is therefore needed to assess which of the established methods transfer to a different problem structure, to what extent, and with what expected errors. Two constraints have to be accounted for from the start:

- **Classically**, high angle resolution on a moving target requires coherent integration over a long effective aperture, which requires knowing the target trajectory to within a fraction of the wavelength. The achievable precision of phase-based trajectory correction therefore sets the ceiling on resolution and coherence; this ceiling is what the deformable-objects note derives.
- **In ML approaches**, the physics induces data limitatons, and therefore fundamental tradeoffs and expectations on performance of learning approaches. The structure of radar data (coherent phase, multipath ghosts, the limited intrinsic precision of each measurement) induces the behavior of learned methods: which targets are learnable at all and reasonable under data available, how much data of what quality, variation, and preprocessing precision each target requires, and which errors are irreducible because they enter with the input. The choice of a learned target is therefore a tradeoff between its potential performance and its requirements on data quantity, quality, environmental variation, and acceptable error, and this tradeoff can be estimated from the physics before any dataset is collected.

The first-principles description in these notes serves to approach the problem at this level: derive the standard machinery (range/Doppler/angle resolution, calibration, phase-based trajectory correction) in a form where the error terms are explicit, so that potential approaches can be assessed against them and their expected errors evaluated.

## The notes

Suggested reading order:

1. **[`01_core_definitions.md`](01_core_definitions.md)**: the foundation. Derives the received FMCW signal from scratch: why the linearly increasing frequency and the hardware mixing make range compression a single Fourier transform (an interference of quadratic phases), how backprojection over the radar trajectory reconstructs the image and when it fails to (the linear-trajectory circle degeneracy), and how the same trick resolves Doppler velocity over slow time and angle over antenna positions. Establishes the signal model and the notation ($\tau$, $\Phi(\tau)$, $\alpha$, $f_0$, $\vec u$, $\vec d$) used in the other notes.

2. **[`02_calibration_error_budget.md`](02_calibration_error_budget.md)**: an error analysis of imperfect calibration. Real calibration is done against a physical target, and the target is never placed perfectly nor is it an ideal point: the note quantifies the residual calibration error caused by target misplacement and geometry uncertainty, working out how precise the placement must be, and, by the same analysis, how small (angularly) the target must be, to stay under a given maximally acceptable residual error. For scale: at 77 GHz with a 5 cm aperture, keeping the residual channel phase under 10 degrees requires pointing the calibration target to about 0.06 degrees, i.e. millimeter placement at 3 m. It also shows how the choice of procedure affects the achievable precision (relative-to-reference-channel calibration versus absolute).

3. **[`03_deformable_objects_perception_and_corrections.md`](03_deformable_objects_perception_and_corrections.md)**: derives the phase-based fine trajectory correction (autofocus) used to restore coherence in backprojection imaging, and its limitations. A coarse trajectory from range measurements is accurate to centimeters while coherent imaging needs a fraction of the wavelength; the note gives the pipeline that closes this gap using the signal's own phase (coarse compensation, inter-frame phase differencing, smooth model fit, integration). The scalar-vs-vector analysis establishes when a single per-frame range correction is sufficient and when the full vector correction is required, i.e. the regime of target size, angular extent, and residual error within which high-angle-resolution imaging of a moving object remains coherent.

4. **[`04_common_ML_methods_tradeoffs.md`](04_common_ML_methods_tradeoffs.md)**: analyzes common ML approaches on radar data against the physical limits derived in the previous notes. Covers the data requirements of each class of approach and the preprocessing it presumes; what is required for generalization across environments; the errors induced by artifacts such as multipath and by the limited intrinsic precision of the raw data, which are inherited by whatever model is trained on it; and, given these fundamental limitations, what training targets are reasonable to pursue in the first place.

## A primer for readers new to radar

A radar of the kind considered here transmits a *chirp*: a short pulse whose frequency increases linearly in time. The pulse travels to a target, reflects, and comes back. Everything the radar knows, it learns from comparing the returned wave with the transmitted one, in particular from the *phase*, i.e. how much the wave was delayed.

A few ideas carry all the notes:

- **Target's distance, velocity, angle and reconstructable image information is encoded in phase and delay.** A target at distance $R$ delays the echo by $\tau = 2R/c$ (the factor 2 is the round trip). For a carrier frequency $f_0$ with wavelength $\lambda = c/f_0$, this shows up as a phase $\frac{4\pi}{\lambda}R$. At automotive-radar frequencies (~77 GHz), $\lambda \approx 4$ mm, so a *millimeter* of extra distance visibly rotates the phase. This extreme sensitivity is both the superpower (fine measurements) and the curse (tiny setup errors ruin everything) that these notes revolve around.
- **Each measurement dimension is resolved by a Fourier transform.** The radar collects a complex signal indexed by several "axes": time within one chirp (*fast time*), the time across repeated chirps (*slow time*), and which antenna pair received it. A Fourier transform along fast time separates targets by *range*; along slow time, by *radial velocity* (Doppler); across antennas, by *angle*. In the ideal continuous case each transform produces a delta function that picks out one range/velocity/angle; with real, finite, discrete data the delta becomes a $\sin x / x$ peak of finite width, which is exactly what "resolution" means.
- **Hardware (Virtual antennas).** With multiple transmitters and receivers (MIMO), each Tx-Rx pair behaves like a single antenna sitting at the midpoint between the two. A handful of physical antennas thus emulates a larger "virtual array", and the phase differences across it encode the target's angle.
- **Coherence.** Many of these tricks work by *adding complex signals so their phases align* (constructive interference). If phases are off by order $\pi$, because the hardware channels are miscalibrated, or because the target trajectory was mis-estimated by a few millimeters, contributions cancel instead of adding, and the measurement or image falls apart. Two of the notes are about exactly this failure mode.

### Glossary

| Term | Meaning |
|---|---|
| Chirp | One pulse with linearly increasing frequency; the radar's basic transmission |
| Fast time $t$ | Time within a single chirp, over which the scene is effectively frozen; its Fourier transform gives range |
| Slow time $\Delta t$ | Time across repeated chirps, over which the geometry changes; its Fourier transform gives Doppler velocity |
| Mixing (dechirp) | Hardware multiplication of the received signal by the conjugate of the transmitted one |
| Beat frequency $f_b = \alpha\tau$ | Post-mixing constant frequency of a return from delay $\tau$; proportional to range |
| Range compression | Fourier transform over fast time that separates targets by distance |
| Doppler | Frequency shift proportional to the target's radial velocity $v_r$ |
| Virtual antenna $\vec d$ | Effective antenna at the midpoint of a Tx-Rx pair; the set of them forms the virtual array |
| Boresight | The direction the antenna array faces ($\theta = 0$) |
| $\vec u$ | Unit vector along the line of sight to the target |
| Channel gain $g_k$ | Unknown per-channel complex (amplitude + phase) hardware error, removed by calibration |
| Multipath | Echoes arriving via indirect bounce paths, producing ghost targets |
| Backprojection | Imaging by integrating the signal over the trajectory with the geometric phases compensated |
| Autofocus / fine correction | Estimating residual trajectory error from the signal phase itself |
| $\frac{4\pi}{\lambda}R$ | The two-way phase of a return from distance $R$; the quantity everything is measured through |

### Conventions

All notes share the same notation: phases written as $\exp(-j\cdot)$ for received signals, vectors with arrows ($\vec x$), dot products as $\vec u^{\top}\vec d$, two-way path phase $\frac{4\pi}{\lambda} = 2\pi f_0 \cdot \frac{2}{c}$, chirp slope $\alpha$ and start frequency $f_0$. Overall amplitude constants don't matter for the algorithms and are dropped without comment: imaging restores relative amplitudes and phases, not absolute values.
