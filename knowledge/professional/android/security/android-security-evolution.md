---
layout: master
title: android-security-evolution
---

# Overview

Android provides a multi-layered security model described in the Android Security Overview. 
Each update to Android includes dozens of security enhancements to protect users. The following are some of the security enhancements introduced in Android versions 1.5 through 4.1:

## Android 1.5
- ProPolice to prevent stack buffer overruns (-fstack-protector)
- safe_iop to reduce integer overflows
- Extensions to OpenBSD dlmalloc to prevent double free() vulnerabilities and to prevent chunk consolidation attacks. Chunk consolidation attacks are a common way to exploit heap corruption.
- OpenBSD calloc to prevent integer overflows during memory allocation

## Android 2.3

- Format string vulnerability protections (-Wformat-security -Werror=format-security)
- Hardware-based No eXecute (NX) to prevent code execution on the stack and heap
- Linux mmap_min_addr to mitigate null pointer dereference privilege escalation (further enhanced in Android 4.1)

## Android 4.0

- Address Space Layout Randomization (ASLR) to randomize key locations in memory

## Android 4.1

- PIE (Position Independent Executable) support
- Read-only relocations / immediate binding (-Wl,-z,relro -Wl,-z,now)
- dmesg_restrict enabled (avoid leaking kernel addresses)
- kptr_restrict enabled (avoid leaking kernel addresses)

## Android 4.2

**Application verification** - Users can choose to enable “Verify Apps" and have applications screened by an application verifier, prior to installation. App verification can alert the user if they try to install an app that might be harmful; if an application is especially bad, it can block installation.

**More control of premium SMS** - Android will provide a notification if an application attempts to send SMS to a short code that uses premium services which might cause additional charges. The user can choose whether to allow the application to send the message or block it.

**Always-on VPN** - VPN can be configured so that applications will not have access to the network until a VPN connection is established. This prevents applications from sending data across other networks.

**Certificate Pinning** - The Android core libraries now support certificate pinning. Pinned domains will receive a certificate validation failure if the certificate does not chain to a set of expected certificates. This protects against possible compromise of Certificate Authorities.

**Improved display of Android permissions** - Permissions have been organized into groups that are more easily understood by users. During review of the permissions, the user can click on the permission to see more detailed information about the permission.

**installd hardening** - The installd daemon does not run as the root user, reducing potential attack surface for root privilege escalation.

**init script hardening** - init scripts now apply O_NOFOLLOW semantics to prevent symlink related attacks.
FORTIFY_SOURCE - Android now implements FORTIFY_SOURCE. This is used by system libraries and applications to prevent memory corruption.

**ContentProvider default configuration** - Applications which target API level 17 will have "export" set to "false" by default for each Content Provider, reducing default attack surface for applications.
Cryptography - Modified the default implementations of SecureRandom and Cipher.RSA to use OpenSSL. Added SSL Socket support for TLSv1.1 and TLSv1.2 using OpenSSL 1.0.1

**Security Fixes** - Upgraded open source libraries with security fixes include WebKit, libpng, OpenSSL, and LibXML. Android 4.2 also includes fixes for Android-specific vulnerabilities. Information about these vulnerabilities has been provided to Open Handset Alliance members and fixes are available in Android Open Source Project. To improve security, some devices with earlier versions of Android may also include these fixes.


## Android 4.3

**Android sandbox reinforced with SELinux**. This release strengthens the Android sandbox using the SELinux mandatory access control system (MAC) in the Linux kernel. SELinux reinforcement is invisible to users and developers, and adds robustness to the existing Android security model while maintaining compatibility with existing applications. To ensure continued compatibility this release allows the use of SELinux in a permissive mode. This mode logs any policy violations, but will not break applications or affect system behavior.

**No setuid/setgid programs**. Added support for filesystem capabilities to Android system files and removed all setuid/setguid programs.  This reduces root attack surface and the likelihood of potential security vulnerabilities.

**ADB Authentication**. Since Android 4.2.2, connections to ADB are authenticated with an RSA keypair. This prevents unauthorized use of ADB where the attacker has physical access to a device.

**Restrict Setuid from Android Apps**. The /system partition is now mounted nosuid for zygote-spawned processes, preventing Android applications from executing setuid programs. This reduces root attack surface and the likelihood of potential security vulnerabilities.

**Capability bounding**. Android zygote and ADB now use prctl(PR_CAPBSET_DROP) to drop unnecessary capabilities prior to executing applications. This prevents Android applications and applications launched from the shell from acquiring privileged capabilities.

**AndroidKeyStore Provider**. Android now has a keystore provider that allows applications to create exclusive use keys. This provides applications with an API to create or store private keys that cannot be used by other applications.

**KeyChain isBoundKeyAlgorithm**. Keychain API now provides a method (isBoundKeyType) that allows applications to confirm that system-wide keys are bound to a hardware root of trust for the device. This provides a place to create or store private keys that cannot be exported off the device, even in the event of a root compromise.

**NO_NEW_PRIVS**. Android zygote now uses prctl(PR_SET_NO_NEW_PRIVS) to block addition of new privileges prior to execution application code. This prevents Android applications from performing operations which can elevate privileges via execve. (This requires Linux kernel version 3.5 or greater).

**FORTIFY_SOURCE enhancements**. Enabled FORTIFY_SOURCE on Android x86 and MIPS and fortified strchr(), strrchr(), strlen(), and umask() calls. This can detect potential memory corruption vulnerabilities or unterminated string constants.

**Relocation protections**. Enabled read only relocations (relro) for statically linked executables and removed all text relocations in Android code. This provides defense in depth against potential memory corruption vulnerabilities.

**Improved EntropyMixer**. EntropyMixer now writes entropy at shutdown / reboot, in addition to periodic mixing. This allows retention of all entropy generated while devices are powered on, and is especially useful for devices that are rebooted immediately after provisioning.

**Security Fixes**. Android 4.3 also includes fixes for Android-specific vulnerabilities. Information about these vulnerabilities has been provided to Open Handset Alliance members and fixes are available in Android Open Source Project. To improve security, some devices with earlier versions of Android may also include these fixes.

## Android 4.4

**Android sandbox reinforced with SELinux**. Android now uses SELinux in enforcing mode. SELinux is a mandatory access control (MAC) system in the Linux kernel used to augment the existing discretionary access control (DAC) based security model. This provides additional protection against potential security vulnerabilities.

**Per User VPN**. On multi-user devices, VPNs are now applied per user. This can allow a user to route all network traffic through a VPN without affecting other users on the device.

**ECDSA Provider support in AndroidKeyStore**. Android now has a keystore provider that allows use of ECDSA and DSA algorithms.

**Device Monitoring Warnings**. Android provides users with a warning if any certificate has been added to the device certificate store that could allow monitoring of encrypted network traffic.

**FORTIFY_SOURCE**. Android now supports FORTIFY_SOURCE level 2, and all code is compiled with these protections. FORTIFY_SOURCE has been enhanced to work with clang.

**Certificate Pinning**. Android 4.4 detects and prevents the use of fraudulent Google certificates used in secure SSL/TLS communications.

**Security Fixes**. Android 4.4 also includes fixes for Android-specific vulnerabilities. Information about these vulnerabilities has been provided to Open Handset Alliance members and fixes are available in Android Open Source Project. To improve security, some devices with earlier versions of Android may also include these fixes.

## Android 5.0

**Encrypted by default**. On devices that ship with L out-of-the-box, full disk encryption is enabled by default to improve protection of data on lost or stolen devices. Devices that update to L can be encrypted in Settings > Security.

**Improved full disk encryption**. The user password is protected against brute-force attacks using scrypt and, where available, the key is bound to the hardware keystore to prevent off-device attacks. As always, the Android screen lock secret and the device encryption key are not sent off the device or exposed to any application.

**Android sandbox reinforced with SELinux**. Android now requires SELinux in enforcing mode for all domains. SELinux is a mandatory access control (MAC) system in the Linux kernel used to augment the existing discretionary access control (DAC) security model. This new layer provides additional protection against potential security vulnerabilities.

**Smart Lock**. Android now includes trustlets that provide more flexibility for unlocking devices. For example, trustlets can allow devices to be unlocked automatically when close to another trusted device (via NFC, Bluetooth) or being used by someone with a trusted face.

**Multi user, restricted profile, and guest modes for phones & tablets**. Android now provides for multiple users on phones and includes a guest mode that can be used to provide easy temporary access to your device without granting access to your data and apps.

**Updates to WebView without OTA**. WebView can now be updated independent of the framework and without a system OTA. This will allow for faster response to potential security issues in WebView.

**Updated cryptography for HTTPS and TLS/SSL**. TLSv1.2 and TLSv1.1 is now enabled, Forward Secrecy is now preferred, AES-GCM is now enabled, and weak cipher suites (MD5, 3DES, and export cipher suites) are now disabled. See https://developer.android.com/reference/javax/net/ssl/SSLSocket.html for more details.

**non-PIE linker support removed**. Android now requires all dynamically linked executables to support PIE (position-independent executables). This enhances Android’s address space layout randomization (ASLR) implementation.

**FORTIFY_SOURCE improvements**. The following libc functions now implement FORTIFY_SOURCE protections: stpcpy(), stpncpy(), read(), recvfrom(), FD_CLR(), FD_SET(), and FD_ISSET(). This provides protection against memory-corruption vulnerabilities involving those functions.

**Security Fixes**. Android 5.0 also includes fixes for Android-specific vulnerabilities. Information about these vulnerabilities has been provided to Open Handset Alliance members, and fixes are available in Android Open Source Project. To improve security, some devices with earlier versions of Android may also include these fixes.

## Reference

* [Security Enhancements](http://source.android.com/devices/tech/security/enhancements/index.html)
* 
* 


