https://defender268.medium.com/mastering-nmap-essential-and-advanced-commands-115b6b37fc10

- Ping scan: This will perform a ping scan on the IP address 192.168.1.1 to check if it’s online.
  - `nmap -sn <target>`

- TCP SYN Scan: lso known as half-open scanning, this command sends TCP SYN packets to identify open ports on a target system.
  - nmap -sS <target>

- UDP Scan: This command focuses on UDP ports, often overlooked but crucial for some services.
  - nmap -sU <target>