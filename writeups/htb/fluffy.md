---
title: Fluffy 
layout: page
permalink: /writeups/htb/fluffy
---

![2025-07-23 17_18_36-Window.png](attachment:4ad43588-fea3-4a08-84c0-a50378e82ca0:2025-07-23_17_18_36-Window.png)

Looking at the nmap scan, we can see a couple of things. We see kerberos is running… maybe kerbroasting? SMB is running maybe there's something interesting in that. Let's investigate further. 

![2025-07-23 17_18_53-Window.png](attachment:c847597f-ef17-4b46-bb88-d123c002b182:2025-07-23_17_18_53-Window.png)

Here we can see after listing the file shares with smbclient, we have read+write access to the IT file share. 

![2025-07-23 17_27_45-Window.png](attachment:6f009e57-a030-442a-b2f0-1a8d43295c5a:2025-07-23_17_27_45-Window.png)

There is a pdf in the file share named “Upgrade_Notice.pdf”. 

![2025-07-23 17_27_28-Window.png](attachment:dda69f7c-8695-4738-abbb-3c49f1d07835:2025-07-23_17_27_28-Window.png)

The pdf is a list of CVEs scheduled to be patched in the next update. After researching each of these, CVE-2025-24071 seems to be the most interesting in this case. 

This vulnerability arises from automatic parsing of file paths in .library-ms files in Windows Explorer, which means we can pass in a UAC file path and potentially force NTLM authentication. Since we have write access over the ‘IT’ SMB share we have a method of delivering this malicious file.

![2025-07-25 21_41_37-xD [Running] - Oracle VirtualBox.png](attachment:bdfd19f1-3696-417d-855c-ab2cbb53d499:2025-07-25_21_41_37-xD_Running_-_Oracle_VirtualBox.png)

We can clone in the PoC of this CVE then upload the file to the ‘IT’ share as I mentioned earlier. In addition, we will launch responder which should capture the NTLM Auth request from our victim user. 

![2025-07-25 21_42_54-xD [Running] - Oracle VirtualBox.png](attachment:fe8d4bcc-0c66-4e36-ac2b-1aa7cae663a4:80ed3f69-ecdc-4059-8921-ae91f62cd8ac.png)

![2025-07-25 21_41_53-xD [Running] - Oracle VirtualBox.png](attachment:b220016f-297b-48a1-85d8-06ded83d9506:2025-07-25_21_41_53-xD_Running_-_Oracle_VirtualBox.png)

We captured a hash of the user ‘p.agila’

After Cracking this hash, we are left with hashcat mode 5600 we are left with the credentials p.agila / prometheusx-303

![2025-07-23 18_15_09-Window.png](attachment:1eaa9b36-f16f-459e-9c54-8347c3c04998:2025-07-23_18_15_09-Window.png)

After running bloodhound with either the first given credentials or the new ones we have just discovered after running the exploit, we see that p.agila is a member of the ‘service account managers’ group. In addition, she has genericAll rights to all three service accounts in the domain. 

![2025-07-25 21_50_55-xD [Running] - Oracle VirtualBox.png](attachment:d7cf2c15-fd5d-4a76-a39c-9f3e64ba8cec:2025-07-25_21_50_55-xD_Running_-_Oracle_VirtualBox.png)

We can then access these service accounts by adding p.agila to service accounts, abusing our genericAll privs from the “service account managers” group. Then we can use shadow credentials attack to access one of the service accounts, in this case “winrm_svc”. With this attack, I am explicitly mapping a certificate to this user and abusing my GenericWrite privs.

![2025-07-25 21_59_07-xD [Running] - Oracle VirtualBox.png](attachment:71334c2b-ddcb-4446-a58f-89da385415c6:2025-07-25_21_59_07-xD_Running_-_Oracle_VirtualBox.png)

![2025-07-25 21_59_19-xD [Running] - Oracle VirtualBox.png](attachment:957bcbd7-3491-4c1c-b485-e45c74de2a94:2025-07-25_21_59_19-xD_Running_-_Oracle_VirtualBox.png)

Now we have a valid NT hash that we can use to authenticate. If we back track to our nmap scan, we see port 5985 open. This, along with the obvious “winrm_svc” account being present means WinRM is actively running, so we can use evil-winrm.

![2025-07-23 20_36_09-xD [Running] - Oracle VirtualBox.png](attachment:f624eae8-19b5-4e80-ba2a-03875cd1cf2b:2025-07-23_20_36_09-xD_Running_-_Oracle_VirtualBox.png)

We successfully authenticated with winrm and get user. 

The “winrm_svc” account has essentially nothing interesting associated with it. So, shifting our attention back to bloodhound to see if anything else catches my attention. 

![2025-07-25 22_04_15-xD [Running] - Oracle VirtualBox.png](attachment:9e279d14-19fb-4b7d-8466-a8715e78e512:2025-07-25_22_04_15-xD_Running_-_Oracle_VirtualBox.png)

We can see from this screenshot, “ca_svc” is part of the “Cert Publishers” group. What does this mean exactly?

Here we can see that “Members of this group are permitted to publish certificates to the directory.”

![2025-07-25 22_07_30-xD [Running] - Oracle VirtualBox.png](attachment:57131e1c-6ec8-413c-9db3-580845ec97dc:2025-07-25_22_07_30-xD_Running_-_Oracle_VirtualBox.png)

We can use certipy again to look for a vulnerable certificate template. We first need to repeat the same steps we used for “winrm_svc” to get authentication for the “ca_svc” service account:

![2025-07-25 22_13_59-xD [Running] - Oracle VirtualBox.png](attachment:f13db534-7b6f-4e69-9cde-1b636c6c49ee:2025-07-25_22_13_59-xD_Running_-_Oracle_VirtualBox.png)

We can use certipy-ad find with the “-vulnerable” flag to look for vulnerable certificate templates. 

![2025-07-25 22_16_54-xD [Running] - Oracle VirtualBox.png](attachment:ecc8914c-64a5-4c5e-8e91-429e73ed93c6:2025-07-25_22_16_54-xD_Running_-_Oracle_VirtualBox.png)

Viewing the output, we find that there's a certificate vulnerable to ESC16 with a security extension disabled. The exact security extension is OID 1.3.6.1.4.311.25.2.

What does this do? 
The output shows that the Security Extension is disabled, specifically, the OID 1.3.6.1.4.1.311.25.2 is not set. This OID is crucial for SID-based object mapping in ADCS. When 1.3.6.1.4.1.311.25.2 is enabled, certificates are mapped to Active Directory objects using the requester’s immutable SID. Without this extension, ADCS falls back to using the requester’s UPN, which can be changed if an attacker has GenericWrite permissions over the target object. This makes it easy to impersonate other users. To exploit this, we simply need to locate a certificate template we have enrollment rights over. The User template, in this case, allows all Domain Users to enroll, making it a possible candidate for the ESC16 attack.

![2025-07-25 22_24_40-xD [Running] - Oracle VirtualBox.png](attachment:591f6341-3559-4dc1-8195-538d2b2d4304:2025-07-25_22_24_40-xD_Running_-_Oracle_VirtualBox.png)

![2025-07-25 22_24_55-xD [Running] - Oracle VirtualBox.png](attachment:2ed15120-6456-42af-9b6a-17cc4e2f6fe3:2025-07-25_22_24_55-xD_Running_-_Oracle_VirtualBox.png)

We first update the UPN of “ca_svc” to that of “administrator”. Then we request a certificate for “ca_svc” using the “user” template. 

![2025-07-25 19_52_46-xD [Running] - Oracle VirtualBox.png](attachment:74cb0aac-8ec5-4a68-9634-308509fadd3d:e3680d7d-4a88-41c4-bd74-fc75aa2bf62e.png)

Next, we must put the upn back to “ca_svc” and then we can force authentication as admin using the ‘administrator.pfx’ we made previously. 

We now have a hash for that of the admin account. We can use evil-winrm again to get the root flag for this box. 

![2025-07-25 22_30_19-xD [Running] - Oracle VirtualBox.png](attachment:6dd7eb9a-7d9e-44e1-9adc-8fc2a560c663:2025-07-25_22_30_19-xD_Running_-_Oracle_VirtualBox.png)

Ez gg.
