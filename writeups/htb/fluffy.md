---
title: Fluffy 
layout: page
permalink: /writeups/htb/fluffy
---

<img width="1033" height="696" alt="image" src="https://github.com/user-attachments/assets/41ba983f-9eb1-48c3-b50f-096a3d10638b" />

Looking at the nmap scan, we can see a couple of things. We see kerberos is running… maybe kerbroasting? SMB is running maybe there's something interesting in that. Let's investigate further. 

<img width="768" height="196" alt="image" src="https://github.com/user-attachments/assets/2ebf5e9a-4d6c-43c5-a553-fc71f9ee7286" />

Here we can see after listing the file shares with smbclient, we have read+write access to the IT file share. 

<img width="1014" height="279" alt="image" src="https://github.com/user-attachments/assets/57c32ca2-e27c-41a6-beb1-d22092b70578" />

There is a pdf in the file share named “Upgrade_Notice.pdf”. 

<img width="610" height="635" alt="image" src="https://github.com/user-attachments/assets/d174b30b-7ec0-4d3e-abc6-7b92813946be" />

The pdf is a list of CVEs scheduled to be patched in the next update. After researching each of these, CVE-2025-24071 seems to be the most interesting in this case. 

This vulnerability arises from automatic parsing of file paths in .library-ms files in Windows Explorer, which means we can pass in a UAC file path and potentially force NTLM authentication. Since we have write access over the ‘IT’ SMB share we have a method of delivering this malicious file.

<img width="611" height="740" alt="image" src="https://github.com/user-attachments/assets/43dcbdea-19e5-4263-98ec-90e427e13dca" />

We can clone in the PoC of this CVE then upload the file to the ‘IT’ share as I mentioned earlier. In addition, we will launch responder which should capture the NTLM Auth request from our victim user. 

<img width="788" height="194" alt="image" src="https://github.com/user-attachments/assets/c1a13fb4-417d-4704-8e8d-1742c53842ed" />

<img width="1077" height="309" alt="image" src="https://github.com/user-attachments/assets/62ae91c2-9007-4556-98cf-e94b2cd28021" />

We captured a hash of the user ‘p.agila’

After Cracking this hash, we are left with hashcat mode 5600 we are left with the credentials p.agila / prometheusx-303

<img width="1142" height="421" alt="image" src="https://github.com/user-attachments/assets/c5367c3d-74ca-47c6-80cf-5d3f3f4b6b67" />

After running bloodhound with either the first given credentials or the new ones we have just discovered after running the exploit, we see that p.agila is a member of the ‘service account managers’ group. In addition, she has genericAll rights to all three service accounts in the domain. 

<img width="594" height="220" alt="image" src="https://github.com/user-attachments/assets/5330f100-9dd1-4614-84e4-ec8808d5e712" />

We can then access these service accounts by adding p.agila to service accounts, abusing our genericAll privs from the “service account managers” group. Then we can use shadow credentials attack to access one of the service accounts, in this case “winrm_svc”. With this attack, I am explicitly mapping a certificate to this user and abusing my GenericWrite privs.

<img width="1073" height="81" alt="image" src="https://github.com/user-attachments/assets/76367771-d7d6-4d8c-89e2-951f11fddbf1" />

<img width="1066" height="340" alt="image" src="https://github.com/user-attachments/assets/a4924716-0ddc-4380-a734-9c0258f13e49" />

Now we have a valid NT hash that we can use to authenticate. If we back track to our nmap scan, we see port 5985 open. This, along with the obvious “winrm_svc” account being present means WinRM is actively running, so we can use evil-winrm.

<img width="1081" height="440" alt="image" src="https://github.com/user-attachments/assets/23b0ebf0-2905-4566-ad25-ab3c865679b6" />

We successfully authenticated with winrm and get user. 

The “winrm_svc” account has essentially nothing interesting associated with it. So, shifting our attention back to bloodhound to see if anything else catches my attention. 

<img width="1163" height="468" alt="image" src="https://github.com/user-attachments/assets/117a1644-7dce-4a37-9e16-45cf15ed24e8" />

We can see from this screenshot, “ca_svc” is part of the “Cert Publishers” group. What does this mean exactly?

Here we can see that “Members of this group are permitted to publish certificates to the directory.”

<img width="396" height="282" alt="image" src="https://github.com/user-attachments/assets/bbacf081-3e33-4ff7-b8af-95c5593e8a59" />

We can use certipy again to look for a vulnerable certificate template. We first need to repeat the same steps we used for “winrm_svc” to get authentication for the “ca_svc” service account:

<img width="1080" height="407" alt="image" src="https://github.com/user-attachments/assets/53484a8b-432c-4fec-9b09-24b79dcf3d8d" />

We can use certipy-ad find with the “-vulnerable” flag to look for vulnerable certificate templates. 

<img width="1038" height="267" alt="image" src="https://github.com/user-attachments/assets/09df799b-cde0-404d-8785-64018fbcc020" />

Viewing the output, we find that there's a certificate vulnerable to ESC16 with a security extension disabled. The exact security extension is OID 1.3.6.1.4.311.25.2.

What does this do? 
The output shows that the Security Extension is disabled, specifically, the OID 1.3.6.1.4.1.311.25.2 is not set. This OID is crucial for SID-based object mapping in ADCS. When 1.3.6.1.4.1.311.25.2 is enabled, certificates are mapped to Active Directory objects using the requester’s immutable SID. Without this extension, ADCS falls back to using the requester’s UPN, which can be changed if an attacker has GenericWrite permissions over the target object. This makes it easy to impersonate other users. To exploit this, we simply need to locate a certificate template we have enrollment rights over. The User template, in this case, allows all Domain Users to enroll, making it a possible candidate for the ESC16 attack.

<img width="1091" height="133" alt="image" src="https://github.com/user-attachments/assets/57320d11-f8af-42ae-a6f2-459d30254930" />

<img width="1109" height="186" alt="image" src="https://github.com/user-attachments/assets/6ff03ece-3000-4a95-b9dd-aafd19f56e1e" />

We first update the UPN of “ca_svc” to that of “administrator”. Then we request a certificate for “ca_svc” using the “user” template. 

<img width="1083" height="324" alt="image" src="https://github.com/user-attachments/assets/7d73a3c3-e0db-411c-bab1-ecefee82e966" />

Next, we must put the upn back to “ca_svc” and then we can force authentication as admin using the ‘administrator.pfx’ we made previously. 

We now have a hash for that of the admin account. We can use evil-winrm again to get the root flag for this box. 

<img width="1128" height="431" alt="image" src="https://github.com/user-attachments/assets/47b607b2-4ee1-407f-8e22-e9557580729d" />

Ez gg.
