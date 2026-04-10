# Tiyatro LaTeX Sistemi — Claude Code Devam Notları

## Proje Nedir?
XeLaTeX ile tiyatro oyunu yazmak için özel bir LaTeX sınıfı/paket sistemi.
3 dosyadan oluşuyor:
- `tiyatro.cls` — ana sınıf, book tabanlı, draft/final mod
- `tiyatro-core.sty` — persona sistemi, diyalog formatı, sahne/perde sistemi
- `tiyatro-theme.sty` — font, sayfa düzeni, kapak, satır numaraları
- `main.tex` — test dosyası

Derleme: `xelatex main.tex` (iki kez — sahne cast listeleri .aux'tan okunuyor)

## Tamamlanan Düzeltmeler

### 1. `\ifnum` hatası (ÇÖZÜLDÜ)
- **Sorun:** `\ifdraftmode` XeTeX primitive ile çakışıyordu + `.cls` dosyasında encoding sorunu vardı
- **Çözüm:** Tüm dosyalarda `\ifdraftmode` → `\iftiyatrodraft`, `\draftmodetrue` → `\tiyatrodrafttrue` olarak yeniden adlandırıldı. `tiyatro.cls` sıfırdan encoding-safe yazıldı.

### 2. Font fallback (ÇÖZÜLDÜ)
- **Sorun:** Calibri/Times New Roman her sistemde yok
- **Çözüm:** `\IfFontExistsTF` ile fallback zinciri: Calibri → TeX Gyre Heros (draft), TNR → TeX Gyre Termes (final). Rusça için DejaVu Serif fallback.

### 3. `\@rufont` already defined (ÇÖZÜLDÜ)
- **Sorun:** `\newfontfamily\@rufont` ikinci kez tanımlanınca hata
- **Çözüm:** `\AtBeginDocument` içinde `\let\@rufont\relax` sonra `\newfontfamily`

### 4. Satır numaraları karakter isimleriyle çakışma (ÇÖZÜLDÜ)
- **Sorun:** Draft modda satır numaraları diyalog isimlerinin üstüne biniyordu
- **Çözüm:** `\AtBeginDocument{\setlength\linenumbersep{\dimexpr\maxcharlength+12pt\relax}}` — linenumbersep dinamik olarak karakter adı genişliğine göre ayarlanıyor

### 5. Türkçe İ/ı scshape sorunu (ÇÖZÜLDÜ)
- **Sorun:** `\scshape` ile "ALİ" yerine "ALI" basılıyordu
- **Çözüm:** Tüm scshape font komutlarına `\addfontfeature{Language=Turkish}` eklendi (Calibri bu feature'ı destekliyor)

### 6. Kapakta yıldızlı karakter adı (ÇÖZÜLDÜ)
- **Sorun:** `\karakter*` ile tanımlanan karakterler kapakta görünmüyordu
- **Çözüm:** `\@starrednamelist` macro'su ile preamble'da starred isimler biriktiriliyor, `\maketitle` içinde basılıyor

### 7. Sahne 1 cast listesi boş (ÇÖZÜLDÜ — iki-pass)
- **Sorun:** Cast listesi "bir önceki sahnenin" bilgisini gösteriyordu, ilk sahne hep boştu
- **Çözüm:** `.aux` dosyasına `\@tiyatro@savedcast{act-scene}{id1,id2,...}` yazılıyor, sonraki derlemede okunuyor. `\sahne`, `\perde` ve `\AtEndDocument` sırasında flush yapılıyor.

### 8. Zincirleme `\aliasdo` formatı (ÇÖZÜLDÜ)
- **Sorun:** `\alido\aysedo\mustdo Girerler.` sadece "Girerler." basıyordu, isimler kayboluyordu
- **Kök neden 1:** `\csgdef{#3do}` içindeki `\if@hasbeenformatted...\fi` bloğunda `\futurelet` sonraki token olarak `\fi`'yi görüyordu
- **Kök neden 2:** `\@action@peekdecide` içindeki `\ifcat...\fi` bloğunda `formataction` çağrılınca `\fi` token'ı metne karışıyordu
- **Çözüm:**
  - `\@tiy@afterfi` pattern: `\long\def\@tiy@afterfi#1\fi{\fi#1}` — `\@action@addname` çağrısını `\fi`'nin dışına çıkarıyor
  - `\@tiy@doaction` wrapper: `\if` bloğunu ayrı bir fonksiyonda tutuyor
  - `\@action@peekdecide` içinde `\let\@tiy@peekresult` ile kararı `\fi`'den sonra çalıştırıyor

### 9. Smallcaps Türkçe İ — Perde/Sahne başlıkları (ÇÖZÜLDÜ)
- **Sorun:** `\acttitlefont` ve `\scenetitlefont`'ta `\addfontfeature{Language=Turkish}` eksikti, "BİRİNCİ" → "BIRINCI" basılıyordu
- **Çözüm:** Her iki font komutuna ve kapaktaki `\scshape`'e `\addfontfeature{Language=Turkish}` eklendi

### 10. Perde Sonu ayrı sayfaya düşüyor (ÇÖZÜLDÜ)
- **Sorun:** `\vspace` + metin sayfaya sığmayınca yeni sayfaya atılıyordu
- **Çözüm:** `\@tiyatro@openact` ve `\AtEndDocument` içinde `\nopagebreak` + esnek `\vskip` kullanıldı

### 11. Satır numarası hizalama (ÇÖZÜLDÜ)
- **Sorun:** Draft modda satır numaraları karakter isimlerine yakın kalıyordu
- **Çözüm:** `\linenumbersep` padding `12pt` → `24pt` artırıldı

### 12. Kapak sayfası yeniden tasarım (ÇÖZÜLDÜ)
- **Sorun:** Draft kapakta dev TASLAK yazısı ve yıldızlı karakter isimleri vardı
- **Çözüm:** Draft: TASLAK küçük (`\small`), başlık altında ortalanmış, yıldızlı karakter yok. Final: example PDF gibi — yıldızlı karakterler ortada kalın `\Large\bfseries`

### 13. `\aliasdo` hizalama sorunu (ÇÖZÜLDÜ)
- **Sorun:** `\aliasdo` ve `\action{}` ile oluşan stage direction metni diyalog metniyle aynı hizada değildi — çok sağda kalıyordu
- **Kök neden:** `formatdialogue`'da karakter adı `\parindent=-\maxcharlength` + minipage ile **sol margin'e** taşınır; diyalog metni sol kenardan (position 0) başlar. Eski `formataction`'da `\kern\maxcharlength` eklenmişti — bu action metnini position `\maxcharlength`'e itiyordu, yani diyalog metninden `\maxcharlength` kadar sağda.
- **Çözüm:** `formataction` ve `\action{}` içindeki `\kern\maxcharlength` kaldırıldı. Bunun yerine `\leftskip 0pt` + `\parindent 0pt` ile `\par` iç grup içinde çağrıldı:
  ```latex
  % formataction (tiyatro-core.sty):
  \vskip 0.5ex%
  {\leftskip 0pt\parindent 0pt\actionfont(text)\par}%
  \vskip 0.5ex%
  \endgroup%
  ```
- **Kritik TeX davranışı:** `\leftskip` satır kırımı sırasında (`\par` anında) kullanılır, horizontal liste oluşturulurken değil. `\par` iç grup içinde çağrılmalı ki doğru `\leftskip` değeri geçerli olsun. Kern/leftskip ile indent eklemeye gerek yok — diyalog metni de position 0'dan başlıyor.

## Bilinen Kalan Sorunlar / İyileştirmeler

### `\ali{...}` vs `\ali metin` sözdizimi (DOĞRULANDI)
- Her iki sözdizimi de çalışıyor ve aynı çıktıyı üretiyor
- `\ali{metin}` braces ile, `\ali metin` ^^M ile yakalanan — görsel fark yok
- Kod değişikliği gereksiz

## Mimari Notlar

### Persona Sistemi (tiyatro-core.sty)
- `\karakter[Tam Ad][Açıklama]{Script Adı}{alias}` — karakter tanımlama
- Her karakter için otomatik türetilen komutlar: `\alias` (diyalog), `\aliasdo` (action), `\aliasfont`
- `\@persona@makemacros` — tüm türetmeyi yapan iç fonksiyon
- Karakter ID'leri Roma rakamı: `\Roman{totalcharcounter}`

### Diyalog Mekanizması
- `\ali{metin}` veya `\ali metin` → `formatdialogue` çağrılır
- `formatdialogue` `^^M` (satır sonu) ile metni yakalar
- Sol sütun: minipage ile karakter adı, `\maxcharlength` genişliğinde
- Sağ sütun: diyalog metni

### Action/Stage Direction
- `\aliasdo metin` → `formataction` çağrılır
- Zincirleme: `\alido\aysedo\mustdo metin` → accumulator sistemi
- `\@action@addname` → `\futurelet` → `\@action@peekdecide` → `\@action@fire`
- `\@tiy@afterfi` pattern ile `\if...\fi` bloğundan güvenli çıkış
- `\action{metin}` — isimsiz sahne yönü ({} sözdizimi)
- Hizalama: `\leftskip 0pt` + `\parindent 0pt`, `\par` iç grup içinde → diyalog metniyle aynı position 0'dan başlar
- `formatdialogue`'da karakter adı sol margin'e düşer (`\parindent=-\maxcharlength` + minipage); diyalog metni ve action metni ikisi de position 0'dan başlar

### İki-Pass Cast Sistemi
- Her sahne/perde değişiminde `\@tiyatro@flushcasttoaux` çağrılır
- `.aux`'a `\@tiyatro@savedcast{I-1}{1,2}` gibi veriler yazılır
- Sonraki derlemede `\@tiyatro@showsavedcast{I-1}` ile okunur
- `\@tiy@renderids` virgüllü ID listesini isimlere çevirir

### Draft/Final Mod
- `\iftiyatrodraft` flag'i `tiyatro.cls`'de tanımlanır
- Draft: Aptos (fallback: Calibri → TeX Gyre Heros), 1.5 satır aralığı, satır numaraları, kapakta küçük "TASLAK"
- Final: Times New Roman, tek satır aralığı, satır numarası yok, temiz kapak

###TODO

- theatre.cls'den grup karakterler konseptini persona sistemine ekle

---

## Gelecek Geliştirme Planları

### Branch A — Yazma Deneyimi İyileştirmeleri

**1. Bölüm: Baskı ve Format Özellikleri**

1. **Booklet baskı modu** — A4 yatay, her yüzde 2 sayfa → arkalı önlü basımda 1 kağıtta 4 script sayfası. Ortadan zımbalanınca gerçek kitapçık. Sayfa sırası özel hesaplanır (1,8,2,7,3,6...). `pdfbook2` harici aracı veya LaTeX içi çözüm. Tiyatro prodüksiyonlarında standart format.
2. **Header/Footer** — Her sayfada: sayfa no + perde adı + taslak versiyon no.
3. **Diyalog sayfa kırılım koruması** — Karakter adı + en az 2 satır replika aynı sayfada kalmalı.
4. **Sahne başlığına yer/zaman** — `\sahne[İç, Balıkçı Çiftliği — Gece]{1}` opsiyonel argüman.
5. **`\beat` / `\sus`** — Kısa sessizlik işareti → *(Sus.)* tek satır iş.
6. **`(devam)` işareti** — Stage direction sonrası aynı karakter devam edince otomatik "ALİ (devam)".

### Branch B — Kopya Sistemleri

1. **Personal Copy** — `\personalcopy{ali}` komutu: o karakteri starred yapar, kapağa "Ali'nin Kopyası" yazar. Dışarıdan script N kez xelatex çalıştırır, her karakter için ayrı PDF.
2. **Rehearsal mode** — Starred karakterin replikaları beyaz/gizli basılır, ezber çalışması için.
3. *(Düşük öncelik)* TikZ sahne diyagramları — sahne yönetmeninin işi, çok karmaşık.