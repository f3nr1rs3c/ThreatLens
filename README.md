# 🛡️ Sysmon IR Konfigürasyonu (Low Noise, High Signal)

Bu repo, **Incident Response (IR)** ve **tehdit tespiti** odaklı optimize edilmiş bir Sysmon konfigürasyonu içerir.

Amaç:

* 🔍 Saldırgan davranışlarını görünür kılmak
* ⚡ Gürültüyü (noise) minimumda tutmak
* 🎯 Gerçek dünya saldırı tekniklerine odaklanmak

---

# 📌 Özellikler

## ✅ Process İzleme

Şüpheli süreçleri tespit eder:

* PowerShell, CMD, rundll32, mshta
* Script engine’leri (wscript, cscript)
* LOLBins (Living-off-the-land binary’ler)

Ayrıca şunları yakalar:

* Encoded komutlar (`-enc`)
* `Invoke-Expression`, `DownloadString`, `IEX`

---

## 🌐 Network İzleme

Şu süreçlerin dış bağlantılarını loglar:

* PowerShell
* CMD
* rundll32
* mshta

➡️ Tespit edilebilecekler:

* Reverse shell
* C2 (Command & Control) trafiği

---

## 📂 Dosya Oluşturma İzleme

Yüksek riskli dizinlere odaklanır:

* `AppData`
* `Temp`

Takip edilen dosyalar:

* `.exe`
* `.dll`

➡️ Kullanım:

* Dropper tespiti
* Malware staging

---

## 🔑 Registry Persistence Tespiti

Şu anahtarları izler:

* `Run`
* `RunOnce`
* `Policies`
* `Winlogon`

➡️ Tespit:

* Kalıcılık (persistence)
* Arka kapı (backdoor)

---

## 🧠 Process Injection Tespiti

Şüpheli erişim haklarını yakalar:

* `0x1F`
* `0x1FFFFF`

➡️ Tespit:

* Credential dumping
* Process injection teknikleri

---

## 🚗 Driver Yükleme İzleme

Tüm driver yüklemelerini loglar:

➡️ Kullanım:

* Rootkit
* Kernel-level saldırılar

---

## 📦 DLL İzleme (Image Load)

Tüm DLL yüklemelerini loglar.

⚠️ Not:
Bu özellik:

* Çok güçlü görünürlük sağlar
* Ancak yüksek gürültü üretir

➡️ Ortama göre optimize edilmesi önerilir

---

# ⚙️ Kurulum

## 1. Sysmon İndir

Microsoft Sysinternals üzerinden indir.

## 2. Konfigürasyonu Uygula

Eğer Sysmon kurulu değilse:

```powershell id="m1n4og"
.\sysmon64.exe -i config.xml
```

Eğer zaten kuruluysa:

```powershell id="d9z2ue"
.\sysmon64.exe -c config.xml
```

> ⚠️ PowerShell’i **Yönetici olarak** çalıştır

---

# 📊 Log Konumu

Olay Görüntüleyicisi yolu:

```id="slm2rj"
Uygulama ve Hizmet Günlükleri
→ Microsoft
→ Windows
→ Sysmon
→ Operational
```

---

# 🎯 Tespit Senaryoları

Bu konfig ile şunları tespit edebilirsin:

* 🧨 PowerShell saldırıları
* 🎭 LOLBins kullanımı
* 🔁 Persistence mekanizmaları
* 🌐 C2 trafiği
* 🧬 Process injection
* 📦 Malware dropper

---

# ⚠️ Önemli Notlar

* Bu config:

  * Detection odaklıdır
  * Minimal log için değildir

* Ortama göre:

  * Whitelist eklenmeli
  * Gürültü azaltılmalı

* Yüksek log üreten ortamlarda:

  * SIEM entegrasyonu önerilir

---

# 🚀 Önerilen Geliştirmeler

Daha profesyonel kullanım için:

* SIEM entegrasyonu (Wazuh, Elastic, Splunk)
* Threat intelligence entegrasyonu
* Alert/detection rule yazımı

---

# 🤝 Katkı Sağlama

Katkılarınızı bekliyoruz:

* Detection geliştirme
* Noise azaltma
* Yeni saldırı teknikleri ekleme

Pull request gönderebilirsiniz.

---

# 📄 Lisans

MIT License

---

# ⭐ Faydalı olduysa

Projeye ⭐ vererek destek olabilirsin.
