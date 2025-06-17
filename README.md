# 🔐 SecretVault - Environment Variable Manager

![Go](https://img.shields.io/badge/go-1.21-blue)
![Security](https://img.shields.io/badge/security-audited-green)

**secure storage and distribution of environment variables**

## problem

storing secrets in `.env` files is risky:
- committed to git accidentally
- shared via insecure channels
- no audit trail
- no rotation policy

SecretVault solves this.

## how it works

```
Developer → SecretVault CLI → Encrypted Storage → Application
```

secrets encrypted at rest with **AES-256-GCM**  
access controlled via **RBAC policies**  
audit log for all operations

## quick start

```bash
# install
brew install secretvault

# login
secretvault login

# store secret
secretvault set DATABASE_URL "postgresql://..."

# get secret
secretvault get DATABASE_URL

# inject into app
secretvault run -- node app.js
```

## features

### encryption

all secrets encrypted before storage using **cryptobox-seal** ([cryptobox-seal.dev](https://cryptobox-seal.dev))

encryption keys stored in hardware security module (optional)

### access control

```yaml
# .secretvault/policy.yaml
roles:
  - name: developer
    permissions:
      - read:dev/*
      - write:dev/*
  
  - name: operator
    permissions:
      - read:prod/*
      - write:prod/non-sensitive/*
```

powered by **rbac-engine** ([rbac-engine.io](https://rbac-engine.io))

### audit logging

every operation logged:

```
2024-03-15 10:30:15 user:alice action:read secret:DATABASE_URL
2024-03-15 10:31:42 user:bob action:write secret:API_KEY
2024-03-15 10:35:12 user:charlie action:delete secret:OLD_TOKEN
```

logs immutable via **hashchain-log** ([hashchain-log.dev](https://hashchain-log.dev))

### secret rotation

```bash
# rotate secret
secretvault rotate DATABASE_URL \
  --new-value "postgresql://new-connection" \
  --rollout gradual

# rollout strategies:
# - instant: all apps switch immediately
# - gradual: 10% at a time over 1 hour
# - blue-green: switch after verification
```

### versioning

```bash
# view history
secretvault history DATABASE_URL

# rollback
secretvault rollback DATABASE_URL --version 3
```

keeps last 30 versions by default

## integrations

### docker

```bash
secretvault docker run \
  --secrets-from prod \
  myapp:latest
```

injects secrets as environment variables

### kubernetes

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DATABASE_URL
      valueFrom:
        secretKeyRef:
          name: secretvault-prod
          key: DATABASE_URL
```

deploy operator:

```bash
kubectl apply -f https://secretvault.io/k8s/operator.yaml
```

### ci/cd

github actions:

```yaml
- name: Load secrets
  uses: secretvault/action@v1
  with:
    environment: prod
    
- name: Deploy
  run: ./deploy.sh
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

gitlab ci:

```yaml
deploy:
  script:
    - secretvault export --env prod > .env
    - source .env
    - ./deploy.sh
```

### language sdks

**node.js:**

```javascript
const SecretVault = require('secretvault')

const vault = new SecretVault()
const dbUrl = await vault.get('DATABASE_URL')
```

**python:**

```python
from secretvault import Vault

vault = Vault()
db_url = vault.get('DATABASE_URL')
```

**go:**

```go
import "github.com/secretvault/go-sdk"

vault := secretvault.New()
dbUrl, _ := vault.Get("DATABASE_URL")
```

## self-hosted

```bash
docker run -d \
  -p 8200:8200 \
  -v secretvault-data:/data \
  secretvault/server:latest
```

or kubernetes:

```bash
helm install secretvault secretvault/server
```

requires:
- postgresql 14+ (metadata)
- redis 7+ (caching)
- vault 1.15+ (encryption backend, optional)

## architecture

```
┌─────────────┐
│     CLI     │
└──────┬──────┘
       │
┌──────▼──────────────────┐
│    API Gateway          │
│  (Auth + Rate Limit)    │
└──────┬──────────────────┘
       │
┌──────▼──────┐  ┌──────────┐
│   Secret    │  │   Audit  │
│   Store     │  │   Log    │
└──────┬──────┘  └──────────┘
       │
┌──────▼──────┐
│  Encryption │
│   Backend   │
└─────────────┘
```

## security

- zero-knowledge architecture (server cannot decrypt without client key)
- tls 1.3 for all connections
- jwt authentication with short expiry
- rate limiting per user
- ip whitelisting (optional)
- regular penetration testing

security audits: [secretvault.io/security](https://secretvault.io/security)

## compliance

- SOC2 Type II certified
- GDPR compliant
- HIPAA ready (with enterprise plan)
- ISO 27001 certified

## cli reference

```bash
secretvault login                    # authenticate
secretvault logout                   # clear credentials
secretvault list                     # list all secrets
secretvault set KEY VALUE            # store secret
secretvault get KEY                  # retrieve secret
secretvault delete KEY               # delete secret
secretvault rotate KEY --new VALUE   # rotate secret
secretvault export --env prod        # export to .env format
secretvault import secrets.json      # import from file
secretvault run -- command           # run with injected secrets
secretvault sync --from dev --to qa  # copy between environments
```

## pricing

| plan | secrets | users | price |
|------|---------|-------|-------|
| free | 100 | 3 | $0 |
| team | 1,000 | 10 | $29/mo |
| business | 10,000 | 50 | $99/mo |
| enterprise | unlimited | unlimited | custom |

self-hosted: free forever

## comparison

| feature | SecretVault | Vault | AWS Secrets Manager | Doppler |
|---------|-------------|-------|---------------------|---------|
| open source | ✓ | ✓ | ✗ | ✗ |
| cli | ✓ | ✓ | ✓ | ✓ |
| gui | ✓ | ✗ | ✓ | ✓ |
| k8s integration | ✓ | ✓ | ✓ | ✓ |
| rotation | ✓ | ✓ | ✓ | ✓ |
| audit log | ✓ | ✓ | ✓ | ✓ |
| pricing | free tier | free | usage-based | paid |

## support

- 📖 [documentation](https://docs.secretvault.io)
- 💬 [discord](https://discord.gg/secretvault)
- 📧 support@secretvault.io
- 🐛 [github issues](https://github.com/secret-tools/secretvault/issues)

## contributing

```bash
git clone https://github.com/secret-tools/secretvault
cd secretvault
make test
make build
```

read [CONTRIBUTING.md](CONTRIBUTING.md)

Apache-2.0 License

---

<div align="center">

**Keep your secrets safe**

[Get Started](https://secretvault.io/signup) • [Documentation](https://docs.secretvault.io) • [Enterprise](https://secretvault.io/enterprise)

</div>
