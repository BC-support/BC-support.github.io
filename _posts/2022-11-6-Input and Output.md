When the standard BTE headset is connected to the PHL, there are 4 microphones and 2 receivers available to use, along with 2 line inputs and 2 line outputs. The 'route' plugin, which is documented in openMHA_plugins.pdf, determines the mapping between these external physical signals and the internal channels of the algorithm running in the PHL.     

Once you're familiar with the 'route' plugin and you've seen a few examples it's fairly easy to understand. Initially, however, there are a few quirks that make it tricky to learn. 

## All about the indexes 
The key to understanding 'route' is to realize it's just a way of rearranging vectors using indexing.     
* There are two vectors involved: the original vector and the output vector. The original vector represents the order of the channels at this point in the signal flow. The output vector is the order of the channels from this point forward.
* The indexing is zero based. 
* The indexes (:0, :1, :2, etc.) represent positions in the original vector.
* Psuedo example:   
If there are three channels, [Left  Right  Sum], going through an algorithm and then 'route' is called with   
mha.route.out = [ :0  :2  :1]   
then the order of the channels gets changed to [Left  Sum  Right]   

## Physical input/output indexes

(All the examples and explanations here are for the in/out library (iolib) MHAIOalsa. The same basic concepts apply to the Jack iolib, although the nomenclature is different.)    
Inputs:

| Signal |  Index |
|--------|--------|
| BTE Left Front Mic | :0 |
| BTE Right Front Mic | :1 |
| Line in Left | :2 |
| XXX | XXX|
| BTE Left Rear Mic | :4 |
| BTE Right Rear Mic | :5 |
| Line in Right | :6 |


Outputs:

| Signal | Index |
|------|-------|
|BTE Left Receiver| :0 |
|BTE Right Receiver| :1 |
|Line out Left| :2 |
|XXX|XXX|
|XXX|XXX|
|XXX|XXX|
|Line out Right| :6 |

## Examples 
In **index.cfg**, from the default **generic hearing aid configuration** that comes with the PHL, the route plugin is used as follows:   
**mha.sort_input.out=[:0 :4 :1 :5 :2 :6]**   
where 'sort_input' is an alias for 'route'. Looking at the indexes for the direct inputs, this results in a vector of signals in the following order:   
**[Left Front Mic, Left Rear Mic, Right Front Mic, Right Rear Mic, Line in Left, Line in Right]**    

--------------------------------------------------------------------------------------------------   

 
The **simple_compressor.cfg** example only uses two front mics, so the statement    
**mha.sort_input.out = [:0 :1]**
where 'sort_input' is again an alias for 'route', results in   
**[Left Front Mic, Right Front Mic]**   

--------------------------------------------------------------------------------------------------   

On the output side, the default **generic hearing aid configuration** has four channels, two for the BTE receivers and two for the line outputs. They're in the following order as they're processed by the algorithm:   
**[BTE Left Receiver, BTE Right Receiver, Line Out Left, Line Out Right]**   
Due to the details of the Alsa interface, to send these signals to the proper outputs we need to specify a vector of length 8. The positions in this vector match up with the output table above. This results in:   
**mha.sort_output.out = [:0 :1 :2 X  X  X :3 X]**   
Here, 'sort_output' is an alias for 'route'. Also, we can't use an X as shown here for the 'dummy' positions, so we use the :0 signal channel as a placeholder that should be harmless (in theory). So the actual statement in the .cfg file is:   
**mha.sort_output.out = [:0 :1 :2 :0 :0 :0 :3 :0 ]**   
So, it's a little tricky but hopefully it makes sense.   

--------------------------------------------------------------------------------------------------   

As a final example, if you wanted to change the **simple_compressor.cfg** example so that it used the Line Inputs instead of the BTE mics, you'd change the previous 'sort_input' line to:   
**mha.sort_input.out = [:2  :6]**   
to produce the signal vector   
**[Line In Left, Line In Right]** 

  




 
