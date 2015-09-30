---
layout: master
title: Android keystore and Keymaster
---

## Overview

- Keystore: A keystore is a signed collection of public keys.
- Keymaster: is a newly introduced key management Hardware Abstraction Layer (HAL)
 component.

## Types

Types of keymaster HAL

### Software-based Keymaster
Uses the OpenSSL software implementation. Jelly Bean comes
with a default softkeymaster module that does all key operations in software only.

### Hardware-based keymaster
Uses TZ application APIs (keymaster application). Hardware
keymaster support essentially ensures that the key stored is not accessible in HLOS.
Regardless of key type (RSA/EC), the keyblob generated is encrypted by a key accessible by
TZ software only and stored in the File System (FS) on the HLOS end.

## Technology

###Key blobs

Keymaster v1.0 key blobs are wrapped inside keystore blobs.

**keystore blobs**
in turn stored as files in /data/misc/keystore/user_X, as before (where X is
the Android user ID, starting with 0 for the primary user).

	261typedef struct {
	262    const uint8_t* key_material;
	263    size_t key_material_size;
	264} keymaster_key_blob_t;

**FORMAT**

Keymaster blobs are variable size and employ a tag-length-value
(TLV) format internally. 

They include 

	a version byte, 
	a nonce, 
	encrypted key material, 
	a tag for authenticating the encrypted key, 
	as well as two authorization sets (enforced and unenforced), which contain the key's properties. 

**encryption**

1. Key material is encrypted using AES in OCB [http://web.cs.ucdavis.edu/~rogaway/ocb/ocbfaq.
htm] mode,  which automatically authenticates the cipher text and produces an
authentication tag upon completion. 



2. Each key blob is encrypted with a dedicated key encryption key (KEK), which is derived by hashing a binary tag representing the key's root of trust (hardware or software), concatenated with they key's authorization sets. Finally, the resulting hash value is encrypted with the master key to derive the blob's KEK.

The current software implementation deliberately uses a 128bit AES zero key, and employs a constant, allzero nonce for all keys. 

The current format is quite easy to decrypt, and while this will likely change in the final M version, in the mean time you can decrypt
keymaster v1.0 blobs using the keystoredecryptor [https://github.com/nelenkov/keystoredecryptor] tool. 



### Keymaster 0.3

- L: /hardware/libhardware/include/hardware/keymaster.h
- M: /hardware/libhardware/include/hardware/keymaster0.h

	
	generate_keypair
	import_keypair
	get_keypair_public
	delete_keypair
	delete_all
	sign_data
	verify_data

### Keymaster 1.0

**Changes**

	1. introduces several new keystore features into the framework API
	2. generating and using symmetric keys that are protected by the system keystore.
	3. introduces a keystorebacked symmetric KeyGenerator, and adds support for the KeyStore.SecretKeyEntry JCA class, which allows storing and retrieving symmetric keys via the standard java.security.KeyStore JCA API
	4. 'key characteristics'. there are a lot more parameters you can set when generating (or importing a key). Along with basic properties such as key size and alias, you can now specify the supported key usage(encryption/decryption or signing/verification), block mode, padding, etc.Those properties are stored along with the key, and the system will disallow key usage which doesn't match the key's attributes.
	5. requiring use authentication before allowing a particular key to be used, and specifying the authentication validity period.


**New keystore APIs**

- The new interface allows for setting finegrained key properties (also called 'key characteristics' internally), 
- supports breaking up cryptographic operations that manipulate data of unknown or large size into multiple steps using the familiar begin/update/finish pattern.

	
	The keymaster HAL and its reference implementation have, however,
	been completely redesigned. The 'old' keymaster HAL is retained for
	backward compatibility as version 0.3, while the Android M version has
	been bumped to 1.0, and offers a completely different interface. 


#### Key Tags/ key characteristics

Key properties are stored as a series of tags along with the key, and form an authorization
set when combined. 

	frameworks/base/core/java/android/security/keymaster/KeymasterDefs.java

**Tag Types**

Each tag has an associated type. For example:

- ■ KM_TAG_PURPOSE and KM_TAG_MODE tags have values from specified enumerations
- ■ KM_TAG_ALL_USERS is boolean, either all device users are allowed to use the key, or not
- ■ KM_TAG_ACTIVE_DATETIME is a timestamp specifying when the key becomes active

Some tags can be repeated, such as KM_TAG_PURPOSE

Current list of tag types is: 

	KM_ENUM, KM_ENUM_REP, KM_INT, KM_INT_REP, KM_LONG, KM_DATE, KM_BOOL, KM_BIGNUM, KM_BYTES

##### 1 Key Authorization Tags

In principle, each tag associated with a key represents a capability, a way in which that key may be used, eg:

- ■ An AES key with the KM_TAG_PURPOSE tag, with value KM_PURPOSE_ENCRYPT may be used to encrypt data
- ■ If the AES key does not have the KM_TAG_MODE tag with

the value KM_MODE_ECB, that key may not be used to encrypt data in ECB mode

Authorizations are cryptographically bound to keys

##### 2 Descriptive Tags

Some tags aren’t actual “authorizations” but mere descriptions. The authorization tag mechanism is
used anyway, for simplicity and consistency. For example:

- ● KM_TAG_KEY_SIZE
- ● KM_TAG_RSA_PUBLIC_EXPONENT
- ● KM_TAG_ORIGIN

##### 3 Parameter Tags

Some tags are used to pass parameters. Again, this is for simplicity and consistency, to have a
single mechanism. Examples:

- ● KM_TAG_NONCE
- ● KM_TAG_ADDITIONAL_DATA (for authenticated encryption with additional data block cipher modes)

##### Code analyzing


##### struct keymaster_key_param_t
hardware/libhardware/include/hardware/keymaster_defs.h

		232typedef struct {
		233    keymaster_tag_t tag;
		234    union {
		235        uint32_t enumerated;   /* KM_ENUM and KM_ENUM_REP */
		236        bool boolean;          /* KM_BOOL */
		237        uint32_t integer;      /* KM_INT and KM_INT_REP */
		238        uint64_t long_integer; /* KM_LONG */
		239        uint64_t date_time;    /* KM_DATE */
		240        keymaster_blob_t blob; /* KM_BIGNUM and KM_BYTES*/
		241    };
		242} keymaster_key_param_t;

		244typedef struct {
		245    keymaster_key_param_t* params; /* may be NULL if length == 0 */
		246    size_t length;
		247} keymaster_key_param_set_t;


##### class AuthorizationSet

	Contains():Returns true if the set contains the specified tag and value
	GetTagValue():If the specified integer-typed \p tag exists, places its value in \p val and returns true

#### keymaster_key_characteristics_t;

		256typedef struct {
		257    keymaster_key_param_set_t hw_enforced;
		258    keymaster_key_param_set_t sw_enforced;
		259} keymaster_key_characteristics_t;


#### different interface

- M: /hardware/libhardware/include/hardware/keymaster1.h

- ■ Key characteristics are specified in a tag-value list.
- ■ Crypto operations follow a begin / update / finish model, rather than being single-shot
- ■ Implementations are free to consume only a portion of data provided to “update”; caller will resend
- remainder


##### generate_key


GenerateKey

SoftKeymasterDevice->android_keymaster->rsa_keymaster0_key.cpp(KM_ALGORITHM_RSA).GenerateKey

- params in generate_key in SoftKeymasterDevice

		params into GenerateKeyRequest in key_description and into AuthorizationSet key_description;
		request.key_description.Reinitialize(*params);
			struct GenerateKeyRequest {AuthorizationSet key_description;}

- rsa_keymaster0_key.GenerateKey

		hw_enforced: Putting TAG_ALGORITHM/TAG_RSA_PUBLIC_EXPONENT/TAG_KEY_SIZE/TAG_ORIGIN in the **hw_enforced**

- CreateKeyBlob with key_description in SoftKeymasterContext
	
		
		SetAuthorizations: check and ignore some TAG/ Everything else TAG just copy into **sw_enforced**
		BuildHiddenAuthorizations
		SerializeIntegrityAssuredBlob


- SerializeIntegrityAssuredBlob in integrity_assured_key_blob.cpp


		94    uint8_t* p = key_blob->writable_data();
		95    *p++ = BLOB_VERSION;
		96    p = key_material.Serialize(p, key_blob->end());
		97    p = hw_enforced.Serialize(p, key_blob->end());
		98    p = sw_enforced.Serialize(p, key_blob->end());
		99
		100    return ComputeHmac(key_blob->key_material, p - key_blob->key_material, hidden, p);
  
AuthorizationSet


##### get_key_characteristics

SoftKeymasterDevice->android_keymaster.cpp

- GetKeyCharacteristics/android_keymaster.cpp
- ParseKeyBlob
- ParseKeyBlob/soft_keymaster_context.cpp

	 DeserializeIntegrityAssuredBlob(blob, h


##### export_key

key_to_export


**How to get and handle Key Tags in export_key**

- Where to modify on getting and handling key tag?

	Better to do it here

		/system/keymaster/android_keymaster.cpp
		void AndroidKeymaster::ExportKey(const ExportKeyRequest& request, ExportKeyResponse* response) {
	
- How to get additional key tag?
	
	Use hw_enforced and sw_enforced gotten from ParseKeyBlob, and addtional SOTER KM_TAG are all in sw_enforced
	
- How to check if a specific KM_TAG_XXX is exsit or not?

	use Contains method from AuthorizationSet

        if (sw_enforced.Contains(KM_TAG_XXX, 1))
        {
			//KM_TAG_XXX is exsit
			...
		}

##### begin

Begins a cryptographic operation using the specified key.  If all is well, begin() will return KM_ERROR_OK and **create an operation handle** which must be passed to subsequent calls to update(), finish() or abort().

It is critical that each call to begin() be paired with a subsequent call to finish() or abort(), to allow the keymaster implementation to clean up any internal operation state. Failure to do this may leak internal state space or other internal resources and may eventually cause begin() to return KM_ERROR_TOO_MANY_OPERATIONS when it runs out of space for

Key authorization enforcement is performed primarily in begin .

**Private key operations** ( KM_PURPOSE_DECYPT and KM_PURPOSE_SIGN) require
authorization of digest and padding, which means that the specified values must be in
the key authorizations. If not, the method must return
KM_ERROR_INCOMPATIBLE_DIGEST or KM_ERROR_INCOMPATIBLE_PADDING, as
appropriate. 

**Public key operations** ( KM_PURPOSE_ENCRYPT and KM_PURPOSE_VERIFY)
are permitted with unauthorized digest or padding.

**operation_handle**

		keymaster_operation_handle_t* operation_handle
		typedef uint64_t keymaster_operation_handle_t;

**operation_table_**

**operation**

operation->SetAuthorizations(key->authorizations());


##### update

Provides data to, and possibly receives output from, an ongoing cryptographic operation begun with begin().

The operation is specified by the operation_handle parameter.

To provide more flexibility for buffer handling, implementations of this method have the
option of consuming less data than was provided.


Implementations may also choose how much data to return, as a result of the update.
This is only relevant for encryption and decryption operations, since signing and verification return no data until finish .


The caller must provide the auth token to every call to update and finish .

##### finish

Finalizes a cryptographic operation begun with begin() and invalidates \p operation_handle.
processing all of the as-yet-unprocessed data provided by update (s).

Key authorization enforcement is performed primarily in begin .


**How to get and handle Key Tags in finish()**

- Where to modify on getting and handling key tag?

	Better to do it here

		/system/keymaster/android_keymaster.cpp
		void AndroidKeymaster::FinishOperation(const FinishOperationRequest& request,
                                       FinishOperationResponse* response) {
	
- How to get additional key tag?
	
	use it from operation->authorizations()
	
- How to check if a specific KM_TAG_XXX is exsit or not?

	use Contains method from AuthorizationSet

        if (operation->authorizations().Contains(KM_TAG_XXX, 1))
        {
			//KM_TAG_XXX is exsit
			...
		}

implementation 1: Is the purpose in begin() is signing && the key has param of KM_TAG_SOTER_IS_FROM_SOTER

		if ( operation->purpose() == KM_PURPOSE_SIGN && operation->authorizations().Contains(KM_TAG_SOTER_IS_FROM_SOTER, 1))
		{
			//purpose in begin() is signing && the key has param of KM_TAG_SOTER_IS_FROM_SOTER
			...
		}


#### key characteristics

The new interface allows for setting finegrained key properties (also called 'key characteristics' internally), 

Key properties are stored as a series of tags along with the key, and form an authorization set when combined. 


#### breaking up cryptographic operations
and supports breaking up cryptographic operations that manipulate data of unknown or large size into multiple steps using the familiar begin/update/finish pattern. 

### software reference keymaster

AOSP includes a pure software reference keymaster implementation which implements cryptographic operations using OpenSSL and encrypts key blobs using a provided master key.

/system/keymaster/soft_keymaster_device.cpp

**keymaster1_device_t**

keymaster/include/keymaster/soft_keymaster_device.h

**SoftKeymasterDevice**

contains all keymaster0_device_t and keymaster1_device_t.

contains UniquePtr<AndroidKeymaster> impl_;

keymaster1_device_t converting to SoftKeymasterDevice* * dev,


**AndroidKeymaster**

class AndroidKeymaster { class in namespace:keymaster 

	31 * This is the reference implementation of Keymaster.  In addition to acting as a reference for
	32 * other Keymaster implementers to check their assumptions against, it is used by Keystore as the
	33 * default implementation when no secure implementation is available, and may be installed and
	34 * executed in secure hardware as a secure implementation.
	35 *
	36 * Note that this class doesn't actually implement the Keymaster HAL interface, instead it
	37 * implements an alternative API which is similar to and based upon the HAL, but uses C++ "message"
	38 * classes which support serialization.
	39 *
	40 * For non-secure, pure software implementation there is a HAL translation layer that converts the
	41 * HAL's parameters to and from the message representations, which are then passed in to this
	42 * API.
	43 *
	44 * For secure implementation there is another HAL translation layer that serializes the messages to
	45 * the TEE. In the TEE implementation there's another component which deserializes the messages,
	46 * extracts the relevant parameters and calls this API.



## gatekeeper/Perkey authorization

**What**

setUserAuthenticationRequired() method of the key
parameter builder allows you to require that the user authenticates before they are authorized to use a certain
key

**How**

The system keystore service now holds an authentication token table, and a key
operation is only authorized if the table contains a matching token. Tokens include an HMAC and thus can
provide a strong guarantee that a user has actually authenticated at a given time, using a particular
authentication method.


Authentication tokens are now part of Android M, and currently support two authentication methods:
password and fingerprint.

Tokens are generated by a newly introduced system component, called the gatekeeper. The gatekeeper
issues a token after it verifies the userentered password against a previously enrolled one.

## Test

### verify the keymaster APIs

/hardware/libhardware/tests/keymaster/

Run the commands to verify the keymaster APIs 

	adb shell /data/keymaster_test
	adb shell keymaster_test --gtest_list_tests
	adb shell keymaster_test --gtest_filter=*GetKeypairPublic_RSA_Success*


### Run CTS test suite

1. Install the CTS test suite from http://source.android.com/compatibility/cts-intro.html. 
the keystore test cases :android-cts\repository\testcases.
2. Install the keystore test cases on the device using adb install CtsKeystoreTestCases.apk.
3. Run the command from the CTS shell to test the keystore APIs:
run cts -c android.keystore.cts.KeyChainTest –m testIsBoundKeyAlgorithm_RequiredAlgorithmsSupported
4. Check adb logcat on the status of the keystore test cases for results.
## Reference

* [jelly-bean-hardware-backed-credential](http://nelenkov.blogspot.com/2012/07/jelly-bean-hardware-backed-credential.html)
* [keystore-redesign-in-android-m](http://nelenkov.blogspot.com/2015/06/keystore-redesign-in-android-m.html)
* 


