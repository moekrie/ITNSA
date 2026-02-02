Tentu, ini adalah panduan konfigurasi BIND9 yang sudah saya rapikan ke dalam format Markdown agar lebih mudah dibaca dan disalin.

---

# Konfigurasi BIND9 DNS Server

Berikut adalah langkah-langkah instalasi dan konfigurasi DNS Server menggunakan BIND9.

### 1. Instalasi Paket

Langkah pertama, instal aplikasi BIND9 dan tools pendukungnya:

```bash
apt update && apt install bind9 bind9utils dnsutils -y

```

---

### 2. Konfigurasi Options

Edit file `/etc/bind/named.conf.options` untuk mengatur hak akses query dan forwarder.

```bash
nano /etc/bind/named.conf.options

```

**Isi Konfigurasi:**

```bind
options {
    directory "/var/cache/bind";
    recursion yes;

    // Tentukan network yang diizinkan melakukan query
    allow-query { 
        127.0.0.1;
        192.168.10.0/24;
        192.168.20.0/24; 
    };

    // IP address server yang mendengarkan request
    listen-on { 192.168.10.10; };
    
    dnssec-validation no;
};

```

---

### 3. Konfigurasi Zone Lokal

Daftarkan domain Anda di file `/etc/bind/named.conf.local`.

```bash
nano /etc/bind/named.conf.local

```

**Isi Konfigurasi:**

```bind
zone "lab.local" {
    type master;
    file "/etc/bind/zones/db.lab.local";
};

```

---

### 4. Pembuatan File Database Zone

Sebelum mengedit, pastikan folder `zones` sudah ada, lalu buat file database-nya.

```bash
mkdir -p /etc/bind/zones
nano /etc/bind/zones/db.lab.local

```

**Isi Konfigurasi:**

```bind
$TTL 86400
@ IN SOA dns.lab.local. admin.lab.local. (
    2026020101 ; Serial
    3600       ; Refresh
    1800       ; Retry
    604800     ; Expire
    86400 )    ; Negative Cache TTL

; Name Server
@    IN NS    dns.lab.local.

; Alamat IP (A Record)
dns  IN A     192.168.1.10
mail IN A     192.168.1.20
web1 IN A     192.168.1.30
web2 IN A     192.168.1.31

; Mail Exchange & Alias
@    IN MX 10 mail.lab.local.
www  IN CNAME web1.lab.local.

```

---

### 5. Verifikasi dan Restart

Setelah selesai, jangan lupa cek konfigurasi dan restart layanan:

* **Cek Syntax:** `named-checkconf`
* **Restart Service:** `systemctl restart bind9`

> **Catatan Kecil:** Saya perhatikan di `named.conf.options` Anda menggunakan network **192.168.10.0**, tapi di file zone IP-nya diarahkan ke **192.168.1.x**. Pastikan IP tersebut selaras dengan topologi jaringan yang Anda gunakan ya!

---

Apakah Anda ingin saya buatkan juga konfigurasi untuk **Reverse DNS** (agar IP bisa diterjemahkan kembali ke nama domain)?
