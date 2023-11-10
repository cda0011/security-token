# security-token

[![CircleCI](https://circleci.com/gh/manetu/security-token/tree/master.svg?style=svg)](https://circleci.com/gh/manetu/security-token/tree/master)

The manetu-security-token CLI is a simple utility to manage Manetu Service Account credentials within a [PKCS11](https://en.wikipedia.org/wiki/PKCS_11) compatible [Hardware Security Module](https://en.wikipedia.org/wiki/Hardware_security_module) (HSM).  It offers support for creating, reading, enumerating, and deleting "security tokens," which are simply a public/private key pair and a self-signed x509.

Users register these security tokens with the Manetu Realm Portal as a credential for a Service Account within the product's Identity and Access Management (IAM) function.

This utility also offers a 'login' function as a convenience, which will initiate a Service Account login via the OAUTH [private_key_jwt](https://openid.net/specs/openid-connect-core-1_0-15.html#ClientAuthentication) authentication flow.  If successful, the resulting [JWT](https://en.wikipedia.org/wiki/JSON_Web_Token) based Access Token will be sent to stdout, making this function suitable as both an example as well as an integration for other applications that cannot perform the HSM and JWT operations natively.

# Getting Started

## Prerequisites

You will need a PKCS11 compatible HSM and its SDK.  Examples:

- [YubiHSM](https://www.yubico.com/products/hardware-security-module/)
- [CloudHSM](https://aws.amazon.com/cloudhsm/)

If you don't have access to a physical HSM, you may also use the [SoftHSM2](https://github.com/opendnssec/SoftHSMv2) emulator.  SoftHSM2 provides a full PKCS11 compatible interface but stores key material on the host's filesystem, so it is not as secure as an actual tamper-resistant hardware device.  YMMV.

## Setup

You may configure the tool for your environment by creating a security-tokens.yml file with your specifics.  These details minimally include the HSM configuration.  Example:

```yaml
pkcs11:
  path: "/usr/local/Cellar/softhsm/2.6.1/lib/softhsm/libsofthsm2.so"
  tokenlabel: "manetu"
  pin: "1234"
```

Please consult the documentation for your selected HSM for the details in the pkcs11 section.

### SoftHSM2
If you have opted to use the SoftHSM2 emulator, the following may be helpful to get you started:
```shell
softhsm2-util --init-token --slot 0 --label "manetu"
```
The tool will ask you to select a PIN.  Be sure to update the security-tokens.yml with your selection.

## Building

Make sure the following software is installed
 
* Golang env version 1.18 or above

```shell
make bin
```

Executing this command should result in the binary 'manetu-security-token' within your current working directory.

# Usage

## help

```shell
$ ./manetu-security-token help
NAME:
   manetu-security-token - A new cli application

USAGE:
   manetu-security-token [global options] command [command options] [arguments...]

COMMANDS:
   generate  Generate a new security token
   show      Display the PEM encoded x509 public key for the specified security token
   list      Enumerate available security tokens
   delete    Remove a security token
   login     Acquires an access token from a security token
   help, h   Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h  show help (default: false)
```

## generate

The generate command will create a new security token consisting of an ECC P.256 public/private key pair and a self-signed x509.  You must specify the target realm with either --realm or by setting the MANETU_REALM environment variable.

```shell
$ ./manetu-security-token generate --realm myrealm
Serial: EF:6D:9B:CA:93:7E:FA:C5:6C:A8:EC:0A:A0:86:ED:FE:5F:22:7A:41:D8:0D:C0:86:17:B5:DC:DD:D7:4A:8D:AF
fingerprint: 51:E1:05:87:20:49:DB:57:C3:06:75:0D:84:07:95:CE
-----BEGIN CERTIFICATE-----
MIIBejCCAR+gAwIBAgIhAO9tm8qTfvrFbKjsCqCG7f5fInpB2A3Ahhe13N3XSo2v
MAoGCCqGSM49BAMCMCQxIjAgBgNVBAoTGU1hbmV0dSBTZWN1cml0eSBUb2tlbiBD
TEkwIBcNMjEwOTA5MTQyMjAwWhgPMDAwMTAxMDEwMDAwMDBaMCQxIjAgBgNVBAoT
GU1hbmV0dSBTZWN1cml0eSBUb2tlbiBDTEkwWTATBgcqhkjOPQIBBggqhkjOPQMB
BwNCAASUDTuj0McGdZXBC/lsO1EULMUQh0vCxHBgWSEvimdEUUHb+k1yHZucRs5q
MgKI62hNTYCbi4XbE5MGRVL+PwO+oyAwHjAOBgNVHQ8BAf8EBAMCB4AwDAYDVR0T
AQH/BAIwADAKBggqhkjOPQQDAgNJADBGAiEAl1UXyFkmkKTlrmodzpaEddDJGDTJ
nUc7u4tiOxuFyH0CIQCZJeNuvU5NLseI9EO7u5hWClSoFKoIJUoiiqvuqk585w==
-----END CERTIFICATE-----
```

The resulting PEM is suitable for pasting in the IAM portal.  The Serial Number embedded within the x509 is a consistent reference within the CLI and IAM.

## list

You may list the inventory of security tokens stored within your configured HSM.

```shell
$ ./manetu-security-token list
+-------------------------------------------------------------------------------------------------+-------------+-------------------------------+
|                                             SERIAL                                              |    REALM    |            CREATED            |
+-------------------------------------------------------------------------------------------------+-------------+-------------------------------+
| 81:AD:CE:D8:29:B5:47:2F:3C:55:2F:C0:35:E9:AB:CA:21:94:6F:84:AB:E9:0B:4A:69:BB:CF:18:4E:60:C2:97 | acmelender  | 2022-04-14 14:24:24 +0000 UTC |
| C3:22:F2:90:CD:8F:41:9B:6C:1A:FC:F9:5D:77:17:30:B9:5D:64:49:EB:C1:88:EB:E3:C5:3F:1A:5D:BD:32:C2 | data-loader | 2022-04-13 22:51:51 +0000 UTC |
+-------------------------------------------------------------------------------------------------+-------------+-------------------------------+
```

## show

You may always re-export an x509 from your inventory:

```shell
$ ./manetu-security-token show --serial 9C:AA:50:2C:B5:1B:01:E2:3D:A6:03:D9:C3:0A:82:6C:F8:8F:6F:D7:B2:E3:CF:05:29:2C:20:F1:AE:C4:7A:72
-----BEGIN CERTIFICATE-----
MIIBejCCAR+gAwIBAgIhAJyqUCy1GwHiPaYD2cMKgmz4j2/XsuPPBSksIPGuxHpy
MAoGCCqGSM49BAMCMCQxIjAgBgNVBAoTGU1hbmV0dSBTZWN1cml0eSBUb2tlbiBD
TEkwIBcNMjEwOTA5MDIyODMzWhgPMDAwMTAxMDEwMDAwMDBaMCQxIjAgBgNVBAoT
GU1hbmV0dSBTZWN1cml0eSBUb2tlbiBDTEkwWTATBgcqhkjOPQIBBggqhkjOPQMB
BwNCAATR5/Q6O7PsSi3rKYrwgfItYzHXgcbGUcLgUeiq42uqZHXsgguzmsSiGwq9
ootrd6xIMx8Tys4NPfPxxu925m+CoyAwHjAOBgNVHQ8BAf8EBAMCB4AwDAYDVR0T
AQH/BAIwADAKBggqhkjOPQQDAgNJADBGAiEAtGOj3SW/X+SmbSHjOmkO1zPXdqKu
JFvrFI4HoO1q2qQCIQD91troFP880DAytLBMQ1iDfunitDE7jCQ+oer8lyj3jQ==
-----END CERTIFICATE-----
```

### Helpful Tip

You can pipe 'show' into tools such as *openssl* to further decode the x509

```shell
$ ./manetu-security-token show --serial 9C:AA:50:2C:B5:1B:01:E2:3D:A6:03:D9:C3:0A:82:6C:F8:8F:6F:D7:B2:E3:CF:05:29:2C:20:F1:AE:C4:7A:72 | openssl x509  -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            9c:aa:50:2c:b5:1b:01:e2:3d:a6:03:d9:c3:0a:82:6c:f8:8f:6f:d7:b2:e3:cf:05:29:2c:20:f1:ae:c4:7a:72
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: O=Manetu Security Token CLI
        Validity
            Not Before: Sep  9 02:28:33 2021 GMT
            Not After : Jan  1 00:00:00 1 GMT
        Subject: O=Manetu Security Token CLI
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:d1:e7:f4:3a:3b:b3:ec:4a:2d:eb:29:8a:f0:81:
                    f2:2d:63:31:d7:81:c6:c6:51:c2:e0:51:e8:aa:e3:
                    6b:aa:64:75:ec:82:0b:b3:9a:c4:a2:1b:0a:bd:a2:
                    8b:6b:77:ac:48:33:1f:13:ca:ce:0d:3d:f3:f1:c6:
                    ef:76:e6:6f:82
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature
            X509v3 Basic Constraints: critical
                CA:FALSE
    Signature Algorithm: ecdsa-with-SHA256
         30:46:02:21:00:b4:63:a3:dd:25:bf:5f:e4:a6:6d:21:e3:3a:
         69:0e:d7:33:d7:76:a2:ae:24:5b:eb:14:8e:07:a0:ed:6a:da:
         a4:02:21:00:fd:d6:da:e8:14:ff:3c:d0:30:32:b4:b0:4c:43:
         58:83:7e:e9:e2:b4:31:3b:8c:24:3e:a1:ea:fc:97:28:f7:8d
```

## delete

You may delete security tokens that are no longer needed.

```shell
$ ./manetu-security-token delete --serial 9C:AA:50:2C:B5:1B:01:E2:3D:A6:03:D9:C3:0A:82:6C:F8:8F:6F:D7:B2:E3:CF:05:29:2C:20:F1:AE:C4:7A:72
```

You can confirm deletion using `list` command.

### Helpful Tip

The following command will clear out any existing tokens for development with SoftHSM2.

```shell
$ softhsm2-util --token manetu --delete-token
```

You may then repeat the --init-token flow to set up a fresh HSM instance.

## login

A Service Account must be created in the Manetu Realm UI using an x509 certificate prior to login. The certificate is obtained through different means for HSM and PEM key based logins.

### hsm

Create the Service Account using the X509 certificate from the [generate](#generate) step.

You may login using a serial number from the [list](#list) command. You must specify the --url or set MANETU_URL pointing to your Manetu instance.

N.B. that the `serial` parameter is optional; if you don't specify, it will pick one from the HSM.  This is primarily useful for cases where you only have one token.

```shell
$ ./manetu-security-token login --url https://manetu.instance hsm --serial 9C:AA:50:2C:B5:1B:01:E2:3D:A6:03:D9:C3:0A:82:6C:F8:8F:6F:D7:B2:E3:CF:05:29:2C:20:F1:AE:C4:7A:72
```

The output is a `jwt` that can be used in Manetu API invocation.

### pem

A standard PEM encoded key-pair, such as generated with openssl, may be used for cases where access to a real HSM is limited or overkill.  PEMs trade lower security for increased convenience, and thus you are encouraged to leverage HSMs for production use whenever possible.

If you understand the tradeoffs but still wish to proceed, you may reference the following script to generate the key-pair and x509 certificate:

```shell
$ openssl ecparam -genkey -name prime256v1 -noout -out key.pem
$ openssl req -new -x509 -key key.pem -out cert.pem -days 365 -subj "/O=the-realm-of-the-service-account"
```

Replace `the-realm-of-the-service-account` appropriate realm ID.

Log into the realm in Manetu Realm UI and create the Service Account using `cert.pem`.

You may login using the key/cert. You must specify the --url or set MANETU_URL pointing to your Manetu instance.

```shell
$ ./manetu-security-token login --url https://manetu.instance pem --key /path/to/key.pem --cert /path/to/cert.pem --path
```

Raw strings are assumed if `--path` is omitted. 

The output is a `jwt` that can be used in Manetu API invocation.
