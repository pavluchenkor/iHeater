Controls two iHeaters in tandem to achieve a target chamber temperatur, adding smart LED controls:
  LEFT: Fan status
  CENTER: Target temperature set
  RIGHT: Heater active (power >30%)
I've also added logic to check if iHeater has AC power. If not, it will kill the heat up process.

Based on the original single iHeater.cfg

I tested on my SV08 MAX. LED controls are not perfect, since the SV08 MAX doesn't have the latest version of Klipper so we cannot do duplicate_pin for correct heater / fan activity indicators.
I do a 2:1 weighted average for the two chamber thermistors, one in the designated hole on the back panel, the other moves around attached to the center of the gantry...
Please refer to my video on YouTube for my arrangement of the two iHeaters: https://www.youtube.com/watch?v=KfKujRUKnWg 
