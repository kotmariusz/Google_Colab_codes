# Raman Fiber Amplifier Simulator

Interactive Jupyter tool for modeling a Raman fiber amplifier with
stimulated Brillouin scattering (SBS) suppression. The amplifier is
described by a set of coupled power-propagation equations, solved as a
boundary value problem with `scipy.integrate.solve_bvp`.

## Features

- **Manual parameter entry** — every physical and numerical parameter
  (fiber length, attenuation, mode area, gain coefficients, wavelengths,
  powers, solver settings) is entered through text boxes, not sliders.
- **Practical units** — each field uses the unit that field is normally
  quoted in, so you can type numbers directly off a datasheet or a
  measurement instead of converting them by hand:
  - **Pump power(s)** — Watts [W] (typical EDFA pump powers are a few W).
  - **Signal seed power** and **SBS/Brillouin seed power** — milliwatts
    [mW].
  - **Fiber loss** (pump and signal) — dB/km, the standard way fiber
    attenuation is specified on a datasheet (e.g. ~0.2 dB/km near
    1550 nm).
  - Internally, everything is converted to base SI units (W and 1/m)
    before being handed to the solver — the physics is unchanged, only
    the input/display units are more convenient.
  - **Watch out:** one easy mistake when trying to reproduce results
    from [1] is entering the fiber loss straight from the
    datasheet/paper (in dB/km) without converting it to 1/m.
- **Three pumping configurations**:
  - **Forward pumping** — pump co-propagates with the signal, launched
    at `z = 0`.
  - **Backward pumping** — pump counter-propagates the signal, launched
    at `z = L`.
  - **Bidirectional pumping** — pump launched simultaneously from both
    ends, with an independent power [W] set for each end, modeled as
    two separate pump waves whose Raman gain contribution to the signal
    adds together.
- **Two operating modes**:
  1. **Simulate amplifier** — sweeps the fiber length and plots the
     signal power (at the output end) and the SBS power (at the input
     end) as a function of length, with an optional pump-power curve
     (or curves, for bidirectional pumping) showing how the pump
     depletes over the fiber length.
  2. **Extract Raman gain coefficient** — given the measured signal
     power [mW] and pump power [W] at each end of a fiber of known
     length, computes the Raman gain coefficient `gr` (forward or
     backward pumping only):
     - a fast analytic estimate (undepleted-pump / effective-length
       approximation), and
     - a numerically exact value obtained by fitting the full
       pump-depletion-aware propagation model.
- **Numerically robust length sweep** — each length step is solved
  using continuation (warm-started from the previous converged
  solution) instead of a fixed initial guess, with automatic rejection
  of non-converged or non-physical (energy-violating) solver output.

## Usage

1. Open `raman_fiber_amplifier_simulation.ipynb` in **Google Colab**
   (or any other Jupyter environment — Notebook, JupyterLab, VS Code).
   Colab already has all the required packages (`numpy`, `scipy`,
   `matplotlib`, `ipywidgets`) preinstalled, so there is nothing to set
   up first.
   - From GitHub: open the notebook's page, click **Raw**, copy the
     URL, then in Colab go to *File → Open notebook → GitHub* and
     paste the repository/notebook path — or click an "Open in Colab"
     badge if you've added one to your repo (see note below).
2. Click **Run** on the single code cell (or *Runtime → Run all*). This
   loads the model and displays the control panel below the cell —
   nothing is computed yet at this point.
3. In the control panel, pick a **Mode** and **Pump direction**, fill
   in the parameter fields in their labeled units, and click the
   **Run** button *inside the panel* to generate the plot (Simulate
   mode) or the `gr` result (Extract mode). You can change values and
   click that Run button again as many times as you like.

> **Adding a Colab badge:** once this notebook is pushed to GitHub, you
> can add a one-click badge at the top of this README:
> `[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/<username>/<repo>/blob/main/raman_fiber_amplifier_simulation.ipynb)`
> (replace `<username>/<repo>` with your GitHub path).

If you'd rather run it locally instead of in Colab: `pip install -r requirements.txt`, then open the notebook the same way.

## Physical model

**Forward / backward pumping** (3 coupled waves: signal `Pr`, pump `Pp`,
SBS `Psbs`):

```
dPr/dx   =  (eps*gr*Pp*Pr)/Aeff - (gbeff*Pr*Psbs)/Aeff - alfas*Pr
dPp/dx   =  pump_sign * [ (lambdR/lambdP)*(eps*gr*Pr*Pp)/Aeff
                         + (lambdR/lambdP)*(eps*gr*Pp*Psbs)/Aeff
                         + alfap*Pp ]
dPsbs/dx = -(eps*gr*Pp*Psbs)/Aeff - (gbeff*Pr*Psbs)/Aeff + alfas*Psbs
```

- `Pr` — Raman signal power, always seeded at `z = 0`.
- `Pp` — pump power. `pump_sign = +1` for backward pumping (launched at
  `z = L`), `-1` for forward pumping (launched at `z = 0`).
- `Psbs` — backscattered SBS power, always seeded at `z = L` (it
  counter-propagates relative to the signal).

**Bidirectional pumping** (4 coupled waves: signal `Pr`, forward pump
`Ppf`, backward pump `Ppb`, SBS `Psbs`). Each pump wave attenuates and
depletes along its own direction of travel, but the Raman gain both
provide to the signal and SBS waves adds, since gain only depends on
total local pump intensity:

```
Pp_tot   =  Ppf + Ppb
dPr/dx   =  (eps*gr*Pp_tot*Pr)/Aeff - (gbeff*Pr*Psbs)/Aeff - alfas*Pr
dPpf/dx  = -[ (lambdR/lambdP)*(eps*gr*Pr*Ppf)/Aeff
             + (lambdR/lambdP)*(eps*gr*Ppf*Psbs)/Aeff + alfap*Ppf ]
dPpb/dx  =  [ (lambdR/lambdP)*(eps*gr*Pr*Ppb)/Aeff
             + (lambdR/lambdP)*(eps*gr*Ppb*Psbs)/Aeff + alfap*Ppb ]
dPsbs/dx = -(eps*gr*Pp_tot*Psbs)/Aeff - (gbeff*Pr*Psbs)/Aeff + alfas*Psbs
```

`Ppf` is launched at `z = 0`, `Ppb` is launched at `z = L`. Setting one
of the two pump powers to zero reduces this exactly to the single-
direction model above.

Common parameters: `gr` — Raman gain coefficient [m/W], `gbeff` —
effective Brillouin gain coefficient [m/W], `eps` — polarization
factor, `Aeff` — mode area, `alfap`/`alfas` — fiber attenuation at the
pump/signal wavelength (converted internally from dB/km to 1/m),
`lambdP`/`lambdR` — pump/signal wavelength.

### Origin of the Raman gain coefficient (`gr`)

The `gr` parameter is the small-signal Raman gain coefficient of the
fiber core glass, and its analytical derivation is laid out in Singh,
Gangwar, and Singh [2]. That paper works from the same kind of coupled
pump/signal power-propagation equations used in this simulator and
solves them under a set of standard simplifying assumptions:
steady-state, continuous-wave operation (no pulse/transient effects);
a single transverse mode, so the interaction is treated as uniform
over the fiber's effective core area `Aeff` rather than the true
transverse intensity profile; and the small-signal, undepleted-pump
regime, where the pump is assumed to decay only through the fiber's
linear attenuation and is not yet significantly depleted by the
Raman/Brillouin transfer to the signal. Integrating the coupled
equations over the fiber length under these assumptions — and taking
the long-fiber limit where the effective interaction length reduces
to `1/alfa` (the reciprocal of the small-signal attenuation constant)
— gives a closed-form relationship between the gain coefficient, the
fiber loss, and the effective area at the *threshold* (critical) pump
power for the scattering process, of the same form as Smith's classic
result: `Pcrit ≈ 20 * Aeff * alfa / gr`. This lets `gr` be inferred
analytically from geometric/loss parameters and a measured or
specified threshold power, rather than requiring a separate direct
gain measurement. The same paper also compares this analytical
Raman-scattering treatment against the corresponding one for Brillouin
scattering, highlighting that SRS has a much broader gain bandwidth
(driven by optical-phonon coupling) than SBS (driven by acoustic
phonons), and that — unlike SBS, which is inherently a backward
process — SRS can build up in both the forward and backward
directions, which is exactly why this simulator offers forward,
backward, and bidirectional pumping for the Raman interaction.

## Notes on the gain-coefficient extraction mode

The numeric fit solves the reduced two-wave (signal + pump, no SBS)
version of the model and root-finds `gr` so that the simulated output
signal power matches your measured value, for the given input signal
power, launch pump power, fiber length, attenuation, mode area, and
pumping direction. It currently only supports forward or backward
pumping (single measured pump end); bidirectional extraction would
need pump measurements at both ends plus a different fit, so the mode
dropdown automatically restricts the direction options when this mode
is selected. If the numeric fit doesn't converge, check that the
measured output power is physically reachable (e.g. not larger than
what the launched pump power could produce), or widen `gr_bounds` in
`extract_gr_numeric()`.

## Default parameters

The default parameter values (4 W pump, ~6.5 mW signal seed, 1651 nm
signal, 1530 nm pump, ~0.2/0.3 dB/km loss, dispersion-shifted fiber)
are based on the Raman amplifier described in [1] below, built for
methane-sensing photoacoustic spectroscopy.

## References

1. R. Bauer et al., "Miniaturized Photoacoustic Trace Gas Sensing
   Using a Raman Fiber Amplifier," in Journal of Lightwave
   Technology, vol. 33, no. 18, pp. 3773-3780, 15 Sept.15, 2015,
   doi: 10.1109/JLT.2015.2443377.
2. Sunil Singh, Ramgopal Gangwar, and Nar Singh, "Nonlinear Scattering
   Effects in Optical Fibers," Progress In Electromagnetics Research,
   Vol. 74, 379-405, 2007. doi:10.2528/PIER07051102

## Requirements

See `requirements.txt`. Widgets require an environment that supports
`ipywidgets` (Jupyter Notebook/Lab, VS Code's notebook interface with
the Jupyter extension, or Google Colab).
