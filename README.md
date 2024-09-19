# Upgrading to Jetpack 6 on Aetina Jetson Devices
**1.0 Release Note: Currently this only reflects the process for updating Orin Nano & NX, once AGX files are available this guide will be updated**

This guide will walk you through getting started with OnLogic’s Aetina Jetson devices. Getting familiar with SDK Manager and updating to Jetpack 6. This guide is specific to Aetina Jetson products, if you are using a Nvidia Dev Kit, refer to the Nvidia documentation as this guide has extra steps to enable Jetson production systems.

## First Steps <br/>

In order to get started, you will need the following:
- A PC running Ubuntu 22.04 (Host PC)
 An Aetina Jetson Device
- A USB-C to USB-A Cable
- Mouse, Keyboard, Monitor (Ideally 2 sets)

1. Power on both the Host PC and the Jetson
2. Attach the USB-A end of the cable to the host PC and the USB-C connector into the port marked ‘OTG’ on the Aetina Jetson device.
![Picture the OTG port](/assets/otgport.jpg) 

## Setting up the Host PC <br/>

We will use the host PC to update our Jetson system to Jetpack 6. This will require downloading some large files, so ensure you have a good Internet connection.

### Step 1: Make sure the PC is up to date <br/>
Assuming that this is a fresh install, first we will ensure that there are no out of date packages. <br/>
Open a Terminal and run the following command: ``` sudo apt-get upgrade ```

### Step 2: Installing Nvidia SDKManager <br/>

Next we will move onto installing Nvidia’s SDKManager which will allow us to download the necessary files (including the BSP) to update our Jetson system. You will need a Nvidia Developer account, if you don’t have one you will be prompted to create one when downloading the ```.deb``` file.

[Navigate to the SDKManager instructions page on your web browser](https://developer.nvidia.com/sdk-manager)

1. Download the install .deb file using the green button marked ‘.deb Ubuntu’
2. Right click in the downloads folder and click ‘Open in Terminal’
3. Run the install command based on the name of the file we downloaded ```sudo apt install ./sdkmanager_[Your File Version]_amd64.deb```
4. Run the following commands to grab the files from a network repository
    - ```wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb```
    - ```sudo dpkg -i cuda-keyring_1.1-1_all.deb```
    - ```sudo apt-get update```
    - ```sudo apt-get -y install sdkmanager```

### Step 3: Setup SDKmanager, Download, and Setup Install Directories <br/>
1. In Terminal, run ```sdkmanager``` to open the program
2. You will be prompted to enter your Nvidia Developer account info
3. We should see our Jetson system appear in the target hardware box
    - If it does not appear try flipping the USB-C cable over in the port or using a different cable.
    - If everything looks good, we can click the ‘continue to step 2’ button.
 ![Screenshot showing what screen should look like at this step](/assets/initsdk.png) 
> [!WARNING]
> We will need to patch the downloaded files before installation so please read the following instructions carefully.
4. We need to have SDKManager create directories the install directories, leave the box, “Download now. Install Later”, unchecked, however we will not be flashing the system yet.
5. It is up to you if you’d like to install the Runtime, SDK Compontents, Platform Services. For now we will just install the needed Host PC components and download the Jetson install files.
6. Click the Continue button, it will first prompt you to create the install directories, then again click it again to start the download process. You will be prompted for the root password.
7. At this point, the system will download and install the Host PC files and download the Jetson Image. This will take a while.
8. Once we get to the screen that will allow us to flash the system, click the x at the top of the window to skip flashing the Jetson system.
![Screenshot showing what screen should look like at this step](/assets/cancelpic.png)
9. The installation will continue to install the Host PC components.
10. Once the installation of the host components is complete we can verify the installed files on the host PC.
    - If you see any errors or missing dependencies, don’t panic. Close SDKManager, open a terminal, and use ```sudo apt-get update``` then reboot the system before opening SDKManager again.
11. If everything worked correctly, and there are no errors, your screen should look like this.
![Screenshot showing what screen should look like at this step](/assets/initcomplete.png)
12. Click Finish and Exit

## Patch & Install Jetpack 6 on Target Device <br/>
Now that we’ve got SDKManager setup and the Jetson image files downloaded, we can move onto to installing Jetpack 6. Since the image files (BSP) we’ve downloaded are designed to work with a Nvidia Dev kit, we need to apply the patch specific to Aetina devices.

### Step 1: Download the patch files <br/>
1. Download the correct patch for your system. The Orin Nano, NX, and AGX have different files. You can download those directly from the OnLogic product page of your system.
2. Copy the patch into the Host PC path ```/home/nvidia/nvidia_sdk``` using file manager

### Step 1: Unzip the downloaded files in the /home/nvidia/nvidia_sdk directory <br/>
2. For the Patch ```sudo tar zxvf PATCH_[Your File Version].tar.gz```

### Step 3: Patch Update <br/>
1. Open the new folder that has appeared in the current directory. You went to the right place if you see a file named setup.sh.
2. Open a new terminal window in this folder or use cd to change the directory.
3. Run the command ```sudo ./setup.sh```
   - This may open a Nvidia License agreement window, if so click accept to continue the installation
   - You should see _Updated Successfully_ as the last line.

### Step 4: Target Jetson Setup <br/>
Setup the Jetson in recovery mode. If you’ve got a monitor hooked up to the Jetson device already, option 1 is the fastest.
- Option 1 (Terminal):
  - **On the Jetson:** Run the following command: ```sudo reboot --force forced-recovery```
- Option 2 (Hardware):
  - Locate the holes on the side of the Jetson chassis marked reset and recovery
  - Restart the system, While booting
    - Press the reset button, the press the recovery button
    - Release the reset button, the release the recovery button
  - This will boot the device in recovery mode

Even though the screen is not displaying on the Jetson, we need to make sure the USB device is still connected. **On the Host PC**, open terminal and run the command ```lsusb``` <br/><br/>
If you can still see a device with the name _Nvidia Corp. APX_, and does not have _L4T (Linux for Tegra) running on Tegra_ after it, the system has entered recovery mode correctly.
![Screenshot showing what screen should look like at this step](/assets/patch7.png)

### Step 5: Jetson Image Installation <br/>
1. On the Host PC, we will navigate to the BSP install directory: ```/home/nvidia/nvidia_sdk/Jetpack_6.0_Linux_JETSON_ORIN_NX_TARGETS/Linux_for_Tegra/```
    - Your directory will look slightly different if on a Nano or AGX
2. Open a new terminal window at this directory and run the following command:
   ```
   sudo ./tools/kernel_flash/l4t_initrd_flash.sh --external-device nvme0n1p1 -p "-c ./bootloader/generic/cfg/flash_t234_qspi.xml" -c ./tools/kernel_flash/flash_l4t_t234_nvme.xml --showlogs --network usb0 jetson-orin-nano-devkit nvme0n1p1
   ```
   - Make sure this stays formatted correctly, when copying to a terminal window
   - This will take a while and the Jetson may reboot a few times.

![Screenshot showing what screen should look like at this step](/assets/patch8.png)
4. Once this command finishes and you see _Flash is successful_, the Jetson device will reboot and finalize setup. The install is complete.

### Step 6: Verify Install Details on Jetson <br/>
1. To Verify Jetpack version we will run: ```cat /etc/nv_tegra_release```
    - This should show a message like the following. Using the [Jetpack Archive](https://developer.nvidia.com/embedded/jetpack-archive). We can decode this information to be R36 3.0 which corresponds to Jetpack 6.0 [L4T 36.3]
     ![Screenshot showing jetpack version verification](/assets/jetpackverify.png)
3. To Verify Patch installation we will run: ```cat /proc/device-tree/nvidia,dtsfilename```
    - If we see a filename with Aetina, that means the patch was loaded successfully. Since we see that the name of the file starts with R36_3_0 that matches the Jetpack info we saw above.
      ![Screenshot showing jetpack version verification](/assets/patchverify.png)
  
**We have now successfully upgraded our unit to Jetpack 6!**

## Addendum: Installing SDK Components using SDKManager <br/>
If you want to install SDK components using sdkmanager, we can now do that.
1. Make sure the Jetson PC is powered on and that we can see the device on the host PC (using ```lsusb```)
2. On the Host PC, open ```sdkmanager```. Make sure the Jetson appears as the target device, proceed to step 2
3. Select any of the SDK components you wish to install.
   
> [!WARNING]
> Unselect the Jetson Linux OS image, this is critical as we can corrupt the image we installed
4. Click Continue to Step 3
5. You will now be prompted to enter info about the system. The patching will have set the username and password on the Jetson to ‘nvidia’ for both. Make sure the connection type is set to USB and the address listed is 192.168.55.1.
![Screenshot showing result for addendum step 5](/assets/adden3.png)





