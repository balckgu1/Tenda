**Buffer Overflow Vulnerability in Tenda FH1203 V2.0.1.3(1079)**

**Vulnerability Description:** A buffer overflow vulnerability exists in Tenda FH1203 V2.0.1.3(1079), which stems from a failure to properly validate the length of user input data in a call to the strcpy function in the /goform/R7WebsSecurityHandler file, and can be exploited by an attacker to execute arbitrary code on the system or cause a denial of service attack.

 

**Vulnerability Location:** R7WebsSecurityHandler function in /bin/httpd

**![img](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)**

When defining the v27 variable on line 29, the array length is fixed.

**![img](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image004.png)**

The vulnerability is caused by the fact that the src parameter in line 112 is entered by the user without limiting the size of the input, and the length validation of the src parameter in line 129 when copying src to v27 using the strcpy function is also not done.

 

**Exploit Replication (requires an ubuntu22 VM with qemu & binwalk installed, and IDA Pro with the mips plugin)**

1. Download the Tenda FH1203 V2.0.1.3(1079) firmware and unpack it using binwalk:

https://www.tenda.com.cn/download/detail-2317.html

Unpacked directory:

![img](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image006.png)

 

2. Use IDA Pro to open the unpacked file: bin/httpd, Ctrl+f on the left to search for the main function and enter the main function:

![img](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image008.png)

3. Patch the check_network function on line 22 of the main function, remove the while loop, and the result of the patch is as follows (here you can install the keypatcher plugin to facilitate the patch):

![img](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image010.png)

The corresponding assembly code is as follows:

![img](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image012.png)

4. Use IDA Pro to open lib/libCfm.so file, search for ConnectCfm function, after finding it, disassemble ConnectCfm function by f5, and patch this function to modify its return value to 1. The result after patching is shown below (the purpose of patch in steps 3-4 is for qemu simulation) (the purpose of patch in step 3-4 is for qemu simulation to run the firmware):

![img](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image014.png)

The corresponding assembly code is as follows:

![img](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image016.png)

 

5. Create a virtual NIC br0 in ubuntu, the IP address can be the current same network segment, my network segment here is 192.168.153.0/24, so I set br0 ip to 192.168.153.100:

**sudo brctl addbr br0**

**sudo ifconfig br0 192.168.153.100/24**

Use the ifconfig command after successful setup to see the following display:

![img](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image018.png)

6. Run httpd using qemu:

**sudo cp $(which qemu-mipsel-static) .**

**sudo chroot . ./qemu-mipsel-static ./bin/httpd**

After running httpd successfully, you can see the following interface:

![img](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image020.png)

7. Construct the POC as follows and save it as poc1203.py:

![img](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image021.png)

8. Running POC: `python3 poc1203.py` triggered a denial of service and the program crashed.

![img](file:///C:/Users/user/AppData/Local/Temp/msohtmlclip1/01/clip_image023.png)