# Array Calibration as an Error Budget

> Question answered by this note: given a maximum acceptable residual channel-phase error, how accurately must a calibration target be placed, and how small must it be?

## Why calibration exists

Angle estimation reads direction out of phase differences across the virtual array (see the core definitions note). Real hardware corrupts each channel with an unknown complex gain from cable lengths, mixer phases, antenna patterns. To see exactly what calibration must fix, let us write where the channel measurement comes from.

The contribution of a single scatterer to the (range-compressed) signal is its reflectivity with the carrier phase of the two-way path:

$$S \approx \sigma\frac{1}{R_0^2}\exp\left(-j\frac{4\pi}{\lambda}R_0\right), \qquad \lambda = \frac{c}{f_0},$$

where $R_0 = \|\vec{R}_0\|$ and $\vec{R}_0$ is the line-of-sight vector from the array center to the scatterer. For the channel whose virtual element sits at offset $\vec{d}_k$ from the array center, the path is taken from the element rather than from the center, so to first order in the small offset

$$R_k = \|\vec{R}_0 + \vec{d}_k\| \approx R_0 + \vec{u}^{\top}\vec{d}_k, \qquad \vec{u} = \frac{\vec{R}_0}{R_0},$$

and the channel-dependent part of the phase is $\exp\left(-j\frac{4\pi}{\lambda}\vec{u}^{\top}\vec{d}_k\right)$. Collecting everything independent of the channel into a single factor $S_0$ (reflectivity, spreading, the common range phase), the ideal channel response is

$$S_k^{\text{ideal}} = S_0\exp\left(-j\frac{4\pi}{\lambda}\vec{u}^{\top}\vec{d}_k\right), \qquad S_0 = \sigma\frac{1}{R_0^2}e^{-j\frac{4\pi}{\lambda}R_0}.$$

Uncalibrated hardware multiplies each channel by its unknown complex gain, so the measured response is

$$S_k = g_k S_0 \exp\left(-j\frac{4\pi}{\lambda}\vec{u}^{\top}\vec{d}_k\right),$$

with $g_k \in \mathbb C$ the hardware error to be calibrated out. If the electronics make the error separable, then $g_k = g_{\text{tx}(k)} g_{\text{rx}(k)}$, but nothing below in derivations requires that.

Uncalibrated $g_k$ phases are indistinguishable from geometric phases: they masquerade as angle errors, sidelobes, and ghost targets. The point of calibration is to measure $g_k$ once against a known scene and divide it out of all subsequent data.

## The procedure

The calibration commonly happens by using a reference scatterer in a known position. In the simplest case we place a single strong (ideally point) scatterer at a known direction $\vec{u}_0$ and record $S_k$. 

1. With the signal that comes from calibration target we can recover the calibration correcting matrix $g_k$by multiplying the signal the conjugate of the known geometric phase:

$$a_k = S_k \exp\left(+j\frac{4\pi}{\lambda}\vec{u}_0^{\top}\vec{d}_k\right) = g_k S_0.$$

2. Then we normalize it to a reference element to get rid of the common factor $S_0$:

$$\hat g_k = \frac{a_k}{a_1} = \frac{g_k}{g_1}.$$

Correcting future data as $S_k^{\text{corr}} = S_k / \hat g_k$ then restores the ideal geometric phases up to a single global constant $g_1$, which is invisible to any algorithm that only uses relative phases, i.e., to all of them.

Note that normalizing without first stripping the geometry doesnt give us desired result and leaves the ratio contaminated by a direction-dependent term,

$$\frac{S_k}{S_1} = \frac{g_k}{g_1}\exp\left(-j\frac{4\pi}{\lambda}\vec{u}_0^{\top}(\vec{d}_k - \vec{d}_1)\right),$$

which vanishes only in the special case where the target sits exactly on boresight and the array lies in the plane perpendicular to the line of sight. Even small variation from this special case from unprecise placement gives an additional error that lead to a calibration bias.

## Propagating setup errors into residual phase

The important part of the procedure and the analysis is to understnad how big an error of calibration is produced from different imperfectoins, e.g. how precisely we should place a target, measure position and how close to point-like object the scatterer should be.

To analyse this, lets perturb the direction $\vec{u}_0$ and the element positions $\vec{d}_k$ by $\Delta\vec{u}$ and $\Delta\vec{d}_k$ and expand to first order. The geometric phase applied in step 1 is then wrong by

$$\delta\phi_k = \frac{4\pi}{\lambda}\left(\vec{u}_0^{\top}\Delta\vec{d}_k + \Delta\vec{u}^{\top}\vec{d}_k\right),$$

and after normalization the residual error on element $k$ relative to the reference is

$$\delta\phi_k^{\text{rel}} = \frac{4\pi}{\lambda}\left(\vec{u}_0^{\top}\Delta\vec{b}_k + \Delta\vec{u}^{\top}\vec{b}_k\right), \qquad \vec{b}_k = \vec{d}_k - \vec{d}_1.$$

Element-position errors $\Delta\vec{b}_k$ are a manufacturing question; the setup-dependent term is the second one. If the dominant setup error is a mispointing $\delta\theta$ of the target direction along the array axis, and $b$ is the baseline from the reference to the farthest element,

$$\big|\delta\phi^{\text{rel}}\big| \approx \frac{4\pi}{\lambda} b \sin(\delta\theta).$$

## Inverting the budget to estimate required placement precision and maximal scatterer size 

The valuable direction from this formula is to estimate the required setup precision from fixed residual phase we can tolerate:

$$\delta\theta \lesssim \frac{\lambda\delta\phi_{\max}}{4\pi b}.$$

At 77 GHz, $\lambda \approx 3.9$ mm and $\frac{4\pi}{\lambda} \approx 3.2\times 10^{3}$ rad/m. For a virtual aperture of $b = 5$ cm:

| Tolerated residual $\delta\phi_{\max}$ | Required pointing accuracy $\delta\theta$ |
|---|---|
| $30^\circ$ (coarse beamforming) | $\lesssim 0.19^\circ$ |
| $10^\circ$ (clean sidelobes) | $\lesssim 0.06^\circ$ |
| $3^\circ$ (coherent imaging) | $\lesssim 0.02^\circ$ |

A pointing accuracy of $0.02^\circ$ at a target 3 m away corresponds to knowing the target's transverse position to about **1 mm**. Therefore, the central practical fact is that at mmWave, to "put the corner reflector roughly at boresight" is not a correct calibration procedure because of an unaceptable error it would generate. An estimate provides practical requirement of a placement precision and of en experimental calibratoin procedure.

**Finite target size enters the same budget.** A calibration target of transverse extent $w$ at range $R_t$ has no single direction: its effective phase center is uncertain within an angle $\sim w/(2R_t)$, which plays exactly the role of $\delta\theta$ above. The same row of the table that dictates pointing accuracy therefore also dictates the maximum target size (or minimum standoff range): for the $10^\circ$ budget at 3 m, $w \lesssim 6$ mm, which generate thee requirement on point-like scatterer error and distance of scatterer.

**Why not calibrate absolutely?** One might try to skip step 2 and use the known target range to predict $S_0$. The same $\frac{4\pi}{\lambda}$ factor makes this hopeless: a 1 cm error in the assumed line-of-sight distance produces

$$\delta\phi \approx 3.2\times 10^{3} \times 0.01 \approx 32\ \text{rad},$$

five full wraps of ambiguity. Absolute phase at mmWave is unmeasurable at any realistic setup tolerance, while relative channel ratios cancel the common range phase identically. 


Note that all the calculations should be considered as first order estimate only since in reality there are more complex calibration procedures that mitigate part of the error and give slightly loosen requirements on placement, size and distance. Hovewer, from this first analysis it is already seen how strict they are anyway, which mean before collecting data to any specific calibration procedure these maximally acceptable errors should be estimated. 