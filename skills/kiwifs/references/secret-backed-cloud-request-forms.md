# Secret-backed cloud request forms in KiwiFS

Use this when a wiki page is a request/intake form for cloud automation (OCI, AWS, GCP, etc.) and some fields are credentials or paths to credentials.

## Durable pattern

1. Treat the wiki as a coordination document, not a secret store.
2. Never paste private keys, API secrets, MFA codes, console passwords, cookies, or bearer tokens into KiwiFS.
3. For credentials required by automation, store only:
   - execution location (`host`, `container`, `runner`, etc.)
   - secret file path as visible from that execution location
   - config file path when relevant
   - non-secret identifiers such as region, tenancy/account ID, user ID, fingerprint, compartment/project ID
4. If both host and container paths matter, record both and name which one the actual CLI should use.
5. Prefer absolute paths over `~` in machine-consumed config because tilde expansion is shell-dependent.
6. If a generated/private key is encrypted, record that passphrase handling is required; for unattended automation, recommend a dedicated non-passphrase API key with tight scope or a secret manager workflow.
7. When the form is partially filled, preserve unknowns explicitly as `미확인` / `TBD` rather than guessing.

## Cloud-form workflow

1. Read the intake form and relevant canonical guide/checklist.
2. Pull non-secret values from local config only when available and safe to display (e.g. `region`, `fingerprint`, `tenancy`, `user`, `key_file` path). Do not print private-key contents.
3. Fill deterministic defaults in the form:
   - root compartment may equal tenancy/account ID only when explicitly framed as a root-compartment assumption.
   - storage and retry policies should include a stop condition for quota/billing risk.
4. Leave values that require console/UI inspection as remaining inputs, with exact UI path to obtain them.
5. Lint the page after writing.

## OCI-specific notes captured from session

- OCI API key fingerprint can be read from `.oci/config` (`fingerprint=...`) or computed from the API public key. The private key itself must not be pasted into the wiki.
- In a containerized Hermes environment, host `~/works/.oci/oci_api_key.pem` may appear as `/workspace/host/works/.oci/oci_api_key.pem`; record the container path if Hermes will run OCI CLI.
- OCI `config` should use the execution-context path: `key_file=/workspace/host/works/.oci/oci_api_key.pem` when the CLI runs inside that container.
- For OCI A1.Flex Always Free request forms, explicitly include storage: `boot_volume_size_gb=200, extra_block_volume_size_gb=0` for simple single-disk, or `boot 50GB + block 150GB` for OS/data split.
- If VCN already exists, do not tell the user to create another one; next step is to open the existing VCN and copy the Public Subnet OCID.
