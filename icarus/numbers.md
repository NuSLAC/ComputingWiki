### Drift time / velocity
Drift velocity: $1.6\text{mm}/ \mu s$ (0.154 cm / us exactly)

Drift distance (cathode to anode): 1.5m (148.5 cm exactly)

Drift time (cathode to anode): ~1ms (965 us exactly)

### TPC / PMT waveform
Recording window: 2ms

Recording speed for PMT: 500 MHz (2ns/sample)

Enabled at -1150 us before trigger, continues for 3450 us

Recording speed for TPC: 2.5 MHz for 1638.4 us (4096 samples)

Index 0 at 340 us (850 samples) before trigger

TPC readout window size: 4096

1150$\mu s$ Difference between ICARUS readout time and GEANT4 Simulation Start Time.
$T_0^{GEANT4} = 1150\mu s$, $T_0^{readout} = 0$. 

* For a MIP going perpendicular to a unit signal for single wire: 18000 $e^{-}$/cm and 6000 $e^{-}$/wire
* Track on YZ plane w/ some angles from the subject wire: 
	* expect larger cross-section between the track and a wire via projection, so more electrons
	* lower signal on induction plane
* Non-MIP: more electrons



Cosmic Discriminator: 2000 Samples = 4 microsecond (because 500Mhz)

Beam Discriminator: 11000 Samples = 22 microsecond (because 500MHz)

### Cosmic rays in TPC/PMT
Time range of cosmic rays seen by PMT = [-1150, 2300] us around trigger time (in simulation)

Time range of cosmic rays seen by TPC = [-340-965(drift time), 1638.4 - 340] = [-1305, 1298.4] us around trigger

10-20 interactions per event. 
