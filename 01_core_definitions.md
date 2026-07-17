# Radar perception and imaging: Core & definitions

This is a concise theoretical introduction to radar imaging and the most important concepts.

## The measurement

An FMCW radar collects information by transmitting pulses that reflect from the environment and get back to the radar, carrying information about distances of the objects around.

The overall picture to have in mind is the following: pulses that come back to the radar contain reflection information from all the different objects around the radar; by selecting only the part of the signal that came at time $t = 2R/c$ from the start of the signal (the factor 2 is the round trip) we can select only the part of the information that belongs to objects at distance $R$ from the radar. To have imaging, however, we need to resolve each point on an object in 3D space (or 2D if the object is a surface), which means we need to resolve the signal inside of this part of the pulse that merged all parts of the surface at distance $R$ together. We can do that by moving the radar along some trajectory: then signals that were fused together at one part of the trajectory no longer belong to the same $R = R_0$ parts of the signal, but to different ones. Then, by carefully accounting for phases, we can resolve each point in the object (= perform the imaging). Let's formalize this intuition.

If a radar is trying to image a surface with reflectivity $\sigma(\vec x)$, where $\vec x$ is the position of a point on the surface, while the radar is in position $x_r$, at time $t$ it receives a signal

$$S(x_r, t) = \int d^2x \sigma(\vec{x})\frac{1}{R^2}\exp\left(-j2\pi\frac{2R}{c}f_0\right)p(t-\tau)$$

where $p(t-\tau)$ is the phase of the transmitted signal at the moment of emission $t-\tau$ of the part of the signal that traveled to the point and back, covering distance $2R$, $R = |\vec x - \vec x_r|$, in time $\tau = 2R/c$.

The engineering of the radar is commonly made to multiply this incoming signal by the conjugate of the outgoing one (which is done at the hardware level), $p^{*}(t)$, for convenience reasons we will see later, so the signal is

$$S(x_r, t) = \int d^2x \sigma(\vec{x})\frac{1}{R^2}\exp\left(-j2\pi\frac{2R}{c}f_0\right)p(t-\tau)p^{*}(t)$$

Another common engineering trick of the radar is to emit a pulse with linearly increasing frequency, from $f_0$ to $f_1$ at some constant rate $\alpha$ (the duration is therefore $T_c = (f_1-f_0)/\alpha$). This is needed for resolution of signals reflected from objects at different distances from the radar, as we will see later, so the phase has the form

$$p(t) = e^{j \cdot 2\pi \int(f_0 +\alpha t) dt}=  e^{j \cdot 2\pi (f_0 t + \tfrac{1}{2}\alpha t^2)}, \qquad 0 \le t < T_c.$$

so that the signal takes the form

$$S(x_r, t)
= \int d^2x \sigma(\vec{x})\frac{1}{R^2}\exp\left(-j2\pi\tau f_0 - j2\pi \alpha(\tau^2/2 - t\tau)\right)$$

Such an at first glance arbitrary choice of both linearly increasing frequency and multiplying the incoming signal by the outgoing one makes it very easy to resolve the signal that traveled from different distances $R$ by performing a simple Fourier transform:

$$S(x_r, f = \alpha\tau') = \int S(x_r,t)\exp\left(j2\pi \alpha\tau' t\right)dt$$ $$ = \int d^2x \sigma(\vec{x})\frac{1}{R^2}\exp\left(-j2\pi\tau f_0 - j2\pi \alpha\tau^2/2\right)\int \exp\left(j2\pi \alpha(\tau'-\tau)t\right)dt$$

$$= \alpha^{-1}\int d^2x \sigma(\vec{x})\frac{1}{R^2}\exp\left(-j2\pi\Phi(\tau)\right)\delta(\tau-\tau')$$

$$= \alpha^{-1}\int_{R = c\tau'/2} d^2x \sigma(\vec{x})\frac{1}{R^2}\exp\left(-j2\pi\Phi(\tau)\right)$$

where $\delta(\tau-\tau')$ is a sign that we sum only over the signal that comes from distance $R = c\tau'/2$, and we denoted

$$\Phi(\tau) = f_0\tau + \alpha\tau^2/2.$$

Note on the delta function: $\delta(\nu)$ is defined as the infinitely narrow peak concentrated at $\nu = 0$ and normalized by $\int\delta(\nu)d\nu = 1$, so that under an outer integral it keeps only the contributions with $\nu = 0$. The common expression for it, and the one behind every delta below, is $\int e^{j2\pi\nu t}dt = \delta(\nu)$: for $\nu \neq 0$ the integral of the rotating phase vanishes, as the known property of summing complex numbers $|z| = 1$ uniformly over the circle, so the only case in which the integral is not zero (and accumulates without bound) is $\nu = 0$. Every resolution in this note is a statement of this form: an integral of the signal against a phase, proportional to a delta function of the quantity being selected.

Note that this integration is a slightly simplified case of receiving a continuous signal during an extended period of time, while in reality the signal received by the radar is discretized and, instead of integration, summing over a finite amount of time of different signals is performed; the overall picture is the same, however, with two corrections: the finite duration turns the delta function into a $\sin x/x$ peak of finite width (this width *is* the range resolution, $\delta R = c/2B$ with $B = f_1 - f_0$ the swept band), and the discreteness of sampling makes distances beyond a maximum unambiguous range alias back into the picture.

For a reader carefully tracing all the derivations, you would notice the role the linearly increasing phase played in what happened. In the simplest scheme of a non-extended, instantaneous signal, we resolve the signal that comes from distance $R$ by selecting the signal that arrives at time $t = 2R/c$; however, this limits the maximum energy each point receives (at the hardware level there is a limit on the power a transmitter can emit instantaneously, and the received energy is power × duration) and in practice makes the signal from each point not bright enough for imaging manipulations. Changing the scheme from an instantaneous to a continuous-in-time signal means that at time $t$ we receive both the signal that was emitted at time $0$ and came from distance $R = tc/2$, and signals that came from shorter distances $R'$ but were emitted at some time $t' = t - 2R'/c > 0$ after the pulse start, so signals from different distances get mixed together in time.

The linearly increasing frequency is what resolves this mixing, and the picture is an interference of quadratic phases. Mixing interferes the arriving chirp with the conjugate of the one being emitted now: two identical parabolic phases shifted by $\tau$, and a parabola minus its shifted copy is exactly a line,

$$\frac{(t-\tau)^2}{2} - \frac{t^2}{2} = -\tau t + \frac{\tau^2}{2}.$$

For $\tau = 0$ the phase and its conjugate cancel identically; a shift leaves the linear residual $e^{-j2\pi\alpha\tau t}$, and a phase linear in time integrates to the famous zero, $\int e^{jat}dt = 0$: the phasor runs uniformly around the circle $|z|=1$ and averages out. The Fourier kernel above merely re-tilts the line, so at the trial delay $\tau' = \tau$ the phasor stands still and accumulates while at every other delay it keeps circling and vanishes: this standing versus circling is the delta function $\delta(\tau - \tau')$. Meanwhile the signal is long in time, so each point accumulates energy for the full $T_c$ instead of an instant, a gain of $BT_c$ over the instantaneous pulse.

Radar notation: multiplying the received signal by the conjugate of the emitted one, $p^{*}(t)$, is called mixing (or dechirping), $f_b = \alpha\tau$ is called the beat frequency, and $f_0$ is the carrier frequency.

Separate note: the quadratic residual $\alpha\tau^2/2$ (the *residual video phase*) is small for short ranges and is commonly ignored throughout; overall amplitude constants can also be dropped, since imaging means restoring relative amplitudes and phases across an imaged object $\sigma(\vec x)$, not absolute values.

# Image reconstruction (SAR)

Now the next part of the promised program: we resolved signals that come from different distances $R$, and what remains is to resolve a specific point $\vec x$ within the sphere $|\vec x - \vec x_r| = R$, using the fact that if we move the radar, points that were mixed together by lying on the same sphere of radius $R$ now belong to different spheres and get sorted into different $R'$ and $R''$. All these signals arrive with phases $\exp\left(-j\frac{4\pi}{\lambda}R(\vec x, \vec x_r)\right)$ (the carrier term of $\Phi(\tau)$ dominates, $\lambda = c/f_0$), and it is by these phases that we will do the sorting.

To formalize: in exact analogy with range compression, we integrate the range-compressed signal over the radar positions with the corresponding phases compensated, following for each hypothesized point $\vec x'$ its own range bin $\tau'(\vec x_r) = 2R(\vec x', \vec x_r)/c$ along the trajectory:

$$\hat\sigma(\vec x') = \int d\vec x_r S\left(\vec x_r,\ \tau'(\vec x_r)\right)\exp\left(+j2\pi\Phi(\tau'(\vec x_r))\right)$$

$$= \alpha^{-1}\int d^2x\sigma(\vec x)\frac{1}{R^2}\int d\vec x_r\exp\left(j2\pi\left[\Phi(\tau') - \Phi(\tau)\right]\right)$$

where the inner $\vec x_r$-integral runs over the positions at which the point $\vec x$ falls into the range cell of $\vec x'$ (for points in different cells the integrand simply vanishes, since they never appear in the followed bin). Within one range cell the difference of the full phases is dominated by the carrier term, $\Phi(\tau') - \Phi(\tau) \approx f_0(\tau' - \tau) = \frac{2}{\lambda}\left[R(\vec x', \vec x_r) - R(\vec x, \vec x_r)\right]$, so the inner integral is

$$\int d\vec x_r\exp\left(j\frac{4\pi}{\lambda}\left[R(\vec x', \vec x_r) - R(\vec x, \vec x_r)\right]\right) \propto \delta(\vec x - \vec x'),$$

in complete analogy with range compression (passing from the delta function of the phases to the delta function of the coordinates is the standard change of variables inside a delta function). Therefore $\hat\sigma(\vec x') \propto \sigma(\vec x')$: we resolved the specific point $\vec x$, as we wanted. This procedure is called **backprojection**; specific SAR algorithms are computational shortcuts for this integral.

Note: the statement that the integral selects only the one point $\vec x = \vec x'$ (where the phase vanishes at every position, while for other points the phases are in general random and interfere to zero) is correct only for a general enough trajectory: matching the whole function $R(\vec x, \vec x_r)$ along a curve wandering in space overdetermines $\vec x$. However, if even a part of the trajectory is linear, $\vec x_r = s\hat e$, other points survive the integral just as well as $\vec x'$ itself: a whole circle of them, because

$$R^2(\vec x, s) = |\vec x|^2 - 2s\hat e^{\top}\vec x + s^2$$

depends on $\vec x$ only through the pair $\left(\hat e^{\top}\vec x,\ |\vec x|\right)$, so all points of the circle formed by intersecting the range sphere with the plane through $\vec x'$ perpendicular to the trajectory have *identical* range histories along that stretch and are inseparable by any processing. Even in this case, though, when the imaged object is a surface, the selection of the whole circle reduces to one point: the surface crosses each such circle in two points, one on either side of the trajectory, so if the whole object lies on one side of the radar, the imaging is unambiguous, which is exactly why airborne SAR looks sideways rather than straight down.

Note also that, as before, the integral over the trajectory is in reality a sum over its finite, discretely sampled length $L$, which turns the delta into a $\sin x/x$ peak of finite width, set by the inverse extent of the integration domain: this is the cross-range resolution

$$\delta_{\perp} \sim \frac{\lambda}{2\Delta\theta}, \qquad \Delta\theta = \frac{L}{R_0},$$

with $\Delta\theta$ the angle the trajectory subtends as seen from the target: the trajectory acts as a lens whose aperture is its angular extent.

One cost is hidden in the coherence of the sum: the compensating phase must be right to a fraction of $\pi$, i.e. $R(\vec x, \vec x_r)$ (the radar's trajectory and any motion of the target) must be known to a fraction of the wavelength, millimeters at mmWave; known only to centimeters, the terms arrive with random phases and the image does not blur but disappears. Closing this precision gap from the signal itself is the subject of the fine-trajectory-correction note.

The two sections below are the two most important special cases of this machinery: the cases where the range history $R(\vec x, x_r)$ is *linear* in the parameter being scanned (slow time, or antenna position), so the matched-filter sum for all hypotheses at once collapses into a single Fourier transform.

# Doppler and angle resolution

## Doppler (radial velocity) resolution

Note that from the received signal we can also reconstruct the velocity of the target in the direction of $\vec R$, using the same trick: repeat the pulse over some period of time and resolve over the repetition time $\Delta t$. If the target (or radar) travels little during this time ($v_r \Delta t \ll R$, so every point still corresponds to the same $R$), the only change between repetitions is the carrier phase of the accumulated range shift $\delta R = v_r\Delta t$:

$$S(\Delta t, \tau) \approx e^{-j2\pi\Phi(\tau_0)}\exp\left(-j\frac{4\pi}{\lambda} v_r\Delta t\right)\int_{R = R_0} d^2x\frac{\sigma(\vec x)}{R_0^2},$$

a phase once again linear in the scanned variable, so the Fourier transform over the repetition time selects the radial velocity:

$$S(f_d', \tau) = \int d(\Delta t) S(\Delta t, \tau)e^{j2\pi f_d'\Delta t} \propto \delta\left(f_d' - \frac{2 v_r}{\lambda}\right),$$

separating the signal that comes from a specific distance *and* a specific radial velocity. Again, the real repetitions are finite in number and discrete in time, producing the $\sin x/x$ peak of width $\delta v_r = \frac{\lambda}{2 T_{\text{obs}}}$ with $T_{\text{obs}}$ the total observation time: the longer we watch, the finer the measurement, and, being a carrier-phase measurement, it improves with shorter wavelength, unlike range resolution which cares only about bandwidth.

Radar notation: the repeated pulses are the same chirps, repeated at a fixed interval. The time $\Delta t$ across the repetitions, over which the geometry changes (the distances $R$, the positions of radar and target), is called *slow time*, as opposed to the *fast time* $t$ inside one pulse, over which the scene is effectively frozen and only the waveform evolves. The radial velocity is called *Doppler* velocity and the corresponding frequency $f_d = \frac{2v_r}{\lambda}$ the *Doppler frequency*.

## Angle resolution

The same trick applies one more time: we can also resolve the direction to the target, this time using, instead of repetitions in time, several antennas at different positions, each transmitting and receiving pulses in every combination, so that every (transmitter, receiver) pair is a separate measurement channel. For a target far away in direction $\vec u$ (a unit vector), moving an antenna by $\vec d$ from the array center changes the one-way path by $\vec u^{\top}\vec d$; since the deviation is accumulated once on the way out (transmitter position) and once on the way back (receiver position), the pair together picks up the extra two-way path

$$\vec u^{\top}\vec d_{\text{tx}} + \vec u^{\top}\vec d_{\text{rx}} = 2\vec u^{\top}\vec d, \qquad \vec d = \frac{\vec d_{\text{tx}} + \vec d_{\text{rx}}}{2}.$$

So the pair behaves exactly as a single antenna sitting at the midpoint $\vec d$, and the pairs together emulate an array of such elements, larger than anything physically built. Each element at $\vec d$ carries the channel phase $\exp\left(-j\frac{4\pi}{\lambda}\vec u^{\top}\vec d\right)$.

For a linear array along the $x$ axis, $\vec u^{\top}\vec d = d_x \sin\theta$ with $\theta$ the target direction measured from the normal of the array: once again a phase *linear* in the scanned coordinate $d_x$, with slope set by $\sin\theta$. Denoting $u = \sin\theta$, the signal from a given distance and radial velocity across the elements is

$$S(d_x, \tau) \approx e^{-j2\pi\Phi(\tau_0)}\exp\left(-j\frac{4\pi}{\lambda} d_x u\right)\int_{R = R_0,\ v_r} d^2x\frac{\sigma(\vec{x})}{R_0^2},$$

and the Fourier transform over the element positions separates the directions:

$$S(u', \tau) = \int dd_x S(d_x, \tau) e^{j\frac{4\pi}{\lambda} d_x u'} \propto \delta\left(\tfrac{2}{\lambda}\left(u' - u\right)\right),$$

sorting targets that share both distance and radial velocity by their direction $\theta$. As with fast and slow time, the finite extent $L$ of the array turns the delta function into a $\sin x/x$ beam of finite width $\delta u = \frac{\lambda}{2L}$ in the variable $u = \sin\theta$.

If the elements also have $y$-offsets, the corresponding part of the phase, $\frac{4\pi}{\lambda} d_y \sin\theta_y$, is untouched by the transform over $d_x$: it must either be compensated explicitly by summing with the correct phases for an assumed $\theta_y$, or resolved by a second Fourier transform over $d_y$ when the array is two-dimensional.

Radar notation: radars with several transmitters and receivers used in every combination are called *MIMO* (multiple input, multiple output); the effective element at the midpoint $\vec d$ of a pair is a *virtual antenna*, and the set of them the *virtual array*; the direction $\theta = 0$ along the array normal is called *boresight*.

So the complete picture: **range, velocity, and angle are the three directions along which a scatterer's phase is linear in an available coordinate** (fast time, slow time, antenna position), each resolved by its own Fourier transform; and full imaging of extended, moving geometry is the general matched-filter sum of the SAR section, of which these three transforms are the linear special cases.
