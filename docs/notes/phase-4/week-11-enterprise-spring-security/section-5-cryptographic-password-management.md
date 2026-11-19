---
id: phase-4-week-11-section-5-cryptographic-password-management
aliases: []
tags:
- spring-security
- password-encoder
- bcrypt
- argon2
- pbkdf2
- scrypt
- salt
- work-factor
- compromised-password-checker
- k-anonymity
- phase-4
---

# Section 5: Cryptographic Password Management, Adaptive Hashing & Breach Screening

## 1. Architectural Problem Statement: The Plaintext Credential Catastrophe

In enterprise software engineering, storing authentication credentials in plaintext is the single most severe architectural vulnerability an organization can introduce. Storing raw passwords in persistent storage exposes applications to catastrophic failure modes across multiple vectors:

```mermaid
flowchart TD
    subgraph Vectors["Compromise Threat Vectors"]
        direction TB
        V1["SQL Injection (SQLi) Dump"]
        V2["Internal Malicious Actor (Rogue DBA / DevOps)"]
        V3["Unencrypted Database Backups & S3 Snapshots"]
        V4["Application Log Pollution (logger.info logging DTOs)"]
    end

    subgraph Database["Database State: PLAINTEXT"]
        DB["users table<br/>username: john@example.com<br/>password: 'EazyBytes@12345'"]
    end

    subgraph Impact["Catastrophic Fallout"]
        direction TB
        I1["Instant Credential Harvesting across All Accounts"]
        I2["Global Credential Stuffing on Third-Party Services"]
        I3["Severe Regulatory Penalties (GDPR, PCI-DSS, HIPAA)"]
        I4["Complete Loss of Enterprise Brand Trust"]
    end

    V1 --> DB
    V2 --> DB
    V3 --> DB
    V4 --> DB
    DB --> I1
    DB --> I2
    DB --> I3
    DB --> I4
```

### 1.1 Real-World Exploit Vectors

1. **SQL Injection (SQLi) Exfiltration:** An attacker injects SQL payloads into an unescaped query parameter, extracting full relational dumps of the credentials table. If stored in plaintext, every credential in the system is instantly readable without cryptanalysis.
2. **Insider Threat (The Rogue DBA / SRE Dilemma):** Database administrators, cloud engineers, and platform developers frequently possess read privileges on production replicas and database backups. Plaintext storage grants every privileged employee uninhibited access to customer and executive accounts.
3. **Storage Snapshot and Cold Storage Leaks:** Database snapshots exported to Amazon S3, Google Cloud Storage, or off-site backup tapes often lack the stringent identity-access boundaries of production OLTP clusters. A misconfigured bucket policy turns into a public corporate disaster.
4. **Application Log Ingestion Pollution:** When developers log incoming payloads (`log.info("Registering user: {}", userRegistrationDto)`), unmasked passwords propagate into centralized logging pipelines (Elasticsearch, Datadog, Splunk, CloudWatch), making plaintext secrets accessible to anyone with log-viewing privileges.

### 1.2 Regulatory and Compliance Liabilities
Enterprise standards explicitly mandate cryptographic protection of credentials at rest:

- **PCI-DSS 4.0 (Requirement 8.3.4):** Passwords and passphrases must be rendered unreadable using strong, one-way cryptographic functions.
- **GDPR (Article 32):** Demands "pseudonymisation and encryption of personal data" as appropriate technical and organisational measures. Storing plaintext passwords guarantees maximum tier-two administrative fines (up to €20 million or 4% of total worldwide annual turnover).
- **NIST SP 800-63B (Section 5.1.1.2):** Mandates that credential verifiers store secrets using salted, computationally difficult key derivation or password-hashing functions.

---

## 2. The Cryptographic Trilemma: Encoding vs. Encryption vs. Hashing

Software engineers frequently conflate three distinct data-transformation mechanisms: **Encoding**, **Encryption**, and **Cryptographic Hashing**. Choosing the wrong primitive for password storage creates critical vulnerabilities.

```mermaid
flowchart TD
    subgraph EncodingConcept["1. Encoding (e.g. Base64, ASCII, Hex)"]
        direction TB
        E1["Plaintext: 'admin:12345'"] --> E2["Transformation: Fixed Algorithm (No Key)"]
        E2 --> E3["Encoded Text: 'YWRtaW46MTIzNDU='"]
        E3 -->|"Trivial Reverse (Zero Keys)"| E1
        E_NOTE["Zero Security. No Confidentiality.<br/>Used strictly for data transport & serialization."]
    end

    subgraph EncryptionConcept["2. Encryption (e.g. AES-256-GCM, RSA-4096)"]
        direction TB
        ENC1["Plaintext: 'SecretData'"] --> ENC2["Cipher + Secret / Public Key"]
        ENC2 --> ENC3["Ciphertext: Binary / Hex Bytes"]
        ENC3 -->|"Requires Secret / Private Key"| ENC1
        ENC_NOTE["Two-Way Cryptography.<br/>Fatal for passwords: Key compromise exposes ALL secrets."]
    end

    subgraph HashingConcept["3. Cryptographic Hashing (e.g. BCrypt, Argon2)"]
        direction TB
        H1["Plaintext: 'P@ssword123'"] --> H2["One-Way Mathematical Trapdoor Function"]
        H2 --> H3["Digest: '$2a$12$...'"]
        H3 -.->|"MATHEMATICALLY IMPOSSIBLE TO REVERSE"| H1
        H_NOTE["One-Way Digestion.<br/>Authentication verified by re-computing hash over input."]
    end
```

### 2.1 Encoding: Data Representation without Confidentiality
**Encoding** transforms data from one representation into another using a publicly known, standardized algorithm without secret keys.

- **Primary Use Case:** Ensuring binary data survives network protocols designed strictly for ASCII text (e.g., transmitting binary images inside JSON or HTTP Basic Authentication headers).
- **Mechanics:** Base64 takes 3 bytes of binary data (24 bits) and splits them into 4 groups of 6 bits, mapping each 6-bit value to an index in a 64-character table (`A-Z`, `a-z`, `0-9`, `+`, `/`).
- **Security Reality:** Encoding provides **zero confidentiality**. Any tool, script, or browser can decode it instantaneously:
  ```bash
  echo -n "YWRtaW46MTIzNDU=" | base64 --decode
  # Outputs: admin:12345
  ```

### 2.2 Encryption: Two-Way Confidentiality and the Key Custody Trap
**Encryption** transforms plaintext into ciphertext using an encryption algorithm and a secret cryptographic key. It is explicitly designed to be **two-way (reversible)** by authorized entities holding the decryption key.

- **Symmetric Encryption (e.g., AES-256-GCM, ChaCha20):** A single secret key is used for both encryption and decryption. Ideal for bulk data at rest (e.g., disk volume encryption, credit card vault storage).
- **Asymmetric Encryption (e.g., RSA, ECC):** A mathematically linked key pair: a public key encrypts data, while a private key decrypts data. Ideal for establishing secure transport channels (TLS handshake) and digital signatures.

#### Hands-On Demonstration: OpenSSL AES-256-CBC Encryption & Decryption
To understand why encryption is two-way, consider encrypting a plaintext password file using AES-256:

```bash
# 1. Create a plaintext password file
echo -n "EazyBytes@12345" > plain.txt

# 2. Encrypt using AES-256-CBC with PBKDF2 key derivation and a secret passkey
openssl enc -aes-256-cbc -salt -in plain.txt -out encrypted.txt -pass pass:12345 -pbkdf2 -base64

# 3. Inspect encrypted output (Base64-encoded ciphertext)
cat encrypted.txt
# Output: U2FsdGVkX1+v8R6uF1i... (Unreadable scrambled ASCII)

# 4. Decrypt using the identical secret key
openssl enc -aes-256-cbc -d -in encrypted.txt -out decrypted.txt -pass pass:12345 -pbkdf2 -base64

# 5. Verify the recovered plaintext
cat decrypted.txt
# Output: EazyBytes@12345

# 6. Attempt decryption with an incorrect secret key
openssl enc -aes-256-cbc -d -in encrypted.txt -out failed.txt -pass pass:WrongKey -pbkdf2 -base64
# Output: bad decrypt / message authentication failed
```

#### Why Encryption Fails as a Password Storage Strategy
Many developers mistakenly believe: *"If we encrypt the passwords in the database with AES-256, attackers cannot read them without the key!"*

In an enterprise application, this introduces the **Key Custody Dilemma**:

1. **The Application Must Possess the Key:** In order to authenticate users, the backend must either decrypt stored passwords to compare them, or encrypt user input to compare ciphertexts (which fails if randomized initialization vectors / IVs are used). Therefore, the backend application holds the decryption key in memory.
2. **Single Point of Complete Failure:** The key must reside somewhere: in `application.properties`, environment variables, a HashiCorp Vault, or an AWS KMS key ring. If an attacker achieves Remote Code Execution (RCE), compromises an environment variable, dumps process memory, or accesses the KMS configuration, they acquire the master key.
3. **Total Database Recovery:** Once the master key is leaked, the attacker can decrypt **every single password in the entire database in a single automated batch**. The confidentiality of millions of users is destroyed in seconds.
4. **Insider Decryption:** Any system administrator or DBA with access to the key management service can decrypt user passwords at will, violating privacy regulations.

!!! caution "The Two-Way Encryption Anti-Pattern"
    Never store user credentials using two-way encryption. An authentication system does not need to know what the user's password is; it only needs to verify whether a presented password is correct. Reversible encryption creates a master key whose compromise completely breaks all stored user credentials.

### 2.3 Cryptographic Hashing: The One-Way Mathematical Trapdoor
**Cryptographic Hashing** is an irreversible, one-way mathematical function that maps an arbitrary-length input string to a fixed-size byte digest.

```mermaid
flowchart LR
    INPUT["Arbitrary Input Data<br/>(1 Character or 50 GB)"] --> HASH["One-Way Hash Function<br/>H(x)"]
    HASH --> OUTPUT["Fixed-Size Digest<br/>(e.g., 256 bits / 32 bytes)"]
    OUTPUT -.->|"COMPUTATIONALLY INFEASIBLE"| INPUT
```

#### Core Mathematical Properties of Cryptographic Hash Functions

1. **Determinism:** Passing the identical input $x$ to $H(x)$ will **always** produce the identical output digest $y$.
2. **Fixed Output Length:** The length of the output digest is constant regardless of input size (e.g., SHA-256 produces 256 bits / 64 hexadecimal characters whether hashing a single character or a 10 GB file).
3. **Pre-image Resistance (One-Way):** Given a digest $y$, it is computationally impossible to invert the function to calculate the input $x$ such that $H(x) = y$.
4. **Second Pre-image Resistance (Weak Collision Resistance):** Given an input $x_1$, it is computationally impossible to find a distinct input $x_2 \neq x_1$ such that $H(x_1) = H(x_2)$.
5. **Strong Collision Resistance:** It is computationally impossible to find any two arbitrary distinct inputs $x_1$ and $x_2$ such that $H(x_1) = H(x_2)$.
6. **The Avalanche Effect:** Changing even a single bit in the input produces a radically uncorrelated output digest (typically flipping ~50% of the output bits).

#### The Blender Analogy
Consider putting fresh fruits into a blender. The blender processes the fruit into juice. It is physically impossible to pour the juice back into the blender, reverse the motor, and reassemble the original whole fruits. Similarly, a hash function irreversibly crushes and mixes input bits.

#### Hands-On Demonstration: OpenSSL Cryptographic Hashing
```bash
# 1. Compute SHA-256 digest of password string
echo -n "EazyBytes@12345" | openssl dgst -sha256
# Output: (stdin)= 19f7f45c82deea31a0e8d05fc1f1f96e47e30d704ba4fc1f24d9c44566c5d9e5

# 2. Modify a single character (change '5' to '6')
echo -n "EazyBytes@12346" | openssl dgst -sha256
# Output: (stdin)= a72d02c6fe9bf18df790eb9361ad2da8ef2e25916027c95e1ee76742a12906e4
# The output is completely uncorrelated (Avalanche Effect).
```

### 2.4 Cryptographic Trilemma Comparison Matrix

| Architectural Dimension | Encoding (e.g. Base64) | Encryption (e.g. AES-256) | Cryptographic Hashing (e.g. BCrypt) |
| :--- | :--- | :--- | :--- |
| **Primary Purpose** | Safe data transmission across diverse network protocols. | Confidentiality of data that must be recovered by authorized parties. | Irreversible credential verification and integrity validation. |
| **Mathematical Nature** | Deterministic substitution / transposition table. | Mathematical permutation / substitution network using a key. | One-way compression function / non-linear mathematical trapdoor. |
| **Reversibility** | **Trivial:** Completely reversible by anyone without keys. | **Reversible:** Decryptable using the matching cryptographic key. | **Irreversible:** Mathematically impossible to reverse. |
| **Key Requirement** | None. Algorithm is public and open. | Mandatory (Symmetric secret key or Asymmetric private key). | None required for raw hashing; salts are generated dynamically. |
| **Output Size** | Expands input size by $\approx 33\%$ (4 bytes per 3 bytes). | Variable; roughly equivalent to plaintext padded to block size. | Strictly fixed length (e.g., 256 bits for SHA-256). |
| **Password Storage Suitability** | **PROHIBITED:** Zero security. | **DANGEROUS:** Central key store creates single point of failure. | **MANDATORY:** The only approved cryptographic approach. |

---

## 3. The Dual Fatal Flaws of Naive Cryptographic Hashing

Having established that hashing is the correct primitive, why is standard, standalone hashing (such as raw `SHA-256` or `MD5`) completely insufficient for production password storage?

### Vulnerability 1: Determinism & Precomputed Rainbow Tables
Because hash functions are deterministic, the same plaintext password always yields the exact same hash:
$$\text{SHA-256}("12345") = \text{5994471abb01112afcc18159f6cc74b4f511b99806da59b3caf5a9c173cacfc5}$$

If 10,000 users across an organization select `Password123`, all 10,000 database rows contain identical hash strings. An attacker observing the database instantly deduces that all these accounts share a password.

#### The Rainbow Table Attack
A **Rainbow Table** is a precomputed lookup database that trades precomputation time for instantaneous cracking speed. 

```mermaid
flowchart LR
    A["Plaintext Password<br/>'Spring2026'"] -->|"Hash Function H()"| B["Hash Digest<br/>'d7a8fbb...'"]
    B -->|"Reduction Function R()"| C["New Candidate Plaintext<br/>'K9#mQ2'"]
    C -->|"Hash Function H()"| D["Hash Digest<br/>'3c2e11a...'"]
    D -->|"Repeated N Times"| END_HASH["Final Digest"]
```

Rather than storing billions of pairs of (password $\to$ hash), rainbow tables generate long hash chains using alternating hash functions and mathematical reduction functions ($R$). By storing only the initial password and the final digest of each chain:

- Attackers compress millions of password-hash combinations into modest storage files.
- When an attacker steals a database dump containing unsalted `SHA-256` or `MD5` hashes, they query the rainbow table.
- A lookup that would take years of brute-force computation resolves in **milliseconds**, revealing the plaintext passwords instantly.

### Vulnerability 2: Extreme Computational Velocity & Hardware Acceleration
Standard hash algorithms (MD5, SHA-1, SHA-256, SHA-512) were engineered for high-throughput checksumming of files, network streams, and TLS records. They are designed to be **as fast as possible**.

What is a virtue for data transmission is a fatal vulnerability for credential storage:

- A modern consumer GPU (e.g., NVIDIA RTX 4090) can compute over **20 billion SHA-256 hashes per second**.
- An 8-character password consisting of lowercase letters and numbers has $36^8 \approx 2.8 \times 10^{12}$ combinations.
- An attacker with a dual-GPU cracking rig running Hashcat can exhaust the entire keyspace and crack the password in **under 2.5 minutes**.

```text
Algorithm Crack Speed Comparison (Modern Consumer GPU - NVIDIA RTX 4090):
-------------------------------------------------------------------------
SHA-256:              ~22,000,000,000 hashes / second  (Instantaneous cracking)
MD5:                  ~65,000,000,000 hashes / second  (Instantaneous cracking)
BCrypt (Cost 12):             ~12,000 hashes / second  (Millions of times slower!)
Argon2id (64MB RAM):             ~250 hashes / second  (Completely paralyzes GPUs!)
```

---

## 4. Cryptographic Countermeasures: Salt & Adaptive Work Factors

To neutralize both rainbow table lookups and hardware-accelerated brute force, modern password security incorporates two non-negotiable architectural countermeasures: **Cryptographic Salt** and **Adaptive Work Factors (Key Stretching)**.

```mermaid
flowchart TD
    subgraph Countermeasure1["Countermeasure 1: Cryptographic Salt"]
        direction TB
        CS1["128-bit CSPRNG Salt<br/>Generated per account"] --> CS2["Concatenated with Raw Password<br/>H(Salt || Password)"]
        CS2 --> CS3["Unique Hash even for Identical Passwords"]
        CS3 --> CS4["Precomputed Rainbow Tables Rendered USELESS"]
    end

    subgraph Countermeasure2["Countermeasure 2: Adaptive Work Factor"]
        direction TB
        WF1["Configurable Cost Parameter (e.g., 2^12 Rounds)"] --> WF2["CPU & Memory Hardness Enforced"]
        WF2 --> WF3["Hashes Take ~100-250ms on CPU"]
        WF3 --> WF4["Parallel GPU/ASIC Exhaustion Defeated"]
    end

    Countermeasure1 --> SECURE_HASH["Production-Ready Password Hash"]
    Countermeasure2 --> SECURE_HASH
```

### 4.1 Defense 1: Cryptographically Secure Salt
A **Salt** is a cryptographically strong, pseudo-random sequence of bytes generated uniquely for every user account during registration or password updates.

$$\text{Digest} = H(\text{Salt} \parallel \text{Password})$$

#### Why Salts Completely Defeat Rainbow Tables
A rainbow table is precomputed against a specific, fixed hashing function. If an application appends a unique 128-bit random salt to the password:

1. Two users with identical passwords (`Password123`) receive different salts ($S_1 \neq S_2$).
2. Therefore, $H(S_1 \parallel \text{Password123}) \neq H(S_2 \parallel \text{Password123})$. Their stored hashes in the database are entirely different.
3. An attacker cannot use a global precomputed rainbow table. To attack salted hashes with rainbow tables, the attacker would have to compute a dedicated rainbow table **for every single user's unique salt**. With a 128-bit salt ($2^{128} \approx 3.4 \times 10^{38}$ variations), storing or precomputing tables is physically and mathematically impossible.

#### The Salt Secrecy Fallacy: Why Salts are Public
A common architectural misconception is that salts must be stored in secret vaults. **Salts are not secrets.**

- The salt is stored directly alongside the hash (or embedded within the hash string itself).
- Kerckhoffs's Principle applies: The security of the system depends solely on the secrecy of the password, not the secrecy of the salt.
- The purpose of a salt is **guaranteed uniqueness**, not confidentiality.

!!! warning "Why Salt Alone Is Not Enough"
    Salt neutralizes rainbow tables and dictionary lookups across multiple users. However, salt **does not stop targeted offline brute-force attacks**. If an attacker steals a database dump containing a salted SHA-256 hash and its salt, the attacker can still feed the known salt and candidate passwords into a GPU computing 20 billion hashes per second. Salt must be paired with an adaptive work factor!

### 4.2 Defense 2: Adaptive Work Factors (Key Stretching & Memory Hardness)
To prevent high-speed hardware brute force, password hashing algorithms must be intentionally **slow**.

- An authentication delay of $100\,\text{ms}$ is imperceptible to a human logging into a web portal.
- But to an attacker testing billions of passwords, $100\,\text{ms}$ per candidate reduces testing throughput from 20 billion guesses per second to **10 guesses per second per CPU core**. Exhausting an 8-character keyspace would require decades or centuries.

#### Types of Hardness

1. **CPU Hardness (Time Complexity):** The algorithm executes iterative loops or key expansion rounds (e.g., $2^{\text{cost}}$ in BCrypt, 310,000 iterations in PBKDF2).
2. **Memory Hardness (Space Complexity):** Modern algorithms like **Argon2** and **SCrypt** allocate large arrays in RAM (e.g., 64 MB per hash) and perform pseudo-random memory lookups. GPUs have thousands of compute cores but very limited local cache memory per core. Forcing the hash to consume 64 MB of RAM completely chokes GPU parallelism, rendering GPU and ASIC cracking rigs ineffective.

---

## 5. The Spring Security `PasswordEncoder` Interface & Lifecycle Contract

Spring Security formalizes cryptographic password management through the `org.springframework.security.crypto.password.PasswordEncoder` interface.

```java
package org.springframework.security.crypto.password;

public interface PasswordEncoder {

    /**
     * Encode the raw password. Generally, a good encoding algorithm applies a SHA-1
     * or greater hash combined with an 8-byte or greater randomly generated salt.
     */
    String encode(CharSequence rawPassword);

    /**
     * Verify the encoded password obtained from storage matches the submitted raw
     * password after it too is encoded. Returns true if the passwords match, false if
     * they do not. The stored password itself is never decoded.
     */
    boolean matches(CharSequence rawPassword, String encodedPassword);

    /**
     * Returns true if the encoded password should be encoded again for better security,
     * otherwise false. The default implementation always returns false.
     */
    default boolean upgradeEncoding(String encodedPassword) {
        return false;
    }
}
```

### 5.1 Deep Dive into the Interface Methods

#### 1. `encode(CharSequence rawPassword)`

- **Input:** Takes a `CharSequence` representing the raw password. Using `CharSequence` rather than `String` allows passing mutable character buffers (`char[]`) that can be explicitly cleared from memory after hashing to minimize heap exposure.
- **Action:** Generates a secure random salt, applies key expansion and adaptive hashing rounds, and formats the result into a serialized string containing the algorithm identifier, cost parameters, salt, and resulting digest.
- **Output:** A self-contained encoded string safe for database persistence.

#### 2. `matches(CharSequence rawPassword, String encodedPassword)`

- **Action:** Never attempts to decrypt the `encodedPassword` (which is impossible). Instead:
  1. Extracts the salt and configuration parameters from `encodedPassword`.
  2. Runs the incoming `rawPassword` through the identical hashing algorithm using the extracted salt and parameters.
  3. Compares the newly computed digest with the stored digest.
- **Critical Security Implementation (Constant-Time Comparison):**
  To prevent **side-channel timing attacks**, `matches()` must never use standard `String.equals()`. Standard string comparison terminates immediately upon encountering the first non-matching character (early exit), allowing an attacker measuring microsecond response times over millions of requests to determine how many leading characters of a hash match.
  Spring Security uses `MessageDigest.isEqual()`:
  ```java
  public static boolean isEqual(byte[] digesta, byte[] digestb) {
      if (digesta == digestb) return true;
      if (digesta == null || digestb == null) return false;
      if (digesta.length != digestb.length) return false;

      int result = 0;
      for (int i = 0; i < digesta.length; i++) {
          result |= digesta[i] ^ digestb[i]; // Bitwise OR over entire length
      }
      return result == 0;
  }
  ```
  Execution time is invariant with respect to whether characters match, eliminating timing leaks.

#### 3. `upgradeEncoding(String encodedPassword)`

- **Purpose:** Enables seamless, zero-downtime security upgrades.
- If an enterprise raises its BCrypt work factor from `10` to `12`, or migrates from legacy SHA-256 to Argon2id, `upgradeEncoding()` inspects the stored hash. If the hash was created with obsolete parameters, it returns `true`.
- The authentication provider can then re-encode the plaintext password (which is available in memory during a successful login) and persist the updated hash without prompting the user to reset their password!

### 5.2 Who Invokes `matches()` in the Framework?
In the standard Spring Security pipeline, `matches()` is invoked by `DaoAuthenticationProvider`:

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant Filter as Security Filter (Basic / Form)
    participant AuthManager as ProviderManager
    participant DaoProvider as DaoAuthenticationProvider
    participant UserDetailsSvc as UserDetailsService
    participant Encoder as PasswordEncoder

    Client->>Filter: HTTP Request (Authorization Header)
    Filter->>AuthManager: authenticate(UsernamePasswordAuthenticationToken)
    AuthManager->>DaoProvider: authenticate(auth)
    DaoProvider->>UserDetailsSvc: loadUserByUsername(username)
    UserDetailsSvc-->>DaoProvider: UserDetails (Contains Hashed Password from DB)
    DaoProvider->>DaoProvider: additionalAuthenticationChecks(userDetails, auth)
    DaoProvider->>Encoder: matches(rawPassword, userDetails.getPassword())
    Encoder-->>DaoProvider: true / false
    alt Credentials Valid
        DaoProvider-->>AuthManager: Fully Authenticated Token
        AuthManager-->>Filter: Authentication Object
        Filter->>Filter: Store in SecurityContextHolder
    else Password Mismatch
        DaoProvider-->>AuthManager: BadCredentialsException("Bad credentials")
        AuthManager-->>Filter: AuthenticationException
    end
```

---

## 6. Comprehensive Taxonomy & Comparison of Spring Security PasswordEncoders

Spring Security provides multiple concrete implementations of the `PasswordEncoder` contract. A senior architect must understand the internal mechanics, algorithmic strengths, and tradeoffs of each.

```mermaid
classDiagram
    class PasswordEncoder {
        <<interface>>
        +encode(CharSequence) String
        +matches(CharSequence, String) boolean
        +upgradeEncoding(String) boolean
    }

    class BCryptPasswordEncoder {
        -int strength
        -BCryptVersion version
        -SecureRandom random
        +encode(CharSequence) String
        +matches(CharSequence, String) boolean
        +upgradeEncoding(String) boolean
    }

    class Argon2PasswordEncoder {
        -int saltLength
        -int hashLength
        -int parallelism
        -int memory
        -int iterations
        +encode(CharSequence) String
        +matches(CharSequence, String) boolean
    }

    class Pbkdf2PasswordEncoder {
        -SecretKeyFactory secretKeyFactory
        -int iterations
        -int hashWidth
        +encode(CharSequence) String
        +matches(CharSequence, String) boolean
    }

    class SCryptPasswordEncoder {
        -int cpuCost
        -int memoryCost
        -int parallelization
        -int keyLength
        -int saltLength
        +encode(CharSequence) String
        +matches(CharSequence, String) boolean
    }

    class DelegatingPasswordEncoder {
        -String idForEncode
        -PasswordEncoder defaultPasswordEncoderForMatches
        -Map~String, PasswordEncoder~ idToPasswordEncoder
        +encode(CharSequence) String
        +matches(CharSequence, String) boolean
    }

    class NoOpPasswordEncoder {
        +encode(CharSequence) String
        +matches(CharSequence, String) boolean
    }

    PasswordEncoder <|.. BCryptPasswordEncoder
    PasswordEncoder <|.. Argon2PasswordEncoder
    PasswordEncoder <|.. Pbkdf2PasswordEncoder
    PasswordEncoder <|.. SCryptPasswordEncoder
    PasswordEncoder <|.. DelegatingPasswordEncoder
    PasswordEncoder <|.. NoOpPasswordEncoder
```

### 6.1 `BCryptPasswordEncoder` (The Enterprise Default)
Developed by Niels Provos and David Mazières in 1999, BCrypt is based on the **Blowfish** symmetric block cipher's key expansion routine ("Eksblowfish" — Expensive Key Schedule Blowfish).

#### Internal Structure of a BCrypt Hash String
When `BCryptPasswordEncoder` outputs a hash, it serializes all cryptographic metadata into a standard 60-character Modular Crypt Format string:

```text
$2a$12$88.f6upbBvy0okEa7OfHFuorV29qeK.sVbB9VQ6J6dWM1bW6Qef8m
\__/\__/\____________________/\_____________________________/
  |   |            |                          |
 (1) (2)          (3)                        (4)
```

1. **`$2a$` — Specification Identifier:** Defines the BCrypt algorithm revision.
   - `$2$` — Original specification (obsolete).
   - `$2a$` — Fixes handling of null characters and 8-bit character sets.
   - `$2b$` — Modern standard in OpenBSD and Spring Security.
   - `$2y$` — Specific to PHP crypt library workarounds.
2. **`$12$` — Cost / Logarithmic Work Factor:** Defines the number of iteration rounds as a power of 2 ($2^{12} = 4,096$ key expansion iterations).
   - Allowed range: $4$ to $31$. Default in Spring Security: $10$ ($2^{10} = 1,024$ rounds).
   - Enterprise banking recommendation: $12$ ($2^{12} = 4,096$ rounds, $\approx 150-250\,\text{ms}$ execution time).
3. **`88.f6upbBvy0okEa7OfHFuo` — Cryptographic Salt (22 Characters):**
   - Radix-64 encoded representation of a 128-bit (16-byte) cryptographically secure random salt.
4. **`rV29qeK.sVbB9VQ6J6dWM1bW6Qef8m` — Ciphertext Checksum (31 Characters):**
   - Radix-64 encoded representation of the final 184-bit hash digest computed by encrypting the 24-character magic text `"OrpheanBeholderScryDoubt"` through Blowfish 64 times.

#### The 72-Byte Password Limit Trap
!!! important "Critical BCrypt Architectural Constraint"
    The Blowfish cipher upon which BCrypt is built has an internal key schedule limit of **72 bytes**. If a user submits a password longer than 72 bytes, `BCryptPasswordEncoder` silently truncates all characters beyond the 72nd byte! A password of 72 characters and an identical password with 50 additional characters appended will generate the exact same BCrypt hash. If your system accepts arbitrarily long passwords (e.g., passphrases), pre-hash passwords with SHA-256 before passing them to BCrypt, or adopt **Argon2id**.

### 6.2 `Argon2PasswordEncoder` (The Gold Standard)
**Argon2** is the winner of the international Password Hashing Competition (PHC, 2015). It is currently recognized by OWASP and cryptography researchers as the most secure password hashing algorithm available.

#### Why Argon2 Surpasses BCrypt: Memory Hardness
BCrypt is purely CPU-intensive; it requires virtually no RAM. Attackers can build custom ASIC chips containing thousands of parallel Blowfish execution pipelines on a single circuit board.
Argon2 is explicitly **memory-hard**. It requires significant amounts of RAM to compute each hash, preventing hardware parallelization.

#### Argon2 Profiles

- **Argon2d:** Maximizes resistance against GPU cracking by using data-dependent memory access. Vulnerable to side-channel cache-timing attacks. Suitable for cryptocurrency proof-of-work, not general web authentication.
- **Argon2i:** Uses data-independent memory access, completely eliminating side-channel timing attacks. Preferred for password hashing in shared cloud environments.
- **Argon2id:** A hybrid combining Argon2i on the first pass and Argon2d on subsequent passes. **This is the recommended standard adopted by Spring Security.**

#### Four Tuning Parameters
```java
Argon2PasswordEncoder argon2 = new Argon2PasswordEncoder(
    16,     // Salt length in bytes (128 bits)
    32,     // Hash length in bytes (256 bits)
    1,      // Parallelism: Number of concurrent threads
    65536,  // Memory cost: 65,536 KB (64 MB of RAM per hash)
    3       // Iterations / Time cost: 3 passes over memory
);
```

### 6.3 `Pbkdf2PasswordEncoder` (Federal / FIPS Compliance)
**PBKDF2** (Password-Based Key Derivation Function 2) is standardized in RFC 2898 and NIST SP 800-132.

- **Mechanism:** Applies a pseudorandom function (HMAC-SHA256 or HMAC-SHA512) repeatedly over thousands of iterations combined with a salt.
- **Enterprise Relevance:** While less resistant to GPU attacks than Argon2 or Scrypt (due to zero memory hardness), PBKDF2 is **FIPS-140-2 validated**. Financial institutions, military applications, and government contractors governed by federal compliance standards that prohibit non-NIST algorithms must deploy `Pbkdf2PasswordEncoder`.
- **Iterations:** Modern NIST guidelines mandate a minimum of **600,000 iterations** for HMAC-SHA256. Spring Security defaults to 310,000 iterations.

### 6.4 `SCryptPasswordEncoder`
Developed by Colin Percival in 2009 for the Tarsnap backup system:

- Introduced the concept of memory hardness to password derivation before Argon2.
- Parameters: CPU/Memory cost parameter $N$ (must be a power of 2, e.g., $2^{16} = 65,536$), block size $r$ (e.g., 8), parallelization $p$ (e.g., 1).
- Highly effective against custom ASIC chips, but superseded by Argon2 due to simpler, more predictable parameter calibration in Argon2id.

### 6.5 Deprecated Legacy Encoders (Security Hazards)

- **`NoOpPasswordEncoder`:** Performs raw plaintext equality (`rawPassword.toString().equals(encodedPassword)`). Marked `@Deprecated(since = "5.0")`. **Never deploy in production under any circumstance.**
- **`StandardPasswordEncoder`:** Uses SHA-256 with 1,024 iterations and an 8-byte salt. Deprecated because 1,024 iterations is trivial for modern hardware to crack.
- **`Md4PasswordEncoder` / `MessageDigestPasswordEncoder`:** Legacy algorithms with proven mathematical collision vulnerabilities.

---

### 6.6 Algorithmic Comparison Matrix

| Encoder Implementation | Underlying Primitive | Memory Hardness | Time / Work Factor | NIST / FIPS Approved | GPU / ASIC Resistance | Max Password Length | Architectural Recommendation |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`BCryptPasswordEncoder`** | Eksblowfish (Blowfish cipher) | Low ($\approx 4\,\text{KB}$) | Configurable ($2^4$ to $2^{31}$) | No (Not FIPS) | Moderate | **72 bytes** | **Default Enterprise Standard.** Highly battle-tested, supported natively on all platforms. |
| **`Argon2PasswordEncoder`** | Blake2b permutation network | **High** (Configurable, e.g. 64 MB) | Iterations + Parallelism + Memory | Emerging (RFC 9106) | **Maximum** | Unlimited | **Highest Security Tier.** Recommended for new greenfield architectures. |
| **`Pbkdf2PasswordEncoder`** | HMAC-SHA256 / HMAC-SHA512 | None ($0\,\text{MB}$) | Iterations ($\ge 310,000$) | **Yes (FIPS-140-2)** | Low to Moderate | Unlimited | **Mandatory for FIPS / Defense / Regulatory Compliance.** |
| **`SCryptPasswordEncoder`** | PBKDF2 + Salsa20/8 core | High (Configurable) | $N, r, p$ parameters | No | High | Unlimited | Strong, but Argon2id is technically superior. |
| **`StandardPasswordEncoder`** | SHA-256 (1024 iterations) | None | Fixed (1024) | No | Extremely Vulnerable | Unlimited | **DEPRECATED.** Do not use. |
| **`NoOpPasswordEncoder`** | Plaintext String Equals | None | None ($0\,\text{ms}$) | No | **Zero Security** | Unlimited | **STRICTLY PROHIBITED.** Only for non-production tests. |

---

## 7. `DelegatingPasswordEncoder` & Zero-Downtime Migration Architecture

In legacy Spring Security versions (pre-5.0), developers registered a specific encoder bean directly:

```java
// ANTI-PATTERN: Hardcoding a concrete PasswordEncoder bean
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

### The Architectural Failure of Hardcoded Encoders

1. **Database Lock-in:** Storing raw BCrypt hashes (`$2a$12$...`) assumes the application will run BCrypt forever.
2. **Zero-Downtime Migration Impossibility:** If quantum computing or cryptographic breakthroughs compromise BCrypt in 5 years, how do you migrate to Argon2?
   - You cannot simply change the bean to `Argon2PasswordEncoder`. The moment you do, every existing user attempting to log in will have their BCrypt hash evaluated by the Argon2 validator, causing all logins to fail!
   - You would be forced to trigger a mandatory password reset for 100% of your user base.

### The Solution: Prefix Routing via `DelegatingPasswordEncoder`
Spring Security solves this via `DelegatingPasswordEncoder`, introduced in Spring Security 5.0. It prepends an **algorithm identifier prefix** enclosed in braces (`{id}`) to the encoded password string:

```text
{bcrypt}$2a$12$88.f6upbBvy0okEa7OfHFuorV29qeK.sVbB9VQ6J6dWM1bW6Qef8m
{argon2}$argon2id$v=19$m=65536,t=3,p=1$eG9y...
{pbkdf2}c1926b6fa3...
{noop}EazyBytes@12345
```

```mermaid
flowchart TD
    DB_HASH["Stored Password in DB:<br/>'{argon2}$argon2id$v=19$m=65536...'"] --> EXTRACT["Extract Prefix Identifier: 'argon2'"]
    
    EXTRACT --> ROUTER{"DelegatingPasswordEncoder Router"}
    
    ROUTER -->|"{bcrypt}"| BC["BCryptPasswordEncoder.matches()"]
    ROUTER -->|"{argon2}"| AR["Argon2PasswordEncoder.matches()"]
    ROUTER -->|"{pbkdf2}"| PB["Pbkdf2PasswordEncoder.matches()"]
    ROUTER -->|"{scrypt}"| SC["SCryptPasswordEncoder.matches()"]
    ROUTER -->|"{noop}"| NO["NoOpPasswordEncoder.matches()"]
    ROUTER -->|"No Prefix"| DEF["defaultPasswordEncoderForMatches"]
```

### 7.1 Standard Enterprise Configuration
```java
package com.eazybytes.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class ProjectSecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        // Creates a DelegatingPasswordEncoder with bcrypt as default
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}
```

Behind the scenes, `PasswordEncoderFactories.createDelegatingPasswordEncoder()` executes:
```java
public static PasswordEncoder createDelegatingPasswordEncoder() {
    String encodingId = "bcrypt";
    Map<String, PasswordEncoder> encoders = new HashMap<>();
    encoders.put(encodingId, new BCryptPasswordEncoder());
    encoders.put("ldap", new org.springframework.security.crypto.password.LdapShaPasswordEncoder());
    encoders.put("MD4", new org.springframework.security.crypto.password.Md4PasswordEncoder());
    encoders.put("MD5", new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("MD5"));
    encoders.put("noop", org.springframework.security.crypto.password.NoOpPasswordEncoder.getInstance());
    encoders.put("pbkdf2", Pbkdf2PasswordEncoder.defaultsForSpringSecurity_v5_8());
    encoders.put("scrypt", SCryptPasswordEncoder.defaultsForSpringSecurity_v5_8());
    encoders.put("argon2", Argon2PasswordEncoder.defaultsForSpringSecurity_v5_8());

    return new DelegatingPasswordEncoder(encodingId, encoders);
}
```

- When `encode(rawPassword)` is called: It invokes the encoder registered under `encodingId` (`bcrypt`), appends `{bcrypt}` to the front, and returns the combined string.
- When `matches(rawPassword, storedPassword)` is called: It extracts the prefix between `{` and `}`, fetches the corresponding encoder from the map, and delegates execution. If the database record contains `{noop}secret`, it uses `NoOpPasswordEncoder`; if it contains `{bcrypt}$2a$...`, it uses `BCryptPasswordEncoder`.

### 7.2 Zero-Downtime Multi-Generation Migration Architecture
How does an enterprise migrate an existing database of 5 million legacy `{noop}` or `{sha256}` hashes to `{bcrypt}` without downtime or forced password resets?

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant App as Custom Authentication Filter / Provider
    participant DelEnc as DelegatingPasswordEncoder
    participant DB as User Database

    User->>App: Login with "myPassword123"
    App->>DB: Fetch user by email
    DB-->>App: UserDetails (Stored Hash: "{noop}myPassword123")
    App->>DelEnc: matches("myPassword123", "{noop}myPassword123")
    Note over DelEnc: Delegates to NoOpPasswordEncoder -> Returns TRUE
    DelEnc-->>App: Credentials Valid (true)
    
    rect rgb(230, 245, 230)
    Note over App: Check if password encoding needs upgrade!
    App->>DelEnc: upgradeEncoding("{noop}myPassword123")
    DelEnc-->>App: Returns TRUE (Prefix is not modern 'bcrypt')
    App->>DelEnc: encode("myPassword123")
    DelEnc-->>App: New Hash: "{bcrypt}$2a$12$e8i..."
    App->>DB: UPDATE users SET pwd = '{bcrypt}$2a$12$e8i...' WHERE id = ?
    end

    App-->>User: Authentication Successful (Session / Token Issued)
```

By leveraging `upgradeEncoding()`, users are transparently and lazily upgraded to modern hashing algorithms on their next login event without any operational interruption.

---

## 8. Spring Security 6.3+ Breach Defense: `CompromisedPasswordChecker` & $k$-Anonymity

Even if an application uses `BCrypt` with cost 14 or `Argon2id`, cryptographic strength is meaningless if a user chooses a password that has already been published in public data breach dumps (e.g., `Password123`, `P@ssword2024`, `admin123`).
Attackers run automated **Credential Stuffing** bots that replay billions of username/password pairs harvested from past breaches against login endpoints.

### 8.1 NIST SP 800-63B Compliance Requirement
NIST Special Publication 800-63B (Section 5.1.1.2) mandates:
> *"When processing requests to establish and change credentials, verifiers SHALL compare the prospective secrets against a list that contains values known to be commonly-used, expected, or compromised."*

Starting in **Spring Security 6.3**, the framework introduces first-class compromised password screening via the `CompromisedPasswordChecker` abstraction.

### 8.2 The Privacy Paradox: The $k$-Anonymity Protocol
How can an enterprise check if a customer's proposed password exists in an external breach repository containing billions of compromised passwords (such as Troy Hunt's HaveIBeenPwned API) without sending the customer's plaintext password over the Internet to an external service?

Sending the password (even over HTTPS) violates privacy laws and creates a massive surveillance vulnerability.
The solution is the mathematical privacy model known as **$k$-Anonymity**:

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Backend as Spring Security Backend
    participant HIBP as HaveIBeenPwned Range API

    User->>Backend: Register: pwd = "Password123!"
    Note over Backend: 1. Compute SHA-1 of password:<br/>SHA-1("Password123!") = 5BAA61E4C9B93F3F0682250B6CF8331B7EE68FD8
    Note over Backend: 2. Split Hash into Prefix (5 chars) & Suffix (35 chars):<br/>Prefix: 5BAA6 | Suffix: 1E4C9B93F3F0682250B6CF8331B7EE68FD8
    
    Backend->>HIBP: GET https://api.pwnedpasswords.com/range/5BAA6
    Note over HIBP: External service receives ONLY the 5-char prefix.<br/>It cannot determine the remaining 35 chars or plaintext.
    HIBP-->>Backend: Returns ~500-1000 matching hash suffixes with breach counts:<br/>0018A45C4D17B5CE514F72016012F983621:2<br/>1E4C9B93F3F0682250B6CF8331B7EE68FD8:84321<br/>...
    
    Note over Backend: 3. Local In-Memory Match:<br/>Compare local suffix against API list.<br/>Match found! Breach count = 84,321 times.
    Backend-->>User: 400 Bad Request: "Password has been compromised in public breaches!"
```

Because the external API only ever receives the first 5 hexadecimal characters ($16^5 = 1,048,576$ possible buckets), it returns a list of several hundred matching suffixes. The external service has mathematically zero knowledge of which suffix belongs to the user, preserving absolute end-user privacy.

### 8.3 Configuring `CompromisedPasswordChecker` in Spring Security 6.3+

```java
package com.eazybytes.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.password.CompromisedPasswordChecker;
import org.springframework.security.web.authentication.password.HaveIBeenPwnedRestApiPasswordChecker;

@Configuration
public class ProjectSecurityConfig {

    @Bean
    public CompromisedPasswordChecker compromisedPasswordChecker() {
        // Enforces k-Anonymity checks against the HaveIBeenPwned API
        return new HaveIBeenPwnedRestApiPasswordChecker();
    }
}
```

### 8.4 Integrating Password Screening into Registration Controller

```java
package com.eazybytes.controller;

import com.eazybytes.model.Customer;
import com.eazybytes.repository.CustomerRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.password.CompromisedPasswordChecker;
import org.springframework.security.authentication.password.CompromisedPasswordDecision;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
@RequiredArgsConstructor
public class UserController {

    private final CustomerRepository customerRepository;
    private final PasswordEncoder passwordEncoder;
    private final CompromisedPasswordChecker compromisedPasswordChecker;

    @PostMapping("/register")
    public ResponseEntity<String> registerUser(@RequestBody Customer customer) {
        try {
            // Step 1: Pre-Screen Password against Public Breach Corpuses via k-Anonymity
            CompromisedPasswordDecision decision = 
                    compromisedPasswordChecker.check(customer.getPwd());

            if (decision.isCompromised()) {
                log.warn("Registration rejected for user {}: Submitted password is known compromised.", 
                        customer.getEmail());
                return ResponseEntity
                        .status(HttpStatus.BAD_REQUEST)
                        .body("The submitted password has appeared in public data breaches. " +
                              "In accordance with security policies, please choose a unique, stronger password.");
            }

            // Step 2: Hash the Verified Password using DelegatingPasswordEncoder (BCrypt default)
            String hashPwd = passwordEncoder.encode(customer.getPwd());
            customer.setPwd(hashPwd);

            // Step 3: Persist Entity
            Customer savedCustomer = customerRepository.save(customer);

            if (savedCustomer.getId() > 0) {
                return ResponseEntity
                        .status(HttpStatus.CREATED)
                        .body("Given user details are successfully registered");
            } else {
                return ResponseEntity
                        .status(HttpStatus.BAD_REQUEST)
                        .body("User registration failed");
            }

        } catch (Exception ex) {
            log.error("Exception during user registration: ", ex);
            return ResponseEntity
                    .status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("An exception occurred: " + ex.getMessage());
        }
    }
}
```

---

## 9. End-to-End Runtime Execution Trace & Debugger Walkthrough

To verify how cryptographic password management operates inside the JVM, let us trace the lifecycle of user registration and subsequent login through a step-by-step debugger walkthrough (corresponding to Lecture 42).

### 9.1 The Registration Phase (`encode` Lifecycle)

1. **HTTP Post Dispatch:** Postman submits `POST /register` with payload:
   ```json
   {
     "email": "smith@example.com",
     "pwd": "EazyBytes@12345",
     "role": "admin"
   }
   ```
2. **Controller Invocation:** `UserController.registerUser()` invokes `passwordEncoder.encode("EazyBytes@12345")`.
3. **Delegating Router:** `DelegatingPasswordEncoder` determines the default encoding ID is `"bcrypt"`.
4. **BCrypt Salt Generation:** `BCryptPasswordEncoder` calls `BCrypt.gensalt(10)`.
   - Invokes `java.security.SecureRandom.nextBytes(byte[] bytes)` to generate 16 cryptographically random bytes.
   - Encodes the 16 bytes into a 22-character Radix-64 salt: `88.f6upbBvy0okEa7OfHFuo`.
5. **Key Expansion & Hashing:** `BCrypt.crypt_raw()` executes $2^{10} = 1,024$ rounds of Blowfish key expansion using the password and salt.
6. **Persistence:** The resulting string `{bcrypt}$2a$10$88.f6upbBvy0okEa7OfHFuorV29qeK.sVbB9VQ6J6dWM1bW6Qef8m` is saved into the `customer.pwd` column.

### 9.2 The Authentication Phase (`matches` Lifecycle)
When the user subsequently authenticates via HTTP Basic (`Authorization: Basic c21pdGhAZXhhbXBsZS5jb206RWF6eUJ5dGVzQDEyMzQ1`):

```mermaid
flowchart TD
    REQ["HTTP GET /myAccount with Basic Auth Header"] --> FILTER["BasicAuthenticationFilter"]
    FILTER --> EXTRACT["Extract: 'smith@example.com' : 'EazyBytes@12345'"]
    EXTRACT --> PROV_MGR["ProviderManager.authenticate()"]
    PROV_MGR --> DAO["DaoAuthenticationProvider.authenticate()"]
    
    DAO --> LOAD["EazyBankUserDetailsService.loadUserByUsername('smith@example.com')"]
    LOAD --> RETRIEVE["Fetch Customer Entity from DB"]
    RETRIEVE --> DETAILS["Construct UserDetails with Stored Hash"]
    DETAILS --> CHECKS["additionalAuthenticationChecks()"]
    
    CHECKS --> MATCHES["passwordEncoder.matches(rawPassword, storedHash)"]
    
    MATCHES --> DEL_MATCHES["DelegatingPasswordEncoder.matches()"]
    DEL_MATCHES --> EXTRACT_ID["Extract Prefix: 'bcrypt'"]
    EXTRACT_ID --> BCRYPT_MATCHES["BCryptPasswordEncoder.matches()"]
    
    BCRYPT_MATCHES --> BCRYPT_CHECK["BCrypt.checkpw('EazyBytes@12345', storedHash)"]
    BCRYPT_CHECK --> PARSE_SALT["Extract Salt: '88.f6upbBvy0okEa7OfHFuo' & Cost: '10'"]
    PARSE_SALT --> COMPUTE_HASH["Compute Blowfish Hash over Input with Extracted Salt"]
    COMPUTE_HASH --> CONSTANT_CMP["MessageDigest.isEqual(computedHash, storedHash)"]
    
    CONSTANT_CMP -->|Match == true| AUTH_OK["Authentication SUCCESS -> Set SecurityContextHolder"]
    CONSTANT_CMP -->|Match == false| AUTH_FAIL["Throw BadCredentialsException -> 401 Unauthorized"]
```

#### Step-by-Step Debugger Inspection inside `BCrypt.checkpw()`
Setting a breakpoint inside `BCrypt.java`:

1. `rawPassword` is evaluated: `"EazyBytes@12345"`.
2. `encodedPassword` is evaluated: `"$2a$10$88.f6upbBvy0okEa7OfHFuorV29qeK.sVbB9VQ6J6dWM1bW6Qef8m"`.
3. Validation verifies the string starts with `$2a$` and cost is between $4$ and $31$.
4. The exact 22-character salt is isolated.
5. The incoming password `"EazyBytes@12345"` is converted to UTF-8 bytes and hashed using that exact extracted salt and cost factor `10`.
6. The resulting string is compared in constant time with `encodedPassword`.
7. Because both match, `checkpw()` returns `true`. `DaoAuthenticationProvider` creates a fully-populated `UsernamePasswordAuthenticationToken` marked `authenticated = true`, and returns it to `FilterChainProxy`.

---

## 10. Production Hardening Checklist for Security Architects

### 10.1 Calibrating Work Factors for Production Environments
Selecting a work factor requires balancing security against application throughput and latency budgets:

```text
Target Authentication Compute Latency:  100 ms to 250 ms per login attempt.
```

- **If too low (< 20 ms):** Offline cracking resistance is severely degraded; attackers can brute-force stolen hashes on consumer GPUs.
- **If too high (> 500 ms):** Legitimate users experience slow logins, and malicious actors can flood the login endpoint with requests to induce **CPU Denial of Service (DoS)** on server worker threads.

#### Benchmarking Harness for Work Factor Calibration
Run this test on your production hardware container image to determine the appropriate cost:

```java
public class PasswordEncoderBenchmark {
    public static void main(String[] args) {
        String testPassword = "EnterpriseBankingSecurePassword2026!";
        
        for (int cost = 10; cost <= 15; cost++) {
            BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(cost);
            long start = System.nanoTime();
            encoder.encode(testPassword);
            long elapsed = (System.nanoTime() - start) / 1_000_000; // ms
            System.out.printf("BCrypt Cost %2d: %5d ms%n", cost, elapsed);
        }
    }
}
```

*Typical server benchmark output (Intel Xeon / AMD EPYC modern cloud instance):*
```text
BCrypt Cost 10:    42 ms  (Acceptable for low-compute microservices)
BCrypt Cost 11:    85 ms
BCrypt Cost 12:   172 ms  (OPTIMAL FOR ENTERPRISE BANKING)
BCrypt Cost 13:   345 ms
BCrypt Cost 14:   690 ms  (Risk of CPU thread exhaustion under load)
BCrypt Cost 15:  1380 ms  (Severely impairs concurrent user logins)
```

### 10.2 Defeating CPU Starvation DoS on Password Endpoints
Because password hashing functions are computationally expensive by design, attackers can exploit login endpoints for asymmetric Resource Exhaustion DoS: an attacker sends 1,000 rapid HTTP Basic requests with arbitrary passwords, causing server CPUs to spike to 100% processing BCrypt rounds.

**Architectural Countermeasures:**

1. **Pre-Authentication Rate Limiting:** Implement IP-based and username-based token buckets (e.g., Bucket4j + Redis) *before* `DaoAuthenticationProvider` is invoked. Limit failed attempts to 5 per minute per IP.
2. **Dedicated Virtual Thread / Reactive Execution:** Offload authentication verification away from non-blocking Netty / reactive event loops into dedicated worker pools to prevent blocking gateway threads.
3. **CAPTCHA / Proof-of-Work Step-Up:** Trigger Cloudflare Turnstile verification after 3 consecutive failed login attempts on an account.

### 10.3 Memory Zeroing & Heap Sanitization
In high-assurance banking environments, raw password strings stored in Java `java.lang.String` objects represent a security concern because `String` is immutable. A plaintext string remains on the JVM heap until garbage collection occurs, making it readable via heap dump analysis tools.

- Where feasible, accept passwords as `char[]` arrays.
- Immediately after hashing or verification, overwrite the array:
  ```java
  Arrays.fill(passwordCharArray, '\0');
  ```
- Clear all DTO fields holding sensitive authentication credentials immediately after the filter pipeline completes.
