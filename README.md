#Simple Scan - a minimal software to scan from network#

##Introduction##
WSD (Web Services on Devices) is a common protocol used by network-based scanners that is not natively supported on Linux.
Because of this, most persons cannot scan directly from MFPs (multi-function printer) to their Linux machines.

Simple scan implements a small subset of the complex WSD protocol.
While being VERY simple, it has enough functionality to successfully initiate scan and save resultant image, at least on the Xerox WorkCentre 3315 MFP.

This software should work on any operating system on which Python 2.7.x is available.

This software is not related in any way to the GNU software of the same name.

##Customizing the 'scan' script for your use##

I have no idea if, after customizing this script it will work for you - but if it does, please open an issue, so other can share the insight you gained on your scanner. Also, please open an issue if you can get your modified script to work - I may be able to help.

We will need to following information:

- What is the IP number or host name of your printer ?
- Is your MFP/scanner using the WSD protocol at all ?
- What port is used for scanning.
- What is the URL used to access WSD on your printer ?


###Finding the IP or host name of your printer:###
Normally, you can find this information by looking at the settings of the printer.
Also, you might find this information by looking at your central router.


###Finding whether your printer uses WSD:###
There are few ways, which we will explore below.
In any case, if your scanner does NOT work with WSD, this software is not of any use to you.

So, here are the various methods:

  - The easy way: finding this information in the documentation of your printer/manufacturer site, etc
  - Another easy way: use the WS-discovery protocol.  
    This network-protocol can discover WSD-capable devices.   
    For example,you can get the [python-ws-discovery](https://github.com/andreikop/python-ws-discovery.git) project from github, and run "python-ws-discovery/wsdiscovery/cmdline.py". This should report the IP number of your scanner. Also,note the port number.
  - The final alternative is to record and analyze the network traffic of a scan-session, probably initiated by a windows machine, The good news is that the analysis will get us all the answers we need to proceed. The bad news is that the procedure is a bit complex. Consider this a learning experience :-)  
    A detailed 'how-to' for the network scan is present below. 

###Finding the port used for scanning:###
If you run the [python-ws-discovery](https://github.com/andreikop/python-ws-discovery.git), described above, it will show you the port number. 

Another approach is to run nmap on the IP of the printer. The command will look like this:
   nmap -p- <IP of your scanner>
nmap will probably report many ports, and the port that we need is one of those.
Like before, the final alternative is to record and analyze the network traffic of a scan-session, as detailed below.


###Recording and analyzing the network traffic###
This procedure requires some knowledge, effort and maybe dedicated hardware.However,it is the best procedure to give us ALL the answers we need. Also, if you have never done this sort of thing, you might find it really interesting,as we will be looking at TCP/IP traffic, which is arguably the fundamental technology driving the Internet.

So, here is the procedure:

1. Capture the session of a working scan client  
   The detailed explanation of how to perform this task is beyond this document, but Google (or DuckDuckGo) is your friend. We strive to record a complete interaction between the scanner and a working client. Therefore, we need to start recording packets, then scan a page, then stop recording. 
   Here are a few points that might help:
   
   - We strive to produce a .pcap or .pcapng file.
   - Normally,those files are produced using the "Wireshark" or "tcpdump" software, but there are other alternatives.
   - The procedure is different when capturing wired or wireless communications.
   - For wired communications, the devices [Throwing Star LAN Tap](https://www.greatscottgadgets.com/throwingstar/).  
     Another option is "SharkTap Network Sniffer", which can be bought on Amazon. I am not affiliated in anyway with any of those devices and there are many other alternatives.
   - When performing the recording, try to record only the network traffic going to the printer. This will make the analysis simpler. Also, if you need to transfer the recording to other persons, it won't contain unintended additional information.
   - Do not scan anything personal, for the same reason. 
   - To be on the safe side, make more than one recording.

2. Open the recording for analysis:
   Open the .pcap or .pcapng file with Wireshark. You should see a long list of packets
3. Filter for HTTP packets
   We are looking for packets that belong to the WSD protocol. Unfortunately, Wireshark does not know how to filter for this protocol. However, as WSD, like many other protocols,is build on top of another protocol: HTTP. So, we will filter by HTTP. This means that some of the packets we see will not be relevant. Filtering to see HTTP packet is simply done by typing "http" at the top bar and pressing "Enter". You will now hopefully see many packets, all having "HTTP..." in the "Protocol" column.
4. Getting the information we need:
   Looking at the very last column, titled "info", we will see, for some of the packets something like "POST /wsd/scan".  
   This is the path! - specifically in this example, the path is "/wsd/san". And we can see to which IP and port this packet is sent - this will be the host and the port of the scanner. 
5. Export the HTTP packets
   We might need to customize the actual commands parameters (which are stored in the .xml files). For this end,we will now export the packets: In Wireshark, choose from the menu bar at the top: "File" --> "Export Objects" --> "HTTP". Then select "Save all", select a directory and click on "Open" to save all the packets.
   
###Modify the 'scan' script to fit your scanner###
  Using any editor, change the values of the variables "host", "port" and "path".
Step 5: Run the modified "./scan" script
  The scanner should start scanning and save the image as "image.jiff".
  Troubleshooting TBD

###Creating new .xml files###
TBD

# Author

**Shalom Mitz** - [shalommmitz](https://github.com/shalommmitz)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE ) file for details.
