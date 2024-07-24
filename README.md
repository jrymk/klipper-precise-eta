# klipper-precise-eta
Includes print preparation procedures in the print progress estimation, and gives an overall percentage for accurate ETA.



- samples history print preparation procedure' time and takes average
- allow multiple stages of print preparation procedure to add granularity to the progress display while preparing
- allow enable/disable stages conditionally
- samples history slicer estimate to actual print time-ratio

Before calibration, with precise ETA enabled:\
(orange is the "precise" percentage, bluish green is the M73 percentage from the slicer, ignore the bumps, magenta is the total print time estimated from the slicer, because the M73 command only gives remaining time in 1 minute increments)
![image](https://github.com/user-attachments/assets/5ad9c864-b94c-4eb6-9ce7-72747107e163)
After one calibration, different print
![image](https://github.com/user-attachments/assets/600c5db2-0e8d-4e9c-9707-0e37f56f74b5)
