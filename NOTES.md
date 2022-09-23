# Assimetric keys

## The problem:

Is this really the right one?

How can we make sure that the key is indeed the one that is supposed to be? Same problem with hashes on web pages belonging to a downloadable package. What if the page has been hacked?

## Ask for a certificate!

What cerificates are doing?

Certificates are issued based on some evidence that the private key is in posession of the entity who claims it.
Common misunderstandings: key <> csr <> certificate

## Generate a private key and create a CSR

### This is only a private key!

```sh
$ openssl genrsa -out vm.znorovszky.hu.key.pem 2048
```

### Generate a CSR (Certificate Signing Request)
In this step we add the details of the person or organization for whom we request a certification.

**Important:** The key DOES NOT leave the organization's storage!!

The CSR contains all the necessary details to link the key and the certificate and ensure that the key posession is proven.

```sh
$ openssl req -config ../int/openssl.cnf -key vm.znorovszky.hu.key.pem -new -sha256 -out vm.znorovszky.hu.csr.pem
```

To check what is inthere:

```sh
$ openssl req -in vm.znorovszky.hu.csr.pem -noout -text
```

Send the CSR to the desired CA

## CA procedure
CA is the Cerificate Authority, often bound with the RA which is the Registration Authority

**RA** is responsible to register the customer and make sure of it's identity
**CA** is responsible to issue the certificate based on the RA decision.

Some words on the security

## Set up a CA
CA needs to be set up according to the regulations

Key players in European and Hungarian regulations:
- [eIDAS (electronic IDentification, Authentication and trust Services)](https://en.wikipedia.org/wiki/EIDAS) - The law (EU)
- [ETSI](https://en.wikipedia.org/wiki/ETSI) - Standards
- [NMHH](https://nmhh.hu/cikk/187339/Bizalmi_szolgaltatasok_es_elektronikus_alairas) - Hungary

### CA generates a root key + selfs igned certificate
The root key and certificate is not in everyday use, it is kept secure, offline, etc. The creation is strictly audited by the authorities (NMHH and appointed auditor company).
As it is the root of a trust chain, it can only be self-signed.

Create key:
```sh
$ openssl genrsa -out ca-root.key.pem 4096
```

Self-sign cert
```sh
$ openssl req -config openssl.cnf -key private/ca-root.key.pem -new -x509 -days 7300 -sha256 -extensions v3_ca -out certs/ca-root.crt.pem
```

Contents of openssl.cnf

### CA generates an intermediate certificate
Why use intermediate certificates?

Generate a separate key then sign it, but now with the root certificate:

```sh
$ openssl req -config openssl.cnf -new -sha256 -key private/int.key.pem -out csr/int.csr.pem
$ openssl req -in csr/int.csr.pem -noout -text
$ openssl ca -config openssl.cnf -extensions v3_intermediate_ca -days 3650 -notext -md sha256 -in ../int/csr/int.csr.pem -out ../int/certs/int.crt.pem
```

Now you can sign requests using the intermediate certificate.

# Links used

[Set up your own PKI](https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html)  
[Setting up a PKI - UK Govt page](https://www.ncsc.gov.uk/collection/in-house-public-key-infrastructure)