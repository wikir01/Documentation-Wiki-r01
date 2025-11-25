
## Activation du FastAccel sur un flux entre deux IPs
Doit être fait sur les deux membres du cluster:

fw ctl fast_accel show_state

fw ctl fast_accel enable

fw ctl fast_accel add @ip1 @ip2 any any

cat $FWDIR/log/fw_fast_accel.log

fw ctl fast_accel show_table
