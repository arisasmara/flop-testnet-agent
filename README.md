# Agent di FLOP Testnet — Dokumentasi Pengalaman

Repo ini mendokumentasikan langkah-langkah yang saya jalankan untuk ikut
FLOP testnet sebagai **Agent**, pada 24 November 2026. Tujuannya belajar
dan jadi perbendaharaan pengalaman — **bukan tutorial resmi**.

Tutorial yang dipakai: [Azet17/flop-testnet-agent-tutorial](https://github.com/Azet17/flop-testnet-agent-tutorial)
(MIT). Status resmi saya verifikasi sendiri ke
[flop.finance/testnet](https://flop.finance/testnet/) sebelum bertindak.

## Status testnet (per 24 Nov 2026)

- Testnet **LIVE** sebagai **"unscored opening"** sejak 24 Nov 2026.
  Enam validator pendiri memproduksi blok. Mainnet direncanakan Q1 2027.
- **Live ≠ scored.** Fase scored — satu-satunya jalan ke genesis airdrop —
  belum mulai dan belum ada tanggal mulainya.
- **Faucet testnet BUKA 25 Nov 2026** (siang saldo tinggal 1 FLOP, sore
  diisi ulang jadi ~9 juta FLOP). Grant agent 10 FLOP per wallet per 24 jam
  (rolling), cooldown 86.400 detik. Klaim lewat `POST
  /v1/testnet/agent-claims` di https://faucet.flop.network/ — lihat
  Langkah 3 di bawah.
- Yang dihitung buat agent: **compute yang dibeli di sesi inference yang
  settled**, pro-rata dari pool draft 1,2 miliar FLOP. Klaim faucet,
  megang token, bikin DID, atau chat doang **tidak dihitung**.
- Airdrop-nya nanti **terkunci penuh, cuma bisa dipakai buat inference**
  (1 FLOP kebuka tiap 3 FLOP dipakai buat inference).
- Satu peserta dihitung satu skor — dompet di bawah kendali yang sama
  dihitung sebagai satu peserta.

## Kenapa jalur Agent

- **Miner** butuh GPU. **Validator** butuh VPS 24/7 (8+ core CPU, 64 GB RAM,
  4 TB NVMe). Saya cuma punya HP Android — keduanya tidak bisa dijalankan
  dari HP.
- **Agent** cuma butuh DID + wallet, dan langkah-langkahnya berupa perintah
  terminal yang bisa dijalankan lewat Muse.

## Yang dibutuhkan

- Python 3.10+
- Paket Python: `pynacl`, `base58`, `substrate-interface`
- `curl`

## Langkah-langkah yang sudah dijalankan

### 0. Verifikasi status resmi dulu

Cek [flop.finance/testnet](https://flop.finance/testnet/) dan baca apa yang
dihitung. Jangan andalkan status di README tutorial kalau sudah lewat
harinya.

### 1. Clone tutorial, baca script sebelum dijalankan

```bash
git clone https://github.com/Azet17/flop-testnet-agent-tutorial.git
```

`gen_keys.py` dan `tc.py` saya baca dulu sebagai teks biasa sebelum
dijalankan. Ini script pihak ketiga — jangan jalankan sebelum tahu isinya.
Di kasus saya, clone sempat kena pemeriksaan execution-risk di sisi Muse;
saya baca scriptnya dulu, lalu lanjut setelah saya yang memutuskan.

### 2. Install dependensi di venv

```bash
python3 -m venv .venv
.venv/bin/pip install pynacl base58 substrate-interface
```

Yang terpakai: Python 3.12.3, pynacl 1.6.2.

### 3. Buat DID + wallet baru (`gen_keys.py`)

```bash
.venv/bin/python gen_keys.py
```

Hasil, 24 Nov 2026 23:13 WITA:

- **DID:** `did:key:z6MkvRG4ijBPSGy4pXH2PkGjLaDmAdQFH7Mx7rrMJXaHpUv4`
- **Wallet:** `5DJHWhEN9sM3d96G6k7KNxvXZJZbyzRc4M5qrhAsnBQbrsoz`
  (SS58 prefix 42, Sr25519)
- File yang dibuat: `did-key.json` + `wallet.json` (mode `0600`, berisi
  seed/mnemonic — **JANGAN PERNAH** dibagikan atau di-commit; `.gitignore`
  sudah menutup keduanya) dan `public.json` (hanya DID + alamat, boleh
  disebar).
- Identitas agent **the00** — dibuat baru dari nol.

### 4. Check-in di Technocore

```bash
.venv/bin/python tc.py testnet "Halo, agent the00 check-in buat FLOP testnet. DID did:key:z6MkvRG4ijBPSGy4pXH2PkGjLaDmAdQFH7Mx7rrMJXaHpUv4 Wallet 5DJHWhEN9sM3d96G6k7KNxvXZJZbyzRc4M5qrhAsnBQbrsoz Siap beli inference begitu faucet testnet buka."
```

Verifikasi:

```bash
curl -s "https://technocore.chat/r/testnet?limit=10" | grep -i the00
```

Terkonfirmasi: pesan `[19627]`, 2026-11-24T15:13:14Z, penulis
terverifikasi `<z6Mk…pUv4>`.

## Langkah 3 — Klaim faucet testnet (percobaan 25 Nov 2026)

Alur resmi (Agent Guide, `flop.network/llms.txt` bagian "Get Testnet
FLOP"):

1. Cek `GET https://faucet.flop.network/chain`: harus `network: testnet`
   dan `genesisHash`
   `0x918abd58fb16306b30ef3886514cfaebffecb4de43e82b6f8ec4795c43cfd800`
   (cocokkan dengan `chain_getBlockHash(0)` di RPC testnet).
2. Cek `GET https://faucet.flop.network/limits`: `faucet.status` harus
   `open` dan `faucet.agent.claimPath` harus `/v1/testnet/agent-claims`.
3. Tandatangani teks UTF-8 berikut dengan kunci Sr25519 wallet penerima:
   `flop-faucet-agent/v1|<genesis hash>|<alamat SS58>|<nonce>`
   (nonce desimal, maks 19 digit).
4. `POST https://faucet.flop.network/v1/testnet/agent-claims` dengan JSON
   `{"address": "<alamat>", "nonce": "<nonce>", "signature": "0x<hex>"}`.
5. Simpan `txHash` yang dikembalikan. Grant baru bisa dipakai setelah
   transaksinya masuk blok Testnet yang finalized.

**Jangan pakai pola devnet di testnet:** challenge `fund|<alamat>|<nonce>`
+ GET `faucet.technocore.chat/faucet/fund/...` dan room Technocore
`flop-testnet-faucet` sudah ditutup / belum dibuka di testnet — server
menolaknya dengan `claim_signature_invalid` ("Sign the D10 versioned
claim").

**Status 25 Nov 2026 ~21:00 WITA:** klaim pertama DITOLAK dengan HTTP 429
`ip_cooldown` — "This IP funded recently; retry in 1073 min". IP egress
VM Muse dipakai bersama agent lain dan baru saja dipakai klaim — BUKAN
wallet ini (wallet belum pernah klaim). Jangan retry berulang. Retry
dijadwalkan Kamis 26 Nov 2026 15:00 WITA (setelah cooldown habis
~14:53 WITA). Saldo faucet saat percobaan: 8.083.723 FLOP.

## Yang belum / selanjutnya

- **Klaim faucet harian** — retry dijadwalkan 26 Nov 2026 15:00 WITA
  (lihat Langkah 3).
- **Beli inference** lewat gateway `https://api.flop.network/v1` atau SDK
  (`@flop-labs/sdk`, `@flop-labs/client`), begitu fase scored diumumkan
  (cek `flop.finance/testnet/` dan `@flop_labs` di X).
- Hanya sesi yang **settled** yang dihitung. Klaim tanpa belanja = nol.

## Catatan keamanan

- Semua token di sini adalah test FLOP — tidak ada nilainya, tidak ada
  dana asli yang dipakai.
- Jangan pernah kirim dana asli, seed phrase dompet utama, atau private
  key apa pun untuk testnet ini.
- Backup (`backup.py`) hanya kalau diminta — ZIP-nya tidak ber-password,
  simpan di tempat privat, jangan disebar.
- FLOP tidak pernah minta seed phrase / deposit untuk klaim, mint, atau
  daftar apa pun. Situs yang minta begitu adalah scam.
- Belum ada token FLOP mainnet. Situs yang jual / mint / klaim FLOP
  mainnet adalah scam.

## Link

- Testnet: https://flop.finance/testnet/
- Airdrop: https://flop.finance/airdrop/
- Link resmi + verifikasi: https://flop.finance/official/
- Docs agent: https://ask.flop.finance/docs/agent-onboarding
- Docs integrasi: https://ask.flop.finance/docs/agent-integration
- Technocore: https://technocore.chat/
- Faucet: https://faucet.flop.network/
- Dokumentasi: https://docs.flop.network/
- Explorer (testnet): https://testnet.flopscan.io/
- Explorer (devnet): https://explorer.devnet.flop.network/
- Skill untuk AI agent (llms.txt, devnet): https://docs.flop.network/llms.txt?network=devnet
- X: https://x.com/flop_labs
- RPC testnet: `wss://testnet.flop.network/`
- Genesis hash testnet:
  `0x918abd58fb16306b30ef3886514cfaebffecb4de43e82b6f8ec4795c43cfd800`
- Tutorial yang dipakai: https://github.com/Azet17/flop-testnet-agent-tutorial

## Lisensi

MIT — lihat `LICENSE`.
