---
title: Controlling the LED 
date: 2022-11-14
categories:
tags: [leds]     # TAG names should always be lowercase
---
This guide to controlling the LED on the PHL is courtesy of Hendrik Kayser, who knows the PHL inside and out. The push button is also used here to switch things on and off. The basic ideas are as follows:

*  The LED is at '/sys/class/gpio/gpio49/value' Send a 0 or 1 here to turn it OFF or ON.   
```
echo 0 /sys/class/gpio/gpio49/value  # turn LED OFF
echo 1 /sys/class/gpio/gpio49/value  # turn LED ON
```
* The pushbutton is at '/sys/class/gpio/gpio115/value' This value is 0 when the button is pushed.  
```
if [ "$(cat /sys/class/gpio/gpio115/value)" == "0" ]; then...
```


The following three examples show how the LED and push button can be used with the generic hearing aid algorithm in the PHL. The techniques can be applied to other .cfg files as well.   

## Bash script for the coherence filter

```
# query state and store in variable

state="$(echo mha.transducers.mhachain.signal_processing.ola.c.coh.select?val | nc -q 0 localhost 33337)"
# remove the linebreak from openMHA's response
state="${state//[$'\n']}"

if [[ $state == "coherence(MHA:success)" ]]; then
       # COH is on, so turn it off.
       echo mha.transducers.mhachain.signal_processing.ola.c.coh.select=identity | nc -q 0 localhost 33337
       sudo sh -c 'echo 0 > /sys/class/gpio/gpio49/value'
fi
if [[ $state == "identity(MHA:success)" ]]; then
       # COH is off, so turn it on.
       echo mha.transducers.mhachain.signal_processing.ola.c.coh.select=coherence | nc -q 0 localhost 33337
       sudo sh -c 'echo 1 > /sys/class/gpio/gpio49/value'
fi

```

Note that the status is not propagated to the Node-Red GUI and that the 'sudo sh -c' is there to be able to switch the LED status as non-root user.

## Monitoring by the mahalia-batcat system service

Usually, this service monitors the status of the push button and lights it up if pushed. Replacing the content of /usr/share/mahalia-utils/mahalia-batcat with the following bash code will power the button led whenever the bypass of the ADM in the generic hearing aid configuration is set to "0".  Drawback: It queries the state in mha periodically and not only triggered by a change in the configuration.  Advantage: It queries the real state of mha, so it will not change the led in case a command the switch on/off the ADM failed.   

```
#!/usr/bin/env bash

# set GPIO 1_17 to output (initally low)
echo "49" > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio49/direction
echo 0 > /sys/class/gpio/gpio49/value

# set GPIO 3_19 to input
echo "115" > /sys/class/gpio/export
echo in > /sys/class/gpio/gpio115/direction

# wait for AMD to change
while true; do

     # query state of adm
     state="$(echo mha.transducers.mhachain.split.bte.adm.bypass?val | nc -q 0 localhost 33337)"
     # remove the linebreak from openMHA's response
     state="${state//[$'\n']}"


     if [[ $state == "0(MHA:success)" ]]; then
          echo "ADM on"
          echo 1 > /sys/class/gpio/gpio49/value
     fi
     if [[ $state == "1(MHA:success)" ]]; then
          echo "ADM off"
          echo 0 > /sys/class/gpio/gpio49/value
     fi
     sleep 0.5s
done
```
Note that the status is not propagated to the Node-Red GUI and that the
'sudo sh -c' is there to be able to switch the LED status as non-root
user. 


## Switch ADM status via push button   
```
#!/usr/bin/env bash

# set GPIO 1_17 to output (initally low)
echo "49" > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio49/direction
echo 0 > /sys/class/gpio/gpio49/value

# set GPIO 3_19 to input
echo "115" > /sys/class/gpio/export
echo in > /sys/class/gpio/gpio115/direction

# wait for switch to be pressed
while true; do

    if [ "$(cat /sys/class/gpio/gpio115/value)" == "0" ]; then
        echo "switch is pressed!"
        # query state of adm
        state="$(echo mha.transducers.mhachain.split.bte.adm.bypass?val | nc -q 0 localhost 33337)"
        # remove the linebreak from openMHA's response
        state="${state//[$'\n']}"

        if [[ $state == "0(MHA:success)" ]]; then
            # ADM is on, so turn it off.
            echo mha.transducers.mhachain.split.bte.adm.bypass=1 | nc -q 0 localhost 33337
            # Remember to switch decomb filter, too.
            echo mha.transducers.mhachain.signal_processing.ola.c.decomb.select=identity | nc -q 0 localhost 33337

            echo 0 > /sys/class/gpio/gpio49/value
        fi
        if [[ $state == "1(MHA:success)" ]]; then
            # ADM is off, so turn it on. \

            echo mha.transducers.mhachain.split.bte.adm.bypass=0 | nc -q 0 localhost 33337
            # Remember to switch decomb filter, too.
            echo mha.transducers.mhachain.signal_processing.ola.c.decomb.select=equalize | nc -q 0 localhost 33337
            echo 1 > /sys/class/gpio/gpio49/value
        fi
    fi

    sleep 0.2s
done
```


