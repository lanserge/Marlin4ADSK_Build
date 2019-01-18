# Marlin4ADSK_Build
CMake project to build Marlin firmware for Arduino DUE to work with Arduino DUE Shield Kit (ADSK)

Arduino DUE Shield Kit is the set of the available boards on the market.
These boards connected to the Arduino DUE in not complex way and result kit used to control 3D printer.

The core is Arduino CNC shield with next modifications:

From the bottom of the CNC shield cut the 5V pin that goes to Arduino.

On the top of the CNC shield install three jumpers:
  1) between 5V and 3V3 pins,
  2) between D12 and A.STEP
  3) between D13 and A.DIR

