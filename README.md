<div align="center">

# MPVPN

### Multi-Path WireGuard & OpenVPN Gateway

**Spread your traffic across several VPN tunnels at once.**<br>
Automatic failover. Leak-proof by design. Post-quantum where providers allow it.<br>
Runs from RAM – power off and nothing is left.

---

**[mpvpn.dev](https://mpvpn.dev)** · **[Download](https://github.com/KevinGandalf/mpvpn-preview/releases/latest)**

</div>

## What is MPVPN?

MPVPN is a self-hosted VPN gateway for your home or office network. It sits between your
LAN and the internet and sends every connection through one of several VPN tunnels – from
the same or from different providers (Mullvad, NordVPN, IVPN, Surfshark, Proton, your own
VPS …). If a tunnel fails, traffic moves on to the others within seconds. Nothing leaves
your network outside a tunnel.

It is a single static Go binary (userspace WireGuard and a native OpenVPN client, gVisor
netstack) and runs on Alpine Linux – or as a stateless boot image entirely from RAM.

## Download

Programs and boot images are free to download from the
[releases](https://github.com/KevinGandalf/mpvpn-preview/releases). Every release is
built reproducibly, boot-tested and signed (Ed25519).

| File | Use |
|---|---|
| `mpvpn-x86_64.iso` | CD/DVD, `dd` to a USB stick, KVM/QEMU/Proxmox (BIOS + UEFI) |
| `mpvpn-x86_64.qcow2` | KVM disk image |
| `vmlinuz-mpvpn` + `initramfs-mpvpn`, `mpvpn.ipxe` | PXE/iPXE network boot |
| `mpvpn-pro-linux-amd64` | program for a normal installation (Alpine, Debian, Ubuntu, Alma/Rocky) |
| `mpvpn-obfs-<os>-<arch>` | Mac/Linux client: WireGuard as HTTPS to your own relay |
| `mpvpn-knock-<os>-<arch>` | knock tool for the Paranoid mode |
| `SHA256SUMS(.sig)`, `BOOTSUMS(.sig)`, `manifest.json(.sig)`, `*.cdx.json` | checksums, signatures, update manifest, SBOM |

```sh
sha256sum -c SHA256SUMS    # then check SHA256SUMS.sig with the published release key
```

The VPN core runs without a license (Community). The Pro features below need a license –
the program and the image check it.

## Features

### Multipath & routing

| Feature | Description |
|---|---|
| **Multi-path tunnels** | any number of WireGuard, OpenVPN and IPsec/IKEv2 uplinks at the same time |
| **IPsec/IKEv2** | router logins of providers without WireGuard export (e.g. AdGuard VPN); IKEv2 and ESP run inside MPVPN, no strongSwan; form with connection test |
| **6 balancing strategies** | sticky, weighted round-robin, least connections, lowest RTT, failover, bandwidth-aware |
| **Health checks** | probes with RTT (EWMA); dead tunnels drop out in seconds, recover automatically |
| **Rules** | by domain (wildcards), IP/CIDR, port, protocol, client or network – e.g. `*.example.de` → German tunnel |
| **VLANs** (Pro) | 802.1Q VLANs, each with its own tunnels (e.g. VLAN 10 → NordVPN, VLAN 20 → Mullvad; dropped, not rerouted, when they are down), DNS through the same tunnels, own DHCP server, admin-only VLANs, isolation |
| **Split DNS** | templates for streaming, banking, LAN names; DNS cache with IP→name mapping |
| **Lists** | ready-made domain/IP lists, refreshed automatically |
| **Provider assistant** (Pro) | log in at NordVPN or PIA, pick country, city and servers – uplinks are created and PIA keys renewed automatically; Proton via its router configs |
| **Import** | `.conf` (WireGuard) and `.ovpn` (OpenVPN) files from your provider |
| **Gateway mode** | MPVPN as default gateway with its own DHCP server, or next to your router |

### Leak protection

| Feature | Description |
|---|---|
| **Kill switch** | nftables: nothing leaves the LAN outside a tunnel, also during restarts |
| **DNS only through the tunnel** | all port 53 traffic captured, DoT/DoH to third parties blocked |
| **IPv6 off** | disabled in kernel and sysctl – no IPv6 leaks |
| **Leak self-test** | one click: WAN IP vs. exit IP of every tunnel, kill switch and tunnel DNS |

### Post-quantum

| Feature | Description |
|---|---|
| **WireGuard + ML-KEM-1024** | Mullvad ephemeral peer, detected and used automatically |
| **OpenVPN hybrid TLS** | X25519MLKEM768 in the TLS control channel where the server supports it |
| **Encrypted backups** | age with post-quantum recipients (ML-KEM-768 + X25519) |

### Privacy & fingerprint

| Feature | Description |
|---|---|
| **Rotation** (Pro) | switch active tunnels and WireGuard servers from a pool, by hand or on a schedule with jitter; group by provider |
| **Network fingerprint** | random WAN MAC address, neutral DHCP identity, uniform TTL |
| **Obfuscation** (Pro) | WireGuard over TLS/WebSocket on port 443 to your own relay – looks like HTTPS; mode `auto` falls back from UDP |
| **Mac client** | `mpvpn-obfs`: the official WireGuard app on a Mac, tunnelled as HTTPS |
| **No logs** | logs and audit only in RAM, memory locked, no core dumps |

### RAM-disk appliance (Pro)

| Feature | Description |
|---|---|
| **Stateless** | the whole system runs from RAM; no swap, no writes to any disk |
| **LUKS USB key** | configuration on an encrypted stick, opened only for seconds at boot and when saving |
| **Network vault** | alternatively an encrypted vault on a web server or KVM disk |
| **Verified boot files** | signed hashes of kernel and initramfs checked before unlocking |
| **Boot media** | ISO (USB/CD), qcow2, PXE/iPXE from one build |

### Emergency (Pro)

| Feature | Description |
|---|---|
| **Panic button** | dashboard, shell or knock: lock down or wipe immediately |
| **USB pull** | removing the key ends the session |
| **Duress passphrase** | a second passphrase that wipes instead of unlocking |
| **Dead-man switch** | wipe when you stop checking in |
| **Paranoid mode** | dead-man switch kept alive by knocks that look like failed IMAP/SMTP/DNS logins |

### Administration (Pro)

| Feature | Description |
|---|---|
| **Web interface** | HTTPS on port 8443, dark theme, German and English |
| **Login** | bcrypt, lockout, 2FA with TOTP and WebAuthn/passkeys |
| **Live apply** | changes take effect at once, with automatic rollback if the box loses its tunnels |
| **Remote access** | optional management WireGuard: admin UI and SSH from outside only through an own tunnel, keys per device with QR code |
| **Network page** | every device of the LAN and the VLANs as map, table and gateway star (which devices bypass the VPN via the router): names (DHCP, mDNS, NetBIOS), vendor, device type, tunnels used; RAM only, discovery, scan and history each switchable off |
| **Everything from the shell** | every setting also via `mpvpn-pro -get/-set` or the config file |
| **Signed updates** | update notice in UI, CLI and login; optional auto-update with rollback watchdog |
| **Backups** | encrypted with age, to file or download |

### Supply chain

Reproducible builds (same commit → same SHA-256), pinned dependencies and GitHub actions,
`govulncheck` on every build, CycloneDX SBOM for every binary, offline root signing key
with short-lived CI subkeys.

## Editions

| | **Community** | **Pro** |
|---|:---:|:---:|
| Price | free | announced at launch |
| WireGuard multipath, all strategies, rules, split DNS | ✅ | ✅ |
| Kill switch, DNS/IPv6 leak protection | ✅ | ✅ |
| Post-quantum WireGuard (Mullvad) | ✅ | ✅ |
| Network fingerprint, config vault | ✅ | ✅ |
| Shell configuration | ✅ | ✅ |
| **Web interface** (2FA, live apply, updates, backups) | – | ✅ |
| **OpenVPN uplinks** (incl. post-quantum TLS) | – | ✅ |
| **IPsec/IKEv2 uplinks** | – | ✅ |
| **RAM-disk appliance** (LUKS key, emergency functions) | – | ✅ |
| **Obfuscation** + Mac client | – | ✅ |
| **Rotation** | – | ✅ |
| **Remote access** via management WireGuard | – | ✅ |
| **VLANs** with own tunnels, DHCP, admin VLAN, isolation | – | ✅ |
| **Network page** (devices, VLAN editor) | – | ✅ |

The number of tunnels is never limited. A license is bound to one device and can be moved.

Privacy-respecting licensing: the periodic check sends only a machine ID and the program's own
checksum – never your traffic, DNS or browsing. Because every release is reproducible and
signed, that checksum can be verified against the public release.

## Platforms

| System | Support |
|---|---|
| **Alpine Linux** | reference platform (OpenRC) |
| RAM-disk image | Alpine-based, x86_64 |
| Debian 13, Ubuntu 24.04, AlmaLinux/Rocky 10 | tested; `mpvpn-hostcheck` adjusts systemd-resolved and sudo |

Needs `/dev/net/tun`, `nftables` and `iproute2`. 1 vCPU and 512 MB RAM are enough.

## Architecture

```
LAN client ─▶ TUN ─▶ gVisor netstack ─▶ DNS (cache, split DNS) ─▶ Rules ─▶ Balancer
                                                                               │
              ┌────────────────────┬──────────────────────┬────────────────────┘
              ▼                    ▼                      ▼
        WireGuard uplink     WireGuard uplink       OpenVPN uplink        … any number
        (PQ: ML-KEM-1024)    (obfuscated: WSS/443)  (PQ: X25519MLKEM768)
              └──────── Kill switch (nftables) – nothing leaves outside a tunnel ────────

RAM-disk:  Boot ─▶ verify boot files ─▶ unlock LUKS key ─▶ load vault into RAM ─▶ run
```

## Coming next

Censorship-resistant transports (Hysteria2, VLESS + Reality, Shadowsocks-2022, CDN fronting),
AmneziaWG, post-quantum for more providers (NordVPN, IVPN, Windscribe), a post-quantum PSK
service on your own VPS.

---

<div align="center">

**Self-hosted. Multipath. Privacy-first.**

**[mpvpn.dev](https://mpvpn.dev)**

© 2026 MPVPN

</div>
