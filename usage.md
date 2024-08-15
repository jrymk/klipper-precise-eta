## Usage

Use the following command to set the current time. Note that this macro won't be run after the `PRINT_START` macro has fully finished, so it is recommended to run this just before printing the file, or after the `PRINT_START` macro finishes.\
This is optional, if you don't run this, you will only see the remaining time in hours, minutes, and seconds, and not the estimated print end time in the form of "22:36:21, tomorrow", for example.
```gcode
IT_IS T=044540
```
