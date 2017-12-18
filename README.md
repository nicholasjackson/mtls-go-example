# MTLS Example
Simple example to demonstrate how to use Mutual Authentication with Golang HTTP servers.

## Generating certificates
Generating the necessary certificates for this example can be performed by running the `./generate.sh` command and providing the domain name to create the cert 
for and the password for the keys.

```bash
./generate.sh localhost password
```

A certificate is only valid if the domain matches the hosted domain of the server, for example a certificate issue to the domain www.example.com would raise an exception
if you attempted to run `curl https://localhost`.

The script generates a root certificate and key, an intermediary, application certificate and a client certificate.  Both the application and client certificate are generated from the 
intermediary this would allow the client to authenticate any server which uses the intermediary chain.  It is possible to lock a client certificate down to a particular application 
by signing it with the applications certificate rather than the intermediary.

## Running the server using a self signed certificate
Start the server  
```bash
$ go run main.go -domain localhost
```

When calling the endpoint it is requred to add the ca-chain cert to the curl command as this is a self signed certificate.

```bash
$ curl -v --cacert 2_intermediate/certs/ca-chain.cert.pem https://localhost:8443/

#...
Hello World% 
```

## Running the server with Mutual TLS Authentication and a self signed certifcate
Start the server  
```bash
$ go run main.go -domain localhost -mtls true
```

Call the endpoint providing the certificates generated for the client, for the server to validate the request the user must provide its 
certifcate and private key.
```bash
$ curl -v --cacert 2_intermediate/certs/ca-chain.cert.pem --cert 4_client/certs/localhost.cert.pem --key 4_client/private/localhost.key.pem https://localhost:8443/

#...
Hello World% 
```

Calling the endpoint without providing the certificates

```bash
$ curl -v --cacert 2_intermediate/certs/ca-chain.cert.pem https://localhost:8443/

#...
curl: (35) error:14094412:SSL routines:SSL3_READ_BYTES:sslv3 alert bad certificate
```
