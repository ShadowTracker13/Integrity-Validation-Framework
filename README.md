# Automated Integrity Validation Framework for Secure USB Storage

**Research project \| Global AI Summit 2026 \| Paper ID: 249**

An application-level security framework for USB flash storage that
combines authenticated encryption, digital signatures, file-integrity
hashing, and event-driven automation. Its goal is to detect unauthorized
changes to protected files and prevent tampered data from being released
as trusted plaintext.

> **Repository status:** This repository is currently a public project
> overview. The complete source code, setup instructions, and
> reproducibility materials are not available yet and will be published
> here in a later update. This README is not an installation guide or a
> claim that a ready-to-run release is available.

------------------------------------------------------------------------

## Research paper

-   **Title:** Automated Integrity Validation Framework for Secure USB
    Storage
-   **Author:** Rahul Kumar Yadav
-   **Co-authors:** Shahana Gajala Qureshi, Pushpinder Singh Patheja,
    Praveen Lalwani, and Sasmita Padhy
-   **Affiliation:** VIT Bhopal University
-   **Conference:** International Conference on Artificial Intelligence
    & Emerging Technology 3.0 --- Global AI Summit 2026
-   **Paper ID:** 249

## 1. Motivation

USB flash drives are portable and convenient, but their physical
accessibility creates risks. An attacker who temporarily obtains an
unattended drive may copy, replace, insert, or modify files before the
device is reconnected to a trusted computer.

Conventional encryption primarily protects confidentiality. This project
focuses on an additional storage-level security layer: validating the
authenticity and integrity of protected files before releasing
plaintext.

The framework is designed to complement---not replace---full-disk
encryption, endpoint protection, or host-side USB security controls.

## 2. Core security components

  -----------------------------------------------------------------------
  Component                           Role in the framework
  ----------------------------------- -----------------------------------
  **BLAKE3**                          Used in the prototype for
                                      password-based 256-bit key
                                      derivation and file-integrity
                                      hashing.

  **AES-256-EAX**                     Authenticated encryption. EAX
                                      produces an authentication tag that
                                      is checked during decryption.

  **RSA-2048**                        Digital signatures. The private
                                      signing key remains on the trusted
                                      host; the corresponding public key
                                      is used for verification.

  **fswatch + macOS LaunchAgents**    Event-driven monitoring and
                                      automation for USB/file events on
                                      macOS.
  -----------------------------------------------------------------------

The framework uses existing cryptographic primitives rather than
proposing a new cryptographic algorithm.

## 3. High-level workflow

### Lockdown --- encryption

1.  A USB connection or file-change event is detected by the automation
    layer.
2.  The user provides a password.
3.  The prototype derives a 256-bit AES key using BLAKE3.
4.  Files are encrypted using AES-256-EAX, producing ciphertext and an
    authentication tag.
5.  A BLAKE3 integrity reference is calculated.
6.  The encrypted payload and associated metadata are signed using the
    RSA private key held on the trusted host.
7.  The framework completes its re-encryption procedure before the USB
    drive is unmounted.

### Unlock --- validation and decryption

1.  On USB reconnection, the event handler triggers the validation
    workflow.
2.  The corresponding RSA public key is used to verify the signature.
3.  The user enters the password and the AES key is derived at runtime.
4.  AES-EAX decryption and authentication-tag validation are performed.
5.  The BLAKE3 hash is recalculated and compared with the stored
    reference.
6.  If validation succeeds, plaintext can be released. If a check fails,
    execution stops and the configured response policy is applied.

**Important:** The detailed decryption procedure in the paper places
BLAKE3 hash recomputation after authenticated decryption. Plaintext
release remains conditional on successful validation.

## 4. Response policies

The research describes two response modes.

### Quarantine Mode

-   Intended as a non-destructive response.
-   When tampering is detected, decryption is refused and an alert is
    issued.
-   The affected data is handled through the quarantine response rather
    than the logical-wipe response.
-   A mistaken password should not be treated as proof of tampering or
    trigger a wipe. This is the intended mode policy described for this
    project, not a separately reported experimental result in the paper.

### High-Assurance Mode

-   Intended for situations where protection takes priority over
    recovery.
-   The reported response to detected tampering is to initiate a logical
    wipe.
-   A logical wipe is not guaranteed physical destruction: residual data
    may remain recoverable from NAND flash using advanced forensic
    techniques.

Both modes are designed to prevent unvalidated plaintext from being
released.

## 5. Experimental evaluation reported in the paper

The paper reports:

-   **50 tamper-detection trials** across Quarantine and High-Assurance
    modes.
-   Simulated cases including unauthorized file insertion, signature
    tampering, and file-data/hash mismatch.
-   Every tampering case included in the evaluation was detected.
-   Representative encryption/decryption performance measurements across
    payload sizes from 10 MB to 500 MB, with values reported as the
    sample mean over **N = 3 independent repetitions**.
-   A separate 60 MB baseline measurement: 7.69 seconds total
    write-and-encrypt duration, and 0.579 seconds for read,
    verification, and decryption.
-   USB attachment-to-validation trigger delay of approximately 0.50
    seconds.

These results should be interpreted in the context of the stated test
environment and repetition count; they are not universal benchmarks.

## 6. Test environment and technology stack

The research paper reports the following test environment:

  Item                Reported configuration
  ------------------- ------------------------------------------------------
  Host                Apple MacBook Pro A2141
  CPU                 Intel Core i7-9750H @ 2.60 GHz, 6 cores / 12 threads
  Memory              16 GB DDR4-2667 MHz
  Operating system    macOS 26.6.2
  Removable storage   64 GB SanDisk Cruzer Blade
  File system         exFAT, 128 KB allocation units
  Language            Python 3.9.6
  Cryptography        PyCryptodome 3.23.0
  Hashing             BLAKE3 1.0.5 with AVX2 acceleration
  Automation          fswatch 1.17.1 and macOS LaunchAgents
  Timing              Python `time.perf_counter_ns()`

Test payloads included text, document, and pseudorandom binary files.

## 7. Limitations

The paper identifies several limitations:

-   The current event-driven implementation is macOS-specific; Windows
    and Linux support require porting the automation layer.
-   BLAKE3 is not memory-hard. Offline password guessing remains a
    concern when passwords are weak or reused.
-   A memory-hard password KDF such as Argon2id is proposed for
    production-oriented deployments, while BLAKE3 remains useful for
    file-integrity hashing.
-   High-Assurance Mode performs a logical wipe; NAND flash remnants may
    still be recoverable.
-   Very large datasets may introduce staging bottlenecks during signing
    and encryption.
-   The framework does not claim to prevent every physical attack,
    compromised-host scenario, or offline cryptanalysis attempt.

## 8. Planned future work

The paper identifies the following directions:

-   Evaluate Argon2id for password-based key derivation.
-   Evaluate Ed25519 signatures as an alternative to RSA-2048.
-   Explore cryptographic shredding for faster key-based data
    invalidation.
-   Port event-driven automation to Linux and Windows.
-   Explore a hardware kill-switch approach.
-   Investigate file-system/partition encryption such as AES-XTS or LUKS
    to protect metadata.

These are proposed future improvements, not features claimed as
completed in the current prototype.

## 9. Repository contents and availability

At this stage, this repository is a project landing page and research
overview.

The following materials are planned for later publication:

-   Source code and modular implementation.
-   Installation and configuration instructions.
-   Demonstration and usage examples.
-   Reproducibility notes and supporting documentation.

**The complete code is not available in this repository yet.** This
README will be updated as the materials are prepared and released.

## 10. Citation

Rahul Kumar Yadav, Shahana Gajala Qureshi, Pushpinder Singh Patheja,
Praveen Lalwani, and Sasmita Padhy, "Automated Integrity Validation
Framework for Secure USB Storage," International Conference on
Artificial Intelligence & Emerging Technology 3.0, Global AI Summit
2026, Paper ID 249.

------------------------------------------------------------------------

## Acknowledgement

Developed as a research project at **VIT Bhopal University** and
presented at **Global AI Summit 2026**.
