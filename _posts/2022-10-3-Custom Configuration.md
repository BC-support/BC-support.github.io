The PHL comes installed with a default hearing aid program that runs when the device is powered up. It's also possible to write a your own default program and this post outlines the steps involved in running a custom configuration. 


The steps necessary to port the simple compressor example, defined in a previous post, are described here. This is just a step by step description of what’s needed, the reasoning behind these steps can be found in other documents. 

The following skills, which are covered in other documents, are needed to successfully get this example running on the PHL:

* Ability to log in remotely to the PHL using ssh and copy files from your host computer to the PHL using scp.
* Very basic knowledge about navigating around Linux directories once you’re on the PHL (cd, ls, etc.).
* Use an editor (vi, nano, etc.) to make simple changes to text files on the PHL.

# Get the example custom .cfg files on your host computer
1. The file used in this example, simple_compressor.cfg, is included here in the appendix and is known to work. A simple compressor is set up with appropriate gains for the PHL BTE headset so it’s obvious when the device is running correctly. Download the files to your PC (the host computer).

# Create the new directory structure on the PHL
2. After your host computer is connected to the PHL via WiFi, login to the PHL using ssh (user:mha password: mahalia) and from the directory /etc, create a new directory, mahalia_example. Since /etc belongs to root, use the sudo command
mha@mahalia:/etc sudo mkdir mahalia_example

3. Change the owner of the new directory to user 'mha'.
mha@mahalia:/etc sudo chown mha:mha mahalia_example

4. Copy the config file from the default /etc/mahalia directory into mahalia_example
 mha@mahalia:/etc/mahalia_example cp ../mahalia/config .

5. Edit the /etc/mahalia_example/config file to make sure the settings are correct for the ALSA based simple_compressor.cfg. The JACK_* /_ settings don't matter here since this example uses the ALSA interface, but it's important that 
NOJACK = 1
is set in the config file. Also, it's easier to debug your programs if initially you set 
OPENMHA_OPTS="--daemon "

6. Create the directory mha_configuration in mahalia_example
mha@mahalia:/etc/mahalia_example mkdir mha_configuration 


