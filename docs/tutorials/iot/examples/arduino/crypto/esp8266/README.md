---
title: "Arduino Cpp-Crypto - ESP8266 Example"
---

# Crypto: ESP8266 Arduino Example Sketch

> you can find the Sketch `ESP8266.ino` [here](/iot/examples/arduino/crypto/assets/esp8266.ino).

```cpp
/**
 * This file is part of Solar Cpp Crypto.
 *
 * (c) Solar <info@solar.network>
 *
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 **/

/**
 * ESP8266 Cpp-Crypto Usage Sketch
 *
 * This sketch covers how to use the Cpp-Crypto library.
 * It allows your ESP8266 use Solar cryptographic protocols.
 */

 /**
 * NOTE: At the time of this writing, the Cpp-Crypto library requires running the 'ARDUINO_IDE.sh' bash script located in the 'extras' folder.
 * This converts our library to be compatible with the Arduino IDE.
 */

/****************************************/

/**
 * This is where you include the 'solarCrypto.h' header.
 * This allows your project to use Solar Cpp-Crypto.
 */
#include <solarCrypto.h>
/**/

/****************************************/

void checkCrypto() {
  /**
   * This is how you can check the default 'Network' "Transaction 'Fees' by type.
   * In this example, it should return a 'uint64_t' integer of '10000000' as the default 'Fee' for a 'Transaction' of 'Type' '0'.
   */
    Solar::Crypto::Configuration::Fee fee;
    unsigned long typeZeroTransactionFee = fee.get(0);
    Serial.print("\n Type 0 default Transaction Fee: ");
    Serial.println(typeZeroTransactionFee); // The response is a 'uint64_t' integer.

  /**/

  /********************/

  /**
   * The following methods allows you to create an Solar address.
   * This is done by passing a 12-word 'Passphrase' and the 'Network' 'Version' "byte".
   * The 'Version" "byte" is a BASE58 P2PKH byte. Solar Devnet is '0x1E'; Solar Mainnet is '0x17'.
   *
   * Given the passphrase ""bullet parade snow bacon mutual deposit brass floor staff list concert ask",
   * and the 'Devnet' 'Version' byte (0x1E); the Solar address should be "DStZXkgpEjxbG355nQ26vnkp95p24U9tsV"
   */
  const auto passphrase = "bullet parade snow bacon mutual deposit brass floor staff list concert ask";
  const uint8_t networkVersion = 0x1E;

  Address solarAddress = Address::fromPassphrase(passphrase, networkVersion);
    Serial.print("\nSolar Address: ");
    Serial.println(solarAddress.toString().c_str()); // The 'Address' object is a type. Use 'toString()' to view the output. Arduino requires a 'c_str()' to 'print'.
  /**/


  /********************/

  /**
   * The following methods allows create a 'PrivateKey'.
   * This is done by passing a 12-word 'Passphrase'.
   *
   * Given the passphrase ""bullet parade snow bacon mutual deposit brass floor staff list concert ask",
   * the 'PrivateKey" should be "950981ce17df662dbc1d25305f8597a71309fb8f7232203a0944477e2534b021".
   * This is a 'SHA256' of your "Passphrase".
   */
  const auto passphrase2 = "bullet parade snow bacon mutual deposit brass floor staff list concert ask";
  PrivateKey privateKeyFromPassphrase = PrivateKey::fromPassphrase(passphrase2);
    Serial.print("\nPrivateKey from Passphrase: ");
    Serial.println(privateKeyFromPassphrase.toString().c_str()); // The 'PrivateKey' object is a type. Use 'toString()' to view the output. Arduino requires a 'c_str()' to 'print'.
  /**/

  /********************/

  /**
   * The following methods allows create a 'PublicKey'.
   * This is done by passing a 12-word 'Passphrase'.
   *
   * Given the passphrase ""bullet parade snow bacon mutual deposit brass floor staff list concert ask",
   * the 'PublicKey" should be "029fdf41a7d69d8efc7b236c21b9509a23d862ea4ed8b13a56e31eee58dbfd97b4".
   */
  const auto passphrase3 = "bullet parade snow bacon mutual deposit brass floor staff list concert ask";
  PublicKey publicKeyFromPassphrase = PublicKey::fromPassphrase(passphrase3);
    Serial.print("\nPublicKey from Passphrase: ");
    Serial.println(publicKeyFromPassphrase.toString().c_str()); // the 'PublicKey' object is a type. Use 'toString()' to view the output. Arduino requires a 'c_str()' to 'print'.
  /**/

  /********************/

  /**
   * The following methods allows create a 'WIF'-style "PrivateKey".
   * 'WIF' stands for "Wallet Import Format"
   * This is done by passing a 12-word 'Passphrase' and the 'Network' 'WIF' "byte".
   * The 'WIF" "byte" is a BASE58 WIF byte. Solar Devnet is '0xaa'; Solar Mainnet is also '0xaa'.

   *
   * Given the passphrase ""bullet parade snow bacon mutual deposit brass floor staff list concert ask",
   * and the 'Devnet' 'WIF' byte (0xaa);
   * The 'WIF" should be "SEZuJZouNK8GLXNApjciH4QnSKiNr971exVcL2Y6XfrDF5o977zB".
   */
  const auto passphrase4 = "bullet parade snow bacon mutual deposit brass floor staff list concert ask";
  const uint8_t wifByte = 0xaa;
  WIF wifFromPassphrase = WIF::fromPassphrase(passphrase4, wifByte);
    Serial.print("\nWIF from Passphrase: ");
    Serial.println(wifFromPassphrase.toString().c_str()); // the 'WIF' object is a type. Use 'toString()' to view the output. Arduino requires a 'c_str()' to 'print'.
  /**/
};

/****************************************/

void setup()
{
  Serial.begin(115200); // Begin your Serial Connection. This allows you to monitor your boards output.

  checkCrypto(); // Begin Crypto example usage.
};

/****************************************/

void loop() {}; // We can leave this empty, as we don't want to repeat anything in this example.

```
