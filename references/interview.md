# The interview

The site is only as true as what the user tells you. Interview in small
groups — 3 to 6 questions per turn, in the user's language (Indonesian by
default) — and move on when you have enough. Every answer below names where it
goes, so nothing asked is wasted and nothing shown is invented.

**What you may write yourself:** headlines, taglines, section intros, button
labels, SEO titles and descriptions, the wording of a service description the
user described in their own words.

**What only the user may supply:** prices, names of people, addresses, phone /
WhatsApp numbers, email, opening hours, legal and license numbers, awards and
certifications, client names and logos, testimonials, ratings, statistics
("10 tahun", "500+ klien"), menu items, product specs. Unknown → ask, or leave
the field or section out. Never a placeholder.

## Group 1 — always

| Ask | Goes to |
|---|---|
| Nama usaha / nama Anda? | `siteName`, navbar `siteName`, `businessProfile.legalName` (only if it is the legal name) |
| Dalam satu-dua kalimat, usaha ini apa dan untuk siapa? | `aiDescription`; hero `headline`/`subheadline` (you word it) |
| Kota / area layanan? | hero or features copy; `businessProfile.address.city` |
| Nomor WhatsApp dan/atau telepon untuk pelanggan? | `businessProfile.phone` (turns on the WhatsApp bubble), `contact.phone` / `contact.whatsappUrl` (`https://wa.me/62…`), cta `buttonUrl` |
| Email? | `businessProfile.email`, `contact.email` |
| Bahasa situs? (Indonesia, English, …) | `language` |

## Group 2 — presence (ask right after group 1)

| Ask | Goes to |
|---|---|
| Alamat lengkap (jalan, kota, kode pos)? | `businessProfile.address` {street, city, region, postalCode, country}; `map.address` and `contact.address` |
| Link Google Maps? | Not needed for the map itself: the `map` section draws the map from `address`. A share link (`maps.app.goo.gl/…`) cannot be embedded — use it only where a variant has a "directions" link field (`mapsLinkUrl`). If they want the exact pin, ask for Google Maps → Bagikan → **Sematkan peta** and put that iframe code in `map.mapEmbedUrl`. |
| Jam buka? | `businessProfile.openingHours` [{days: ["Mo",…], open: "08:00", close: "17:00"}]; `map.hours` as text |
| Instagram / TikTok / Facebook / YouTube / marketplace links? | `businessProfile.socialLinks`, footer `links` |
| Punya logo? Foto usaha / produk / tim? | `asset:<file>` (upload via `begin_asset_upload`), navbar `logoUrl`, hero `images`, gallery |
| Gaya yang diinginkan? (hangat, modern, mewah, ceria, formal) Warna brand? | `theme` (see `site-json.md`) |

## Group 3 — by kind of business

Pick the one that fits (or two, for a hybrid). Ask the first 3–5 questions,
then follow up only on what they have.

### Kuliner (warung, kafe, resto, katering, kue)
| Ask | Goes to |
|---|---|
| Menu andalan beserta HARGA? (nama, harga, deskripsi singkat, foto) | `products` (manual) or `pricing`; a Menu page |
| Bisa pesan antar / GoFood / GrabFood / pre-order? | cta, `contact`, links in hero `ctaUrl` |
| Suasana tempat (dine-in, kapasitas, cocok untuk acara)? | `features-grid`, `gallery` |
| Terima pesanan besar / katering? Minimal order? | `features-grid` or `faq` |
| Ada ulasan pelanggan yang boleh dikutip? (nama + isi) | `testimonials` — only these, verbatim |

### Jasa (bengkel, laundry, servis AC, kontraktor, desain, konsultan)
| Ask | Goes to |
|---|---|
| Layanan apa saja? Untuk tiap layanan: apa yang dikerjakan, tarif atau "mulai dari"? | `features-grid` (services), `pricing` when they have clear packages |
| Area layanan? Datang ke lokasi atau di tempat? | features copy, `faq` |
| Proses kerjanya (pesan → survei → kerja → garansi)? | `steps` |
| Garansi? Lama pengerjaan? | `faq`, features |
| Portofolio / foto hasil kerja? Sebelum-sesudah? | `gallery`, `before-after` |
| Klien yang boleh disebut? Testimoni? | `logo-cloud` (with their logos), `testimonials` |

### Toko (fashion, kosmetik, elektronik, kerajinan)
| Ask | Goes to |
|---|---|
| Produk utama: nama, harga, varian (ukuran/warna), foto? | `products` (manual) — or tell them the dashboard's Produk menu gives real checkout (products are not managed by this skill) |
| Cara beli: WhatsApp, marketplace (Shopee/Tokopedia), atau checkout di situs? | hero/cta `ctaUrl`, footer links |
| Pengiriman dari kota mana? Ongkir, COD, retur? | `faq` |
| Promo yang sedang berjalan (dengan tanggal berakhir)? | site `banner` (chrome), `countdown` if there is a real end date |

### Klinik / kesehatan / kecantikan
| Ask | Goes to |
|---|---|
| Layanan / tindakan dan tarifnya? | `features-grid`, `pricing` |
| Dokter / terapis: nama, gelar, spesialisasi, jadwal praktik, foto? | `team` (members: name, role, bio) |
| Izin / nomor STR / akreditasi yang boleh ditampilkan? | `text-block` or `features-grid` — only exactly as given |
| Cara reservasi (WhatsApp, telepon, form)? | `form` (reservation) or cta to WhatsApp |
| BPJS / asuransi yang diterima? | `faq` |

### Perusahaan (PT/CV, B2B, manufaktur, distributor)
| Ask | Goes to |
|---|---|
| Profil singkat: tahun berdiri, bidang, visi-misi? | `text-block` (Tentang), hero |
| Direksi / manajemen yang ingin ditampilkan (nama, jabatan, foto)? | `team` |
| Legalitas: nama resmi (PT/CV), NIB, sertifikasi (ISO, SNI, halal)? | `businessProfile.legalName`, `text-block`/`features-grid` — exactly as given |
| Produk / layanan utama dan spesifikasinya? | `features-grid`, `products`, `pricing` |
| Klien atau mitra (boleh disebut + logo)? Angka pencapaian (proyek, tahun, kapasitas)? | `logo-cloud`, `stats` — only their numbers |
| Kantor / pabrik / cabang? | `map`, `contact` |

### Sekolah / kursus / pelatihan
| Ask | Goes to |
|---|---|
| Program / kelas: nama, jenjang, durasi, biaya? | `features-grid`, `pricing` |
| Fasilitas? Foto? | `features-grid`, `gallery` |
| Pengajar (nama, bidang, foto)? | `team` |
| Jadwal & cara pendaftaran, periode penerimaan? | `steps`, `form` (pendaftaran), `countdown` for a real deadline |
| Akreditasi / izin? Prestasi siswa? | `text-block`, `stats` — as given |

### Personal (freelancer, kreator, profesional, portofolio)
| Ask | Goes to |
|---|---|
| Nama, profesi, satu kalimat tentang Anda, foto diri? | hero `split-profile` variant (headline = name), `siteType: "personal"` |
| Karya / proyek terbaik (judul, foto, tautan)? | `gallery` |
| Layanan yang ditawarkan dan tarif (kalau mau ditampilkan)? | `features-grid`, `pricing` |
| Pengalaman / klien / penghargaan? | `text-block`, `logo-cloud`, `stats` — as given |
| Cara dihubungi / booking? | `contact`, `form` |

### Event (seminar, konser, pernikahan, pameran)
| Ask | Goes to |
|---|---|
| Nama acara, tanggal & jam, lokasi? | hero, `countdown` (`targetDate` ISO), `map` |
| Susunan acara / pembicara? | `steps` (rundown), `team` (speakers) |
| Tiket: kategori & harga, cara daftar? | `pricing`, `form` or cta |
| Sponsor (logo, boleh ditampilkan)? | `logo-cloud` |
| FAQ (dress code, parkir, anak-anak)? | `faq` |

## Before you build: the confirmation

Show, in the user's language:

```
Situs: Kopi Senja — kopi-senja.wpage.id (belum tayang)
Beranda: Hero (foto kedai) · Menu andalan (6 menu, harga dari Anda) · Kata pelanggan (2 ulasan dari Anda) · FAQ (4) · Ajakan pesan via WhatsApp
Menu: Daftar lengkap 14 item
Kontak: Alamat + peta + jam buka + WhatsApp
Belum ada (dikosongkan): foto tim, promo
```

Anything they correct, correct before `create_site`.
