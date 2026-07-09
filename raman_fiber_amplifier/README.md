# Raman Fiber Amplifier Simulator

Interactive Jupyter tool for modeling a Raman fiber amplifier with
stimulated Brillouin scattering (SBS) suppression. The amplifier is
described by a set of coupled power-propagation equations, solved as a
boundary value problem with `scipy.integrate.solve_bvp`.

## Features

- **Manual parameter entry** — every physical and numerical parameter
  (fiber length, attenuation, mode area, gain coefficients, wavelengths,
  powers, solver settings) is entered through text boxes, not sliders.
- **Pump direction switch** — toggle between **forward** (co-propagating)
  and **backward** (counter-propagating) pumping.
- **Two operating modes**:
  1. **Simulate amplifier** — sweeps the fiber length and plots the
     signal power (at the output end) and the SBS power (at the input
     end) as a function of length, with an optional third curve showing
     how the pump power depletes over the fiber length.
  2. **Extract Raman gain coefficient** — given the measured signal and
     pump power at each end of a fiber of known length, computes the
     Raman gain coefficient `gr`:
     - a fast analytic estimate (undepleted-pump / effective-length
       approximation), and
     - a numerically exact value obtained by fitting the full
       pump-depletion-aware propagation model.
- **Numerically robust length sweep** — each length step is solved
  using continuation (warm-started from the previous converged
  solution) instead of a fixed initial guess, with automatic rejection
  of non-converged or non-physical (energy-violating) solver output.

## Physical model

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
- `gr` — Raman gain coefficient [m/W], `gbeff` — effective Brillouin
  gain coefficient [m/W], `eps` — polarization factor, `Aeff` — mode
  area, `alfap`/`alfas` — fiber attenuation at the pump/signal
  wavelength, `lambdP`/`lambdR` — pump/signal wavelength.

## Usage

1. Install the requirements: `pip install -r requirements.txt`
2. Open `raman_fiber_amplifier_simulation.ipynb` in Jupyter (Notebook,
   JupyterLab, or VS Code) and run the single code cell.
3. Pick a **Mode** and **Pump direction**, fill in the parameter fields,
   and click **Run**.

## Notes on the gain-coefficient extraction mode

The numeric fit solves the reduced two-wave (signal + pump, no SBS)
version of the model and root-finds `gr` so that the simulated output
signal power matches your measured value, for the given input signal
power, launch pump power, fiber length, attenuation, mode area, and
pumping direction. If it doesn't converge, check that the measured
output power is physically reachable (e.g. not larger than what the
launched pump power could produce), or widen `gr_bounds` in
`extract_gr_numeric()`.

## Default parameters

The default parameter values (4 W pump, ~6.5 mW signal seed, 1651 nm
signal, 1530 nm pump, dispersion-shifted fiber) are based on the Raman
amplifier described in [1] below, built for methane-sensing
photoacoustic spectroscopy.

## References

1. R. Bauer, T. Legg, D. Mitchell, G. M. H. Flockhart, G. Stewart,
   W. Johnstone, and M. Lengden, "Miniaturized Photoacoustic Trace Gas
   Sensing Using a Raman Fiber Amplifier," *Journal of Lightwave
   Technology*, vol. 33, no. 18, pp. 3773–3780, 2015.
   https://ieeexplore.ieee.org/document/7120894
2. R. H. Stolen and E. P. Ippen, "Raman gain in glass optical
   waveguides," *Applied Physics Letters*, vol. 22, no. 6, pp. 276–278,
   1973. https://doi.org/10.1063/1.1654496

## Requirements

See `requirements.txt`. Widgets require an environment that supports
`ipywidgets` (Jupyter Notebook/Lab, or VS Code's notebook interface with
the Jupyter extension).
