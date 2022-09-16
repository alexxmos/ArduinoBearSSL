# BearSSL - QL changes
ArduinoECCX08@1.3.6 at the time of copying to local lib.  

1. Comment out line 50 at .pio/libdeps/arduino-esp32-bearssl/ArduinoBearSSL/src/SHA1.h
2. The verify function from the ECC608 has some problems. In file BearSSLClient.cpp replace as below:
```c++
// At lines 55 and 260 replace:
_ecVrfy = eccX08_vrfy_asn1;
// With:
_ecVrfy = br_ecdsa_vrfy_asn1_get_default(); 
```
3. To save stack memory only one instace of BearSSLClient is used for the QuarkLink object. This involve adding the following method to BearSSLClient class:
```c++
void BearSSLClient::setTAs(const br_x509_trust_anchor* myTAs, int myNumTAs) {
  _TAs = myTAs;
  _numTAs = myNumTAs;
}
```
and the declaration to the .h file
```c++
void setTAs(const br_x509_trust_anchor* myTAs, int myNumTAs);
```
4. The default input buffer of BearSSL is too small for the firmware update. Increase its size in BearSSLClient.h, line #33:
```c++
//#define BEAR_SSL_CLIENT_IBUF_SIZE 8192 + 85 + 325 - BEAR_SSL_CLIENT_OBUF_SIZE
/* 16405 bytes is the minimum size for a successfull fw update. Not tested with different firmwares, so it may vary?
 * Can't be too large (e.g 32768) as it does not fit and it fails build */
#define BEAR_SSL_CLIENT_IBUF_SIZE 16709 // slightly larger than minimum to be on the safe side
```

From the og project of [bearssl](https://www.bearssl.org/api1.html):

<i>The maximum size of an SSL/TLS record is 16384 plaintext bytes. With a MAC (at most 48 extra bytes, for HMAC/SHA-384), padding (at most 256 bytes), explicit IV for CBC (16 bytes) and record header (5 bytes), the maximum per-record overhead is 325 bytes. Thus, the buffer for incoming data should ideally have size 16384 + 325 = 16709 bytes. The macro BR_SSL_BUFSIZE_INPUT evaluates to that value.

For output, BearSSL does not use more padding bytes than necessary; but it also enforces the “1/n-1 split” when using TLS-1.0. The resulting overhead is then at most 85 bytes. The buffer for outgoing data should then have size 16384 + 85 = 16469 bytes. The macro BR_SSL_BUFSIZE_OUTPUT evaluates to that value.</i>

Not sure why Arduino reduced that.