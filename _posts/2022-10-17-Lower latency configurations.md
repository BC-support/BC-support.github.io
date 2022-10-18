The latency, or delay, through the PHL depends on several interrelated parameters. This post shows how latency is affected by changing the configuration of a typical multiband compressor based on Short Term Fourier Transform (STFT) using overlapp and add processing. The 'simple-compressor.cfg' program used in other posts is the starting point for the examples here.   

As described in the documentation for the **overlapadd** plugin, the signal is windowed then usually zero padded before going through the FFT. The window is advanced by fragment size and the process is repeated.  
		**M**: Window size   
		**N**: FFT size  
		**P**: Frag size   

The delay due to the signal processing is then   
**M + (N-M)/2**   
In addition to this delay, there is a significant delay associated with the data buffering of the Linux operating system.

In terms of practical implementations, there are some general constraints:   
- The window size **M** is usually a multiple of the frag size **P** (P=2\*M, P=4\*M, etc.)   
- Zero padding is necessary to prevent aliasing in practical systems. This means **N**>**M**, and the length of the zero padding limits the length of the impulse response of the effective filter formed by different band gains.   
- The length of the FFT can affect its computational efficiency. It doesn't need to be a power of two, but it's probably best to keep it a product of small prime numbers. For example, 2^5 * 3 = 96.   
- At a certain point the CPU will run out of cycles when the configuration as a whole proves too much. Sampling rate, fragment size, FFT size and overall algorithm complexity all play a role here.
- And of course, a smaller FFT size results in less frequency resolution for a given sample rate (given the usual zero padding caveats). This may or may not be much of an issue, depending on how your bands are structured.

## Delay examples

The delay through the PHL was measured for different parameter configurations and the results are shown below. The electronic measurement used a soundcard to generate an impulse that was routed through the PHL using the Line input/output connectors. All of these configurations used the same sampling rate, 24kHz, for ease of comparison. 

 
|| M | N | P | Delay | 
|---|---|---|---|---------------|
| **1** | 128 | 256 | 64 | 13ms |
| **2** | 64 | 128 | 32 | 7ms |
| **3**| 48 | 96| 24| 6ms |
| **4\***| 110 | 162 | 55| 10ms |
   
Example **1** uses the parameters in the original 'simple-compressor.cfg' program. Examples **2** and **3** show how the delay decreases as the FFT size and corresponding parameters are made smaller. Example **4\*** was measured using the default hearing aid program that comes installed on the PHL.   

Example **3** was also attempted using a sampling rate of 32kHz and the measured delay was 4.5ms, but the processor couldn't keep up and this caused a very audible sporadic clicking.   




