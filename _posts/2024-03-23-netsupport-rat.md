---
title: NetSupport RAT
date: 2024-03-23 11:15:00 +0530
categories: [Malware]
tags: [rat, netsupport, decoding]
---


## NetSupport RAT Decoding and Analysis

![](assets/images/netsupport_rat/netsupport_rat_decoding.png)

NetSupport Manager is a legitimate application designed to facilitate remote technical support or provide assistance with remote computers. Unfortunately, this application has been exploited and repurposed as a Remote Access Trojan (RAT) for malicious campaigns.

I utilized Python to decode the sample and extract the malicious URLs. Please note that this is not the complete sample analysis from an execution perspective. I drew inspiration after [`watching`](https://youtu.be/CIg4TXFJRK0), but opted to leverage Python to assess its efficacy in processing the payload, resulting in the creation of a jupyter [notebook](https://github.com/zen29d/zen29d.github.io/blob/main/assets/other/NetSupport_RAT_Decoding.ipynb).


`Download` [`Sample`](https://bazaar.abuse.ch/sample/befc7ebbea2d04c14e45bd52b1db9427afce022d7e2df331779dae3dfe85bfab/?ref=embee-research.ghost.io)

First, will load the sample by reading the contents of the file.
![](assets/images/netsupport_rat/load_sample.png)

I verified the hash on [VirusTotal](https://www.virustotal.com/gui/file/2d48b04eb26654eaf3c4431d190257e46c409339426d75e0f0e2ac074dfbab6f) to assess its maliciousness and cross-validate the sample.

![](assets/images/netsupport_rat/virustotal.png) 

You can observe *name obfuscation*, but disregarding the naming convention and focusing on the logic reveals the utilization of *subtraction decoding* with a value of `787`.
![](assets/images/netsupport_rat/load_code.png)

I utilized regex in Python to extract the key and payload, which comprise continuous numeric values. It took some trial and error to refine the regex pattern, as this was my first time using regex in Python.
![](assets/images/netsupport_rat/extract_payload_1.png)

After performing subtraction decoding, obtained PowerShell code, which undergoes two-stage wrapping processes involving Advanced Encryption Standard (AES) and compression. Upon inspecting the code, it becomes evident that AES in Electronic Code Book (ECB) mode is used for encryption followed by GZip compression.
![](assets/images/netsupport_rat/decoding_payload_1.png)

If we examine the decoded code snippet, it reveals the presence of two variables containing base64-encoded data, which are certainly encrypted using a key. Notably, certain key components are highlighted, aiding in the identification of AES encryption parameters such as AES mode, Initialization Vector (IV), encryption key, and the encrypted payload. Subsequently, the decrypted code undergoes GZip decompression.
![](assets/images/netsupport_rat/decoded_stage_1.png)

Now, let's see how we can accomplish this with Python. We'll once again use regex to extract the base64 encoded data. The regex pattern will extract two base64 encoded strings, with the first one likely being the encrypted code and the second one representing the key as its size is 256 bytes,`len(b64decode(aes_key))*8`.
![](assets/images/netsupport_rat/extract_base64.png)

I utilized the cipher module to decrypt the payload. Dropped the first 16 bytes from the payload, which represent the IV. Then, initialized AES with ECB mode. After decryption, knowing that the data would undergo decompression, I dumped the hexadecimal content to identify the magic bytes of Gzip, which are `1F 8B`. This was confirmed by checking the file type.

> _In Electronic Codebook mode, the Initialization Vector is not required. ECB mode does not use an IV because it encrypts each block of plaintext independently, without any dependence on previous blocks._
{: .prompt-tip }

![](assets/images/netsupport_rat/decryption.png)

Decompression was fairly straightforward. I converted the result to a string for further analysis since the decompressed output is in byte stream format.
![](assets/images/netsupport_rat/decompression.png)

We once again received the PowerShell code, which again used subtraction decoding. Additionally, also observed some plaintext within the content.

![](assets/images/netsupport_rat/code_stage_2.png)

This marks the final stage of payload extraction and decoding...
![](assets/images/netsupport_rat/stage_3.png)

Voila! we have malicious URLs.
![](assets/images/netsupport_rat/ioc.png)

Here's a summarized overview of the process:
1. **Initial Analysis**: Extracted the initial payload from the VBScript file and performed subtraction decoding, revealing the PowerShell script.

2. **Base64 Decoding**: From the PowerShell script, the base64 encoded data was decoded, resulting in encrypted code.

3. **Decryption**: AES decryption was performed in ECB mode using the base64 decryption key.

4. **Decompression**: Then, the decrypted data was decompressed using Gzip, revealing PowerShell code once again.

5. **Final Decoding**: Subtraction decoding was performed on the final stage extracted payload, leading to the successful extraction of malicious URLs.
