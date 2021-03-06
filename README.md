OpenOrb (or OOrb) is an open-source orbit-computation package.

More detailed documentation is available by doing

> `cd doc`

> `make pdf`

which should produce a PDF document 'OpenOrb_Tutorial_vN.N.pdf'.

# Introduction #

OOrb contains, e.g., the statistical orbital ranging method (hereafter
referred to as Ranging). Ranging is used to solve the orbital inverse
problem of computing non-Gaussian orbital-element probability density
functions (p.d.f.s) based on input astrometry.

Ranging is optimized for cases where the amount of astrometry is
scarce or spans a relatively short time span. Ranging-based methods
have successfully been applied to a variety of different topics such
as rigorous ephemeris prediction, orbital-element-distribution studies
for trans-neptunian objects, the computation of invariant collision
probabilities between NEOs and the Earth, detecting linkages between
astrometric asteroid observations within an apparition as well as
between apparitions, and in the rigorous analysis of the impact of
orbital arc-length and/or astrometric uncertainty on the uncertainty
of the resulting orbits.

In OOrb, tools for making ephemeris predictions and classification of
objects (i.e., NEO-MBO-TNO) are also available.


# Installation #

## Requirements ##

  * svn
  * make
  * a Fortran 90/95 compiler
  * ncftpget
  * gnuplot
  * latex
  * dvips
  * ...


## Compiling ##

In the root directory of your OOrb installation (=`OORBROOT`) do

> `make all_clean`

> `./configure COMPILER TYPE`

where `COMPILER` is `gfortran`, `g95`, `intel`, `absoft`, `compaq`, `ibm`, `lahey`, or `sun`, and `TYPE` is either `opt` for optimized code or `deb` for code including debugging information. Then do

> `cd main/`

> `make oorb`

Now you have a working executable called `oorb` in the `OORBROOT/main/` directory. To do something useful, you need to provide the software additional data files which will be prepared in the next section.


## Generating and updating additional data files ##

### Planetary ephemerides ###

The [DE405](http://ssd.jpl.nasa.gov/?planet_eph_export) planetary ephemerides provided by the [Solar System Dynamics Group](http://ssd.jpl.nasa.gov/) at the Jet Propulsion Laboratory need to be converted to binary format (e.g., `de405.dat`) only once by doing

> `cd OORBROOT/data/JPL_ephemeris/`

> `make`

> `make test`

### IAU/MPC Observatory codes and positions ###

The [Minor Planet Center](http://www.cfa.harvard.edu/iau/mpc.html) updates the [observatory codes](http://www.cfa.harvard.edu/iau/lists/ObsCodes.html) on a daily basis, but an update is not necessarily required until you stumble upon observations from an observatory which isn't listed in your version of the file.

> `cd OORBROOT/data/`

> `./updateOBSCODE`

updates a file called `OBSCODE.dat`.

### ET minus UT ###

Update via the OOrb git repository by

> `git pull ET-UT.dat`

### TAI minus UTC ###

Update via the OOrb git repository by

> `git pull TAI-UTC.dat`

## Setting environment variables ##

Finally, you need to tell `oorb` where to find the different files. This is easiest to do through environment variables which you declare in the configuration file for the shell (e.g., `.profile` on Mac OS X and `.bash_profile` on Linux). For the Bash shell you need to add the following lines to the configuration file of your shell:

> `export OORB_DATA=OORBROOT/data`

> `export OORB_CONF=OORBROOT/main/oorb.conf`

> `export OORB_GNUPLOT_SCRIPTS_DIR=OORBROOT/gnuplot/`

# Using oorb #

The full path to the OOrb configuration file is specified by

  1. the `--conf=CONFIGURATIONFILE` command-line parameter
  1. the `$OORB_CONF` environment variable
  1. the current directory assuming the default name (`oorb.conf`)

in this order. That is, option #1 overrides #2 which overrides #3. The path to the default configuration file is `OORBROOT/main/oorb.conf`.

## --task=ranging ##

To compute an orbital solution given astrometric observations, do

> `oorb --task=ranging --obs-in=OBSERVATIONFILE --orb-out=ORBITFILE`

where `OBSERVATIONFILE` (use of suffix, such as **.mpc or**.des, is mandatory!) contains the input astrometry and `ORBITFILE` contains the resulting sampled orbital-element probability-density function (PDF) in OOrb format. The orbits will be written to standard out if `--orb-out=` is omitted.

If astrometry of several different objects is included in OBSERVATIONFILE, then a command like

> `oorb --task=ranging --obs-in=OBSERVATIONFILE --separately`

will process each object separately and write the output to a separate set of files. The separation into different objects is done using the numbers and/or designations. If both are specified for a line of astrometry, then the number overrides the designation.

## --task=lsl ##

> `oorb --task=lsl --obs-in=OBSERVATIONFILE --orb-in=ORBITFILEIN --orb-out=ORBITFILEOUT`


## --task=propagation ##

> `oorb --task=propagation --orb-in=ORBITFILEIN --epoch-mjd-tt=MJD --orb-out=ORBITFILEOUT`


## --task=ephemeris ##

Topocentric ephemerides _without_ uncertainty information for, e.g., Mauna Kea (observatory code 568) for the orbital-element epoch are generated by issuing the command:

> `oorb --task=ephemeris --code=568 --G=GVALUE --orb-in=ORBITFILE`

where `ORBITFILE` (use of suffix, either .orb or .des, is mandatory!) contains the orbits in either the OpenOrb format or DES format and `GVALUE` refers to the slope parameter in the H,G system (default=0.15). The default for `--code` is 500, which corresponds to the geocenter. It is also possible to compute ephemerides simultaenously for a range of dates:

> `oorb --task=ephemeris --code=568 --timespan=TIMESPAN --step=STEP --orb-in=ORBITFILE`

Here `TIMESPAN` specifies how many days into the past (`TIMESPAN` < 0 days) or future (`TIMESPAN` > 0 days) you wish to compute ephemerides, and `STEP` specifies the time interval (in days) between ephemerides. The default is to use an integrator for propagations of the orbital elements, but it is also possible to use the analytical two-body approach by making changes to the configuration file.

The ephemerides are written to stdout unless the `--separately` option is specified (in which case every object in the input file gets its own output file).

The output is divided into the following columns (as marked by a single-line header)
  * **Designation** is the designation for the orbit/object as specified in the input file,
  * **Code** is the official observatory code assigned by IAU/MPC,
  * **MJD UTC/UT1** is the UTC (or UT1 before year 1972) ephemeris date (Modified Julian Date),
  * **Delta**, **RA** and **Dec** are the topocentric equatorial spherical coordinates (AU, deg, deg),
  * **dDelta/dt**, **dRA/dt** and **dDec/dt** are the instantaneous topocentric equatorial spherical sky velocities (for coordinate velocities, divide **dRA/dt** with the cosine of **Dec**) (AU/day, 2 x deg/day),
  * **VMag** is the apparent brightness (mag),
  * **Alt** is the altitude of the object (pure geometric altitude where Earth is assumed spherical) (deg),
  * **Phase** is the phase angle of the object (deg),
  * **LunarElon** is the lunar elongation (angular distance between the Moon and the object) (deg),
  * **LunarAlt** is the lunar altitude (pure geometric altitude where Earth is assumed spherical) (deg),
  * **LunarPhase** is the lunar phase where 0 is new moon and 1 is full moon,
  * **SolarElon** is the solar elongation (angular distance between the Sun and the object) (deg),
  * **SolarAlt** is the altitude of the Sun (pure geometric altitude where Earth is assumed spherical) (deg),
  * **r**, **HLon**, and **HLat** are the heliocentric ecliptic spherical coordinates (AU, deg, deg),
  * **TLon** and **TLat** are the topocentric ecliptic spherical coordinates (2 x deg),
  * **TOCLon** and **TOCLat** are the topocentric opposition-centered ecliptic spherical coordinates (2 x deg),
  * **HOCLon** and **HOCLat** are the heliocentric opposition-centered ecliptic spherical coordinates (2 x deg),
  * **TOppLon** and **TOppLat** are the topocentric ecliptic spherical coordinates for the opposition direction (2 x deg),
  * **HEclObj X Y Z dX/dt dY/dt dZ/dt** are the heliocentric ecliptic cartesian coordinates for the object (3 x AU, 3 x AU/day), and
  * **HEclObsy X Y Z** are the heliocentric ecliptic cartesian coordinates for the observer (3 x AU).


# FAQ #

## Installation ##

### Which OS + compiler combos seem to work with OpenOrb? ###

  * Fedora Core 9 + intel versions 11.8
  * Linux + intel 10.1
  * Mac OS X 10.5.8 + gfortran 4.3.4 and 4.4.0
  * Mac OS X 10.5.8 + g95 0.91

### Which OS + compiler combos have problems with OpenOrb? ###

  * Mac OS X 10.5.7 + gfortran versions 4.2.3 and 4.3.1
  * Fedora Core 9 + gfortran 4.3.0

## Usage ##

### Can I specify a date for which I want an ephemeris to be computed? ###

No. At the moment the only option is to first explicitly propagate the orbits to the desired date by using, e.g.,

> `oorb --task=propagation --epoch-mjd-utc=DATE --orb-in=ORBITFILEIN --orb-out=ORBITFILEOUT`

and then compute the ephemeris using, e.g.,

> `oorb --task=ephemeris --orb-in=ORBBITFILEOUT`

Note that there is no algorithmic reason why you couldn't specify an option like `--epoch-mjd-utc=DATE` to `--task=ephemeris`. The option just doesn't exist (yet!).

### I've been seeing the following warning message quite frequently: "Could not find inverse of inverse covariance matrix." Should I be concerned, or is this normal? ###

This is normal and has to do with the numerical instability of the matrix
inversion. The information matrix, for which the inversion fails, typically has a large condition number, that is, the inversion results are not accurate, and in this particular case a solution cannot even be found. For a sampling method the impact of the failure on the overall results is expected to be negligible because a successful solution can probably be obtained in the immediate vicinity of the failed one.


### Is there an OOrb Users mailing list? ###

Yes there is. Go to http://groups.google.com/group/oorb.


## Miscellaneous ##

### Why are the Python wrappers not working anymore? ###

The Python wrappers are under heavy development and change fairly frequently. Use the command-line executable if you need a more stable platform.


# Citation information #

When using this software please cite

	Granvik, M., Virtanen, J., Oszkiewicz, D., Muinonen, K. (2009). 
	OpenOrb: Open-source asteroid orbit computation software including statistical ranging. 
	Meteoritics & Planetary Science 44(12), 1853-1861.


# Licensing information #

OpenOrb is free software: you can redistribute it and/or modify it
under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

OpenOrb is distributed in the hope that it will be useful, but
WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
General Public License for more details.

You should receive a copy of the GNU General Public License along 
with OpenOrb. If not, see <http://www.gnu.org/licenses/>.
