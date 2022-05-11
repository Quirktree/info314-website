# TLS 1.3 Lab Introduction

## Overview

This lab is the start of a multi-week project building a minimal HTTPS client using the TLS 1.3 protocol over TCP. The overall goal will be to complete a TLS 1.3 handshake, send a simple HTTP GET request to the course website over the encrypted channel, and process its response. With the short time remaining, you may not be able to complete the project in its entirety, but you will get plenty of hands-on experience with common protocol implementation challenges.

Be aware that this is a complicated protocol. This guide is provided as a high-level reference, but we will also be providing starter code and walking through code in class. If you find yourself struggling, please review the resources and ask for help.

## Limitations

Our implementation will not be feature complete or even secure on a practical level. The objective is to strip away as many unnecessary details as necessary so in order to handle data for a single HTTP request and response. 

The first detail that we have removed is concurrency. A non-concurrent network application is easier to reason about for a beginner because it is restricted to sequential operations and can't perform reading and writing operations at the same time. We will also ignore the following tasks that would be needed in a real TLS implementation

- handshake verification (security critical)
- certificate verification (security critical)
- signature verification (security critical)
- error handling / alerts
- key updates
- client certificates
- pre-shared keys
- session tickets

## Cryptography

Even with the these omissions, a functional TLS 1.3 implementation is still quite a challenging task due to its use of cryptography to generate keys and encrypt / decrypt application layer data. To simplify the task, the project repository includes a module named `helpers.py` that contains classes that will manage these more complex concerns and allow you to focus on the practical elements of communication Layer 4 - 7 communication.

## Resources

In addition to example code and demonstrations provided in the course repository and lectures, you will need to reference resources that decompose real TLS connections. By imitating the examples, you will be able to create a working version of TLS without having to become a protocol expert in advance. The following two sites will be used extensively.

- [The Illustrated TLS 1.3 Connection](https://tls13.ulfheim.net/)
- [A Walkthrough of the TLS 1.3 Handshake](https://commandlinefanatic.com/cgi-bin/showarticle.cgi?article=art080)

Likewise, RFC 8446 is the definitive resource to learn more about the protocol.

- [The Transport Layer Security (TLS) Protocol Version 1.3 (RFC 8446)](https://datatracker.ietf.org/doc/html/rfc8446)

## Tasks

### Preparation

The code provided for this course relies on Python 3.9+ and one external cryptographic library. Once you've confirmed that you have the proper version of Python installed, use your preferred dependency management workflow to install the `cryptography` package. If you're unfamiliar with project dependency management, you can make the package available to all Python 3 code on your system by running `pip3 install cryptography`. 

### Create TCP Connection

Create a client-based TCP connection using the sockets library. Connecting to the requested host will initiate a TCP handshake. Before the application exits, you should close the socket.

### Establish TLSContext

Before you get too far in the project, you will need to import the TLSContext class from the project directory and create a new context instance that'll help you manage the cryptographic aspects of your connection.

The following snippet demonstrates how you can 

```python
from helpers import TLSContext

def hostname = "info314.tcpip.dev"

# Create a new context
tls = TLSContext(hostname=hostname)

# Generate client_random, private_key, and public_key
tls.generate_handshake_parameters()

# Inspect values that your handshake code will rely on
print(f"{tls.hostname=}")
print(f"{tls.client_random.hex(' ')=}")
print(f"{tls.public_key.hex(' ')=}")
print(f"{tls.cipher_suites=}")
print(f"{tls.signature_algorithms=}")
```

### Begin TLS Handshake

In this assignment, you will focus on generating/sending a ClientHello message and processing the ServerHello message that is sent back in response. 

#### Send Client Hello

Build and send an unencrypted record containing ClientHello message. The ClientHello message initiates the TLS handshake and provides an initial set of parameters that the server can use to determine the capabilities of the client and compute a shared secret. 

Be aware that the ClientHello for TLS 1.3 is disguised to appear as an earlier version of the protocol to avoid problems with legacy _middle boxes_. To complete this disguise, TLS 1.3 will outwardly present TLS 1.2 and also include certain fields that don't provide any function in TLS 1.3.

##### Record and Handshake Headers

As a rather complex protocol, TLS uses multiple layers of messages headers to communicate message structure to software. When we send the ClientHello message on the network, it will require the following parts:

TLS Record Header
: Every TLS message is encoded as a record with a 5-byte header, including a 1-byte record type, a 2-byte protocol version, and a 2-byte length field. The ClientHello record header sets the type to 22 for handshake message and sets the version to TLS 1.0 `b'\x03\x01'`. The length is encoded in network byte order and excludes the 5-bytes of record header.

Handshake Header
: Every TLS handshake message will also have a 4-byte header, including a 1-byte message type and a 3-byte length field. The ClientHello message sets the type to 1. The length is encoded in network byte order and excludes the length of the record and handshake headers.

##### Core Message Structure

Each message type prescribes specific requirements for the structure of its messages. The following fields make up the payload of the ClientHello message. 

Client Version
: The ClientHello version field is a two-byte field set to TLS 1.2 `b'\x03\x03` for backwards compatibility. The real version is sent using the extension mechanism.

Client Random:
: Set to 32 bytes of random data. You may create random bytes using `os.urandom(32)` or use `client_random` from the current TLSContext.

Session ID
: Provide a non-zero 32 byte Session ID value (for backwards compatibility). You may create bytes using `os.urandom(32)`.

Cipher Suites
: Indicates symmetric encryption and hash combinations that are supported by the client. The TLSContext for your session contains a property named `cipher_suites` with a list of 2-byte cipher-suite identifiers supported by instructor-provided TLSCipher class.

Compression Methods
: Indicates compression methods that are supported by the client. This field is included for backwards compatibility since TLS 1.3 does not support compression. The list should only include the _null_ (`b'\x00'`) compression method 

Extensions Length
: The main ClientHello body ends with a 2-byte length field specifying the size of the extension data that is included in this message.

##### Extensions

TLS uses extensions to implement functionality that may not be supported by all clients and servers. Extension syntax varies by type. To enable software to cope with extensions that are not implemented, each extension begins with its own 4-byte header made up from a 2-byte type field and a 2-byte length. In the case of TLS 1.3, extensions are also used to ensure _middle box_ compatibility by disguising the message body as TLS 1.2. As such, all TLS 1.3 implementations are required to include specific extensions within the ClientHello. 

The following extensions should be included in the ClientHello:

Server Name Extension (Type 0)
: Provides the hostname so that the server can select the correct certificates. This mechanism is frequently used with HTTPS servers.

Supported Groups Extension (Type 10)
: Indicates which key exchange algorithms the client supports. For a minimal implementation, we will support the x25519 key exchange, which is identified by the value `0x001D`.

Signature Algorithm Extension (Type 13)
: Identifies signature algorithms that will be accepted for Certificate and CertificateVerify messages. We won't implement any signature algorithms, but we must provide a valid list. 

TLS Version Extension (Type 43)
: Indicates that the client is requesting a TLS 1.3 handshake. Set to the TLS 1.3 version indicator, i.e., `b'\x03\x04'`

Key Share Extension (Type 51)
: Provides the client's public key to the server. The TLSContext for your session contains a property named `public_key` that is set to the byte-level representation of the public key generated for the current session. 

#### Receive Server Hello

Once a ClientHello is properly constructed and sent by a client, a server is expected to reply with a ServerHello message in return. Breaking this message into its parts is a critical step that the client must take before it will be possible to compute encryption parameters needed to read / write any further messages.

TLS Record Header
: Use the 5-byte header TLS record header to ensure determine the length of the message and to verify the type of message being read. For ServerHello the type should be set to 22 (handshake message) and the version will be TLS 1.2 `b'\x03\x03'`. The length should be decoded in network byte order.

Handshake Header
: Use the 4-byte header TLS message header to verify that the handshake message is a ServerHello. To know whether you are processing a ServerHello message, verify that the type is 2. Decode the length to determine the number of bytes expected in the message.

Server Version
: This two-byte version field will have been set to TLS 1.2 (`b'\x03\x03') for backwards compatibility. The real version is sent using the extension mechanism.

Server Random
: The server will have sent 32 bytes of random data generated for this connection. You will not need to work with this data directly, but you may inspect it.

Session ID
: For TLS 1.3, the session id is only included for compatibility with _middle boxes_. The server should return the value originally given in the ClientHello. 

Compression
: The server will return a 1-byte value indicating that it accepted the null compression offer made by the client.

Cipher Suite
: The server will return a 2-byte value indicating which cipher was chosen from the options given in the ClientHello. Save this value to a variable for later use.

The ServerHello message contents include a set of extensions that must be parsed individually. Every extension begins with its own 4-byte header including a 2-byte type and a 2-byte length field. You may ignore all extensions except for the following.

TLS Version Extension (Type 43)
: Indicates that the server agrees to a TLS 1.3 handshake when provided and set to the TLS 1.3 2-byte version (`b'\x03\x04'`)

Key Share Extension (Type 51)
: Provides the server's public key which is needed to complete the key exchange. Verify that a x25519 public key (identified as `0x001d`) is included and save the value to a variable for later use.

## Reference

The followiong tables are provided as a quick reference for many of the standard types/values that you'll encounter when working with TLS.

### TLS Record Types (encode as 1-byte integer) 

| Description                  | Value |
| ---------------------------- | ----- |
| handshake                    |  22   |
| change cipher spec           |  20   |
| application data             |  23   |
| alert                        |  21   |

### TLS Handshake Message Types (encode as 1-byte integer)

| Description                  | Value |
| ---------------------------- | ----- |
| client hello                 |  1    |
| server hello                 |  2    |
| new session ticket           |  4    |
| encrypted extensions         |  8    |
| certificate                  |  11   |
| certificate verify           |  15   |
| handshake finished           |  20   |

### TLS Handshake Extension Types (encode as 1-byte integer)

| Description                  | Value |
| ---------------------------- | ----- |
| server name                  |  0    |
| supported groups             |  10   |
| signature algorithm          |  13   |
| supported versions           |  43   |
| key share                    |  51   |

### Cipher Suites (encode bytes)

| Description                  | Value       |
| ---------------------------- | ----------- |
| TLS_AES_128_GCM_SHA256       | {0x13,0x01} |
| TLS_AES_256_GCM_SHA384       | {0x13,0x02} |
| TLS_CHACHA20_POLY1305_SHA256 | {0x13,0x03} |

### Signature Algorithms (encode as 2-byte integer)

| Description                  | Value    |
| ---------------------------- | -------- |
| ECDSA-SECP256r1-SHA256       |  0x0403  |
| ECDSA-SECP384r1-SHA384       |  0x0503  |
| ECDSA-SECP521r1-SHA512       |  0x0603  |
| ED25519                      |  0x0807  |
| ED448                        |  0x0808  |
| RSA-PSS-PSS-SHA256           |  0x0809  |
| RSA-PSS-PSS-SHA384           |  0x080a  |
| RSA-PSS-PSS-SHA512           |  0x080b  |
| RSA-PSS-RSAE-SHA256          |  0x0804  |
| RSA-PSS-RSAE-SHA384          |  0x0805  |
| RSA-PSS-RSAE-SHA512          |  0x0806  |
| RSA-PKCS1-SHA256             |  0x0401  |
| RSA-PKCS1-SHA384             |  0x0501  |
| RSA-PKCS1-SHA512             |  0x0601  |