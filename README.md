# BlueMoon-2021-Challenge-Writeup
Step-by-step walkthrough for the BlueMoon 2021 Challenge.

---

## 1. Host Discovery

We started by scanning the local subnet to identify the target's IP address:
'''bash
sudo nmap -sn 192.168.56.0/24
<img width="682" height="260" alt="557deedc-9d49-4f64-b495-ab75309222f1" src="https://github.com/user-attachments/assets/fa23c382-93e0-4b52-882e-1f8181210fd7" />
Discovered Target IP: 192.168.56.103

## 2. Reconnaissance & Enumeration

We scanned the target IP ('192.168.56.3') to identify any open ports and services.
<img width="773" height="248" alt="92c84f5f-22dc-4d32-9ba1-b8c2a48b69bd" src="https://github.com/user-attachments/assets/8caa78ce-78e8-4f13-b8c3-4940ae012ba2" />
Found: Port 21(FTP), port 22(SSH), and port 80(HTTP)
* **Port 21 (FTP):** We attempted anonymous login but it resulted in a '530 Permission Denied':
  '''bash
  ftp 192.168.56.103
  <img width="325" height="139" alt="519d471b-1d85-43c8-99e2-5e141ca86fac" src="https://github.com/user-attachments/assets/6e1c0448-0b31-4620-9eeb-fd7abf53f0b8" />
* **Port 80 (HTTP):** We use directory brute-forcing using 'gobuster' and found a hidden directory:
  '''bash
  gobuster dir -u http://192.168.56.103 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
  <img width="856" height="354" alt="0a9fb83d-cdeb-4d55-92be-2308ac175dea" src="https://github.com/user-attachments/assets/91004af2-a36b-4ac7-9fd6-2ae888af8e36" />
  Found: /hidden_text (Status 200)
  
## 3. Finding the Clues 

* Visiting "http://192.168.56.103/hidden_text" displayed a maintenance page with a link labeled "Thank You":.<img width="1277" height="895" alt="d0c9c13b-e82c-4bda-a88e-da07a11c6e4a" src="https://github.com/user-attachments/assets/235268a9-6046-41a2-8811-73b9f122edb5" />
* Clicking it led to a hidden image file named ".QR_C0d3.png":<img width="1279" height="894" alt="77d84fca-5d58-4cd8-981c-c05d22ebfdba" src="https://github.com/user-attachments/assets/b110309f-fdcf-4f9e-9b88-b1a2ead5f5fe" />
* Scanning the QR code revealed hidden text containing FTP credentials and a script snippet:<img width="1080" height="1318" alt="83b4eb80-3ab2-4d13-b49b-6491ba75638e" src="https://github.com/user-attachments/assets/37b41425-05c6-4971-9f9e-41e403531c45" />
  Username: userftp
  Password: ftpp@ssowrd

## 4. FTP Exploitation & Data Harvesting

* Using the newly discovered credentials, we logged into the FTP server:
  '''bash
  ftp 192.168.56.103
  <img width="605" height="265" alt="3866a2e4-e6ba-4479-afd6-fda332786a11" src="https://github.com/user-attachments/assets/af1eae76-3b6e-4ba2-9966-be382b8b81c8" />
* We found two files: 'information.txt' and 'p_lists.txt'. We downloaded them using 'get':<img width="1600" height="191" alt="06371337-ab20-473b-b858-07a9ee2a9894" src="https://github.com/user-attachments/assets/849e04f4-2753-499e-bb2a-035e2940c3d1" />
* 'information.txt' addressed a user named robin regarding password weaknesses while 'p_lists.txt' contained a wordlists of
  potential passwords:<img width="998" height="685" alt="0d16432a-0447-4c91-8520-7e27f49190c9" src="https://github.com/user-attachments/assets/9a01e645-9f02-4f63-ac87-bc2010f98dc6" />

## 5. Credential Brute Forcing & SSH Access

* Using the target username (robin) and the password wordlist (p_lists.txt), we used Hydra to brute-force the SSH service (Port 22):
  '''bash
  hydra -l robin -P p_lists.txt ssh://192.168.56.103
  <img width="1600" height="228" alt="27308f0f-d6c5-4de1-a5bf-62ab19f91388" src="https://github.com/user-attachments/assets/8c7b5f10-2d2f-4838-a042-4fa9a834e9c4" />
  Cracked Password: k4rv3ndh4nh4ck3r
* We logged into SSH using the credentials we have:<img width="674" height="311" alt="5820d868-b409-4add-b876-f5df4235c1e0" src="https://github.com/user-attachments/assets/e07455c0-e60c-49e5-a510-533f9556fc8e" />
* We found the first flag in 'user.txt' file:<img width="373" height="123" alt="3bd29845-1897-438a-a58b-b3956889eac4" src="https://github.com/user-attachments/assets/46292ff0-3488-4017-9932-625b15bb0a41" />
  Flag: Fl4g{u5er1r34ch3d5ucc355fully}

## 6. Privilege Escalation (To Jerry)

* We checked our available sudo privileges as 'robin':
  '''bash
  sudo -l
  <img width="870" height="101" alt="7a8e92b0-031a-4176-ba09-04da6035c1a8" src="https://github.com/user-attachments/assets/8830d80a-05cf-4b28-8ea4-abb82f67763a" />
* We found that 'robin' can execute 'home/robin/project/feedback.sh' as user 'jerry' without a password. Inspecting 'feedback.sh' revealed a command injection vulnerability where the user feedback variable was evaluated directly:
  ''bash
  $feedback 2>/dev/null
  <img width="553" height="215" alt="6202e1ae-be4c-4f47-9a98-1039238da0e7" src="https://github.com/user-attachments/assets/a8784f07-c9e7-4a9c-9f64-7c376d117de9" />
* We executed the script using 'sudo' and injected a shell command '/bin/bash' into the feedback prompt to spawn a shell a as 'jerry':
  <img width="538" height="195" alt="fbac7e3d-17e0-4061-8816-2d37b8cc0dcb" src="https://github.com/user-attachments/assets/b82735d8-9896-49d3-89e6-1d25d9d932d7" />

## 7. Privilege Escalation (To Root)

* Checking 'jerry's group id revealed membership in the 'docker' group. To escalate to root, we spun up an Alpine Docker container with the host's root filesystem (/) mounted to '/mnt'. This instantly dropped us into a  root shell:
  '''bash
  docker run -v /:/mnt --rm -it alpine chroot /mnt sh
  <img width="974" height="118" alt="7e358219-ab2c-4c6a-afd5-8a09a05c95cf" src="https://github.com/user-attachments/assets/f78a4b7d-d426-4e32-af16-56f9419a642c" />
* We then list the directory available and navigated to 'root.txt' file to retrieve the final flag:
  <img width="319" height="369" alt="1199ffda-d8df-483a-b936-08326081b5c0" src="https://github.com/user-attachments/assets/75793b7a-a3ce-41c0-99fd-2eefba2398e2" />
  Flag: Fl4g{r00t-H4ckTh3P14n3t0nc34g41n}
