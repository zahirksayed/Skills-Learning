# Project Plan: Hospital Data Viewer with RBAC (Role-Based Access Control) — Python & Blockchain Edition

This project plan guides you in building a hospital data viewer application with robust role-based access control (RBAC), first as a traditional Python app, then as a blockchain-enhanced system. It is designed for developers seeking hands-on experience in secure application design and blockchain integration, with an emphasis on clarity and practical learning.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Part 1: Traditional Python RBAC Hospital App](#part-1-traditional-python-rbac-hospital-app)
    - [Core Features](#core-features)
    - [Technology Stack](#technology-stack)
    - [Project Structure](#project-structure)
    - [Implementation Steps](#implementation-steps)
3. [Part 2: Blockchain-Enhanced Hospital App](#part-2-blockchain-enhanced-hospital-app)
    - [Why Blockchain?](#why-blockchain)
    - [Key Paradigm Shifts](#key-paradigm-shifts)
    - [Architecture Overview](#architecture-overview)
    - [Smart Contracts & On-Chain Logic](#smart-contracts--on-chain-logic)
    - [Python Integration & Off-Chain Data](#python-integration--off-chain-data)
    - [Security, Privacy & Compliance](#security-privacy--compliance)
    - [Learning Recommendations](#learning-recommendations)
4. [Summary & Next Steps](#summary--next-steps)

---

## Project Overview

Build a hospital data viewer where users (Admins, Doctors, Nurses, Patients) have different permissions to view or manage patient data. Begin with a traditional Python app, then explore how blockchain transforms access control, auditability, and data management.

---

## Part 1: Traditional Python RBAC Hospital App

### Core Features

- **User Authentication:** Login with username and password (passwords stored securely as hashes).
- **User Roles & Permissions:** Define roles (Admin, Doctor, Nurse, Patient) with specific access rights.
- **Data Simulation:** Store users, patients, and medical records in-memory or in a SQLite database.
- **RBAC Enforcement:** Each action (view, add, manage) is permitted based on user role.

### Technology Stack

- **Python 3.x**
- **Database:** 
    - *Option 1:* In-memory (dictionaries/lists)
    - *Option 2:* SQLite (recommended for persistence)
- **Optional Web Framework:** Flask (for web UI)
- **Password Security:** `hashlib` or `bcrypt` for password hashing

### Project Structure

```
hospital_app/
├── models.py         # Data classes: User, Patient, MedicalRecord
├── database.py       # DB setup, data population, password hashing
├── auth.py           # Authentication & RBAC logic
├── main.py           # Main CLI or Flask app
├── data/             # SQLite DB file
│   └── hospital.db
└── README.md
```

### Implementation Steps

1. **Data Models (`models.py`, `database.py`):**
    - Users: `id`, `username`, `password_hash`, `role`
    - Patients: `id`, `name`, `dob`, `contact`, `assigned_doctor_id`, `user_id`
    - MedicalRecords: `id`, `patient_id`, `date`, `diagnosis`, `treatment`, `doctor_id`, `notes`
    - Functions to setup DB and add dummy data.

2. **Authentication & RBAC (`auth.py`):**
    - `authenticate_user(username, password)`: Validate credentials.
    - `has_permission(role, permission)`: Role-permission mapping.
    - Example Permission Dictionary:
        ```python
        PERMISSIONS = {
            "admin": ["view_all_patients", "view_medical_records", "create_medical_record", "admin_users"],
            "doctor": ["view_assigned_patients", "view_medical_records", "create_medical_record"],
            "nurse": ["view_all_patients", "view_medical_records"],
            "patient": ["view_own_patient_data"]
        }
        ```

3. **Main Application (`main.py`):**
    - **CLI Option:** Prompt login, display menu based on role, enforce permissions via `has_permission`.
    - **Flask Option:** Use Flask-Login, Flask-SQLAlchemy, web templates, and custom decorators for RBAC.

#### Example CLI Menu Flow

```python
# Pseudocode
while True:
    username, password = input()
    user = authenticate_user(username, password)
    if user:
        show_menu(user.role)
        # menu options filtered by has_permission
    else:
        print("Invalid login.")
```

#### Example Flask RBAC Decorator

```python
def roles_required(*roles):
    def wrapper(fn):
        @wraps(fn)
        def decorated_view(*args, **kwargs):
            if current_user.role not in roles:
                return redirect(url_for('dashboard'))
            return fn(*args, **kwargs)
        return decorated_view
    return wrapper
```

---

## Part 2: Blockchain-Enhanced Hospital App

### Why Blockchain?

Traditional apps rely on a central database. Blockchain introduces:

- **Decentralization:** No central authority controlling data.
- **Immutability:** Once data is recorded on-chain, it cannot be changed.
- **Auditability:** All actions are permanently logged, improving security and compliance.
- **Automated Access Control:** Smart contracts enforce RBAC and consent.

### Key Paradigm Shifts

- **Identity:** Users identified by blockchain addresses (public keys), not usernames.
- **Data Storage:** Only metadata/hashes on-chain; sensitive patient data is encrypted and stored off-chain.
- **Smart Contracts:** RBAC and consent logic are coded as immutable contracts.

### Architecture Overview

- **On-Chain (Blockchain):**
    - Store hashes of medical records, access grants/revokes, role assignments.
    - Smart contracts for identity, RBAC, and consent management.

- **Off-Chain (Secure Storage):**
    - Actual medical records stored encrypted in a secure server, cloud, or decentralized file system (e.g., IPFS).

- **Python Client:**
    - Authenticates users, interacts with blockchain via SDKs (e.g., web3.py), retrieves/decrypts off-chain data.

#### Recommended Platforms for Python Developers

- **Quorum:** Permissioned Ethereum variant, works with Solidity smart contracts and web3.py.
- **Hyperledger Fabric:** Enterprise-grade, with Python SDK for client interaction.
- **Algorand:** Python-native smart contract language (PyTeal).

### Smart Contracts & On-Chain Logic

**Key Contracts:**

1. **Identity & Role Management**
    - Register users by address, assign roles.
    - Query roles by address.

2. **Consent Management**
    - Patients grant/revoke access to records for specific users.
    - Track permissions as explicit transactions.

3. **Record Metadata**
    - Store hashes and metadata for each medical record creation event.

4. **Access Verification**
    - Function to check if a requester is authorized to access a record, based on roles and patient consent.

#### Example Solidity Function

```solidity
function grantAccess(address patient, bytes32 recordHash, address doctor, uint expiration) public { ... }
function canAccess(address requester, address patient, bytes32 recordHash) public view returns (bool) { ... }
```

### Python Integration & Off-Chain Data

- Use web3.py or platform-specific SDKs to interact with smart contracts.
- Application flow:
    1. User logs in and signs transactions with private key.
    2. Doctor requests data → Python app calls `canAccess()` on smart contract.
    3. If allowed, fetch encrypted record from off-chain storage, decrypt, display.
    4. When adding records, hash & store metadata on-chain, actual data off-chain.

### Security, Privacy & Compliance

- **No PHI on-chain:** Only hashes and metadata.
- **Encryption:** All off-chain data encrypted.
- **Consent & Audit:** Every access grant/revoke and record creation is auditable.
- **Key Management:** Safeguard user private keys.
- **Compliance:** Design for HIPAA/GDPR (data minimization, right to be forgotten managed off-chain).

### Learning Recommendations

- For rapid learning, prototype the smart contract (identity, consent, metadata) in Solidity/Vyper.
- Use a local blockchain (Ganache, Quorum) for testing.
- Build a minimal Python client to interact, focusing on permission checks and metadata.
- Skip full off-chain storage in your first iteration; simulate the encrypted data retrieval.

---

## Summary & Next Steps

This project exposes you to:

- Secure application design and RBAC principles.
- Python-based authentication and authorization.
- Blockchain architecture for auditability and decentralized access control.
- Smart contract development and Python/blockchain integration.

**Suggested Learning Path:**

1. Build the CLI or Flask version with SQLite and RBAC.
2. Prototype smart contracts for role and consent management on a local blockchain.
3. Integrate with Python using web3.py, simulate access checks and metadata handling.

**Resources:**

- [web3.py Documentation](https://web3py.readthedocs.io/)
- [Solidity Docs](https://docs.soliditylang.org/)
- [Hyperledger Fabric Python SDK](https://hyperledger-fabric.readthedocs.io/en/latest/write_first_app.html)
- [Algorand PyTeal](https://developer.algorand.org/docs/get-details/dapps/pyteal/)
