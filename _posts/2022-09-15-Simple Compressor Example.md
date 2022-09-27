---
title: 
date: 2022-09-15
---

The default program that comes installed on the PHL can be overwhelming for new users trying to understand the openMHA framework. A less complex program is presented here that implements a compressor without any other processing. 

(If you're interested in more fully documented openMHA programs, check out the /openMHA/examples directory in your installation. Just remember, most of that code is designed to run on a PC. New users often get confused by openMHA details that don't translate between PC and PHL.)    

The configuration files for this Simple Compressor are located at the end of this post and also at [https://github.com/BC-support/Simple-Compressor](https://github.com/BC-support/Simple-Compressor). 

## Signal Flow

![simple compressor flow](/assets/images/simple-compressor.drawio.png)

* Only the front mics of the BTEs are used here. The output goes to the respective BTE receivers along with the headphone line out.
```terminal
mha.sort_input.out = [:0 :1]
mha.sort_output.out = [:0 :1 :0 :0 :0 :0 :1 :0]
```
* The FFT length is 256 samples. 

```terminal
mha.transducers.mhachain.overlapadd.fftlen = 256
mha.transducers.mhachain.overlapadd.wnd.type = hanning
mha.transducers.mhachain.overlapadd.wnd.user = []
mha.transducers.mhachain.overlapadd.wnd.len = 128
```
* There are 9 bands in the compressor.

```terminal
mha.transducers.mhachain.overlapadd.mhachain.mbc.f = [250 500 1000 1500 2000 3000 4000 6000 8000]
```
* The gain is currently set to 12dB linear.

```terminal
mha.transducers.mhachain.overlapadd.mhachain.mbc.dc_simple.g50 =                 [ 12 12 12 12 12 12 12 12 12      ...

                                           12 12 12 12 12 12 12 12 12]
mha.transducers.mhachain.overlapadd.mhachain.mbc.dc_simple.g80 =                 [ 12 12 12 12 12 12 12 12 12      ...
                                           12 12 12 12 12 12 12 12 12]
```
* The EQ stage (equalization) settings are currently flat and are read in from an extenal file.
```terminal
mha.transducers.mhachain.overlapadd.mhachain.equalize?read:/etc/mahalia/mha_configuration/eq_gains.cfg
```

## Running the code on the PHL
To run this code on the PHL instead of the default generic hearing aid application a number of changes need to be made to settings and files on the device. It's not that complicated though, and there will be another blog post detailing the exact steps needed to set up the PHL to run this or any other configuration the user desires. 


### simple-compressor.cfg
```terminal
nchannels_in = 2
nchannels_in = 2
fragsize = 64
srate = 24000 
iolib = MHAIOalsa
io.in.device=hw:0
io.out.device=hw:0
io.priority=90

mhalib = mhachain 

mha.algos = [ route:sort_input transducers route:sort_output]

mha.transducers.plugin_name = mhachain 

mha.transducers.mhachain.algos = [ overlapadd ]

mha.sort_input.out = [:0 :1]
mha.sort_output.out = [:0 :1 :0 :0 :0 :0 :1 :0]

mha.transducers.calib_in.peaklevel = [115 115]
mha.transducers.calib_in.fir = [[1 0 0];[1 0 0]]
mha.transducers.calib_out.peaklevel = [ 108]
mha.transducers.calib_out.fir = [[]]
mha.transducers.calib_out.softclip.tau_attack = 0.002
mha.transducers.calib_out.softclip.tau_decay = 0.005
mha.transducers.calib_out.softclip.threshold = 0.6
mha.transducers.calib_out.softclip.slope = 0.5
mha.transducers.calib_out.softclip.tau_clip = 1
mha.transducers.calib_out.softclip.max_clipped = 1
mha.transducers.calib_out.do_clipping = yes 

mha.transducers.mhachain.overlapadd.plugin_name = mhachain

mha.transducers.mhachain.overlapadd.fftlen = 256
mha.transducers.mhachain.overlapadd.wnd.type = hanning
mha.transducers.mhachain.overlapadd.wnd.user = []
mha.transducers.mhachain.overlapadd.wnd.len = 128
mha.transducers.mhachain.overlapadd.wnd.pos = 0.5
mha.transducers.mhachain.overlapadd.wnd.exp = 1
mha.transducers.mhachain.overlapadd.zerownd.type = rect
mha.transducers.mhachain.overlapadd.zerownd.user = []
mha.transducers.mhachain.overlapadd.mhachain.algos = [multibandcompressor:mbc equalize]

mha.transducers.mhachain.overlapadd.mhachain.mbc.unit = Hz
mha.transducers.mhachain.overlapadd.mhachain.mbc.f = [250 500 1000 1500 2000 3000 4000 6000 8000]
mha.transducers.mhachain.overlapadd.mhachain.mbc.fscale = log
mha.transducers.mhachain.overlapadd.mhachain.mbc.ovltype = hanning 
mha.transducers.mhachain.overlapadd.mhachain.mbc.ftype = center
mha.transducers.mhachain.overlapadd.mhachain.mbc.plugin_name = dc_simple 

mha.transducers.mhachain.overlapadd.mhachain.mbc.dc_simple.limiter_threshold = [95 95 95 95 95 95 95 95 95  95 95 95 95 95 95 95 95 95 ] 
mha.transducers.mhachain.overlapadd.mhachain.mbc.dc_simple.expansion_slope = [1.5] 
mha.transducers.mhachain.overlapadd.mhachain.mbc.dc_simple.tau_attack = [0.005] 
mha.transducers.mhachain.overlapadd.mhachain.mbc.dc_simple.tau_decay = [0.015]
mha.transducers.mhachain.overlapadd.mhachain.mbc.dc_simple.filterbank = mbc
mha.transducers.mhachain.overlapadd.mhachain.mbc.dc_simple.g50 =                 [ 12 12 12 12 12 12 12 12 12      ...

                                           12 12 12 12 12 12 12 12 12]
mha.transducers.mhachain.overlapadd.mhachain.mbc.dc_simple.g80 =                 [ 12 12 12 12 12 12 12 12 12      ...
                                           12 12 12 12 12 12 12 12 12]

mha.transducers.mhachain.overlapadd.mhachain.mbc.dc_simple.maxgain =              [40 40 40 40 40 40 40 40 40 ...
                                                      40 40 40 40 40 40 40 40 40 ]
mha.transducers.mhachain.overlapadd.mhachain.mbc.dc_simple.expansion_threshold = [40 40 40 40 40 40 40 40 40   40 40 40 40 40 40 40 40 40]


mha.transducers.mhachain.overlapadd.mhachain.equalize?read:/etc/mahalia/mha_configuration/eq_gains.cfg



cmd=start

```
### eq_gains.cfg
```terminal
gains = [[ 1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1     ]; ... 
               [1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1 ];]
```

