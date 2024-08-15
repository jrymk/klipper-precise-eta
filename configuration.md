## Configuration
1. Include the [provided config file](precise_eta.cfg), or paste it in whatever config file you want.
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
