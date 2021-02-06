## Handra

### [Home](/) | Blog | [Disclaimer](/disclaimer) | [Terms and conditions](/tnc)

[<<go back](..)

### Extract PKCS#12 using openssl
In this post, we are going to run through the steps necessary to extract a PKCS#12 digital certificate to TWO (2) separate files. At the end, we will have a **pem** file that contains the public certificate and a **key** file that contains the private key of the certificate.

Before continuing, you have to ensure that you have **openssl** command installed in your system. You may visit this link <https://www.openssl.org/source/> in order to download OpenSSL if you have not had one.

In this post, I will be using OpenSSL version 1.1.1f that has been installed in my system.
```bash
handra@nebula  ~/Downloads/demop12  openssl version
OpenSSL 1.1.1f  31 Mar 2020
```

---
I already have a file named **demop12.p12** here inside my environment that I enrolled using a testing CA system.

```bash
handra@nebula  ~/Downloads/demop12  ll
total 4.0K
-rw-rw-r-- 1 handra handra 2.7K Sep 28 21:23 demop12.p12
```

The password of this file is **foo123**. Below is the information contained inside the **demop12.p12** file.
```bash
handra@nebula  ~/Downloads/demop12  openssl pkcs12 -info -in demop12.p12 
Enter Import Password:
MAC: sha1, Iteration 102400
MAC length: 20, salt length: 20
PKCS7 Data
Shrouded Keybag: pbeWithSHA1And3-KeyTripleDES-CBC, Iteration 51200
Bag Attributes
    friendlyName: demop12
    localKeyID: D0 97 8C 31 AD 7F 87 9F 69 26 C7 59 5D 17 E1 95 21 E1 2E 8C 
Key Attributes: <No Attributes>
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIHjME4GCSqGSIb3DQEFDTBBMCkGCSqGSIb3DQEFDDAcBAirrDCWSAdoOAICCAAw
DAYIKoZIhvcNAgkFADAUBggqhkiG9w0DBwQIvA7ye3T5jDcEgZDdN0/t+p7UGkEt
St2Sf1gXoObY9kAF5SlqIydcv3angcCoVfBwPaWhCUT0o5Y5msVodoNFLFWlBCAZ
OXczUSwELAYTrsVyaKpLU/AE3OIjYR54uHd+yhoLD0GAQO2ICk5js2OrQ9tSKgf/
Zj6Uq/N+fJRlNtVFPjcDpPeiYxbR8+PPUpNpWC0PivCVbFPmXFU=
-----END ENCRYPTED PRIVATE KEY-----
PKCS7 Encrypted data: pbeWithSHA1And40BitRC2-CBC, Iteration 51200
Certificate bag
Bag Attributes
    friendlyName: demop12
    localKeyID: D0 97 8C 31 AD 7F 87 9F 69 26 C7 59 5D 17 E1 95 21 E1 2E 8C 
subject=CN = demop12

issuer=UID = c-0fd6a8f76519120e5, CN = ManagementCA, O = EJBCA Container Quickstart

-----BEGIN CERTIFICATE-----
MIIDNTCCAZ2gAwIBAgIUR4biv3HyQk62vYih+ul+qE0n/Z0wDQYJKoZIhvcNAQEL
BQAwYTEjMCEGCgmSJomT8ixkAQEME2MtMGZkNmE4Zjc2NTE5MTIwZTUxFTATBgNV
BAMMDE1hbmFnZW1lbnRDQTEjMCEGA1UECgwaRUpCQ0EgQ29udGFpbmVyIFF1aWNr
c3RhcnQwHhcNMjAwOTI4MTMyMTA2WhcNMjIwOTI4MTMxMzE0WjASMRAwDgYDVQQD
DAdkZW1vcDEyMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEUiG6+FyglHA/3K8q
owLnOzPwRSXdoJo1QFmvOFSOjliZTFCqJJhj4vc11lXWoIKo1PoLtqSDSUCcBaMf
Gu0gyKN/MH0wDAYDVR0TAQH/BAIwADAfBgNVHSMEGDAWgBTb1DXMmZI0nDym8CDo
PhjhPexMEDAdBgNVHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwQwHQYDVR0OBBYE
FNCXjDGtf4efaSbHWV0X4ZUh4S6MMA4GA1UdDwEB/wQEAwIF4DANBgkqhkiG9w0B
AQsFAAOCAYEAt+4Bs88JyyaE5HFy2JvMIs2l9pxRYmZTmr24iQRnBYNuiS0xpjg4
tDIFRCB7vraid0+mirbuRF157eK5IHwZXyafTlmY8t/2Jubu/usdiCWPIdzMUeC/
BuVx0pIaBHAn/MMVPrRb2FDFvi0BdYcw4fdfmSfG7JSphwZdHygUyV6CwgYzmLYf
CS298eZT6y3wt0Q2mJbyzR/b5lodeFiJkT4Ms5b3AB+v4Yujlu/LAWaNsFeXsj7i
xm/M9DIEnNZllmRUwr5dJPU/GIHBae8xDYOjCMqXx7dbg57yjwMR0A/VyWzPXudj
Cf1GzLz0Mod9jP96YUHQVV/VlYi8JYUjFjsSSHuEiDDXAOOawXR7ITR8McD/4tVN
zoXO7scYr1Sc6nURBDeJubP/OBrQglg5KdrUrZOUlsi9Ilb4vocAKdSFlIYoMlZp
U+/DwkYOghCscLiVo8VQksT4JiCj7MFbXfphuSTQG2LQdBA2w8ZWV15vrV9QoXY9
VHr8Xt6yOUgy
-----END CERTIFICATE-----
Certificate bag
Bag Attributes
    friendlyName: ManagementCA
subject=UID = c-0fd6a8f76519120e5, CN = ManagementCA, O = EJBCA Container Quickstart

issuer=UID = c-0fd6a8f76519120e5, CN = ManagementCA, O = EJBCA Container Quickstart

-----BEGIN CERTIFICATE-----
MIIEszCCAxugAwIBAgIUDEyeAz3LvwpOu0sBmEu2f6zWfwwwDQYJKoZIhvcNAQEL
BQAwYTEjMCEGCgmSJomT8ixkAQEME2MtMGZkNmE4Zjc2NTE5MTIwZTUxFTATBgNV
BAMMDE1hbmFnZW1lbnRDQTEjMCEGA1UECgwaRUpCQ0EgQ29udGFpbmVyIFF1aWNr
c3RhcnQwHhcNMjAwOTI4MTMyMTA2WhcNMzAwOTI4MTMyMTA2WjBhMSMwIQYKCZIm
iZPyLGQBAQwTYy0wZmQ2YThmNzY1MTkxMjBlNTEVMBMGA1UEAwwMTWFuYWdlbWVu
dENBMSMwIQYDVQQKDBpFSkJDQSBDb250YWluZXIgUXVpY2tzdGFydDCCAaIwDQYJ
KoZIhvcNAQEBBQADggGPADCCAYoCggGBAOxYA5KU6C4eheG5UGmKiwzYvgOv0IH4
Z8dIIvZWrbTzbHDaYIGIlB77A8f5ZIWZrL+LnevNMhIYklv1GkfFQZBL7VkdCONx
rfXgk0fYM051zxt9M4VsSPb0/CVqpPasDWCwt3t1Skat8Vce0SIf5zvQp8aMbJO9
d5afp0tSZV/dP8Gzx2UCjeZopYQWs3cf8648kFXKQ0urKUCK5gU2TncYXCj6ECth
QwYQlIXWO49aOFk7+xiuFj0yTbKwqmIY5HlQTx8/TNlk+nQvdaTOUk0QpGEBSX0K
1mDEYupVkv3EHYjaqjCS7IFfm2utGLIGbdE/ftodyE9mWgpqO4+SEPoGRcqgGnaj
qwb5tnAR6EMu+dYmVGm9tx0rdkmvmAsr84ncJ2V1sm25qAjUZwH+g37SzAGflOji
/ApikLcXR7s2i7hR/WXvcX+fmreRkNRfdZBPKfLKf4RaRKrjMEsk5lDaPqGoLwB7
jTUJydKw5PtPAmnMVr+L2+aSumKYppN0YwIDAQABo2MwYTAPBgNVHRMBAf8EBTAD
AQH/MB8GA1UdIwQYMBaAFNvUNcyZkjScPKbwIOg+GOE97EwQMB0GA1UdDgQWBBTb
1DXMmZI0nDym8CDoPhjhPexMEDAOBgNVHQ8BAf8EBAMCAYYwDQYJKoZIhvcNAQEL
BQADggGBAAT+fVyfkvk/6A4QqmIo67H28V3NneHdQBqMyQ1LakG+rRJIRm3Q9u17
O4GkmjUjc+BKccfRRsRFVdnY+Szur+CCQZEUJGybRuVIViS2LmY0Y6mdO6UIVWp6
YdXtqKeFQXntFSmWRPnsCN2I82g120qOqDKkdNUVLuQThIXt43F+DVZ2oRE7wfor
ez4eN+dbNOGuO/UudnhgweXt9e5BuZgQ7+Weo+FlTc1kIl9uYqrkyCmj/ah38DB4
B8OCKyNUJi0K9V9ql2O93Qd8FZtl1pY3EdLbQ86FXlVuUb9qcsT0/3Z7/APukCV2
YrTUbuJYAlHXCCjnOqQnMxdQa728HSiTN6LHo/xyer6aXlXjFbFG446SbqampABI
M982bWNNDJ0if2gY/dMX3mRLqP6X1r5qqwiacrSnLt6IP9Pa1Z3KXGBoNQ53fIKA
FzfnwbisPM1hUA0+96YcAPhSNC0UnSnjqygkZYLdVFnp2uvHwMbwWXQN4cgylfLx
fUJbuPQRXw==
-----END CERTIFICATE-----
```
---
#### Extracting the public certificate
To extract only the public certificate, you may use the following command:
```bash
openssl pkcs12 -in <input.p12> -out <output.pem> -nokeys
```
You need to replace the **\<input.p12\>** with your p12 file and **\<output.pem\>** with your desired output file name. Below is the command I executed in my environment against my p12 file.
```bash
openssl pkcs12 -in demop12.p12 -out demop12.pem --nokeys
```
It will ask you the password of the **p12** file. You have to enter the correct password to continue. Upon successful extraction, a new file named **demop12.pem** will be created.

Below is the output of my command.
```bash
handra@nebula  ~/Downloads/demop12  openssl pkcs12 -in demop12.p12 -out demop12.pem --nokeys
Enter Import Password:

handra@nebula  ~/Downloads/demop12  ll
total 8.0K
-rw-rw-r-- 1 handra handra 2.7K Sep 28 21:23 demop12.p12
-rw------- 1 handra handra 3.3K Sep 28 21:40 demop12.pem
```

Below is the content of the **demop12.pem** file.
```bash
handra@nebula  ~/Downloads/demop12  cat demop12.pem 
Bag Attributes
    friendlyName: demop12
    localKeyID: D0 97 8C 31 AD 7F 87 9F 69 26 C7 59 5D 17 E1 95 21 E1 2E 8C 
subject=CN = demop12

issuer=UID = c-0fd6a8f76519120e5, CN = ManagementCA, O = EJBCA Container Quickstart

-----BEGIN CERTIFICATE-----
MIIDNTCCAZ2gAwIBAgIUR4biv3HyQk62vYih+ul+qE0n/Z0wDQYJKoZIhvcNAQEL
BQAwYTEjMCEGCgmSJomT8ixkAQEME2MtMGZkNmE4Zjc2NTE5MTIwZTUxFTATBgNV
BAMMDE1hbmFnZW1lbnRDQTEjMCEGA1UECgwaRUpCQ0EgQ29udGFpbmVyIFF1aWNr
c3RhcnQwHhcNMjAwOTI4MTMyMTA2WhcNMjIwOTI4MTMxMzE0WjASMRAwDgYDVQQD
DAdkZW1vcDEyMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEUiG6+FyglHA/3K8q
owLnOzPwRSXdoJo1QFmvOFSOjliZTFCqJJhj4vc11lXWoIKo1PoLtqSDSUCcBaMf
Gu0gyKN/MH0wDAYDVR0TAQH/BAIwADAfBgNVHSMEGDAWgBTb1DXMmZI0nDym8CDo
PhjhPexMEDAdBgNVHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwQwHQYDVR0OBBYE
FNCXjDGtf4efaSbHWV0X4ZUh4S6MMA4GA1UdDwEB/wQEAwIF4DANBgkqhkiG9w0B
AQsFAAOCAYEAt+4Bs88JyyaE5HFy2JvMIs2l9pxRYmZTmr24iQRnBYNuiS0xpjg4
tDIFRCB7vraid0+mirbuRF157eK5IHwZXyafTlmY8t/2Jubu/usdiCWPIdzMUeC/
BuVx0pIaBHAn/MMVPrRb2FDFvi0BdYcw4fdfmSfG7JSphwZdHygUyV6CwgYzmLYf
CS298eZT6y3wt0Q2mJbyzR/b5lodeFiJkT4Ms5b3AB+v4Yujlu/LAWaNsFeXsj7i
xm/M9DIEnNZllmRUwr5dJPU/GIHBae8xDYOjCMqXx7dbg57yjwMR0A/VyWzPXudj
Cf1GzLz0Mod9jP96YUHQVV/VlYi8JYUjFjsSSHuEiDDXAOOawXR7ITR8McD/4tVN
zoXO7scYr1Sc6nURBDeJubP/OBrQglg5KdrUrZOUlsi9Ilb4vocAKdSFlIYoMlZp
U+/DwkYOghCscLiVo8VQksT4JiCj7MFbXfphuSTQG2LQdBA2w8ZWV15vrV9QoXY9
VHr8Xt6yOUgy
-----END CERTIFICATE-----
Bag Attributes
    friendlyName: ManagementCA
subject=UID = c-0fd6a8f76519120e5, CN = ManagementCA, O = EJBCA Container Quickstart

issuer=UID = c-0fd6a8f76519120e5, CN = ManagementCA, O = EJBCA Container Quickstart

-----BEGIN CERTIFICATE-----
MIIEszCCAxugAwIBAgIUDEyeAz3LvwpOu0sBmEu2f6zWfwwwDQYJKoZIhvcNAQEL
BQAwYTEjMCEGCgmSJomT8ixkAQEME2MtMGZkNmE4Zjc2NTE5MTIwZTUxFTATBgNV
BAMMDE1hbmFnZW1lbnRDQTEjMCEGA1UECgwaRUpCQ0EgQ29udGFpbmVyIFF1aWNr
c3RhcnQwHhcNMjAwOTI4MTMyMTA2WhcNMzAwOTI4MTMyMTA2WjBhMSMwIQYKCZIm
iZPyLGQBAQwTYy0wZmQ2YThmNzY1MTkxMjBlNTEVMBMGA1UEAwwMTWFuYWdlbWVu
dENBMSMwIQYDVQQKDBpFSkJDQSBDb250YWluZXIgUXVpY2tzdGFydDCCAaIwDQYJ
KoZIhvcNAQEBBQADggGPADCCAYoCggGBAOxYA5KU6C4eheG5UGmKiwzYvgOv0IH4
Z8dIIvZWrbTzbHDaYIGIlB77A8f5ZIWZrL+LnevNMhIYklv1GkfFQZBL7VkdCONx
rfXgk0fYM051zxt9M4VsSPb0/CVqpPasDWCwt3t1Skat8Vce0SIf5zvQp8aMbJO9
d5afp0tSZV/dP8Gzx2UCjeZopYQWs3cf8648kFXKQ0urKUCK5gU2TncYXCj6ECth
QwYQlIXWO49aOFk7+xiuFj0yTbKwqmIY5HlQTx8/TNlk+nQvdaTOUk0QpGEBSX0K
1mDEYupVkv3EHYjaqjCS7IFfm2utGLIGbdE/ftodyE9mWgpqO4+SEPoGRcqgGnaj
qwb5tnAR6EMu+dYmVGm9tx0rdkmvmAsr84ncJ2V1sm25qAjUZwH+g37SzAGflOji
/ApikLcXR7s2i7hR/WXvcX+fmreRkNRfdZBPKfLKf4RaRKrjMEsk5lDaPqGoLwB7
jTUJydKw5PtPAmnMVr+L2+aSumKYppN0YwIDAQABo2MwYTAPBgNVHRMBAf8EBTAD
AQH/MB8GA1UdIwQYMBaAFNvUNcyZkjScPKbwIOg+GOE97EwQMB0GA1UdDgQWBBTb
1DXMmZI0nDym8CDoPhjhPexMEDAOBgNVHQ8BAf8EBAMCAYYwDQYJKoZIhvcNAQEL
BQADggGBAAT+fVyfkvk/6A4QqmIo67H28V3NneHdQBqMyQ1LakG+rRJIRm3Q9u17
O4GkmjUjc+BKccfRRsRFVdnY+Szur+CCQZEUJGybRuVIViS2LmY0Y6mdO6UIVWp6
YdXtqKeFQXntFSmWRPnsCN2I82g120qOqDKkdNUVLuQThIXt43F+DVZ2oRE7wfor
ez4eN+dbNOGuO/UudnhgweXt9e5BuZgQ7+Weo+FlTc1kIl9uYqrkyCmj/ah38DB4
B8OCKyNUJi0K9V9ql2O93Qd8FZtl1pY3EdLbQ86FXlVuUb9qcsT0/3Z7/APukCV2
YrTUbuJYAlHXCCjnOqQnMxdQa728HSiTN6LHo/xyer6aXlXjFbFG446SbqampABI
M982bWNNDJ0if2gY/dMX3mRLqP6X1r5qqwiacrSnLt6IP9Pa1Z3KXGBoNQ53fIKA
FzfnwbisPM1hUA0+96YcAPhSNC0UnSnjqygkZYLdVFnp2uvHwMbwWXQN4cgylfLx
fUJbuPQRXw==
-----END CERTIFICATE-----
```
You can see that the certificate content is exactly the same as the one inside the **demop12.p12** file shown in the above steps.

---
#### Extracting the private key
Now that we managed to extract the public certificate, we can proceed with extracting the private key. To do that, we can simply issue the following command:
```bash
openssl pkcs12 -in <input.p12> -out <output.key> --nocerts
```
You need to replace the **\<input.p12\>** with your p12 file and **\<output.key\>** with your desired output file name. Below is the command I executed in my environment against my p12 file.
```bash
openssl pkcs12 -in demop12.p12 -out demop12.key --nocerts
```
It will ask you the password of the **p12** file. You have to enter the correct password to continue. Upon successful extraction, a new file named **demop12.key** will be created. The difference on this steps is that, you will be asked to enter the password THREE (3) times. First is to open the P12 file, the second and third is the password to encrypt the private key.

Below is the output of my command.
```bash
handra@nebula  ~/Downloads/demop12  openssl pkcs12 -in demop12.p12 -out demop12.key --nocerts
Enter Import Password:
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:

handra@nebula  ~/Downloads/demop12  ll
total 12K
-rw------- 1 handra handra  537 Sep 28 21:46 demop12.key
-rw-rw-r-- 1 handra handra 2.7K Sep 28 21:23 demop12.p12
-rw------- 1 handra handra 3.3K Sep 28 21:40 demop12.pem
```

Below is the content of the **demop12.key** file.
```bash
handra@nebula  ~/Downloads/demop12  cat demop12.key 
Bag Attributes
    friendlyName: demop12
    localKeyID: D0 97 8C 31 AD 7F 87 9F 69 26 C7 59 5D 17 E1 95 21 E1 2E 8C 
Key Attributes: <No Attributes>
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIHjME4GCSqGSIb3DQEFDTBBMCkGCSqGSIb3DQEFDDAcBAgfGpILPldCbwICCAAw
DAYIKoZIhvcNAgkFADAUBggqhkiG9w0DBwQIBdteJ0jfqN4EgZC8XVnMrm26WXTX
bRdjlVUBpTqJRetgPsLSaQhqoXsnVGcv2n55Q0Vvv/hLGSf01Mek3D5ywJ/sdGyU
V3Dop7KcBMg6w36Ppf0GImBeNhY0raCwoCwxBwKR5HvlhJqsNiNXFP3crUVAVmui
8HYiVDOtkcOaRfxAu8S5Vwc0yQz89RfumjIyEyB217WkwjrTTfw=
-----END ENCRYPTED PRIVATE KEY-----
```

However, as you might have noticed, the private key is in encrypted format. If this is not what you want, you can simply add a new parameter **--nodes** to the **openssl** command as shown below:
```bash
openssl pkcs12 -in <input.p12> -out <output.key> --nocerts --nodes
```
Same like previous, you need to replace the **\<input.p12\>** with your p12 file and **\<output.key\>** with your desired output file name. Below is the command I executed in my environment against my p12 file.
```bash
openssl pkcs12 -in demop12.p12 -out demop12-plain.key --nocerts --nodes
```
It will ask you the password of the **p12** file. You have to enter the correct password to continue. Upon successful extraction, a new file named **demop12-plain.key** will be created.

Below is the output of my command.
```bash
handra@nebula  ~/Downloads/demop12  openssl pkcs12 -in demop12.p12 -out demop12-plain.key --nocerts --nodes
Enter Import Password:

handra@nebula  ~/Downloads/demop12  ll 
total 16K
-rw------- 1 handra handra  537 Sep 28 21:46 demop12.key
-rw-rw-r-- 1 handra handra 2.7K Sep 28 21:23 demop12.p12
-rw------- 1 handra handra 3.3K Sep 28 21:40 demop12.pem
-rw------- 1 handra handra  391 Sep 28 21:54 demop12-plain.key
```

Below is the content of the **demop12-plain.key** file.
```bash
handra@nebula  ~/Downloads/demop12  cat demop12-plain.key 
Bag Attributes
    friendlyName: demop12
    localKeyID: D0 97 8C 31 AD 7F 87 9F 69 26 C7 59 5D 17 E1 95 21 E1 2E 8C 
Key Attributes: <No Attributes>
-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQgOP3zRocKtyQRccWP
bV9K2c+HIfPrsnSidyIWR6qKBt6hRANCAARSIbr4XKCUcD/cryqjAuc7M/BFJd2g
mjVAWa84VI6OWJlMUKokmGPi9zXWVdaggqjU+gu2pINJQJwFox8a7SDI
-----END PRIVATE KEY-----
```
---
That's it. I hope it helps anyone who needs to do this. This is usually useful for some applications that do not provide a way for you to just provide a P12 file, but separate public certificate and private key files.

Cheers

---
> Copyright &copy; 2020-2021 Handra. All Rights Reserved.