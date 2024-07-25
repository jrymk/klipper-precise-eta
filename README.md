# klipper-precise-eta
Includes print preparation procedures in the print progress estimation, and gives an overall percentage for accurate ETA.



- samples history print preparation procedure' time and takes average
- allow multiple stages of print preparation procedure to add granularity to the progress display while preparing
- allow enable/disable stages conditionally
- samples history slicer estimate to actual print time-ratio

![Screenshot 2024-07-25 154802](https://github.com/user-attachments/assets/6888220a-1bc7-47d5-8a0c-c0bcf74d329c)

Orange is the real time print progress, red is the estimated total time, which its flatness directly reflects the accuracy of the estimations. \
The slicer total estimation is based on the slicer M73 percentage and remaining time (rounded down to the minute) and the projection from the current print. The bumps that happen when yellow (remaining minutes reported by the slicer) will be fixed soon.

Legend:\
![image](https://github.com/user-attachments/assets/51d68a42-8f12-4801-bed0-67c1a3283d5a)

We record the average time took in each prepare stage, and the average slicer estimation accuracy, to improve future estimations.
![image](https://github.com/user-attachments/assets/59000595-b91d-4748-a1dc-d262a7afe42c)

```cfg
[gcode_macro _PRECISE_ETA]
variable_project_remaining_weight:     0.5  # 0: fully trust the slicer's remaining time estimate ;  1: scale the current printed percentage to actual time ratio to the rest of the print (the weight will be further scaled by P(2-P) to give low weight when the percentage is low)
variable_slicer_overall_scale_weight:  0.9  # 0: do not apply scaling to the slicer estimates ;  1: apply the scaling factor obtained from previous prints
variable_stages_time_history:          [0,0,0,0,0,0,0,0,0,0,0]
variable_stages_include:               [1,1,1,1,1,1,1,1,1,1,1]  # what stages to include in the estimation. this is also the stage times that will be added to the history
variable_stages_time_current:          []
variable_stages_default:               [0,0,0,0,0,0,0,0,0,0,0]  # a constant value for macros to set their defaults
# To add or change stages, you'll need to update the lists above, and all _PRECISE_ETA_SET_STAGE / _PRECISE_ETA_INCLUDE references. Make sure the last stage is always set.
# 00 - initial_homing
# 01 - clear_nozzle_start
# 02 - clear_nozzle_purge
# 03 - clear_nozzle_wipe
# 04 - clear_nozzle_finish
# 05 - bed_mesh_start
# 06 - bed_mesh_get_zoffset
# 07 - bed_mesh_mesh_offset_probing
# 08 - bed_mesh_finish
# 09 - nozzle_heating_start
# 10 - nozzle_heating_finish
variable_slicer_estimate_time: 5  # placeholder value, should update immediately after a file is selected
variable_prepare_time: 0
variable_scaling: 1
gcode:
```

Stages are updated with the following commands:
```gcode
_PRECISE_ETA_INCLUDE STAGES="11111111111"  ; what stages to include for this print, say we are not doing a nozzle Z calibration in this print, we can update it to "10000111111". 
_PRECISE_ETA_SET_STAGE STAGE=-1
_PRECISE_ETA_SET_STAGE STAGE=0
_PRECISE_ETA_SET_STAGE STAGE=1
...
```

Everything here is configurable, and you can include as many stages as you want.
