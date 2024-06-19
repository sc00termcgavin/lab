
[ASUS RT-AX86U-Pro VLAN Docs](https://www.asus.com/us/support/faq/1049415/)

[UniFi SwitchPort Docs](https://help.ui.com/hc/en-us/articles/9761080275607-UniFi-Network-Creating-Virtual-Networks-VLANs)
## IP's 

 Name         | VLAN ID | Subnet          | Gateway      | Domain     | 
--------------|---------|-----------------|--------------|------------|
 Main         | 21      | 192.168.21.0/24 | 192.168.21.1 | Ian        |
 Roomate      | 22      | 192.168.22.0/24 | 192.168.22.1 | N/a        |
 IOT          | 23      | 192.168.23.0/24 | 192.168.23.1 | N/a        | 
 Guest        | 24      | 192.168.24.0/24 | 192.168.24.1 | N/a        |
 Management   | 1       | 10.0.1.0/24     | 10.0.1.1     | N/a        |
 Fake WAN     | 10      | 10.0.10.0/24    | 10.0.10.1    | N/a        |
 Work LAN     | 20      | 10.0.20.0/24    | 10.0.20.1    | tooter.Ian |
 Security     | 50      | 10.0.50.0/24    | 10.0.50.1    | tooter.Ian |
 Isolated LAN | 99      | 10.0.99.0/24    | 10.0.99.1    | N/a        |

# process
- Configure the VLANs on the Layer 2 Managed Switch
- Configure the VLANs on the Router
- Set Up Trunking Between the Router and the Switch
- Assign Gateway IP Addresses to the VLAN Interfaces on the Router
- Verify the Configuration



