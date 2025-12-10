# PKI-Based 2FA Microservice

This repository contains a Node.js microservice developed for the Partnr PKI-2FA assignment. It performs secure seed decryption, TOTP generation and verification, cron-based logging, and commit signing. The service is fully containerized using Docker.

## Overview

The microservice provides:
- RSA-OAEP (SHA-256) decryption of an encrypted seed
- TOTP code generation (RFC 6238 compliant)
- Verification of TOTP codes with a ±30 second time window
- Cron job that logs the current TOTP code every minute (UTC)
- Persistent storage using Docker volumes
- RSA-PSS commit signature generation

## Features

### Seed Decryption
Decrypts the encrypted seed provided by the instructor API using the student private key.

### TOTP Generation
Generates a 6-digit TOTP code using SHA-1, 30-second time steps, and a Base32 secret.

### TOTP Verification
Validates a given TOTP code with tolerance for small clock drift.

### Cron Logging
A cron task runs every minute and writes the current UTC timestamp and TOTP code to:
```
/cron/last_code.txt
```

### Docker Support
The microservice includes:
- Dockerfile (multi-stage build)
- docker-compose.yml
- Volumes for `/data` and `/cron`

### Commit Signature
Signs a Git commit hash using RSA-PSS (SHA-256) and encrypts the signature with the instructor's RSA-OAEP public key.

## Project Structure

```
.
├── Dockerfile
├── docker-compose.yml
├── cron/
│   └── 2fa-cron
├── scripts/
│   ├── log_2fa_cron.js
│   ├── request_seed.js
│   ├── generate_keys.js
│   └── commit_sign.js
├── src/
│   ├── server.js
│   └── crypto/
│       ├── decryptSeed.js
│       ├── seedStore.js
│       └── totp.js
├── student_private.pem
├── student_public.pem
├── instructor_public.pem
├── package.json
└── README.md
```

## Running with Docker

### Build and start the service

```bash
docker-compose build
docker-compose up -d
```

The service runs at:
[http://localhost:8080](http://localhost:8080)

## API Endpoints

### POST /decrypt-seed

```bash
curl -X POST http://localhost:8080/decrypt-seed \
-H "Content-Type: application/json" \
-d "{"encrypted_seed": "$(cat encrypted_seed.txt)"}"
```

### GET /generate-2fa

```bash
curl http://localhost:8080/generate-2fa
```

### POST /verify-2fa

```bash
CODE=$(curl -s http://localhost:8080/generate-2fa | jq -r '.code')

curl -X POST http://localhost:8080/verify-2fa \
-H "Content-Type: application/json" \
-d "{"code": "$CODE"}"
```

## Commit Signature

Run:

```bash
node scripts/commit_sign.js
```

Outputs:
- Commit hash (SHA-1)
- Encrypted RSA-PSS signature (base64)

Submit both as required.

## Docker Image

```
docker.io/harshaithireddy/pki-2fa:latest
```
