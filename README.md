# Qoder Lab — Setup Lokal

Folder ini berisi eksperimen otomasi akun Qoder (`hirotomasato/qoder-creator`)
dengan **temp mail self-hosted** (`tempik`) yang jalan di Cloudflare.

## Struktur

```
qoder-lab/
├── .venv/                     # Python 3.12.7 (pyenv)
├── qoder-creator/             # repo target
│   ├── config.toml            # nunjuk ke tempik produksi
│   ├── data/accounts.jsonl    # hasil akun
│   └── src/
└── tempik/                    # temp mail (Cloudflare Worker)
    ├── wrangler.toml          # config produksi (kodemaverick.my.id)
    └── wrangler.local.toml    # config lokal (wrangler dev)
```

## Status Infrastruktur

| Komponen | Status | Detail |
|---|---|---|
| Python | ✅ | 3.12.7 via pyenv + `.venv` |
| Playwright Chromium | ✅ | terinstall |
| Google Chrome | ✅ | `/Applications/Google Chrome.app` |
| Tempik worker | ✅ | `https://tempik.kwkbkartono.workers.dev` |
| Email catch-all | ✅ | `*@kodemaverick.my.id` → worker tempik |
| D1 database | ✅ | `tempik-db` (id `d7314464-...`) |

## Domain

| Domain | Fungsi |
|---|---|
| `satgas.ai.id` | Email pribadi (catch-all → Gmail) — JANGAN disentuh |
| `maverig.my.id` | Email pribadi (catch-all → Gmail) — JANGAN disentuh |
| `kodemaverick.my.id` | Temp mail tempik (catch-all → worker) |

> Cloudflare hanya mengizinkan **catch-all di apex domain**, bukan di subdomain.
> Karena itu temp mail pakai seluruh `kodemaverick.my.id`.

## Cara Menjalankan

### Tempik lokal (untuk dev, opsional)
```bash
cd tempik
npx wrangler dev --config wrangler.local.toml --port 8787 --ip 127.0.0.1
```

### qoder-creator
```bash
cd qoder-creator
source ../.venv/bin/activate
python main.py view          # lihat hasil
python main.py signup -n 1   # bikin 1 akun
```

## Hasil Uji

Flow yang **terbukti jalan**:
- ✅ Temp email via tempik (`xxx@kodemaverick.my.id`)
- ✅ Buka halaman signup Qoder
- ✅ Isi form + password
- ✅ Solve slider captcha Aliyun (lokal, image processing)
- ✅ Terima OTP lewat tempik + ekstrak 6 digit

Flow yang **mentok**:
- ❌ Setelah captcha slider, Qoder memunculkan **captcha gambar kedua**
  (Aliyun "punish" page, `action=captchaclick`) → akun belum authenticated
- ❌ PAT creation → 401 "User not authenticated"

Root cause: risk-score Aliyun tinggi → muncul image captcha yang tidak bisa
di-solve repo ini (cuma ada slider solver).

## Catatan Keamanan
- `config.toml`, `accounts.jsonl`, `debug_shots/` berisi data sensitif — gitignored.
- Worker tempik hanya menerima email di `kodemaverick.my.id`.
- Tidak menyentuh Email Routing domain `satgas.ai.id` / `maverig.my.id`.
