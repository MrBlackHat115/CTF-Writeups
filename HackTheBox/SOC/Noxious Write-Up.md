# Scenario:

<img width="575" height="350" alt="image" src="https://github.com/user-attachments/assets/996eea53-39d5-4835-aff1-79758a3f21c5" />

## The IDS device alerted us to a possible rogue device in the internal Active Directory network. The Intrusion Detection System also indicated signs of LLMNR traffic, which is unusual. It is suspected that an LLMNR poisoning attack occurred. The LLMNR traffic was directed towards Forela-WKstn002, which has the IP address 172.17.79.136.  A limited packet capture from the surrounding time is provided to you, our Network Forensics expert.  Since this occurred in the Active Directory VLAN, it is suggested that we perform network threat hunting with the Active Directory attack vector in mind, specifically focusing on LLMNR poisoning.

- **Task 1**
  - It's suspected by the security team that there was a rogue device in Forela's internal network running responder tool to perform an LLMNR Poisoning attack. Please find the malicious IP Address of the machine.
  
    - **Answer:** `172.17.79.135`
    - **Explanation and Breakdown:**

        <img width="239" height="55" alt="image" src="https://github.com/user-attachments/assets/5059e269-f68f-401f-a472-2c16455e87a3" />

        First, we filter the traffic to display only LLMNR traffic, which operates over UDP port 5355. This helps isolate name resolution requests and responses from the rest of the network traffic. 
        Wireshark Display Filter:
        - `udp.port == 5355`

        <img width="1623" height="119" alt="image" src="https://github.com/user-attachments/assets/2aded4b8-b27f-4375-9c68-51014a35ed61" />
        

        After applying the filter, we observe that Forela-Wkstn002 sends an LLMNR Standard Query requesting the hostname DCC01. This query is essentially asking:
        - "Who is DCC01? What is its IP address?"
        
        Since DCC01 is the legitimate owner of that hostname, it is expected to respond with its IP address. The packet capture confirms that DCC01 sends a legitimate LLMNR response.
        However, the capture also shows Forela-Wkstn001 responding to the exact same query. 
        This behavior is suspicious because Forela-Wkstn001 does not own the hostname DCC01 and therefore should not be answering requests for it.

        In conclusion, the malicious host is **172.17.79.135** because it responded to an LLMNR query intended for DCC01, even though it is not the legitimate owner of that hostname.
        

- **Task 2**
  - What is the hostname of the rogue machine?

      - **Answer:** `kali`
      - **Explanation and Breakdown:**

      <img width="1585" height="122" alt="image" src="https://github.com/user-attachments/assets/896bfbbd-9092-4610-977e-6d4029d6a564" />
      
      First, we filter the traffic to display only DHCP packets. DHCP requests contain information about devices joining the network, including the hostname they provide to the DHCP server. Wireshark Display Filter:
      
       - `dhcp`
        
      <img width="307" height="61" alt="image" src="https://github.com/user-attachments/assets/48de8b64-ae00-4909-a929-a1adedc0cd77" />
        
      After applying the filter, locate a DHCP Request packet and inspect its details. Expand the Dynamic Host Configuration Protocol (Request) section and look for the Host Name option. 
      This field contains the hostname that the client identifies itself with when requesting an IP address. 
      In the DHCP Request, the Host Name field is **kali**, indicating that the requesting device identifies itself as **kali**.

- **Task 3**
  - Now we need to confirm whether the attacker captured the user's hash and it is crackable!! What is the username whose hash was captured?

    - **Answer:** `john.deacon`
    - **Explanation and Breakdown:**
    
    <img width="450" height="83" alt="image" src="https://github.com/user-attachments/assets/25780204-5ee9-491d-813a-13be6c73140c" />

    To determine whether the attacker captured a user's NTLM hash, we filtered the network traffic for NTLM authentication packets.
    In Wireshark, we used the following display filter:
    
    - `ntlmssp`
    
    This filter displays packets related to NTLM authentication. We then used the Find Packet option in Wireshark to search through the NTLM traffic and identify the username associated with the captured hash.
    The username found in the NTLM authentication exchange was:
    
    - `john.deacon`
    
    This confirms that the attacker captured an NTLM hash belonging to the user **john.deacon**, which can potentially be cracked offline if the password is weak.
    
  
- **Task 4**
  - In NTLM traffic we can see that the victim credentials were relayed multiple times to the attacker's machine. When were the hashes captured the First time?

      - **Answer:** `2024-06-24 11:18:30`
      - **Explanation and Breakdown:**

      <img width="550" height="97" alt="image" src="https://github.com/user-attachments/assets/787c6940-3213-4de3-8d56-6df597a0a605" />

      To determine when the victim's NTLM hashes were captured for the first time, We filtered the traffic using:
    
      - `ntlmssp`
    
      This allowed us to view NTLM authentication exchanges between the victim and the attacker's machine. By examining the NTLM authentication packets and checking their timestamps, we identified the earliest instance where the victim's credentials were relayed and captured.
    
      <img width="518" height="55" alt="image" src="https://github.com/user-attachments/assets/035a9836-e85e-4643-a5e3-1b1d701c08da" />
    
      The first captured NTLM hash occurred at:
    
      - `2024-06-24 11:18:30`
    
      This timestamp represents the first time the attacker successfully captured the victim's NTLM authentication response.

- **Task 5**
  - What was the typo made by the victim when navigating to the file share that caused his credentials to be leaked?
  
    - **Answer:** `DCC01`
    - **Explanation and Breakdown:**

    <img width="415" height="75" alt="image" src="https://github.com/user-attachments/assets/526bdbf6-ce4a-4bc6-8616-a400ea92b48f" />

    To identify the typo that caused the victim's credentials to be leaked, we analyzed LLMNR traffic in Wireshark. We filtered the packets using:
    
    - `ip.addr == 172.17.79.135 && llmnr`

    <img width="578" height="93" alt="image" src="https://github.com/user-attachments/assets/65f9309b-032a-4f2d-8640-0b0d7880ff16" />

    This allowed us to view the LLMNR requests generated by the victim machine. The traffic showed that the victim attempted to access a file share using the hostname:
    
    - `DCC01`

- **Task 6**
  - To get the actual credentials of the victim user we need to stitch together multiple values from the ntlm negotiation packets. What is the NTLM server challenge value?

      - **Answer:** `601019d191f054f1`
      - **Explanation and Breakdown:**

      <img width="162" height="77" alt="image" src="https://github.com/user-attachments/assets/1405e65e-6222-483f-876c-32724e1a68cb" />

      To find the NTLM server challenge value, we analyzed the NTLM authentication negotiation packets in Wireshark. We filtered the traffic using:
      
      - `ntlmssp`

      <img width="756" height="36" alt="image" src="https://github.com/user-attachments/assets/c2def9a1-af74-4378-b0a8-f2aed264a879" />

      Then, we looked for the SMB2 authentication exchange. Specifically, we identified the packet containing:
      
      - `SMB2 (Server Message Block Protocol version 2)`
      - `STATUS_MORE_PROCESSING_REQUIRED`
      - `Session Setup Response`
      - `MessageId: 2`

      <img width="376" height="47" alt="image" src="https://github.com/user-attachments/assets/885de17a-b804-4bd5-a599-4a723b091599" />
      
      This packet contains the NTLMSSP Challenge message sent by the server during the NTLM authentication handshake.
      Inside the NTLMSSP Challenge packet, we located the Server Challenge field:
      
      - `601019d191f054f1`

- **Task 7**
  - Now doing something similar find the NTProofStr value.

    - **Answer:** `c0cc803a6d9fb5a9082253a04dbd4cd4`
    - **Explanation and Breakdown:**

    <img width="575" height="108" alt="image" src="https://github.com/user-attachments/assets/6706fcb8-2995-4b37-a267-f5f09e4cc1a5" />

    To find the NTProofStr value, we continued analyzing the NTLM authentication packets in Wireshark. We filtered the traffic using:
    
    - `ntlmssp`
    
    Then, we searched within the NTLM authentication packets for:
    
    - `NTProofStr`

    <img width="437" height="38" alt="image" src="https://github.com/user-attachments/assets/c6c8a3e2-54a3-42f1-9a5e-a3150047481d" />
    
    The NTProofStr field is located inside the NTLMv2 Response and is an important component used when reconstructing the NTLMv2 hash for offline password cracking. The value we found was:
    
    - `c0cc803a6d9fb5a9082253a04dbd4cd4`

- **Task 8**
  - To test the password complexity, try recovering the password from the information found from packet capture. This is a crucial step as this way we can find whether the attacker was able to crack this and how quickly.

    - **Answer:** `NotMyPassword0K?`
    - **Explanation and Breakdown:**

    To test whether the captured NTLMv2 hash could be cracked, we used the information extracted from the packet capture and attempted an offline password recovery attack.

    <img width="897" height="322" alt="image" src="https://github.com/user-attachments/assets/4542cce2-666c-4699-a618-2ec2dcadefba" />

    The captured NTLMv2 hash was saved in the format required by Hashcat:
    
    - `username::domain:NTProofStr:ServerChallenge:NTLMv2Response`
    
    We then used Hashcat with the NTLMv2 hash mode:
    
    - `hashcat -m 5600 password.txt rockyou.txt`
    
    - Where:
    
      - `-m 5600` specifies NetNTLMv2 cracking mode.
      - `password.txt` contains the captured NTLMv2 challenge-response hash.
      - `rockyou.txt` is the wordlist used to test possible passwords.
    
    Hashcat successfully recovered the password:
    
    - `NotMyPassword0K?`

- **Task 9**
  - Just to get more context surrounding the incident, what is the actual file share that the victim was trying to navigate to?

      - **Answer:** `\\DC01\DC-Confidential`
      - **Explanation and Breakdown:**

      <img width="221" height="54" alt="image" src="https://github.com/user-attachments/assets/253b7959-ae23-4ec2-ab56-14b5ce9fea77" />

      To determine the file share the victim was attempting to access, we analyzed the SMB traffic in the packet capture using Wireshark. We filtered for SMB2 traffic:
      
      - `smb2`

      <img width="523" height="59" alt="image" src="https://github.com/user-attachments/assets/d71d3b99-e592-416c-8cd2-7cfa1ae4e9eb" />
      
      Then, we examined the SMB2 Tree Connect Request packets. The Tree Connect Request is used by a client to connect to a shared folder on an SMB server and contains the target share path. 
      Within the packet details, we located the share path:
      
      - `\\DC01\DC-Confidential`

      This shows that the victim was attempting to access the confidential file share hosted on the domain controller (DC01). The failed navigation attempt likely caused the LLMNR request, allowing the attacker to respond and capture the victim's NTLM authentication information.
      
