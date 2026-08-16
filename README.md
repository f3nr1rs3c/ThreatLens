# 🛡️ Sysmon Threat Detection Configuration

<div align="center">

![Sysmon](https://img.shields.io/badge/Sysmon-4.90-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Windows-0078D6?style=for-the-badge&logo=windows11&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Maintenance](https://img.shields.io/badge/Maintained-Yes-success?style=for-the-badge)

**Gürültüyü azaltan, sinyali önceliklendiren, saldırgan davranışlarına odaklanan
üretime hazır bir Sysmon tespit yapılandırması.**

[Kurulum](#-kurulum) • [Kapsanan Teknikler](#-kapsanan-teknikler) • [Olay Kuralları](#-olay-kuralları) • [MITRE ATT&CK Eşleştirmesi](#-mitre-attck-eşleştirmesi) 

</div>

---

## 📋 Genel Bakış

Bu repo, [Sysmon (System Monitor)](https://learn.microsoft.com/sysinternals/downloads/sysmon) için
şema sürümü **4.90** ile yazılmış, üretim ortamlarında kullanılabilecek düzeyde ayarlanmış bir
yapılandırma dosyası (`config.xml`) içerir.

Yapılandırmanın temel felsefesi şudur:

- **Gürültüyü azalt** — Bilinen, güvenilir sistem/uygulama davranışlarını (`onmatch="exclude"`)
  loglamadan çıkar.
- **Sinyali önceliklendir** — Saldırgan davranışına özgü, MITRE ATT&CK teknikleriyle eşleşen
  olayları (`onmatch="include"`) yakala.
- **Bağlamı koru** — Her kural grubu, neyi neden yakaladığını açıklayan yorum satırlarıyla belgelenmiştir.

Sonuç: SIEM/SOC ekiplerinin boğulmadan, gerçek tehdit göstergelerine odaklanabileceği bir olay akışı.

> ⚠️ **Not:** Bu yapılandırma yalnızca **savunma amaçlı** tespit ve izleme içindir. Kurallar,
> saldırgan araç ve tekniklerini *engellemek* için değil, *görünür kılmak* için tasarlanmıştır.

---

## ✨ Özellikler

- 🔍 **31 farklı Sysmon olay türü** için özel filtreleme kuralı
- 🧩 **Include / Exclude** kural ayrımıyla dengelenmiş gürültü seviyesi
- 🎯 **LOLBin (Living off the Land)** araçlarının kötüye kullanımına odaklı tespit
- 🕵️ **Kimlik bilgisi hırsızlığı** (LSASS erişimi, SAM/NTDS.dit, tarayıcı credential store'ları)
- 🐚 **C2 framework imzaları** (Cobalt Strike, Meterpreter named pipe desenleri)
- 🔐 **Kalıcılık (persistence)** mekanizmaları (Registry Run key'leri, WMI subscriptions, IFEO, servisler)
- 🧬 **Process injection & tampering** (CreateRemoteThread, hollowing, doppelganging)
- 💣 **Ransomware davranış göstergeleri** (uzantı değişiklikleri, fidye notları, shadow copy silme)
- 📡 **DNS tunneling / C2 beacon** tespiti (script motorlarının anormal DNS sorguları)
- 🧾 **Hash & imza doğrulama** (MD5/SHA1/SHA256/IMPHASH + sertifika iptal kontrolü)

---

## 🚀 Kurulum

### 1. Sysmon'u indirin

```powershell
# Sysinternals paketinden indirin
Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "Sysmon.zip"
Expand-Archive Sysmon.zip -DestinationPath .\Sysmon
```

### 2. Bu depoyu klonlayın

```bash
git clone https://github.com/f3nr1rs3c/ThreatLens.git
cd ThreatLens
```

### 3. Sysmon'u yapılandırmayla birlikte kurun

```powershell
# İlk kurulum
.\Sysmon\Sysmon64.exe -accepteula -i config.xml

# Zaten kuruluysa yalnızca yapılandırmayı güncelleyin
.\Sysmon\Sysmon64.exe -c config.xml
```

### 4. Kurulumu doğrulayın

```powershell
# Servis durumunu kontrol et
Get-Service sysmon64

# Aktif yapılandırma şemasını görüntüle
.\Sysmon\Sysmon64.exe -c
```

### 5. Olayları görüntüleyin

Olaylar şurada toplanır:

```
Applications and Services Logs → Microsoft → Windows → Sysmon → Operational
```

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 20
```

---

## 🧱 Genel Ayarlar

| Ayar | Değer | Açıklama |
|---|---|---|
| `schemaversion` | `4.90` | Sysmon şema sürümü |
| `HashAlgorithms` | `MD5,SHA1,SHA256,IMPHASH` | Dosyalar için hesaplanan hash türleri; `IMPHASH` PE import tablosu üzerinden malware ailesi tanımlamada kullanılır |
| `CheckRevocation` | `True` | İmzalı işlemlerin sertifika iptal durumunu doğrular |

---

## 📡 Olay Kuralları

Aşağıdaki tablo, yapılandırmada tanımlı her kural bloğunu ve amacını özetler.

| # | Sysmon Olayı | Mod | Amaç |
|---|---|---|---|
| 1 | `ProcessCreate` | Exclude | Defender, SearchIndexer, WerFault gibi bilinen sistem gürültüsünü filtreler |
| 2 | `FileCreateTime` | Include | **Timestomping** (dosya zaman damgası manipülasyonu) tespiti |
| 3a | `NetworkConnect` | Exclude | Chrome/Firefox/Edge/svchost gibi güvenilir uygulamaların ağ trafiğini filtreler |
| 3b | `NetworkConnect` | Include | Bilinen C2 portları (4444, 1337, 9001/9050-Tor vb.) ve script motorlarının ağ bağlantıları |
| 4 | `ProcessTerminate` | Include | Kısa ömürlü LOLBin/script motoru süreçlerinin sonlanma davranışı |
| 5 | `DriverLoad` | Exclude | Microsoft/Windows imzalı sürücüleri filtreler; imzasız/3. parti sürücüler loglanır |
| 6 | `ImageLoad` | Include | Şüpheli dizinlerden DLL yükleme, `amsi.dll` yüklemesi (AMSI bypass göstergesi) |
| 7 | `CreateRemoteThread` | Exclude | Defender'ın meşru injection işlemlerini filtreler; diğer tüm remote thread olayları kaydedilir |
| 8 | `RawAccessRead` | Include | Ham disk erişimi — LSASS okuma / disk forensics araçları |
| 9 | `ProcessAccess` | Include | **LSASS erişimi** — Mimikatz/Cobalt Strike imzalı `GrantedAccess` bayrakları |
| 10 | `FileCreate` | Include | Şüpheli konum + uzantı kombinasyonlarında dropper davranışı |
| 11 | `RegistryEvent` | Include | **Kalıcılık** — Run/RunOnce, Winlogon, IFEO, COM hijacking, AppInit_DLLs, Defender/PowerShell politika değişiklikleri |
| 12 | `FileCreateStreamHash` | Include | **NTFS Alternate Data Stream (ADS)** — Mark-of-the-Web bypass tespiti |
| 13 | `PipeEvent` | Include | Bilinen **C2 named pipe** imzaları (Cobalt Strike, Meterpreter) |
| 14 | `WmiEvent` | Include | Fileless kalıcılık için tüm WMI olay abonelikleri |
| 15a | `DnsQuery` | Exclude | Microsoft/Google/Cloudflare/Azure gibi meşru alan adlarını filtreler |
| 15b | `DnsQuery` | Include | Script motorları/LOLBin'lerin anormal DNS sorguları (tunneling/beacon göstergesi) |
| 16 | `FileDelete` | Include | Araç temizleme, log silme (`.evtx`), Volume Shadow Copy silme göstergeleri |
| 17 | `ClipboardChange` | Include | Script motorlarının pano erişimi — clipboard hijacking / keylogger davranışı |
| 18 | `ProcessTampering` | Include | Process hollowing / doppelganging tespiti (genel kapsam) |
| 19 | `FileDeleteDetected` | Include | Kritik dosya silme — `.evtx`, Sysmon logları, `System32`/`SysWOW64` |
| 20 | `FileBlockExecutable` | Include | Temp/Public/ProgramData dizinlerine çalıştırılabilir dosya yazılmasını engeller |
| 21 | `FileBlockShredding` | Include | Araçların güvenli silme (shredding) ile kurtarılamaz hale getirilmesini engeller |
| 22 | `FileExecutableDetected` | Include | Kullanıcı yazılabilir dizinlere PE dosyası düşme anı (fileless → disk geçişi) |
| 23 | `ProcessCreate` | Include | Oturum açma ile ilişkili süreçler (`runas`, `logonui`, `winlogon`, `consent`, yüksek integrity level) |
| 24 | `ProcessCreate` | Include | Brute force / kimlik doğrulama denemeleri (`mstsc`, `ssh`, `plink`, komut satırında parola anahtar kelimeleri) |
| 25 | `ProcessCreate` | Include | **Hesap oluşturma** — `net user /add`, `New-LocalUser`, `New-ADUser`, gruba ekleme |
| 26 | `ProcessCreate` | Include | Hesap silme/devre dışı bırakma — iz örtme göstergesi |
| 27 | `ProcessCreate` | Include | Kritik grup üyeliği değişiklikleri (Domain/Enterprise/Schema Admins, Backup Operators) |
| 28 | `FileCreate` | Include | Hassas dosyalara erişim — `SAM`, `NTDS.dit`, `.kdbx`, `id_rsa`, `unattend.xml`, tarayıcı credential store'ları |
| 29 | `FileCreate` | Include | Dosya maskeleme (masquerading) ve ransomware uzantı/fidye notu göstergeleri |
| 30 | `FileCreate` / `ProcessCreate` | Include | Çıkarılabilir medya (USB) yazma ve buradan çalıştırma — veri sızdırma |
| 31 | `ProcessCreate` / `FileCreate` | Include | SMB ağ paylaşımı erişimi ve **lateral movement** araçları (`psexec`, `wmiexec`, `smbexec`, `atexec`) |

---

## 🎯 Kapsanan Teknikler

Bu yapılandırma, saldırı yaşam döngüsünün aşağıdaki aşamalarına görünürlük sağlar:

```
Initial Access → Execution → Persistence → Privilege Escalation
      ↓               ↓            ↓                ↓
 Defense Evasion → Credential Access → Discovery → Lateral Movement
      ↓                    ↓               ↓            ↓
 Collection → Command & Control → Exfiltration → Impact
```

| Aşama | Örnek Tespitler |
|---|---|
| **Execution** | LOLBin kötüye kullanımı (`regsvr32`, `rundll32`, `mshta`, `certutil`) |
| **Persistence** | Registry Run key'leri, WMI subscriptions, zamanlanmış görevler, servis manipülasyonu |
| **Privilege Escalation** | UAC bypass göstergeleri (yüksek integrity level süreçleri), IFEO hijacking |
| **Defense Evasion** | Timestomping, ADS ile MOTW bypass, log/dosya silme, process tampering |
| **Credential Access** | LSASS erişimi, SAM/NTDS.dit dökümü, tarayıcı/KeePass credential dosyaları |
| **Discovery** | `nltest`, `net group`, `net view`, DNS keşif sorguları |
| **Lateral Movement** | SMB paylaşımı erişimi, `psexec`/`wmiexec`/`smbexec`, RDP/SSH bağlantıları |
| **Command & Control** | Bilinen C2 portları, named pipe imzaları, anormal DNS sorguları |
| **Exfiltration** | USB'ye toplu kopyalama (`robocopy`/`xcopy`), çıkarılabilir medya yazımı |
| **Impact** | Ransomware uzantı değişiklikleri, fidye notları, shadow copy silme |

---

## 🗺️ MITRE ATT&CK Eşleştirmesi

| Kural Bloğu | ATT&CK Tekniği |
|---|---|
| Timestomping Tespiti | [T1070.006](https://attack.mitre.org/techniques/T1070/006/) — Indicator Removal: Timestomp |
| LSASS Erişimi | [T1003.001](https://attack.mitre.org/techniques/T1003/001/) — OS Credential Dumping: LSASS Memory |
| Registry Persistence | [T1547.001](https://attack.mitre.org/techniques/T1547/001/) — Boot or Logon Autostart Execution |
| WMI Event Subscription | [T1546.003](https://attack.mitre.org/techniques/T1546/003/) — Event Triggered Execution: WMI |
| ADS Oluşturma | [T1564.004](https://attack.mitre.org/techniques/T1564/004/) — Hide Artifacts: NTFS File Attributes |
| Named Pipe C2 İmzaları | [T1071.001](https://attack.mitre.org/techniques/T1071/001/) — Application Layer Protocol |
| Hesap Oluşturma | [T1136.001](https://attack.mitre.org/techniques/T1136/001/) — Create Account: Local Account |
| Grup Üyeliği Değişikliği | [T1098](https://attack.mitre.org/techniques/T1098/) — Account Manipulation |
| SMB Lateral Movement | [T1021.002](https://attack.mitre.org/techniques/T1021/002/) — Remote Services: SMB/Windows Admin Shares |
| Ransomware Göstergeleri | [T1486](https://attack.mitre.org/techniques/T1486/) — Data Encrypted for Impact |
| Shadow Copy Silme | [T1490](https://attack.mitre.org/techniques/T1490/) — Inhibit System Recovery |
| DNS Tunneling / Beacon | [T1071.004](https://attack.mitre.org/techniques/T1071/004/) — Application Layer Protocol: DNS |

---

## ⚙️ Özelleştirme

Kendi ortamınıza göre uyarlarken dikkat edilmesi önerilen noktalar:

1. **Exclude listelerini genişletin** — Ortamınıza özgü meşru yazılımları (EDR ajanı, yedekleme
   yazılımı, envanter araçları) `onmatch="exclude"` bloklarına ekleyerek gürültüyü daha da azaltın.
2. **Port/uygulama listelerini gözden geçirin** — Kurumunuzda meşru olarak kullanılan RDP/SSH/WinRM
   trafiği varsa ilgili kuralları buna göre daraltın (ör. yalnızca beklenmeyen kaynak IP'lerden gelen).
3. **SIEM entegrasyonu** — Olayları Winlogbeat, NXLog veya doğrudan Windows Event Forwarding (WEF)
   ile merkezi bir SIEM'e (Splunk, Elastic, Sentinel vb.) yönlendirmeniz önerilir.
4. **Test ortamında doğrulayın** — Yapılandırmayı üretime almadan önce bir test/kabul ortamında
   çalıştırıp yanlış pozitif oranını değerlendirin.

```powershell
# Değişiklik sonrası yapılandırmayı yeniden yükleyin
.\Sysmon64.exe -c config.xml
```

---

## 📁 Depo Yapısı

```
.
├── config.xml     # Ana Sysmon yapılandırma dosyası
└── README.md      # Bu belge
```

---

## 🤝 Katkıda Bulunma

Katkılarınızı memnuniyetle karşılarız:

1. Bu repoyu forklayabilirsiniz
2. Yeni bir dal oluşturun. (`git checkout -b kural/yeni-tespit`)
3. Değişikliklerinizi commit edin. (`git commit -m "Yeni tespit kuralı eklendi"`)
4. Dalınızı push'layın (`git push origin kural/yeni-tespit`)
5. Bir Pull Request açın.

Yeni kural önerirken lütfen:
- İlgili **MITRE ATT&CK tekniğini** belirtin.
- Kuralın **neden** gerekli olduğunu açıklayan bir yorum ekleyin.
- Mümkünse **false positive** senaryolarını da not edin.

---

## ⚖️ Sorumluluk Reddi

Bu yapılandırma yalnızca **meşru güvenlik izleme, tespit mühendisliği ve savunma amaçlı** kullanım
için hazırlanmıştır. Kendi sahip olduğunuz veya izinli olarak yönettiğiniz sistemler dışında
kullanmayın.

## 📄 Lisans

Bu proje [MIT Lisansı](LICENSE) ile lisanslanmıştır.

---

<div align="center">

**⭐ Faydalı bulduysanız bir yıldız bırakmayı unutmayın, emek veriyorum sonuçta :)**

</div>
