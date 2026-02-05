### assign a fixed IP
To set a fixed IP in DietPi, use the dietpi-config tool via SSH or terminal: navigate to Network Options: Adapters, select your adapter (Ethernet/WiFi), change from DHCP to STATIC, enter your desired IP details, and apply the changes. A reboot is required to apply the new IP. 
Steps to Assign a Fixed IP on DietPi:
Open Configuration: Run dietpi-config in the terminal.
Navigate to Network Settings: Go to Network Options: Adapters.
Select Adapter: Choose Ethernet (eth0) or WiFi (wlan0).
Change Mode: Change the mode from DHCP to STATIC.
Configure IP: Enter the desired Static IP, Subnet Mask (usually 255.255.255.0), and Gateway (router IP).
Apply Changes: Select Apply and let the system restart networking.
Reboot: Reboot the system to ensure the changes take effect. 
