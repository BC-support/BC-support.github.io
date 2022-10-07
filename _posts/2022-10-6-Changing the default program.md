The PHL comes installed with a default hearing aid program that runs when the device is powered up. It's also possible to write a your own default program and this post outlines the steps involved in running a custom configuration. 


The steps necessary to load the simple compressor example onto the PHL are described here. This is just a step by step description of what’s needed, the reasoning behind these steps can be found in other documents. 

The following skills are needed to successfully get this example running on the PHL, although the full commands are given here if you're still learning your way around Linux:

* Ability to log in remotely to the PHL using **ssh** and copy files from your host computer to the PHL using **scp**.
* Very basic knowledge about navigating around Linux directories once you’re on the PHL (**cd**, **ls**, etc.).
* Use an editor (**vi**, **nano**, etc.) to make simple changes to text files on the PHL.

## Get the example custom .cfg files on your host computer
**1\.** The files used in this example, **simple_compressor.cfg** and **eq_gains.cfg**, are included here in the appendix and they're known to work. A simple compressor is set up with appropriate gains for the PHL BTE headset so it’s obvious to hear when the device is running correctly. Download the files to your PC (the host computer).


##     Create the new directory structure on the PHL


**2\.** After your host computer is connected to the PHL via WiFi, login to the PHL using **ssh** (user:mha password: mahalia) and from the directory /etc, create a new directory, mahalia_example. Since /etc belongs to root, use the sudo command   
<span style="color:green">mha@mahalia</span>:<span style="color:blue">/etc</span>$ <span style="color:red"> sudo mkdir mahalia_example</span>   

**3**\. Change the owner of the new directory to user 'mha'.   
<span style="color:green">mha@mahalia</span>:<span style="color:blue">/etc</span>$ <span style="color:red"> sudo chown mha:mha mahalia_example</span>   

**4**\. Copy the config file from the default /etc/mahalia directory into mahalia_example

<span style="color:green">mha@mahalia</span>:<span style="color:blue">/etc/mahalia_example</span>$<span style="color:red"> cp ../mahalia/config .</span>

**5**\. Edit the /etc/mahalia_example/config file to make sure the settings are correct for the ALSA based simple_compressor.cfg. The **JACK_*** settings don't matter here since this example uses the ALSA interface, but it's important that 
**NOJACK = 1**
is set in the config file. Also, it's easier to debug your programs if initially you set 
**OPENMHA_OPTS=\"--daemon \"**

**6**\. Create the directory mha_configuration in mahalia_example   
<span style="color:green">mha@mahalia</span>:<span style="color:blue">/etc/mahalia_example</span>$<span style="color:red"> mkdir mha_configuration</span> 

**7**\. Move the .cfg files from the host computer to the PHL directory /etc/mahalia_example/mha_configuration. Do this from the host computer:   

user@host:~$<span style="color:red">scp simple_compress.cfg mha@10.0.0.1:/etc/mahalia_example/mha_configuration</span>      
user@host:~$<span style="color:red">scp eq_gains.cfg mha@10.0.0.1:/etc/mahalia_example/mha_configuration</span>      

**8**\. On the PHL, copy the ‘headsets’ directory (and all its contents) from the default /etc/mahalia directory into mahalia_example.   

<span style="color:green">mha@mahalia</span>:<span style="color:blue">/etc/mahalia_example</span>$<span style="color:red"> cp -r ../mahalia/headsets .</span>   

## Edit mahalia.config

**9**\. On the PHL, edit the file **/configuration/mahalia.config**, adding a new variable   
**mahaliaconfig=example**   
and uncommenting   
**MAHALIA_CONFIG_PREFIX=/etc/mahalia_**      


*mha@mahalia:/configuration/mahalia.config*

```
# Name of the setup to be used   
# mahaliaconfig=generic-hearing-aid   
# mahaliaconfig=calibration   
mahaliaconfig=example      

# Prefix used to locate the directory that contains the openMHA configurations and gui setup   
MAHALIA_CONFIG_PREFIX=/etc/mahalia_   
```


## Restart and run the program

**10**\. Physically turn the PHL OFF then back ON again to cause a bootup with the new
configuration.

**11**\. Connect the host computer to the PHL again via WiFi

**12**\. From the host computer, run nc (or ncat)

user@host:~$<span style="color:red">nc 10.0.0.1 33337</span>

Hit return to verify you’re in interactive mode (should get ‘success’) 

**13**\. From interactive mode, read in the new .cfg file   
<span style="color:red">?read:/etc/mahalia/mha_configuration/simple_compressor.cfg</span>   
Verify no errors, and start   
<span style="color:red">cmd=start</span>      
Verify no errors. You should hear the device running.   
It’s useful to run a new program this way in the interactive mode because it shows any errors that may occur.  
If you encounter errors or want to make changes, quit interactive mode.   
<span style="color:red">cmd=quit</span>       
This will take you out of interactive mode. You can edit your .cfg file on the PHL, making any necessary changes, restart interactive mode (step 12), then repeat this step.

**14**\. If everything is OK, you can have the program run automatically on startup by editing two files on the PHL.   
Change the last line of the /etc/mahalia_example/config file:   
\# openMHA settings   
OPENMHA_INTERFACE="0.0.0.0"   
OPENMHA_PORT="33337"   
<span style="color:red">OPENMHA_OPTS="--daemon ?read:$MHA_CONFIG_PATH/simple_compressor.cfg"</span>     
 
Also, add the following as the last line of simple_compressor.cfg   
(/etc/mahalia_example/mha_configuration/simple_compressor.cfg)   
<span style="color:red">cmd = start</span>      
When you reboot the device, the simple_compressor.cfg program will run automatically.

## Appendix

### simple_compressor.cfg
```
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



#cmd=start

```
### eq_gains.cfg
```
gains = [[ 1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1     ]; ... 
               [1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1  1 ];]


```
