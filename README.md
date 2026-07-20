## Önce karar: Windows 11 mi, Windows 10 mu?

**İlk tercih: Windows 11 Pro 64-bit + SSD/NVMe disk.**

* Microvisioneer yazılımı **yalnızca 64-bit Windows** üzerinde çalışıyor. ([Google Sites][1])
* Windows 10’un normal güvenlik desteği **14 Ekim 2025’te sona erdi**. Bu nedenle internete veya hastane ağına bağlanacak yeni bir sistemde Windows 11 daha doğru seçimdir. ([Microsoft Support][2])
* Ancak **kameranın Windows 11 sürücüsü bulunamazsa**, önce eski sistemle aynı Windows sürümünü kurmak gerekebilir. Bu durumda Windows 10 yalnızca cihazı çalıştırmak için, mümkünse ağdan izole edilmiş bir yedek seçenek olmalıdır.

Yeni disk gerçekten mekanik HDD olacaksa, bunun yerine **en az 1 TB SSD** takılmasını öneririm. Whole-slide dosyaları tipik olarak yüzlerce MB, yüksek büyütmelerde 2 GB veya üzerine çıkabiliyor; tarama yazılımı da disk ve işlemciyi yoğun kullanıyor. ([Microvisioneer][3])

# Uygulanacak doğru sıra

## 1. Eski diski sökmeden önce mümkünse kayıt alın

Bilgisayar hâlâ açılıyorsa teknik personele şunları yaptırın:

1. `manualWSI` veya `mvSlide` programını açın.
2. `Help → About / Hakkında` ekranının fotoğrafını çekin.
3. Program sürümünü kaydedin.
4. Kamera seçim ekranının fotoğrafını çekin.
5. Preset açılır listesindeki bütün objektif ayarlarını görüntüleyin:

   * 4×
   * 10×
   * 20×
   * 40×
   * HE, İHK gibi ayrı profiller varsa bunlar
6. Her preset seçiliyken ekran görüntüsü alın:

   * Exposure/Brightness
   * Gain
   * Red/Blue gain
   * çözünürlük
   * tarama kalitesi
7. Aygıt Yöneticisi’nde kamerayı bulun ve şunları kaydedin:

   * marka-model
   * donanım kimliği
   * sürücü sağlayıcısı
   * sürücü sürümü
   * sürücü tarihi
8. Kamera üzerindeki fiziksel etiketin fotoğrafını çekin:

   * model
   * seri numarası
   * USB 2/USB 3 bağlantısı

Bunlar alınabiliyorsa kurtarma çok kolaylaşır.

---

## 2. Eski diski yalnızca yedek olarak koruyun

Eski diske:

* format atılmamalı,
* `CHKDSK /F` çalıştırılmamalı,
* “Diski onar”, “başlat” veya “initialize” denmemeli,
* üzerine yeni dosya yazılmamalı.

En güvenlisi, diskin ayrıca **tam disk imajının veya birebir klonunun** alınmasıdır. Bir diski çalışma kopyası, diğerini dokunulmamış ana yedek olarak tutun.

BitLocker şifresi sorarsa diski formatlamayın; kurumun BitLocker kurtarma anahtarı bulunmalıdır.

---

## 3. Kurtarma klasörünü oluşturun

Yeni bilgisayarda örneğin:

```text
D:\MVSlide_Kurtarma\
├── 01_Eski_Disk_Imagi
├── 02_Program_Kurulum
├── 03_Lisans
├── 04_Kamera_Surucusu
├── 05_Ayarlar_ve_Presetler
├── 06_Ekran_Goruntuleri
├── 07_Eski_Taramalar
└── 08_Test_Taramalari
```

Eski diskte bulunan dosyaları doğrudan çalıştırmak yerine önce bu klasörlere **kopyalayın**.

---

## 4. Eski Windows sürümünü belirleyin

Eski disk harici olarak bağlandığında örneğin `E:` harfini aldıysa, Yönetici Komut İstemi açarak:

```bat
reg load HKLM\OLD_SOFTWARE "E:\Windows\System32\config\SOFTWARE"
```

Ardından:

```bat
reg query "HKLM\OLD_SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v ProductName
```

```bat
reg query "HKLM\OLD_SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v DisplayVersion
```

Bitince:

```bat
reg unload HKLM\OLD_SOFTWARE
```

Böylece eski sistemin Windows 7, 10 veya 11 olup olmadığı anlaşılır. Windows, çevrimdışı registry hive dosyalarının bu yöntemle yüklenmesini destekler. ([Microsoft Learn][4])

---

## 5. Eski diskte Microvisioneer dosyalarını bulun

Eski diskteki Windows kullanıcılarını kontrol edin:

```text
E:\Users\
```

Eski kullanıcı adı örneğin `Patoloji` ise şu klasörlerin tamamını kopyalayın:

```text
E:\Users\Patoloji\AppData\Local\Microvisioneer
```

```text
E:\Users\Patoloji\AppData\Roaming\Microvisioneer
```

```text
E:\Users\Patoloji\AppData\Local\Apps\2.0
```

```text
E:\ProgramData\Microvisioneer
```

```text
E:\Program Files\Microvisioneer
```

```text
E:\Program Files (x86)\Microvisioneer
```

Özellikle `AppData\Local\Apps\2.0` önemlidir. Eski manualWSI sürümleri ClickOnce ile yalnızca oturum açmış kullanıcıya kurulabiliyordu; program dosyaları ve kullanıcı ayarları klasik `Program Files` yerine kullanıcı profilinde bulunabilir. Microvisioneer ayrıca her Windows kullanıcısının ayrı ayarları olduğunu belirtiyor. ([Google Sites][5])

Aramada şu ifadeleri kullanın:

```text
manualWSI.exe
mvSlide.exe
Microvisioneer
*.lic22
*.lic20
*.lic18
*.license
user.config
portable_settings
settings
preset
setup.exe
```

### PowerShell ile toplu arama

Yönetici PowerShell:

```powershell
Get-ChildItem "E:\" -Recurse -Force -ErrorAction SilentlyContinue |
Where-Object {
    $_.Name -match "manualWSI|mvSlide|Microvisioneer|\.lic22$|\.lic20$|\.lic18$|\.license$|user\.config"
} |
Select-Object FullName, Length, LastWriteTime |
Out-File "D:\MVSlide_Kurtarma\microvisioneer_dosyalari.txt"
```

Arama uzun sürebilir; ancak eski diske yazmaz.

---

## 6. Lisans dosyasını bulun

En sık beklenen konumlardan biri:

```text
E:\Users\<eski-kullanıcı>\AppData\Local\Microvisioneer\manualWSI\license
```

Taşınabilir kurulum kullanıldıysa:

```text
...\portable_settings\license
```

Bulduğunuz lisans dosyasını değiştirmeden kopyalayın.

Dosya uzantısı hangi sürümün kurulacağını gösterir:

| Lisans uzantısı | Önce denenmesi gereken yazılım |
| --------------- | ------------------------------ |
| `.lic22`        | manualWSI 2022                 |
| `.lic20`        | manualWSI 2021                 |
| `.lic18`        | manualWSI 2019                 |
| `.license`      | manualWSI 2017                 |

Microvisioneer’ın eski resmî kurulum sayfası bu sürümlerin kurulum paketlerini lisans uzantısına göre ayrı ayrı listeliyor ve kurulum sonrasında lisans dosyasının seçilmesini istiyor. ([Google Sites][6])

**İlk denemede güncel mvSlide’ı değil, lisansla eşleşen eski manualWSI sürümünü kurun.** Eski lisans güncel mvSlide sürümünde kabul edilmeyebilir.

---

## 7. Eski kamera sürücülerini dışarı aktarın

Eski Windows diski `E:` ise, Yönetici PowerShell’de:

```powershell
New-Item -ItemType Directory -Path "D:\MVSlide_Kurtarma\04_Kamera_Surucusu\Tum_Eski_Suruculer" -Force
```

Ardından:

```powershell
Export-WindowsDriver `
  -Path "E:\" `
  -Destination "D:\MVSlide_Kurtarma\04_Kamera_Surucusu\Tum_Eski_Suruculer"
```

Bu komut eski çevrimdışı Windows kurulumundaki üçüncü taraf sürücüleri dışarı aktarır. ([Microsoft Learn][7])

Ancak bu dosyalar yalnızca `.inf` tabanlı sürücülerdir. Kamera için ayrıca gerekli olabilecek:

* kamera SDK’sı,
* üreticinin görüntüleme programı,
* lisans bileşeni,
* USB servisleri

eski `Program Files`, `ProgramData`, `Downloads` ve kurulum klasörlerinden ayrıca kopyalanmalıdır.

---

## 8. Kamera markasını belirleyin

Microvisioneer eski ve güncel sürümlerde farklı kamera üreticilerini destekliyor. Bunlar arasında:

* Basler
* IDS Imaging
* Lumenera/Infinity
* PointGrey/FLIR
* Olympus/Evident DP serileri
* Zeiss Axiocam

bulunuyor. Ancak destek yalnızca markaya değil, kamera sensörüne ve modele bağlı. ([Microvisioneer][3])

Eski diskte şu klasör ve kelimeleri arayın:

```text
Basler
Pylon
IDS
uEye
Lumenera
Infinity
PointGrey
FLIR
FlyCapture
Olympus
Evident
cellSens
Zeiss
Axiocam
Touptek
Tucsen
Motic
```

Kamera modeli belli olmadan işletim sistemi kurulumu kesinleştirilmemelidir.

---

# Windows seçimi için karar tablosu

| Durum                                                          | Kurulacak sistem                                               |
| -------------------------------------------------------------- | -------------------------------------------------------------- |
| Kamera üreticisinin Windows 11 x64 sürücüsü var                | **Windows 11 Pro 64-bit**                                      |
| Kamera Windows 11’de üretici yazılımında canlı görüntü veriyor | **Windows 11 ile devam**                                       |
| Sadece Windows 10 x64 sürücüsü var                             | Önce Win11’de sürücüyü test edin; çalışmazsa Win10 yedek planı |
| Çok eski, imzasız sürücü kullanıyor                            | Eski Windows sürümünü yeniden oluşturmak gerekebilir           |
| Kamera modeli bilinmiyor                                       | Önce eski disk ve fiziksel etiketten model tespit edilmeli     |

Benim önerim:

> **Önce Windows 11 Pro 64-bit kurun; eski HDD’yi kesinlikle bozmadan saklayın. Kamera Windows 11’de çalışmazsa, aynı bilgisayara ikinci bir SSD üzerinde eski Windows ortamı kurulabilir.**

---

## 9. Windows 11 kurulumundan sonra yapılacaklar

Yeni sistemde şu sıra izlenmeli:

1. Windows 11 Pro 64-bit kurulmalı.
2. Anakart/chipset sürücüleri kurulmalı.
3. USB 3 denetleyici sürücüleri kurulmalı.
4. Windows Update tamamlanmalı.
5. `mvSlide` adlı özel bir yerel Windows kullanıcısı oluşturulmalı.
6. Program her zaman bu kullanıcıyla çalıştırılmalı.
7. Güç planında:

   * Uyku kapatılmalı.
   * USB seçmeli askıya alma kapatılmalı.
   * Tarama sırasında bilgisayarın ekranı veya diski kapanmamalı.
8. Kamera henüz takılmadan üreticinin kamera paketi kurulmalı.
9. Bilgisayar yeniden başlatılmalı.
10. Kamera tercihen eski sistemde kullanılan aynı USB porta takılmalı.

Microvisioneer’ın resmî sırası da önce **kamera sürücüsünün**, sonra tarama yazılımının kurulmasıdır. ([Google Sites][6])

---

## 10. Kamerayı mvSlide’dan önce test edin

Kamerayı önce üreticisinin kendi programında açın:

* Basler ise Pylon Viewer
* Lumenera ise Infinity Analyze
* IDS ise uEye Cockpit
* Olympus ise cellSens
* Zeiss ise ZEN/Axiocam bileşeni
* FLIR ise FlyCapture veya SpinView

Şunlar doğrulanmalı:

* Kamera Aygıt Yöneticisi’nde hatasız görünüyor.
* Canlı görüntü geliyor.
* Çözünürlük seçilebiliyor.
* Pozlama değiştirilebiliyor.
* Gain değiştirilebiliyor.
* Görüntü donmuyor.
* USB bağlantısı kesilmiyor.

**Üretici programında canlı görüntü yoksa manualWSI/mvSlide kurulumuna geçmeyin.**

Eski sürücüyü elle yüklemek gerekirse:

```bat
pnputil /add-driver "D:\MVSlide_Kurtarma\04_Kamera_Surucusu\Tum_Eski_Suruculer\*.inf" /subdirs /install
```

Windows, PnPUtil ile `.inf` sürücü paketlerinin alt klasörlerle birlikte eklenmesini ve uygun cihaza kurulmasını destekler. ([Microsoft Learn][8])

Ancak üreticinin tam kurulum paketi varsa onu kullanmak daha güvenlidir.

---

## 11. Doğru manualWSI sürümünü kurun

Sıra:

1. Lisans dosyasının uzantısını belirleyin.
2. Buna karşılık gelen manualWSI kurulumunu indirin.
3. Kurulumu `mvSlide` adlı Windows kullanıcısıyla yapın.
4. Program ilk kez açıldığında lisans dosyasını gösterin.
5. Lisans kabul edilene kadar güncel mvSlide sürümüne geçmeyin.

Eski Microvisioneer yazılımı kullanıcı bazlı ClickOnce kurulum kullandığından, başka bir Windows hesabıyla kurarsanız ayarlar görünmeyebilir. ([Google Sites][5])

---

## 12. Lisansı geri yükleyin

Program lisansı kendisi sorarsa yedeklediğiniz lisans dosyasını seçin.

Sormaz veya lisansı görmezse, programı kapatıp şu klasörü kontrol edin:

```text
C:\Users\mvSlide\AppData\Local\Microvisioneer\manualWSI\license
```

Klasör yoksa oluşturun ve lisans dosyasını buraya kopyalayın.

Portable kurulumda:

```text
<program-klasörü>\portable_settings\license
```

Aynı kamera kullanılmalıdır. Lisansın kamera seri numarasına bağlı olma ihtimali bulunduğundan, farklı bir kamera ile ilk test yapılmamalıdır.

---

## 13. Objektif presetlerini geri yükleyin

Burada en güvenli yöntem:

1. Programı bir kez açın.
2. Kamerayı seçin.
3. Programı kapatın.
4. Yeni oluşan Microvisioneer ayar klasörünü yedekleyin.
5. Eski ve yeni klasör yapısını karşılaştırın.
6. Aynı program sürümüyse eski:

   * `settings`
   * `presets`
   * `user.config`
   * kamera konfigürasyon dosyaları

yeni kullanıcı klasörüne kopyalanabilir.

Bütün eski `AppData` klasörünü körlemesine yeni sisteme yapıştırmayın. Öncelikle yalnızca Microvisioneer’a ait klasörleri kopyalayın.

Presetler geri gelmezse manuel oluşturulur. Microvisioneer’da her objektif ve hatta her boya için ayrı profil kaydedilebilir; örneğin:

```text
10x 0.25 NA HE
20x 0.40 NA HE
40x 0.65 NA HE
20x IHK
```

Yeni preset, `+` düğmesiyle oluşturulup isim girildikten sonra Enter ile kaydedilir. ([Google Sites][9])

---

## 14. Optik ve kamera kalibrasyonu

Ayarların geri gelmesi tek başına yeterli değildir. Şu sıra izlenmeli:

1. Mikroskop lambasını en az 5 dakika ısıtın.
2. Işığın mümkün olan en büyük kısmını kameraya yönlendirin.
3. İlgili objektifi çevirin.
4. Dokuyu okülerden netleyin.
5. Kamera görüntüsünde tekrar ince netlik yapın.
6. Gain’i mümkün olduğunca düşük tutun.
7. Beyaz alanda histogramın sağ tarafının kesilmediğini kontrol edin.
8. Kırmızı, yeşil ve mavi histogram tepelerini birbirine yaklaştırın.
9. Preseti kaydedin.
10. Aynı işlemi her objektif için tekrarlayın.
11. Kondansör açıklığını kaydedin.
12. Kamera rotasyonunu kontrol edin.

Microvisioneer’ın resmî kurulum sırası; tarama parametrelerini ayarlama, kondansörü hizalama ve kamera rotasyonunu ince ayarlamayı içeriyor. ([Google Sites][10])

---

## 15. Test taraması

İlk testte büyük rezeksiyon değil, küçük bir biyopsi kullanın.

Her objektif için kontrol edin:

* Canlı görüntü akıcı mı?
* Lam sağa hareket ederken görüntü doğru yönde genişliyor mu?
* Dikiş çizgileri oluşuyor mu?
* Görüntü parçaları üst üste doğru oturuyor mu?
* Renkler doğal mı?
* Hareket sırasında bulanıklık oluyor mu?
* Program çöküyor mu?
* Dosya başarıyla kaydediliyor mu?
* Kaydedilen `.svs` veya `.tiff` tekrar açılabiliyor mu?

Önce:

```text
10x küçük alan
```

ardından:

```text
20x küçük alan
```

son olarak:

```text
20x tam küçük biyopsi
```

taraması yapın.

---

## 16. Sistem çalışınca son yedek

Sistem tamamen çalıştığında şu dosyaları tek arşivde saklayın:

```text
MVSlide_Tam_Kurulum_Yedegi
├── manualWSI_kurulum_dosyasi
├── lisans_dosyasi
├── kamera_surucu_paketi
├── kamera_SDK_ve_goruntuleme_programi
├── Microvisioneer_ayar_klasoru
├── objektif_preset_ekran_goruntuleri
├── kamera_model_ve_seri_numarasi
├── Windows_surumu
├── test_tarama
└── kurulum_notlari.txt
```

Ayrıca yeni SSD’nin tam sistem imajını alın. Böylece daha sonraki bir arızada bütün işlemi yeniden yapmak yerine sistemi doğrudan geri yükleyebilirsiniz.

## En kritik dört kural

1. **Eski HDD’ye hiçbir şey yazmayın.**
2. **Lisans uzantısıyla eşleşen eski manualWSI sürümünü önce deneyin.**
3. **Kamera üreticisinin programında canlı görüntü almadan manualWSI kurmayın.**
4. **Windows 11’i tercih edin; fakat kamera sürücüsü uyumsuzsa eski Windows ortamını yedek plan olarak koruyun.**

[1]: https://sites.google.com/a/microvisioneer.com/manual-wsi/compatibility/computer "Manual WSI - Computer"
[2]: https://support.microsoft.com/en-US/Windows/Deployment/Updates-Lifecycle/windows-10-support-has-ended-on-october-14-2025?utm_source=chatgpt.com "Windows 10 support has ended on October 14, 2025"
[3]: https://www.microvisioneer.com/faq "FAQ | Microvisioneer"
[4]: https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/reg-load?utm_source=chatgpt.com "reg load | Microsoft Learn"
[5]: https://sites.google.com/a/microvisioneer.com/manual-wsi/setup/install-software/advanced-installation-information "Manual WSI - Advanced Installation Information"
[6]: https://sites.google.com/a/microvisioneer.com/manual-wsi/setup/install-software "Manual WSI - Install Software"
[7]: https://learn.microsoft.com/en-us/powershell/module/dism/export-windowsdriver?view=windowsserver2025-ps&utm_source=chatgpt.com "Export-WindowsDriver (Dism) | Microsoft Learn"
[8]: https://learn.microsoft.com/en-us/windows-hardware/drivers/devtest/pnputil-command-syntax?utm_source=chatgpt.com "PnPUtil Command Syntax - Windows drivers | Microsoft Learn"
[9]: https://sites.google.com/a/microvisioneer.com/manual-wsi/setup/adjust-and-save-scanning-parameters "Manual WSI - Adjust and save scanning parameters"
[10]: https://sites.google.com/a/microvisioneer.com/manual-wsi/setup?utm_source=chatgpt.com "Manual WSI - Setup - Google Sites"
