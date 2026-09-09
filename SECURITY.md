# Security

A packet capture is a high-value artifact — credentials, session tokens and internal traffic
in the clear. This document describes where kvm-echo puts things, what crosses the wire,
and what the plugin deliberately does not do.

## What the plugin stores, and where

**The encryption key.** A fresh AES-256 key is generated on the Morpheus appliance for every
capture. It is stored in Cypher, the appliance's secret store, under a per-capture entry.
No key is ever reused between captures.

**Getting the key to the host.** The key is delivered to the hypervisor with the plugin
API's `copyToServer` call, written to `/dev/shm/echo-key-<jobId>` at mode 0600. It is never
passed as a command-line argument, on stdin, or through an environment variable — all three
are readable by other processes on the host, argv most obviously so.

The capture wrapper opens that file on file descriptor 3 and immediately `rm -f`s it. The
descriptor keeps the key readable to the running wrapper while the path stops existing, so
the window in which the key is visible on the filesystem is the gap between write and
unlink, not the duration of the capture.

**The ciphertext.** `tcpdump` output is piped directly into AES-256-GCM encryption. The
encrypted stream lands on tmpfs at `/dev/shm/echo/<jobId>.enc` — memory-backed, so it never
reaches persistent storage and does not survive a host reboot. There is no point in the
pipeline at which a cleartext pcap is written to disk on the hypervisor.

**The plaintext pcap.** Exists only in the appliance JVM's memory, during the
decrypt-and-serve flow that runs when you click download. It is not written to appliance
disk, and the AES key does not leave the JVM.

## What leaves the host

Only the AEAD ciphertext, and only when an operator requests an export. It is encrypted
end-to-end: the hypervisor never holds the plaintext, and the transport carries ciphertext
only. Nothing is transmitted on a schedule, and nothing is transmitted as a side effect of
a capture completing.

## Retention

Ciphertext on the hypervisor is cleared on host reboot, since tmpfs does not survive one.

The Cypher key entry expires with the configured retention window — three days by default,
adjustable via the `echo.retention.days` setting. A background sweep refreshes each live
capture's lease so it survives the full window rather than expiring early.

Once a key expires it is gone, and that capture's ciphertext cannot be decrypted by anyone,
including the plugin. This is deliberate: expiry is the mechanism that stops old captures
accumulating as indefinitely-recoverable sensitive data.

## Integrity

Every export is byte-verified before decrypt is attempted. The hypervisor computes the
ciphertext's SHA-256 and reports it alongside the byte count; the appliance compares both
against what actually arrived and refuses to decrypt on any mismatch.

Failures surface as an explicit integrity error with the expected and received sizes and
hashes. This is a correctness guard rather than a security boundary — the ciphertext is
authenticated by AES-GCM regardless — but it means transit corruption is reported as
corruption, rather than as a misleading crypto or format error.

## What the plugin does not do

- **No in-guest agent.** Captures run entirely on the KVM host. Guests are unmodified and
  have nothing installed in them.
- **No persistent cleartext.** At no stage is an unencrypted pcap written to disk, on the
  hypervisor or on the appliance. There is no cleanup step for an operator to forget.
- **No export without an operator action.** Captures do not ship themselves anywhere. A
  scheduled Automation Task produces ciphertext on the host and reports where it is; turning
  that into a pcap still takes a deliberate export.
- **No NOPASSWD sudoers entry.** The plugin uses Morpheus's built-in sudo escalation. There
  is no host-side sudoers file to install, audit, or leave behind after uninstalling.

## Reporting a security issue

Please report security issues through
[**GitHub Security Advisories**](../../security/advisories/new), which keeps the report
private until there is a fix.

**Please do not open a public issue for a security concern.**

This is a community project maintained in spare time, so there is no formal response-time
commitment — but security reports go to the front of the queue.
