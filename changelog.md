# BearSSL - QL changes

1. Comment out line 50 at .pio/libdeps/arduino-esp32-bearssl/ArduinoBearSSL/src/SHA1.h. This singleton causes collisions with the ESP32 core package.
2. The verify function from the ECC608 has some problems. In file BearSSLClient.cpp replace as below:
```c++
// At lines 55 and 260 replace:
_ecVrfy = eccX08_vrfy_asn1;
// With:
_ecVrfy = br_ecdsa_vrfy_asn1_get_default(); 
```
3. To save stack memory only one instace of BearSSLClient is used through the project. This involve adding the following method to BearSSLClient class:
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