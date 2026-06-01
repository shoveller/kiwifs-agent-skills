# Secret path handling in operational request/checklist wiki pages

Use this when a wiki page or Inbox form asks the user to provide credentials, API signing keys, private keys, tokens, or config files for an automation run.

## Rule

Never ask the user to paste private key or token bodies into KiwiFS, Discord, or chat. Ask for a **reference path that is readable from the environment that will execute the automation**.

## Host vs container path pattern

For each secret/config file, record both paths when relevant:

```text
Private Key:
- 본문 공유 안 함
- 실행 위치: <host | container | VPS name>
- host path: <path on the machine where the user copied/stores the file>
- container path: <path visible inside the agent/container, if different>
- config path: <path to config file if applicable>
- config key_file=<execution-environment-visible path>
```

If the automation will run directly on the host, the container path can be omitted or explicitly marked `해당 없음`.

If the automation runs inside Hermes/container, prefer the mounted container path (for example `/workspace/host/works/...`) in the config because libraries and CLIs resolve paths from their own runtime namespace, not the user's local shell namespace.

## Safe verification

It is acceptable to verify existence and permissions, but do not print secret contents.

Good checks:

```bash
realpath <secret-path>
stat -c '%a %U:%G %n' <secret-dir> <secret-path>
grep '^fingerprint=' <oci-config>
grep '^key_file=' <oci-config>
```

Avoid:

```bash
cat <private-key>
head <private-key>
grep -v ... <private-key>
```

## OCI API Signing Key example

For an OCI automation form, distinguish these fields:

- `SSH Public Key`: safe-ish public key line for VM login, e.g. `ssh-ed25519 AAAA...`.
- `API Signing Key public key`: the only key material that should be uploaded/pasted into OCI Console under **Identity & Security → Domains → <domain> → Users → <user> → API Keys → Add API Key**. This is a `-----BEGIN PUBLIC KEY-----` PEM, not the `.oci` directory.
- `API Signing Key private key`: sensitive PEM file used by OCI CLI/API. Provide only a path; never upload it to the console or paste it into chat/wiki.
- `Fingerprint`: after the public key is registered in OCI Console, treat the console-displayed fingerprint as the authoritative value and update `.oci/config` with it. A locally computed candidate can be noted only as a pre-registration hint.
- `key_file`: must point to the private key path as seen by the runtime that calls OCI CLI.

When the user asks “should I upload `.oci` here?”, answer directly: **No — upload/paste only the public key into the user's API Keys page; keep `.oci/config` and the private key on the execution machine/container.**

Example for a container-visible copied OCI config:

```text
Private Key:
- 본문 공유 안 함
- 실행 위치: Hermes/컨테이너
- host path: ~/works/ai-assistant/runtime/.oci/oci_api_key.pem
- container path: /opt/data/.oci/oci_api_key.pem
- OCI config path: /opt/data/.oci/config
- OCI config key_file=/opt/data/.oci/oci_api_key.pem
```

Then update the config if needed using a safe writer/editor rather than printing private key contents. The finished config shape is:

```ini
[DEFAULT]
user=<user_ocid>
fingerprint=<console_api_key_fingerprint>
tenancy=<tenancy_ocid>
region=<region>
key_file=<execution-runtime-visible-private-key-path>
```

## Wiki form guidance

When updating a checklist or answering follow-up questions from it:

1. Identify which document is the execution checklist and which is the background guide.
2. For blank fields, give exact source locations or safe commands.
3. Mark secrets as path-only, not pasteable content.
4. Prefer absolute paths over `~` in machine-consumed config, because tilde expansion is not guaranteed in all tools.
5. Include a stop condition for quota/billing risk, especially when a form touches cloud resources.
