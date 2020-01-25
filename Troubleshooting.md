# Troubleshooting

## RoboRio Communications
### Overview
When the RoboRio is connected to your computer by USB cable, it acts as an ethernet/network connection and 
communication happens over TCP port 1735.  The RoboRio acts as a TCP socket server with the connected computer 
acting as the client.

Refrence [Network Tables Gist](https://gist.github.com/bb010g/99e590de76d651e5b09b) for detailed information 
on the FRC Network Tables protocol.
#### Initial Network Setup
The commands below will need to be run from an elevated privelege Powershell prompt (run as administrator).
##### Change Adapter Name and Alias
To make it easier to recognize the ethernet adapter on your computer lets change the alias:
First, get a list of adapters on your computer, run the following:
```powershell
Get-NetAdapter
````
Find the adapter with the interface descrpition of "National Instruments USBLAN Adapter".  Take the name of the
output and run the following (example if the corresponding name is "Ethernet 3"):
```powershell
Rename-NetAdapter -Name "Ethernet 3" -NewName "RoboRio"
```
### Common Problems
#### Basic IP Connectivity Test
Run the following command at a Powershell or Windows command prompt (RoboRio will always have an IP address of 172.22.11.2):
ping 172.22.11.2
If you don't see a reply from the RoboRio then you don't have basic network connectivity.
#### DHCP Not Enabled on Ethernet Adapter
If DHCP isn't enabled on the ethernet adapter that shows up when the RoboRio is plugged in (interface description 
will be "National Instruments USBLAN Adapter" then communications will be broken.  Run the following Powershell 
command (assuming the adapter was renamed to "RoboRio"):
```powershell
Get-NetIPInterface -InterfaceAlias RoboRio
```
In the output the "Dhcp" column should show "Enabled".  If it doesn't, run the following command:
```powershell
Set-NetIPInterface -InterfaceAlias RoboRio -Dhcp Enabled
```
DHCP is now enabled.  Re-run the "Basic IP Connectivity Test" again.

#### Windows Firewall is Blocking
Change the network connection profile to "Private" so you don't have to diable the Windows Defender firewall 
which isn't secure (assuming the adapter was renamed to "RoboRio"):
```powershell
Set-NetConnectionProfile -InterfaceAlias "RoboRio" -NetworkCategory Private
```
This should get connectivity working through the firewall.  If you still don't have connectivity then add a rule 
for TCP-1735 to be allowed on the network.  Run the following (assuming the adapter was renamed to "RoboRio"):
```powershell
New-NetFirewallRule -DisplayName "FRC Network Tables Protocol" -Profile Private -Direction Outbound -Action Allow -RemotePort 1735 -Protocol TCP
```
