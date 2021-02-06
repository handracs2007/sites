## Handra

### [Home](/) | Blog | [Disclaimer](/disclaimer) | [Terms and conditions](/tnc)

[<<go back](..)

### Check Certificate Revocation Status via CRL
In this post, we are going to learn on how to check certificate revocation status by using Certificate Revocation List (CRL). CRL though is not the only way for us to check for certificate revocation status. There is another protocol called Online Certificate Status Protocol (OCSP), that is more preferred in terms of doing certificate revocation checking. We will discuss on the OCSP in the next blog post.

---
#### Background
People and the internet are becoming more conscious on securing their information online. One of the security mechanism that is very common and used everywhere on the internet is the use of TLS to secure the HTTP transport (HTTPS). Another common use case that is becoming more prevalent is to digitally sign a document using digital certificate. Signing a document using digital certificate instead of printing it and using hand-signature not only help in reducing the paper wastage but also in digitalising the workflow. Everyone, wherever and whenever they are, can now sign documents easily  without the need to send papers everywhere. You may visit [SigningCloud](https://www.signingcloud.com/), one of the good digital signing solutions available on the internet.

The use of digital certificate though imposes another risk. Someone can use an abused digital certificate to be used on the internet. A TLS certificate might be mis-issued or the keys might be leaked, leading to the needs to revoke the certificate and stop it from being used further. It is the responsibility of the clients to check for the certificate revocation status before trusting the certificate for further usage. To do this, we can utilise Certificate Revocation List (CRL) that is maintained and regularly issued by the Certificate Authority (CA).

---
#### CRL
A CRL is a list of revoked certificates. This file contains the list of serial number and the revocation date of the revoked certificates. Based on the [RFC5280](https://tools.ietf.org/html/rfc5280), there are several reasons that a CA can choose when revoking a certificate:
 - unspecified (0)
 - keyCompromise (1)
 - cACompromise (2)
 - affiliationChanged (3)
 - superseded (4)
 - cessationOfOperation (5)
 - certificateHold (6)
 - removeFromCRL (8)
 - privilegeWithdrawn (9)
 - aACompromise (10)

Sometimes, a CA might just want to temporarily halt the use of the certificate. In this case, the certificate itself is not totally revoked, but instead its on-hold (revoke reason *certificateHold*).

Despite the usage, there is still a disadvantage in choosing CRL to do certificate revocation checking. As you might have noticed, CRL is file based, hence the size can be quite big depending on how many certificates have been revoked by the CA. The bigger the size of the CRLs, the longer it takes to loop through its content and check for the certificate revocation status.

Another disadvantage of using CRL is there is a gap between the time a certificate is revoked and the time a new CRL is generated. If a certificate is revoked before CA is scheduled to issue a new CRL, then during this gap period, the revoked certificate will still be trusted. Some clients as well cached the CRL to avoid re-downloading the CRL. In this case, when the downloaded CRL has not yet expired, the clients will not know that a certificate has been revoked due to the cached CRL is still being used for validation.

---
#### Scope
In this post, we are going to just see the CRL handling which is located at HTTP server. CRL can be located anywhere, such as in LDAP, but we will skip that and focus on HTTP only in this blog post.

---
### Get Through the Code
For this demo, I'll be using the Go programming language. We will run through step by step from reading a certificate file and retrieving the CRL endpoints from the certificate data itself.

Before we begin, I have downloaded an SSL certificate from a website https://spring.io/ (accessed: Jan 10, 2021). Below is the content of the SSL certificate.
```
-----BEGIN CERTIFICATE-----
MIIGPDCCBSSgAwIBAgIQCXrf438iChWuzzd/a4XpnDANBgkqhkiG9w0BAQsFADBN
MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMScwJQYDVQQDEx5E
aWdpQ2VydCBTSEEyIFNlY3VyZSBTZXJ2ZXIgQ0EwHhcNMjAwMzEwMDAwMDAwWhcN
MjEwNDI4MTIwMDAwWjB+MQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5p
YTESMBAGA1UEBxMJUGFsbyBBbHRvMR8wHQYDVQQKExZQaXZvdGFsIFNvZnR3YXJl
LCBJbmMuMQ8wDQYDVQQLEwZTcHJpbmcxFDASBgNVBAMMCyouc3ByaW5nLmlvMIIB
IjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0TX7HU9eE/IM4Z8Mqc8NS1Tq
9PPUrMtH6mId2/TG5hCTGgq1Ey/f98vEVE7p7NmzLdNTsr5SlGTEQ/RgXrakpOPX
hWIrTxCanD4734idB+XBfcOM7q1ilSF+lv+FKm0h0MlnKTrn+TrBbkcbNKeZ6KkN
r6L4IXOG6hrDXexPzhVdTezl+AU9lhrIGLZ61LDOwTMZMkQnvR5nkJaX/akpMVnv
+kay5faRpwF22KeyL4axrCOEUHzSEX5ylsTglnpeY3LzauDZixmBydxPJlk/FjIK
ZwBc4i8thsMOR6aAvSQ70QXeZ6Pe522vqDnjYml2aEzk8RquBnlWYGKc5KuUKwID
AQABo4IC5TCCAuEwHwYDVR0jBBgwFoAUD4BhHIIxYdUvKOeNRji0LOHG2eIwHQYD
VR0OBBYEFPu+4DDTmF/qTT9lgfKAfw0jfvcyMCEGA1UdEQQaMBiCCyouc3ByaW5n
LmlvgglzcHJpbmcuaW8wDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUF
BwMBBggrBgEFBQcDAjBrBgNVHR8EZDBiMC+gLaArhilodHRwOi8vY3JsMy5kaWdp
Y2VydC5jb20vc3NjYS1zaGEyLWc2LmNybDAvoC2gK4YpaHR0cDovL2NybDQuZGln
aWNlcnQuY29tL3NzY2Etc2hhMi1nNi5jcmwwTAYDVR0gBEUwQzA3BglghkgBhv1s
AQEwKjAoBggrBgEFBQcCARYcaHR0cHM6Ly93d3cuZGlnaWNlcnQuY29tL0NQUzAI
BgZngQwBAgIwfAYIKwYBBQUHAQEEcDBuMCQGCCsGAQUFBzABhhhodHRwOi8vb2Nz
cC5kaWdpY2VydC5jb20wRgYIKwYBBQUHMAKGOmh0dHA6Ly9jYWNlcnRzLmRpZ2lj
ZXJ0LmNvbS9EaWdpQ2VydFNIQTJTZWN1cmVTZXJ2ZXJDQS5jcnQwDAYDVR0TAQH/
BAIwADCCAQQGCisGAQQB1nkCBAIEgfUEgfIA8AB2APZclC/RdzAiFFQYCDCUVo7j
TRMZM7/fDC8gC8xO8WTjAAABcMU71FQAAAQDAEcwRQIgJ/nE2izuYSct+TPKHmmk
bWur+3YWcXHdQjngYtXS3tgCIQDdcG7Qzm4NUJ1CAipiw2M7MIsVl3ou6wFo8xbz
Gl37KwB2AFzcQ5L+5qtFRLFemtRW5hA3+9X6R9yhc5SyXub2xw7KAAABcMU71KsA
AAQDAEcwRQIgWNC+h5wkYkjl4xEgcGoz55LrMT5L6+vvICZ4+RTGPf8CIQCtCuzx
Ah9yo9Dudt85UKZVnRfHPbYF+s9alT3fO3gzOzANBgkqhkiG9w0BAQsFAAOCAQEA
YRul65az+hBXpTKvwOHFxJyrpDdX4ivzTp9kbPXkf2P3Se9rHwvrsz6P/YeV1Uln
053sVvFVNU3jLEoZtGnFigIkMnPmnrKJ3WH3Kq2Yf+/+DRoKt1jTiUjpMB8tp1UW
W2MZBPxsOj/nXVHWfFallVFC7opWs2lplHSHdzAVZn8DTqqJy0Y/KyVdxa0lIeQe
EL7NZnn5iymWnBAOPpNoLCaaVamGvTvyn5NRBmszaLR91Af4pGhn38QD7zvvOMyf
J/TM+hpq3XNmxzZgwDEztx8f8ax6NNk5ysKW0lPqhXFnVQ4O5p8kR7pzxP89U6Ck
Gm7mjTNXo4pZXoXL06AAEw==
-----END CERTIFICATE-----
```

I have the above certificate file saved as `spring.io.pem`.

---
#### Create Certificate Object
Once we have the certificate data, we can easily parse the certificate data and store it as a `Certificate` object through few simple steps.

First of all, we'll read the whole content of the file and store it into a variable called `fc`.
```go
// Read the certificate file content
fc, err := ioutil.ReadFile("spring.io.pem")
if err != nil {
    log.Printf("Failed to read certificate file: %s\n", err)
    return
}
```

Once we have read the file content, we will need to decode the content. We can simply use the function `Decode` provided in the `pem` package.
```go
// Decode the PEM file
block, _ := pem.Decode(fc)
if block == nil {
    log.Println("Failed to decode certificate data")
    return
}
```

And lastly, we will use the function `ParseCertificate` inside the package `x509` and send the bytes of the certificate data to be parsed. This function returns the certificate object that we can use for further operations.
```go
// Parse the certificate data
cert, err := x509.ParseCertificate(block.Bytes)
if err != nil {
    log.Printf("Failed to parse certificate data: %s\n", err)
    return
}
```

---
#### Get the CRL Endpoints
Now that we have the certificate object, we will need to retrieve the list of CRL endpoints. Lucky for us, Go has provided a convenient way to retrieve list of CRL Distribution Points.

The field `CRLDistributionPoints` contains the CRL information that we can use later on for certificate revocation status checking.
```go
// Retrieve the CRL URLs
crlUrls := cert.CRLDistributionPoints

if crlUrls == nil || len(crlUrls) == 0 {
    log.Println("Unable to check certificate revocation status. No CRL URL defined.")
    return
}
```

---
#### Certificate Revocation Checking
First of all, we will need to download the CRL from the HTTP server and read the CRL data. We are storing the downloaded CRL binary data into a variable `data`.
```go
// Let's download the crl
resp, err := http.Get(url)
if err != nil {
    log.Printf("Failed to download CRL from %s. Skipping this URL.\n", err)
    continue
}

// Download successful, read the body
data, err := ioutil.ReadAll(resp.Body)
if err != nil {
    log.Printf("Failed to read response body %s. Skipping this URL.\n", err)
    continue
}

// Close the connection
_ = resp.Body.Close()
```

Once we have the CRL data, we can use a function `ParseCRL` provided in package `x509` to easily parse the CRL data and get an instance of the `CertificateList`.
```go
// Parse the CRL data
crl, err := x509.ParseCRL(data)
if err != nil {
    log.Printf("Failed to parse CRL data: %s. Skipping this URL.\n", err)
    continue
}
```

We will then loop through each of the `RevokedCertificate` and do comparison based on the certificate serial number.
```go
certSn := cert.SerialNumber // The certificate serial number to check
revokedCerts := crl.TBSCertList.RevokedCertificates
wg := sync.WaitGroup{}

for _, revokedCert := range revokedCerts {
    revokedSn := revokedCert.SerialNumber

    // Spawn a new go-routine to speed-up the checking. Helpful for CRL that contains
    // a lot of items. You might want to limit the number of go-routines based on your
    // circumstances.
    go func() {
        wg.Add(1)
        if certSn.Cmp(revokedSn) == 0 {
            // The certificate is revoked since the serial number is inside the CRL
            revoked = true
        }
        wg.Done()
    }()
}

wg.Wait()
```
From the above code, I am spawning a lot of go-routines for the sake of speeding-up the checking. Instead of sequentially do a serial number comparison, we'll do it concurrently through multiple go-routines. This is useful if your CRL contains a huge number of revoked certificates. Of course, care must be taken especially if you want to limit the number of go-routines created, but that will be out of the scope of this post.

If the serial number can be found inside the CRL file, that means the certificate you are checking has been either revoked or put on-hold. You can then decide the next course of action based on the revocation status result.

---
That's all. It looks very easy and straight-forward. In the next follow-up article, we'll see how can we do certificate revocation status checking by using OCSP instead of CRL.

I hope this is helpful and the source code (including the docker related files) for this is available at Github [here](https://github.com/handracs2007/certcrl){:target=_blank}.

Cheers

---

> Copyright &copy; 2020-2021 Handra. All Rights Reserved.