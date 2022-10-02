## Handra

### [Home](/) | Blog | [Disclaimer](/disclaimer) | [Terms and conditions](/tnc)

[<<go back](..)

### Secure gRPC Client/Server over mTLS
In this post, we are going to run through the process of creating gRPC client/server that is secured using mutual TLS authentication (mTLS). We will start with a little bit introduction on mTLS and gRPC, that is just enough for the sake of understanding this post. More detailed discussion on both is outside the scope of this post.

---
#### Brief introduction to mTLS
In today's web, it is very rare to see a website that is not secured at least using HTTPS (HTTP over TLS). It has been a norm in the industry that websites are ought to use HTTPS instead of just using plain HTTP. Some web browsers have taken further step as well to flag a website as insecure (through the RED warning in the address bar) when users try to access a website using only HTTP.

In a traditional HTTPS, clients (mostly web browsers), verify the identity of a server through its digital certificate. The digital certificate contains information of a website such as the domain name, the organisation, and some other information deemed as necessary. This is known as one-way TLS, whereas it's only the client that verifies the server.

In mTLS, the verification goes both way. Client needs to verify the server, and server needs to verify the client. Both verification is done using digital certificate. Due to the nature of it, mTLS is sometimes known as two-way TLS as well.

---
#### Brief introduction to gRPC
gRPC (gRPC Remote Procedure Calls), is an RPC framework developed initially by Google in 2015. gRPC runs on top of HTTP/2. It is a modern, open source RPC that can run almost anywhere. By default, gRPC is using protocol buffers.

There are several advantages on using gRPC:
- There is a need to have a well defined contract. This ensures every caller has correct expectation on the provided functions, the parameters, including the data type of the parameters.
- Enables streaming of payload, including bi-directional streaming.
- Protocol buffer is an efficient way to serialise structured data.
- Code generator eases developers in doing development of gRPC.

---
#### Generate CA certificate using OpenSSL
At this stage, we will first need to create a new CA certificate. Here, we will be using OpenSSL to generate the CA certificate. You can use any tool in your hand to achieve the same thing. If you already have a CA system installed (such as EJBCA), you can utilise that as well and skip this part.

##### Generate CA key
First of all, let's generate the CA private key using the following command.
```bash
openssl ecparam -name prime256v1 -genkey -noout -out cakey.key
```
This command is going to generate a key using curve named **prime256v1** and save to a file named **cakey.key**.

Below is the sample content of the generated **cakey.key** (note that your value might be different).
```bash
handra@nebula  ~/Downloads/grpc  cat cakey.key 
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIJywhPclt51cOZzNe/AYfTH//dtR/qqv7krhJWYGSndnoAoGCCqGSM49
AwEHoUQDQgAEeSTi1eJ9QImV6jJx6/qO0AX4S6bJjfML0AYn5lY/ibLoMLiMj69U
3PvQcQ4uVFsRe+x89UTbZFKVIaG9On6wgA==
-----END EC PRIVATE KEY-----
```

##### Generate CA certificate
Now, let's generate the CA certificate.
```bash
openssl req -x509 -new -nodes -key cakey.key -subj "/CN=TestCA/C=MY" -days 730 -out cacert.pem
```
This command is used to generate a new certificate with subject DN **CN=TestCA,C=MY** with validity of 730 days. The generate certificate is stored in a file named **cacert.pem**.

Below is the sample output of the generated certificate details.
```
handra@nebula  ~/Downloads/grpc  openssl x509 -in cacert.pem -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            03:8a:45:12:14:da:a7:7f:30:d5:71:44:01:b6:fe:77:10:f2:63:a3
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: CN = TestCA, C = MY
        Validity
            Not Before: Oct 10 07:22:44 2020 GMT
            Not After : Oct 10 07:22:44 2022 GMT
        Subject: CN = TestCA, C = MY
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:79:24:e2:d5:e2:7d:40:89:95:ea:32:71:eb:fa:
                    8e:d0:05:f8:4b:a6:c9:8d:f3:0b:d0:06:27:e6:56:
                    3f:89:b2:e8:30:b8:8c:8f:af:54:dc:fb:d0:71:0e:
                    2e:54:5b:11:7b:ec:7c:f5:44:db:64:52:95:21:a1:
                    bd:3a:7e:b0:80
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                41:D7:BA:F5:AF:D4:11:B9:59:D3:D4:BD:1C:13:91:5B:17:8F:74:4B
            X509v3 Authority Key Identifier: 
                keyid:41:D7:BA:F5:AF:D4:11:B9:59:D3:D4:BD:1C:13:91:5B:17:8F:74:4B

            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: ecdsa-with-SHA256
         30:44:02:20:64:7b:1e:de:e0:af:9d:d6:f3:a1:0d:89:f0:40:
         80:ba:cc:6c:e4:73:94:e0:b0:35:0e:5e:31:ac:85:62:c6:fe:
         02:20:6b:41:a2:4d:45:c1:69:91:d0:62:82:f5:e4:5a:3f:fe:
         87:d4:6c:ea:8b:23:51:e2:f4:4a:8b:36:f0:46:98:12
-----BEGIN CERTIFICATE-----
MIIBkDCCATegAwIBAgIUA4pFEhTap38w1XFEAbb+dxDyY6MwCgYIKoZIzj0EAwIw
HjEPMA0GA1UEAwwGVGVzdENBMQswCQYDVQQGEwJNWTAeFw0yMDEwMTAwNzIyNDRa
Fw0yMjEwMTAwNzIyNDRaMB4xDzANBgNVBAMMBlRlc3RDQTELMAkGA1UEBhMCTVkw
WTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAAR5JOLV4n1AiZXqMnHr+o7QBfhLpsmN
8wvQBifmVj+JsugwuIyPr1Tc+9BxDi5UWxF77Hz1RNtkUpUhob06frCAo1MwUTAd
BgNVHQ4EFgQUQde69a/UEblZ09S9HBORWxePdEswHwYDVR0jBBgwFoAUQde69a/U
EblZ09S9HBORWxePdEswDwYDVR0TAQH/BAUwAwEB/zAKBggqhkjOPQQDAgNHADBE
AiBkex7e4K+d1vOhDYnwQIC6zGzkc5TgsDUOXjGshWLG/gIga0GiTUXBaZHQYoL1
5Fo//ofUbOqLI1Hi9EqLNvBGmBI=
-----END CERTIFICATE-----
```

---
#### Generate SSL certificate using OpenSSL
Ok, so far so good. Now, let's generate an SSL certificate issued by our just newly generated CA. If you already have a CA system to generate the SSL certificate, you may skip this step.

##### Generate SSL certificate key
First, let's generate the key for our new SSL certificate.
```bash
openssl ecparam -name prime256v1 -genkey -noout -out server.key
```
This command is going to generate a key using curve named **prime256v1** and save to a file named **server.key**.

Below is the sample content of the generated **server.key** (note that your value might be different).
```bash
handra@nebula  ~/Downloads/grpc  cat server.key 
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIOQbRyoBmdviBR4L/2GSAOWJ8IGugUd2sQi3kk8TPaj0oAoGCCqGSM49
AwEHoUQDQgAEWXz/qzpr6DAbHMy3iCGUhWQ11LG0kS68+TT+29oArx6RX17lXp8R
RoyiXbS4ub5L0AyXmRHhlSQ3P2jQZyfwew==
-----END EC PRIVATE KEY-----
```

##### Create CSR configuration file
Next, let's create a config file to generate our CSR. The file name will be **csr.conf**.
```
cat > csr.conf <<EOF
[ req ]
default_bits = 256
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = MY
CN = localhost

[ req_ext ]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = localhost
IP.1 = 127.0.0.1

EOF
```

##### Generate CSR
Now that we have the CSR configuration file created, we can proceed with the CSR generation. We can use the below command to generate our CSR.
```bash
openssl req -new -key server.key -out server.csr -config csr.conf
```
This command is going to read the **server.key** file, and together with the **csr.conf** file, it will generate a new CSR and store it into **server.csr**.

Below is the sample content of the **server.csr** file (note that your value might be different).
```bash
handra@nebula  ~/Downloads/grpc  cat server.csr
-----BEGIN CERTIFICATE REQUEST-----
MIIBKjCB0gIBADAhMQswCQYDVQQGEwJNWTESMBAGA1UEAwwJbG9jYWxob3N0MFkw
EwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEWXz/qzpr6DAbHMy3iCGUhWQ11LG0kS68
+TT+29oArx6RX17lXp8RRoyiXbS4ub5L0AyXmRHhlSQ3P2jQZyfwe6BPME0GCSqG
SIb3DQEJDjFAMD4wGgYDVR0RBBMwEYIJbG9jYWxob3N0hwR/AAABMAsGA1UdDwQE
AwIEMDATBgNVHSUEDDAKBggrBgEFBQcDATAKBggqhkjOPQQDAgNHADBEAiApSAIN
ufy3Pj3Rs3Yof5bXyKrgvHHEIX4cpXU8uTZrSgIgB5ZZoJrkRqJNGveIzY0uy2Jj
J0a2Ajf1Gkt4uOVUnc8=
-----END CERTIFICATE REQUEST-----
```

##### Generate SSL certificate
Now that our CSR is ready, we can use the below command to sign the CSR and issue a new certificate for our SSL server usage.
```bash
openssl x509 -req -in server.csr -CA cacert.pem -CAkey cakey.key -CAcreateserial -out server.pem -days 90 -extfile csr.conf -extensions req_ext
```
This command is going to read **server.csr**, **csr.conf**, **cacert.pem**, and **cakey.key** to then issue the SSL certificate. The new SSL certificate will be stored as **server.pem**. Below is the output of the command.
```bash
handra@nebula  ~/Downloads/grpc  openssl x509 -req -in server.csr -CA cacert.pem -CAkey cakey.key -CAcreateserial -out server.pem -days 90 -extfile csr.conf -extensions req_ext
Signature ok
subject=C = MY, CN = localhost
Getting CA Private Key
```

Below is the sample output of the generated SSL certificate.
```
handra@nebula  ~/Downloads/grpc  openssl x509 -in server.pem -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            09:35:57:44:36:fb:a4:d9:2c:9e:97:f7:58:05:6b:c8:5e:78:07:0f
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: CN = TestCA, C = MY
        Validity
            Not Before: Oct 10 08:09:29 2020 GMT
            Not After : Jan  8 08:09:29 2021 GMT
        Subject: C = MY, CN = localhost
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:59:7c:ff:ab:3a:6b:e8:30:1b:1c:cc:b7:88:21:
                    94:85:64:35:d4:b1:b4:91:2e:bc:f9:34:fe:db:da:
                    00:af:1e:91:5f:5e:e5:5e:9f:11:46:8c:a2:5d:b4:
                    b8:b9:be:4b:d0:0c:97:99:11:e1:95:24:37:3f:68:
                    d0:67:27:f0:7b
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Subject Alternative Name: 
                DNS:localhost, IP Address:127.0.0.1
            X509v3 Key Usage: 
                Key Encipherment, Data Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
    Signature Algorithm: ecdsa-with-SHA256
         30:45:02:20:64:05:5e:45:7b:b4:a1:eb:db:88:bf:51:63:dd:
         9b:e4:7a:b4:3a:3a:49:cf:41:ae:cd:27:78:cb:61:a3:c6:c2:
         02:21:00:81:5b:61:d1:0c:5b:c1:5b:92:e4:32:2f:25:48:6a:
         8f:e7:91:0c:b6:93:5d:24:67:43:25:49:08:79:2a:8c:ab
-----BEGIN CERTIFICATE-----
MIIBgTCCASegAwIBAgIUCTVXRDb7pNksnpf3WAVryF54Bw8wCgYIKoZIzj0EAwIw
HjEPMA0GA1UEAwwGVGVzdENBMQswCQYDVQQGEwJNWTAeFw0yMDEwMTAwODA5Mjla
Fw0yMTAxMDgwODA5MjlaMCExCzAJBgNVBAYTAk1ZMRIwEAYDVQQDDAlsb2NhbGhv
c3QwWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAARZfP+rOmvoMBsczLeIIZSFZDXU
sbSRLrz5NP7b2gCvHpFfXuVenxFGjKJdtLi5vkvQDJeZEeGVJDc/aNBnJ/B7o0Aw
PjAaBgNVHREEEzARgglsb2NhbGhvc3SHBH8AAAEwCwYDVR0PBAQDAgQwMBMGA1Ud
JQQMMAoGCCsGAQUFBwMBMAoGCCqGSM49BAMCA0gAMEUCIGQFXkV7tKHr24i/UWPd
m+R6tDo6Sc9Brs0neMtho8bCAiEAgVth0QxbwVuS5DIvJUhqj+eRDLaTXSRnQyVJ
CHkqjKs=
-----END CERTIFICATE-----
```

---
#### Generate client certificate using OpenSSL
At this step, let's generate a client certificate issued by our newly generated CA as well. Like above, if you already have a CA system to generate the SSL certificate, you may skip this step.

##### Generate client certificate key
First, let's generate the key for our new SSL certificate.
```bash
openssl ecparam -name prime256v1 -genkey -noout -out client.key
```
This command is going to generate a key using curve named **prime256v1** and save to a file named **client.key**.

Below is the sample content of the generated **client.key** (note that your value might be different).
```bash
handra@nebula  ~/Downloads/grpc  cat client.key 
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEID46wfEqjHoHx4CzI+GjQmbwrm/9H11HoriwE84QcZNboAoGCCqGSM49
AwEHoUQDQgAEuQFRSskfErYz78k0DFsok1r8h7byL8vH7nNUqKKF4kquXJtgTo+V
RFXt5gaX6qa7Kb4AFjurvFS7ZAbPU8VlEw==
-----END EC PRIVATE KEY-----
```

##### Create CSR configuration file
Next, let's create a config file to generate our CSR. The file name will be **csrclient.conf**.
```
cat > csrclient.conf <<EOF
[ req ]
default_bits = 256
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = MY
CN = client

[ req_ext ]
keyUsage = keyEncipherment
extendedKeyUsage = clientAuth

EOF
```

##### Generate CSR
Now that we have the CSR configuration file created, we can proceed with the CSR generation. We can use the below command to generate our CSR.
```bash
openssl req -new -key client.key -out client.csr -config csrclient.conf
```
This command is going to read the **client.key** file, and together with the **csrclient.conf** file, it will generate a new CSR and store it into **client.csr**.

Below is the sample content of the **client.csr** file (note that your value might be different).
```bash
handra@nebula  ~/Downloads/grpc  cat client.csr
-----BEGIN CERTIFICATE REQUEST-----
MIIBDTCBswIBADAeMQswCQYDVQQGEwJNWTEPMA0GA1UEAwwGY2xpZW50MFkwEwYH
KoZIzj0CAQYIKoZIzj0DAQcDQgAEuQFRSskfErYz78k0DFsok1r8h7byL8vH7nNU
qKKF4kquXJtgTo+VRFXt5gaX6qa7Kb4AFjurvFS7ZAbPU8VlE6AzMDEGCSqGSIb3
DQEJDjEkMCIwCwYDVR0PBAQDAgUgMBMGA1UdJQQMMAoGCCsGAQUFBwMCMAoGCCqG
SM49BAMCA0kAMEYCIQCF3b/O/EoOyYGhtT5IG8y0CdXdjWZl7TZr+Gcm5+z9/gIh
AL4oaI+aOiAncGk17x3fjsT68D1IaBC9gGSq9CvYZTHP
-----END CERTIFICATE REQUEST-----
```

##### Generate client certificate
Now that our CSR is ready, we can use the below command to sign the CSR and issue a new certificate for our client usage.
```bash
openssl x509 -req -in client.csr -CA cacert.pem -CAkey cakey.key -CAcreateserial -out client.pem -days 90 -extfile csrclient.conf -extensions req_ext
```
This command is going to read **client.csr**, **csrclient.conf**, **cacert.pem**, and **cakey.key** to then issue the client certificate. The new client certificate will be stored as **client.pem**. Below is the output of the command.
```bash
handra@nebula  ~/Downloads/grpc  openssl x509 -req -in client.csr -CA cacert.pem -CAkey cakey.key -CAcreateserial -out client.pem -days 90 -extfile csrclient.conf -extensions req_ext
Signature ok
subject=C = MY, CN = client
Getting CA Private Key
```

Below is the sample output of the generated client certificate.
```
handra@nebula  ~/Downloads/grpc  openssl x509 -in client.pem -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            09:35:57:44:36:fb:a4:d9:2c:9e:97:f7:58:05:6b:c8:5e:78:07:10
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: CN = TestCA, C = MY
        Validity
            Not Before: Oct 10 08:30:35 2020 GMT
            Not After : Jan  8 08:30:35 2021 GMT
        Subject: C = MY, CN = client
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:b9:01:51:4a:c9:1f:12:b6:33:ef:c9:34:0c:5b:
                    28:93:5a:fc:87:b6:f2:2f:cb:c7:ee:73:54:a8:a2:
                    85:e2:4a:ae:5c:9b:60:4e:8f:95:44:55:ed:e6:06:
                    97:ea:a6:bb:29:be:00:16:3b:ab:bc:54:bb:64:06:
                    cf:53:c5:65:13
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: 
                Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication
    Signature Algorithm: ecdsa-with-SHA256
         30:44:02:20:57:5d:97:6d:0e:a5:0a:1a:0e:e4:c8:01:2d:a2:
         da:3a:71:20:a6:bb:7b:7b:08:57:8f:6d:d1:15:d4:1a:6e:17:
         02:20:36:b4:04:8c:34:e6:3b:62:42:a2:cc:5c:f3:16:98:0b:
         15:ab:78:f6:30:64:5c:a7:68:2c:fb:20:ad:aa:b3:c0
-----BEGIN CERTIFICATE-----
MIIBYTCCAQigAwIBAgIUCTVXRDb7pNksnpf3WAVryF54BxAwCgYIKoZIzj0EAwIw
HjEPMA0GA1UEAwwGVGVzdENBMQswCQYDVQQGEwJNWTAeFw0yMDEwMTAwODMwMzVa
Fw0yMTAxMDgwODMwMzVaMB4xCzAJBgNVBAYTAk1ZMQ8wDQYDVQQDDAZjbGllbnQw
WTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAAS5AVFKyR8StjPvyTQMWyiTWvyHtvIv
y8fuc1SoooXiSq5cm2BOj5VEVe3mBpfqprspvgAWO6u8VLtkBs9TxWUToyQwIjAL
BgNVHQ8EBAMCBSAwEwYDVR0lBAwwCgYIKwYBBQUHAwIwCgYIKoZIzj0EAwIDRwAw
RAIgV12XbQ6lChoO5MgBLaLaOnEgprt7ewhXj23RFdQabhcCIDa0BIw05jtiQqLM
XPMWmAsVq3j2MGRcp2gs+yCtqrPA
-----END CERTIFICATE-----
```

---
#### Create gRPC contract
Now that we have our certificates ready, let's proceed by completing our gRPC development. First of all, let's create an IDL (Interface Definition Language) as required by gRPC. This file contains the exposed services together with the inputs and outputs.

For simplicity, we are going to just create a simple ```DemoService``` service. This is more than enough to demo the mTLS and gRPC working together.

Below is the content of the proto file. We have an RPC endpoint named ```SayHello``` that accepts ```name``` and ```age``` wrapped as a ```HelloRequest``` and returns a response string wrapped inside the ```HelloResponse``` object.

```proto
syntax = "proto3";

option go_package=".;rpc";

message HelloRequest {
  string name = 1;
  int32 age = 2;
}

message HelloResponse {
  string response = 1;
}

service DemoService {
  rpc SayHello(HelloRequest) returns (HelloResponse);
}
```
For further reference on the protobuf configuration, you can refer [here](https://developers.google.com/protocol-buffers/docs/proto3){:target=_blank}. That's it for the proto file. We will continue with developing both client and server applications. We will be using Go for the development.

---
#### Develop gRPC server
First of all, let's generate the code from our proto file to Go source.

Before we proceed, we will first need to download the dependencies for gRPC with Go.
```bash
go get github.com/golang/protobuf
go get google.golang.org/grpc
go get google.golang.org/protobuf
```

Next, we will need to compile the proto file to Go code. Below is the command I used to compile the proto file. Depending on your proto file name and desired output directory, you might want to adjust the parameters.
```bash
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=../rpc/ --go-grpc_opt=paths=source_relative demo.proto
```

Next, we will need to create an implementation struct for our DemoService. The code is as below.
```go
type GrpcDemoService struct {
    rpc.UnimplementedDemoServiceServer
}

func (service GrpcDemoService) SayHello(context context.Context, request *rpc.HelloRequest) (*rpc.HelloResponse, error) {
    name := request.Name
    age := request.Age
    resp := fmt.Sprintf("Hello %s, you are %d year(s) old.", name, age)

    response := &rpc.HelloResponse{Response: resp}
    return response, nil
}
```
There is nothing special with the above implementation. It will just grab the data from the request and create a response message from the given data as a string and send it back to the caller.

Now that we have the implementation class ready, we will proceed to build our gRPC server.

We will create a main Go file, whose sole purpose is to start the gRPC server. The code can be seen below. I have put some comments at the code itself as well to give clearer explanation on the important parts of the code.
```go
func main() {
    // Load the server certificate and its key
    serverCert, err := tls.LoadX509KeyPair("server.pem", "server.key")
    if err != nil {
        log.Fatalf("Failed to load server certificate and key. %s.", err)
    }

    // Load the CA certificate
    trustedCert, err := ioutil.ReadFile("cacert.pem")
    if err != nil {
        log.Fatalf("Failed to load trusted certificate. %s.", err)
    }

    // Put the CA certificate to certificate pool
    certPool := x509.NewCertPool()
    if !certPool.AppendCertsFromPEM(trustedCert) {
        log.Fatalf("Failed to append trusted certificate to certificate pool. %s.", err)
    }

    // Create the TLS configuration
    tlsConfig := &tls.Config{
        Certificates: []tls.Certificate{serverCert},
        RootCAs:      certPool,
        ClientCAs:    certPool,
        MinVersion:   tls.VersionTLS13,
        MaxVersion:   tls.VersionTLS13,
    }

    // Create a new TLS credentials based on the TLS configuration
    cred := credentials.NewTLS(tlsConfig)

    // Create a listener that listens to localhost port 8443
    listener, err := net.Listen("tcp", "localhost:8443")
    if err != nil {
        log.Fatalf("Failed to start listener. %s.", err)
    }
    defer func() {
        err = listener.Close()
        if err != nil {
            log.Printf("Failed to close listener. %s\n", err)
        }
    }()

    // Create a new gRPC server
    server := grpc.NewServer(grpc.Creds(cred))
    rpc.RegisterDemoServiceServer(server, &GrpcDemoService{}) // Register the demo service

    // Start the gRPC server
    err = server.Serve(listener)
    if err != nil {
        log.Fatalf("Failed to start gRPC server. %s.", err)
    }
}
```

That is all for the server part. Next, we will proceed to create the client part of the gRPC.

---
#### Develop gRPC client
For this part of the code, we will be using the same IDL file to generate the Go code for our client.

Before we proceed, same with the server code, we will first need to download the dependencies for gRPC with Go.
```bash
go get github.com/golang/protobuf
go get google.golang.org/grpc
go get google.golang.org/protobuf
```

Next, we will need to compile the proto file to Go code. Below is the command I used to compile the proto file. Depending on your proto file name and desired output directory, you might want to adjust the parameters.
```bash
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=../rpc/ --go-grpc_opt=paths=source_relative demo.proto
```

What's left is just to develop the code for the gRPC client. Below is the code I used to develop the client.
```go
func main() {
    // Load the client certificate and its key
    clientCert, err := tls.LoadX509KeyPair("client.pem", "client.key")
    if err != nil {
        log.Fatalf("Failed to load client certificate and key. %s.", err)
    }

    // Load the CA certificate
    trustedCert, err := ioutil.ReadFile("cacert.pem")
    if err != nil {
        log.Fatalf("Failed to load trusted certificate. %s.", err)
    }

    // Put the CA certificate to certificate pool
    certPool := x509.NewCertPool()
    if !certPool.AppendCertsFromPEM(trustedCert) {
        log.Fatalf("Failed to append trusted certificate to certificate pool. %s.", err)
    }

    // Create the TLS configuration
    tlsConfig := &tls.Config{
        Certificates: []tls.Certificate{clientCert},
        RootCAs:      certPool,
        MinVersion:   tls.VersionTLS13,
        MaxVersion:   tls.VersionTLS13,
    }

    // Create a new TLS credentials based on the TLS configuration
    cred := credentials.NewTLS(tlsConfig)

    // Dial the gRPC server with the given credentials
    conn, err := grpc.Dial("localhost:8443", grpc.WithTransportCredentials(cred))
    if err != nil {
        log.Fatal(err)
    }
    defer func() {
        err = conn.Close()
        if err != nil {
            log.Printf("Unable to close gRPC channel. %s.", err)
        }
    }()

    // Create the request data
    request := &rpc.HelloRequest{
        Name: "Handra",
        Age:  35,
    }

    // Create the gRPC client
    client := rpc.NewDemoServiceClient(conn)
    response, err := client.SayHello(context.Background(), request)
    if err != nil {
        log.Fatalf("Failed to receive response. %s.", err)
    }

    // Print out response from server
    fmt.Println(response.Response)
}
```

---
#### Testing all together
Now that we have the server and client code ready, we can start doing the integration between client and server.

First, let's start the gRPC server. For the sake of simplicity, I will just use ```go run``` to quickly start the server.
```bash
handra@nebula  /media/handra/DATA/go/src/github.com/handracs2007/demogrpcserver  go run main/main.go
```

Once the server is started, we can run the client using the same command.
```bash
handra@nebula  /media/handra/DATA/go/src/github.com/handracs2007/demogrpcclient  go run main/main.go
```

It will write the output of the gRPC call to the console as seen below.
```
handra@nebula  /media/handra/DATA/go/src/github.com/handracs2007/demogrpcclient  go run main/main.go
Hello Handra, you are 35 year(s) old.
```

---
That's it. Now we have a secure gRPC server that requires clients to provide certificate as well for authentication. This is of course just a simplistic implementation. The next task to do will be on using encrypted private key file, but I will leave that for another blog post.

I hope this is helpful and the source code for this is available at Github as below:
- Server: <https://github.com/handracs2007/demogrpcserver>
- Client: <https://github.com/handracs2007/demogrpcclient>

Cheers

---

> Copyright &copy; 2020-2022 Handra. All Rights Reserved.