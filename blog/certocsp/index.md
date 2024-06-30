## Handra

### [Home](/) | Blog | [Disclaimer](/disclaimer) | [Terms and conditions](/tnc)

[<<go back](..)

### Check Certificate Revocation Status via OCSP
In this post, we are going to learn on how to check certificate revocation status by using Online Certificate Status Protocol (OCSP). For revocation checking via CRL, you can refer to this [blog post](/blog/certcrl).

---
#### Background
People and the internet are becoming more conscious on securing their information online. One of the security mechanism that is very common and used everywhere on the internet is the use of TLS to secure the HTTP transport (HTTPS). Another common use case that is becoming more prevalent is to digitally sign a document using digital certificate. Signing a document using digital certificate instead of printing it and using hand-signature not only help in reducing the paper wastage but also in digitalising the workflow. Everyone, wherever and whenever they are, can now sign documents easily  without the need to send papers everywhere. You may visit [SigningCloud](https://www.signingcloud.com/), one of the good digital signing solutions available on the internet.

The use of digital certificate though imposes another risk. Someone can use an abused digital certificate to be used on the internet. A TLS certificate might be mis-issued or the keys might be leaked, leading to the needs to revoke the certificate and stop it from being used further. It is the responsibility of the clients to check for the certificate revocation status before trusting the certificate for further usage. To do this, we can utilise Online Certificate Status Protocol (OCSP) that is updated regularly by the Certificate Authority (CA) with its OCSP responder.

---
#### OCSP
OCSP is a protocol that is based on HTTP and used to return the revocation status of a digital certificate. An OCSP responder may return the following statuses:
 - *Good*. This indicates that the certificate being checked is not revoked.
 - *Revoked*. This indicates that the certificate being checked is already revoked.
 - *Unknown*. This indicates that the certificate being checked is not known by the OCSP responder. Note though sometimes an OCSP responder might return *Unauthorized* instead for unknown certificates.

---
#### Scope
In this post, we are going to just see the OCSP handling which is located at HTTP server.

---
### Get Through the Code
For this demo, I'll be using the Go programming language. We will run through step by step from reading a certificate file and connecting to the OCSP responder itself to check for the certificate status.

Before we begin, I have downloaded an SSL certificate from a website https://spring.io/ (accessed: Dec 12, 2021). Below is the content of the SSL certificate.
```
-----BEGIN CERTIFICATE-----
MIIGrjCCBZagAwIBAgIQD9F0JnMk5rO5z/HR149CrTANBgkqhkiG9w0BAQsFADBP
MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMSkwJwYDVQQDEyBE
aWdpQ2VydCBUTFMgUlNBIFNIQTI1NiAyMDIwIENBMTAeFw0yMTA0MTAwMDAwMDBa
Fw0yMjA0MTQyMzU5NTlaMGMxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9y
bmlhMRIwEAYDVQQHEwlQYWxvIEFsdG8xFTATBgNVBAoTDFZNd2FyZSwgSW5jLjEU
MBIGA1UEAwwLKi5zcHJpbmcuaW8wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQC+kqKViVNKbqFkK3YlPR8n44TNuuTAYgIwKrvbx1V+k85jWazF9DHI+lFA
rtwtAfXmWyD5SgX+8XYFzPcPVVXiGWlFpLIWJNTKbS7J3aVNr+QoMBqTzIl3XXl5
pP8DIeGrHJdZKv37iFd1LrCXU03mOjLSASSQqLr2tTzG8S1Op3ibuEHDh6/MdLZD
CAqB3zagaeCrUNsRlW1955TcBZzS7+j6sb6TyL49mOiIZqtHUfU44KKmX0kvz4wU
nAhuU0n2Q8HVkbYpKFSotnMp4c734EQpzfNjtjET+0+dprKtsCP5gRSNISScOxm7
phFdax+mBeNF+CBO85CgJLEdR7PrAgMBAAGjggNwMIIDbDAfBgNVHSMEGDAWgBS3
a6LqqKqEjHnqtNoPmLLFlXa59DAdBgNVHQ4EFgQUwmLp872u4jYWRSjg4WPTymHJ
McIwIQYDVR0RBBowGIILKi5zcHJpbmcuaW+CCXNwcmluZy5pbzALBgNVHQ8EBAMC
BaAwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMIGLBgNVHR8EgYMwgYAw
PqA8oDqGOGh0dHA6Ly9jcmwzLmRpZ2ljZXJ0LmNvbS9EaWdpQ2VydFRMU1JTQVNI
QTI1NjIwMjBDQTEuY3JsMD6gPKA6hjhodHRwOi8vY3JsNC5kaWdpY2VydC5jb20v
RGlnaUNlcnRUTFNSU0FTSEEyNTYyMDIwQ0ExLmNybDA+BgNVHSAENzA1MDMGBmeB
DAECAjApMCcGCCsGAQUFBwIBFhtodHRwOi8vd3d3LmRpZ2ljZXJ0LmNvbS9DUFMw
fQYIKwYBBQUHAQEEcTBvMCQGCCsGAQUFBzABhhhodHRwOi8vb2NzcC5kaWdpY2Vy
dC5jb20wRwYIKwYBBQUHMAKGO2h0dHA6Ly9jYWNlcnRzLmRpZ2ljZXJ0LmNvbS9E
aWdpQ2VydFRMU1JTQVNIQTI1NjIwMjBDQTEuY3J0MAwGA1UdEwEB/wQCMAAwggF+
BgorBgEEAdZ5AgQCBIIBbgSCAWoBaAB2ACl5vvCeOTkh8FZzn2Old+W+V32cYAr4
+U1dJlwlXceEAAABeLnAeGsAAAQDAEcwRQIhAJkb6Xz1MP0s1SN0c6FVNAfgJ2nt
+7GZHLiPhhzrimJnAiBQqAOXwv/TfjPDFNzLYPB14raHaa0J4p0k6GoMqZD9oAB2
ACJFRQdZVSRWlj+hL/H3bYbgIyZjrcBLf13Gg1xu4g8CAAABeLnAeEIAAAQDAEcw
RQIgNccLEi5/iK1yI62FrnlIfIAa28lcMQryLoO3DXmdeAUCIQCGVegsJSbbhVmz
Ktedqtaqnf1q7I5r64xY52IQX8hUNwB2AFGjsPX9AXmcVm24N3iPDKR6zBsny/ee
iEKaDf7UiwXlAAABeLnAeJQAAAQDAEcwRQIgMJ5JqsbWy9IFZcJDgizUvfbj3TT3
IPmAvqugeZ+udycCIQCCR2Ot+AQYdZly43JHVjRXvf8Ok6KtQs82dGG0unI9fTAN
BgkqhkiG9w0BAQsFAAOCAQEAfOR9tTYxgQXimta95d6MQtuyAEJm0O2VF2t0BE/c
N9UyzPWjY4YiRFiLXOmFZ1zqY454rMwdSQlcsT7gmhLHgN3n1a/iuNFGiQYmR+hO
ACkZK13rloV1ULqnRz15XojrbFh2x/3TdTJUB91vQ5KRPfZ2+LfdoNbOB7o8re2M
RZRtay5FpI7HPHNxgVQwfvkGHKhvp0AcGHx0hV7muqRGtKZ1gUNMs2TT/WLn7dL5
IAPfvW2njgJM7EZ5Dp8QCsr5SBzZOtJTlg8oXumNhyQZwDOrN3GL6zpINHAc+ksz
zk6W7NQ6yuDi3R03QX30B+hUuVSaD1a/bD2G2yc6yya+lA==
-----END CERTIFICATE-----
```

I have the above certificate file saved as `spring.io.pem`.

I have as well downloaded the issuer certificate of the TLS certificate (accessed: Dec 12, 2021). Below is the content of the issuer certificate:
```
-----BEGIN CERTIFICATE-----
MIIE6jCCA9KgAwIBAgIQCjUI1VwpKwF9+K1lwA/35DANBgkqhkiG9w0BAQsFADBh
MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMRkwFwYDVQQLExB3
d3cuZGlnaWNlcnQuY29tMSAwHgYDVQQDExdEaWdpQ2VydCBHbG9iYWwgUm9vdCBD
QTAeFw0yMDA5MjQwMDAwMDBaFw0zMDA5MjMyMzU5NTlaME8xCzAJBgNVBAYTAlVT
MRUwEwYDVQQKEwxEaWdpQ2VydCBJbmMxKTAnBgNVBAMTIERpZ2lDZXJ0IFRMUyBS
U0EgU0hBMjU2IDIwMjAgQ0ExMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKC
AQEAwUuzZUdwvN1PWNvsnO3DZuUfMRNUrUpmRh8sCuxkB+Uu3Ny5CiDt3+PE0J6a
qXodgojlEVbbHp9YwlHnLDQNLtKS4VbL8Xlfs7uHyiUDe5pSQWYQYE9XE0nw6Ddn
g9/n00tnTCJRpt8OmRDtV1F0JuJ9x8piLhMbfyOIJVNvwTRYAIuE//i+p1hJInuW
raKImxW8oHzf6VGo1bDtN+I2tIJLYrVJmuzHZ9bjPvXj1hJeRPG/cUJ9WIQDgLGB
Afr5yjK7tI4nhyfFK3TUqNaX3sNk+crOU6JWvHgXjkkDKa77SU+kFbnO8lwZV21r
eacroicgE7XQPUDTITAHk+qZ9QIDAQABo4IBrjCCAaowHQYDVR0OBBYEFLdrouqo
qoSMeeq02g+YssWVdrn0MB8GA1UdIwQYMBaAFAPeUDVW0Uy7ZvCj4hsbw5eyPdFV
MA4GA1UdDwEB/wQEAwIBhjAdBgNVHSUEFjAUBggrBgEFBQcDAQYIKwYBBQUHAwIw
EgYDVR0TAQH/BAgwBgEB/wIBADB2BggrBgEFBQcBAQRqMGgwJAYIKwYBBQUHMAGG
GGh0dHA6Ly9vY3NwLmRpZ2ljZXJ0LmNvbTBABggrBgEFBQcwAoY0aHR0cDovL2Nh
Y2VydHMuZGlnaWNlcnQuY29tL0RpZ2lDZXJ0R2xvYmFsUm9vdENBLmNydDB7BgNV
HR8EdDByMDegNaAzhjFodHRwOi8vY3JsMy5kaWdpY2VydC5jb20vRGlnaUNlcnRH
bG9iYWxSb290Q0EuY3JsMDegNaAzhjFodHRwOi8vY3JsNC5kaWdpY2VydC5jb20v
RGlnaUNlcnRHbG9iYWxSb290Q0EuY3JsMDAGA1UdIAQpMCcwBwYFZ4EMAQEwCAYG
Z4EMAQIBMAgGBmeBDAECAjAIBgZngQwBAgMwDQYJKoZIhvcNAQELBQADggEBAHer
t3onPa679n/gWlbJhKrKW3EX3SJH/E6f7tDBpATho+vFScH90cnfjK+URSxGKqNj
OSD5nkoklEHIqdninFQFBstcHL4AGw+oWv8Zu2XHFq8hVt1hBcnpj5h232sb0HIM
ULkwKXq/YFkQZhM6LawVEWwtIwwCPgU7/uWhnOKK24fXSuhe50gG66sSmvKvhMNb
g0qZgYOrAKHKCjxMoiWJKiKnpPMzTFuMLhoClw+dj20tlQj7T9rxkTgl4ZxuYRiH
as6xuwAwapu3r9rxxZf+ingkquqTgLozZXq8oXfpf2kUCwA/d5KxTVtzhwoT0JzI
8ks5T1KESaZMkE4f97Q=
-----END CERTIFICATE-----
```

I have the above certificate file saved as `DigiCert TLS RSA SHA256 2020 CA1.pem`.

---
#### Helper Function to Read Certificate Data
First, let's have a helper function that we can reuse to read certificate file data.
```go
func readCertificate(file string) (*x509.Certificate, error) {
	// Read the certificate file content
	fc, err := os.ReadFile(file)
	if err != nil {
		return nil, err
	}

	// Decode the PEM file
	block, _ := pem.Decode(fc)
	if block == nil {
		return nil, fmt.Errorf("failed to decode certificate data")
	}

	// Parse the certificate data
	return x509.ParseCertificate(block.Bytes)
}
```

The function is fairly simple. It accepts the file name, read the file content and then parse the content as an X509 certificate. If it failes for whatever reason, it shall return an error.

#### Read Certificate Data
Now that we have the helper function ready, let's create certificate objects for both the end entity certificate and the issuer certificate.
```go
cert, err := readCertificate("spring.io.pem")
if err != nil {
    log.Printf("Failed to read end entity certificate: %s", err)
    return
}

issuerCert, err := readCertificate("DigiCert TLS RSA SHA256 2020 CA1.pem")
if err != nil {
    log.Printf("Failed to read issuer certificate: %s\n", err)
    return
}
```

From the above code, we are storing the end entity certificate into `cert` variable and the issuer certificate into `issuerCert` variable.

---
#### Create the OCSP Request
Before we do that, we need to add a dependency to our project. For OCSP, we can use dependency from `golang.org/x/crypto/ocsp`.
```go
go get golang.org/x/crypto/ocsp
```

Now, we need to create the OCSP request data. This is the request that we will send to OCSP responder later. See the below snippet.
```go
// Create the OCSP request.
ocspReq, err := ocsp.CreateRequest(cert, issuerCert, &ocsp.RequestOptions{Hash: crypto.SHA256})
if err != nil {
    log.Printf("Failed to create OCSP request: %s\n", err)
    return
}
```

As can be seen from the above code, to create the OCSP request, we just have to send the certificate that we want to check, the issuer of the certificate (this typically will be the signer of the OCSP response), and the OCSP request options.

---
#### Certificate Revocation Checking
In Go, it is very easy to retrieve the list of OCSP responder URL(s) stored inside the certificate. We can simply call the `OCSPServer` property of the certificate object. It returns slice of the OCSP URL(s).
```go
ocspURLs := cert.OCSPServer
```

For OCSP request, we need to add a specific request headers to signify that the request is an OCSP request. Hence, first, we will create or own HTTP POST request to the OCSP URL.
```go
// Build the OCSP request.
ocspHTTPReq, err := http.NewRequest(http.MethodPost, ocspURL, bytes.NewBuffer(ocspReq))
if err != nil {
    log.Printf("Failed to build OCSP HTTP request: %s\n", err)
    continue
}
```

After that, we add the necessary HTTP headers.
```go
// Add the necessary headers.
ocspHTTPReq.Header.Add("Content-Type", "application/ocsp-request")
ocspHTTPReq.Header.Add("Accept", "application/ocsp-response")
```

Now that we have the HTTP request ready, we can simply use the built-in HTTP client provided by Go to send the OCSP HTTP request.
```go
// Send the OCSP request.
ocspHTTPResp, err := http.DefaultClient.Do(ocspHTTPReq)
if err != nil {
    log.Printf("Failed to send OCSP request: %s\n", err)
    continue
}
defer ocspHTTPResp.Body.Close()

if ocspHTTPResp.StatusCode != http.StatusOK {
    log.Printf("Received HTTP code: %d\n", ocspHTTPResp.StatusCode)
    continue
}
```

If we received an OK status from the OCSP responder, the last thing we need to do is to parse the OCSP response data. This is also can be done faily easily as shown below.
```go
// Read the OCSP response.
ocspResp, err := io.ReadAll(ocspHTTPResp.Body)
if err != nil {
    log.Printf("Failed to read OCSP response: %s\n", err)
    continue
}

// Parse the OCSP response.
resp, err := ocsp.ParseResponse(ocspResp, issuerCert)
if err != nil {
    log.Printf("Failed to parse OCSP response: %s\n", err)
    continue
}
```

Once we finished parsing the OCSP response, we can then check the status of the certificate by calling the `Status` property.
```go
switch resp.Status {
case ocsp.Good:
    log.Println("Certificate status: GOOD")

case ocsp.Revoked:
    log.Println("Certificate status: REVOKED")

case ocsp.Unknown:
    log.Println("Certificate status: UNKNOWN")
}
```

---
That's all. It is very easy and straight-forward.

I hope this is helpful and the source code for this is available at Github [here](https://github.com/handracs2007/certocsp){:target=_blank}.

Cheers

---

> Copyright &copy; 2020-2024 Handra. All Rights Reserved.
