# klipper-precise-eta
Includes print preparation procedures in the print progress estimation, and gives an overall percentage for accurate ETA.\
No modules or Klipper folder modifications needed, it is just a macro.

- samples history print preparation procedures' time and takes average
- allow multiple stages of print preparation procedure to add granularity to the progress display while preparing
- allow enable/disable stages conditionally
- samples history slicer estimate to actual print time-ratio

Estimate: 20:20:22\
![Screenshot 2024-08-15 192007](https://github.com/user-attachments/assets/e0cd865c-e55b-4b28-bc3b-74cfcfd0166c)\
Estimate: 20:19:50\
![Screenshot 2024-08-15 192101](https://github.com/user-attachments/assets/d3bbda9c-2771-4568-87e9-5261d0c3dcb9)\
Estimate: 20:20:15\
![Screenshot 2024-08-15 192225](https://github.com/user-attachments/assets/577a53d5-7324-4df6-b53d-b3ba830bef97)\
Estimate: 20:20:41\
![Screenshot 2024-08-15 194303](https://github.com/user-attachments/assets/4185c5d6-6e9c-45e3-ae0e-bd51ae0e3992)\
Estimate: 20:19:59\
![Screenshot 2024-08-15 213434](https://github.com/user-attachments/assets/ac5acee2-39c6-4afe-89ab-e3eb0c868674)\
Actual: 20:19:59\
![Screenshot 2024-08-15 213425](https://github.com/user-attachments/assets/d8c4d349-42fb-4da3-bd1e-186d9f3a8508)\

The slicer estimate on average overestimates by 1.425-fold:\
![image](https://github.com/user-attachments/assets/b2604729-6531-419c-a228-3e65aad52b7e)

This plugin takes historical data and live data from each print and gives a relatively accurate estimate (off by one minute is expected as the slicer only provides the estimated time in minutes via the M73 gcode, the metadata is not accessible through klipper)\
And unfortunately, the local time is not accessible via klipper as well, so I will add a command: `it_is t214449` to tell the system that it is 21:44:49 at the time the command is sent, for example.

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

### Configuration
1. Include the provided config file, or paste it in whatever config file you want.
2. Enable `[save_variables]` [klipper docs](https://www.klipper3d.org/Config_Reference.html#save_variables)
3. Add the following to the start of the `PRINT_START` macro
```cfg
_PRECISE_ETA_LOAD
_PRECISE_ETA_INCLUDE STAGES="11111111111"
_PRECISE_ETA_SET_STAGE STAGE=-1
```
4. Add the `_PRECISE_ETA_SET_STAGE` command after the stage is complete. You do not have to call this for all the stages (if you're not doing a bed mesh, for example, then stages 5-8 will not be called, and that is fine), only the last one is a must.
```cfg
G28 Z_TILT=1
_PRECISE_ETA_SET_STAGE STAGE=0
```
Another example:
```cfg
[gcode_macro G29]
variable_k:1
gcode:
    _PRECISE_ETA_SET_STAGE STAGE=5
    {% set hotendtemp = params.HOTEND|default(200)|int %}  # hotend temperature during meshing, to save heat up time
    BED_MESH_CLEAR

    G28
    GET_ZOFFSET # now z0 is true zero
    _PRECISE_ETA_SET_STAGE STAGE=6
    MOVE_PROBE_TO_NOZZLE
    G1 Z10 F600
    PROBE_QDPROBE
    SAVE_MESHOFFSET  # store last probe result to SET_MESHOFFSET
    _PRECISE_ETA_SET_STAGE STAGE=7
    G1 Z10 F600
    M104 S{hotendtemp} # begin heat up
    {% if printer["gcode_macro JRYMK_MOD_VARIABLES"].quick_start_enable %}
        _BED_MESH_CALIBRATE PROFILE=quickstart
        SAVE_VARIABLE VARIABLE=profile_name VALUE='"quickstart"'
        SET_GCODE_VARIABLE MACRO="PRINT_START" VARIABLE=quick_start_calibrated_at_bed VALUE={printer["heater_bed"].target}
    {% else %}
        BED_MESH_CALIBRATE PROFILE=kamp
        SAVE_VARIABLE VARIABLE=profile_name VALUE='"kamp"'
        SET_GCODE_VARIABLE MACRO="PRINT_START" VARIABLE=quick_start_calibrated_at_bed VALUE=-1
    {% endif %}
    SET_MESHOFFSET  # just like "zero reference offset"

    _PRECISE_ETA_SET_STAGE STAGE=8
```
5. Make sure you have called the last stage at the end of the `PRINT_START` macro. The time took in each stage will be saved for future estimations here.
```cfg
_PRECISE_ETA_SET_STAGE STAGE=10
```
6. Save the slicer estimation to optimize future estimations with the following command in the start of the `PRINT_END` command:
```cfg
_PRECISE_ETA_SAVE_SLICER_STAT
```
7. You can change what stages are included in this print to make the estimation more accurate, for example, if we are skipping all the pre-print procedure:
```cfg
RESPOND PREFIX="QS: " MSG="Using quick start."
_PRECISE_ETA_INCLUDE STAGES="00000000001"
RESTORE_GCODE_STATE NAME="quickstart" MOVE=1
```
The time it will take for the stages not included will not be counted towards the estimated time, and their time will not be saved for future estimations, of course.
