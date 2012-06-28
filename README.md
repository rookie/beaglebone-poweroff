### Details

This patch allows for 'halt -p' and 'poweroff' commands to completely power down the beaglebone. Running on 5V this has been initially calculated at 0.5mA in the TPS65217 OFF state.

It does not include power button functionality.

It is at a very early state. I haven't even tried to add it into the beaglebone openembedded workflow yet.

### Todo

* Incrementing seconds needs to be propagated all the way back up through min/hour/day/month/year
* Race condition when the seconds increment after we read current value but before we write the next value

* Add instructions to add it into the openembedded patch system

### Details

1. At the end of `setup_beaglebone()` I set `pm_power_off = beaglebone_tps65217_poweroff;` to call that function on poweroff. Search for pm_power_off in the kernel for more details.
2. in `tps65217_set_off_bit()` set the tps65217 PMIC to go into the OFF state when PMIC_POWER_EN is pulled low
    1. Change the tps65217 `struct i2c_client *client;` into a static global `tps65217_client`. (If there is a better normal kernel way to talk to this i2c chip in my function let me know.)
    2. Write the STATUS_OFF bit in the STATUS register
3. Enable pwr_enable_en bit
4. Enable ALARM2 single interrupt
5. Copy over hour/day/month/year
6. Increment seconds and propagate to minutes (TODO for hour...etc)
7. Wait up to one second for poweroff.
8. TODO: we need to check to see if the seconds have changed since we first read them as a race condition check (seconds incremented without triggering interrupt). In this case we can just loop to the top and try again.