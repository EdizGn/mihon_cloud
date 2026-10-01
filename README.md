# mihon_cloud — Cihazlar Arası Kütüphane ve Okuma Senkronizasyonu

Amaç: Bir cihazda kütüphaneye manga eklendiğinde diğerinde de görünsün, bir cihazda
okuduğun yer (okunan bölümler, sayfa ilerlemesi, geçmiş) diğerine aktarılsın.

Mihon'un kendisinde bu özellik **yok**; Mihon yalnızca elle yedekleme/geri yükleme
yapabiliyor. Bu yüzden Mihon tabanlı **[Komikku](https://github.com/komikku-app/komikku)**
fork'unu kullanıyoruz. Komikku, Mihon'un tüm özelliklerine ek olarak yerleşik bir
**Eşitleme (Sync)** sistemi içeriyor ve Mihon yedeklerini doğrudan açabiliyor.

---

## Nasıl çalışıyor?

Her eşitlemede uygulama kütüphanenin bir yedeğini oluşturur ve seçtiğin hizmetteki
(Google Drive / WebDAV / SyncYomi) uzak kopyayla **birleştirir (merge)**:

- **Manga** kaynak + URL ile eşleştirilir. Birinde olup diğerinde olmayan manga eklenir.
- **Bölümler** tek tek karşılaştırılır; hangi cihazdaki değişiklik daha yeniyse
  (okundu işareti, kaldığın sayfa, yer imi) o kazanır.
- Sonuç uzak hizmete geri yüklenir ve yerel kütüphaneye uygulanır.

Yani iki cihazda farklı mangalar okusan bile hiçbir ilerleme ezilmez, birleştirilir.

---

## 1. Kurulum (her iki cihazda)

1. Komikku'nun son sürümünü indir:
   <https://github.com/komikku-app/komikku/releases/latest>
   (Cihazın için genelde `arm64-v8a` APK'sı doğru seçimdir.)
2. Komikku, Mihon'dan farklı bir paket adı kullanır; ikisi aynı anda kurulu kalabilir.
3. Kullandığın **eklentileri (kaynakları) her iki cihaza da kur**. Eşitleme mangaları
   kaynağa göre eşleştirir; bir cihazda olmayan kaynağın mangası orada açılamaz.

## 2. Mevcut Mihon kütüphaneni taşı (yalnızca ana cihazda)

1. **Mihon**: `Daha fazla → Ayarlar → Veri ve depolama → Yedek oluştur`
2. **Komikku**: `Daha fazla → Ayarlar → Veri ve depolama → Yedeği geri yükle`
   → Mihon'dan aldığın `.tachibk` dosyasını seç.

> İkinci cihazı **boş bırak** (ya da aynı yedeği oraya da yükle). Eşitleme
> ilk çalıştığında ikisini zaten birleştirecek.

## 3. Eşitleme hizmetini seç

`Daha fazla → Ayarlar → Veri ve depolama → Eşitleme → Hizmet`

Üç seçenek var; **her iki cihazda da aynı hizmeti ve aynı hesabı** kullanmalısın.

| Hizmet | Ne gerekir | Kimler için |
|---|---|---|
| **Google Drive** (önerilen) | Sadece Google hesabı | Sunucuyla uğraşmak istemeyenler |
| **WebDAV** | Nextcloud, Synology, Koofr vb. bir WebDAV adresi | Zaten bir bulut depolaması olanlar |
| **SyncYomi** | Kendi sunucunda Docker | Tüm veriyi kendinde tutmak isteyenler |

### a) Google Drive

1. `Hizmet → Google Drive`
2. `Giriş yap` → Google hesabını seç ve izin ver.
   (Uygulama yalnızca kendi oluşturduğu dosyaya erişir.)
3. Diğer cihazda **aynı Google hesabıyla** aynı adımları yap.

### b) WebDAV

1. `Hizmet → WebDAV`
2. Sunucu adresi, kullanıcı adı, parola ve veri klasörünü gir
   (örn. Nextcloud için `https://bulutum.com/remote.php/dav/files/KULLANICI/`).
3. Diğer cihazda **aynı bilgileri** gir.

### c) SyncYomi (kendi sunucun)

1. Sunucunda bu depodaki [`syncyomi/docker-compose.yml`](syncyomi/docker-compose.yml)
   dosyasıyla başlat:
   ```sh
   cd syncyomi
   docker compose up -d
   ```
2. Tarayıcıdan `http://SUNUCU-IP:8282` adresine gir, hesap oluştur ve bir
   **API anahtarı** üret.
3. Uygulamada `Hizmet → SyncYomi`, `Sunucu` alanına `http://SUNUCU-IP:8282`
   yaz; `API anahtarı` alanına anahtarı yapıştır veya QR kodunu tara.
4. Diğer cihazda **aynı API anahtarını** kullan. Farklı anahtar = farklı kullanıcı,
   veriler birbirini görmez.

> IP ile bağlanıyorsan sunucudaki `config/config.toml` içinde `host = "0.0.0.0"`
> olmalı. Ev dışından erişmek için bir ters vekil sunucu (reverse proxy) + HTTPS
> ya da Tailscale gibi bir VPN önerilir.

## 4. Neyin eşitleneceğini seç

`Eşitleme → Nelerin eşitleneceğini seçin`

İstediğin şey için şunların açık olması yeterli:

- **Kütüphane girdileri** – eklediğin/çıkardığın mangalar
- **Kategoriler**
- **Bölümler** – okundu durumu ve kaldığın sayfa
- **Geçmiş** – "en son okunanlar" listesi
- **Takip** (AniList/MAL kullanıyorsan)

Uygulama ayarlarını eşitlemek istemiyorsan (her cihazın kendi okuyucu ayarları kalsın
istiyorsan) **Ayarlar** bölümünü kapalı bırak.

## 5. Otomatik eşitlemeyi aç

`Eşitleme → Eşitleme tetikleyicileri oluştur` altında önerilen ayarlar:

- ✅ **Bölüm okununca eşitle** → bir bölümü bitirdiğinde kaldığın yer hemen gönderilir
- ✅ **Uygulama başlatıldığında eşitle**
- ✅ **Uygulamaya geri dönüldüğünde eşitle** → diğer cihazı elinde aldığında güncel olur
- ⬜ Bölüm açılınca eşitle (isteğe bağlı; çok sık ağ trafiği yapar)

Ek olarak `Otomatik Eşitleme → Eşitleme sıklığı` için **30 dakika** veya **1 saat**
seçebilirsin; uygulama arka plandayken de eşitler.

## 6. İlk eşitleme

1. **Ana cihazda** (kütüphanenin dolu olduğu): `Şimdi eşitle`
   → kütüphane buluta yüklenir.
2. **İkinci cihazda**: `Şimdi eşitle` → **iki kez** çalıştır.
   - İlk seferde, cihazda zaten manga varsa yalnızca kendi verisini buluta
     birleştirir.
   - İkinci seferde birleştirilmiş kütüphaneyi indirir.
3. Artık bir cihazda manga ekleyip/okuyunca, diğer cihaz uygulama açıldığında veya
   bir sonraki eşitlemede güncellenir.

---

## 7. Tablet: yan yana çift sayfa

Mihon'da olmayan, Komikku'da bulunan bir özellik: sayfalı okuyucuda iki sayfayı
yan yana gösterme (kitap gibi açık sayfa).

**Ayarlama:** Okuyucuda bir bölüm aç → alttaki ⚙️ (ayarlar) → **Okuma modu** sekmesi
→ **Sayfa düzeni**:

| Seçenek | Davranış |
|---|---|
| Tek sayfa | Her zaman tek sayfa |
| İkili sayfalar | Her zaman iki sayfa yan yana |
| **Otomatik (ekran yönüne göre)** ✅ | Yatayda iki sayfa, dikeyde tek sayfa |

**Telefon + tablet için "Otomatik" seçeneğini öneririm.** Ayarları eşitlesen bile
telefonda dikey tutunca tek sayfa, tablette yatay tutunca çift sayfa görürsün.

Faydalı ek ayarlar (`Ayarlar → Okuyucu → Kopya Ayarları` ve okuyucu ayarları):

- **İkili sayfaları ters çevir** – sağdan sola (manga) okurken sayfa sırası yanlış
  görünüyorsa aç.
- **Bir sayfa kaydır** – kapak/tek sayfa yüzünden çiftler kaymışsa (sol sayfa sağda
  görünüyorsa) çiftleri bir sayfa kaydırır.
- **Alt Okuyucu Butonları** → **Sayfa düzeni** butonunu ekle; okurken tek dokunuşla
  tek/çift sayfa arasında geçersin.
- **Orta boşluk şekli** – katlanabilir cihazlarda iki sayfanın arasına boşluk koyar.

> Çift sayfa yalnızca **sayfalı** okuma modlarında (Soldan sağa / Sağdan sola)
> çalışır; Webtoon / dikey kaydırma modunda geçerli değildir.

---

## Sorun giderme

| Belirti | Çözüm |
|---|---|
| Manga diğer cihazda görünmüyor | Kaynak eklentisi o cihazda kurulu mu? `Şimdi eşitle`'yi tekrar çalıştır. |
| "No data found on remote server" | İlk eşitlemeyi önce dolu cihazdan yap. |
| İlerleme gelmiyor | `Nelerin eşitleneceğini seçin` içinde **Bölümler** ve **Geçmiş** açık mı? |
| Google Drive girişi bozuldu | `Eşitleme verisini Google Drive'dan temizle` → yeniden `Giriş yap`. |
| Her şeyi sıfırlamak istiyorum | Uzak veriyi temizle, ana cihazdan yeniden `Şimdi eşitle`. |

**İpucu:** Eşitlemeye geçmeden önce Mihon'dan aldığın `.tachibk` yedeğini sakla;
bir şey ters giderse oradan geri dönebilirsin.

---

## Alternatif: Sadece okuma ilerlemesi (Mihon'da kalmak istersen)

Mihon'u bırakmak istemiyorsan, **yalnızca okunan bölüm sayısını** eşitleyebilirsin:
`Ayarlar → Takip` altında aynı AniList/MyAnimeList hesabıyla her iki cihazda giriş
yap ve her mangayı takibe ekle. Ancak bu yöntem kütüphaneyi (yeni eklenen mangaları)
ve sayfa düzeyindeki ilerlemeyi **eşitlemez**; tam çözüm için Komikku gerekir.
