Rev 1.1

- Added a third thermistor. It is used as an iHeater activation trigger: heating turns on when the temperature rises above 60 °C and turns off when the bed temperature drops to a consistently low level. Implemented to support users of the standalone firmware.

- Updated the PCB layout with improved component placement and routing.

- Changed the power connection method. The main option is to route the power cable through a cable gland in the enclosure wall and connect it to the screw terminals. This allows convenient installation with power switching through the printer's main power line. Alternative option: a C8 connector can be mounted on the enclosure. The board can be oriented freely.

- Pin-to-pin compatible with the current version, except for support for the third thermistor.
