# Kısım 8: Raporlar — Klasik Listeler ve ALV Grid

*ABAP'ın bir SELECT sorgusunu her SAP kullanıcısının beklediği çıktı ekranına nasıl dönüştürdüğü — ve gerçek iş hayatında biletlerin neden yarısının burada olduğu.*

---

## ☕ ABAP raporu gerçekte nedir?

Aklınıza gelebilecek en basit ve işe yarar programı düşünün: kullanıcı birkaç filtre alanını dolduruyor, bir düğmeye basıyor ve sonuçların bulunduğu bir tablo ekrana geliyor. .NET'te bu bir MVC action'ı ve sorgu formu + sonuç görünümü olabilir. Python'da ise `argparse` + `pandas` `to_string()` çağrısı içeren bir CLI betiği olabilir.

SAP'ta bu kalıbın özel bir adı vardır: **ABAP raporu**. 1990'ların başından bu yana SAP çıktısının bel kemiğidir ve — evet — ilk haftanızda bir tanesini bakıma alacaksınız. Onu yöneten olay döngüsünü anlamak, her şeyin kilidini açar.

---

## 8.1 Rapor Olay Modeli

### 1️⃣ Benzetme

Bir ABAP raporu, görünmez bir **olay dağıtıcısının** içinde çalışır. Çalışma zamanı (runtime), adlandırılmış olayları sabit bir sırayla tetikler; siz de ihtiyaç duyduklarınıza kancalanırsınız. Bir olayı atlarsanız, çalışma zamanı onu sessizce geçer — yalnızca sizin kodunuz çalışır.

Bunu ASP.NET'in istek hattı ara yazılımına (request pipeline middleware) benzetebilirsiniz; ancak tamamen senkron, ekran odaklıdır ve `app.Use(...)` yerine anahtar kelime etiketleriyle tanımlanır.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# konsol uygulaması yaşam döngüsü benzetmesi
static void Main(string[] args)
{
    // INITIALIZATION — varsayılan değerleri ayarla
    string cargoClass = args.FirstOrDefault() ?? "Y";

    // AT SELECTION-SCREEN — doğrula
    if (cargoClass.Length != 1) { Console.Error.WriteLine("Hatalı giriş"); return; }

    // START-OF-SELECTION — işi yap
    var flights = FlightRepo.Query(cargoClass);

    // END-OF-SELECTION — çıktıyı tamamla
    foreach (var f in flights) Console.WriteLine(f);
}
```

```python
# Python karşılığı
import argparse

def main():
    # INITIALIZATION
    parser = argparse.ArgumentParser()
    parser.add_argument("--class", default="Y")
    args = parser.parse_args()       # AT SELECTION-SCREEN (runtime halleder)

    # START-OF-SELECTION
    flights = fetch_flights(args.__dict__["class"])

    # END-OF-SELECTION
    for f in flights:
        print(f)
```

### 3️⃣ ABAP'taki karşılığı

```abap
REPORT zmy_first_report.

" ── INITIALIZATION ──────────────────────────────────────────────
" Selection screen gösterilmeden önce bir kez çalışır.
" Parametreler için varsayılan değerleri burada ayarlayın.
INITIALIZATION.
  p_class = 'Y'.           " aşağıdaki parametreye varsayılan atama

" ── AT SELECTION-SCREEN ─────────────────────────────────────────
" Kullanıcı Execute'e tıkladıktan SONRA, mantık çalışmadan ÖNCE tetiklenir.
" Girişi doğrulamak ve MESSAGE ile temiz şekilde durdurmak için kullanın.
AT SELECTION-SCREEN.
  IF p_class NA 'YBC'.
    MESSAGE 'Kabin sınıfı Y, B veya C olmalıdır' TYPE 'E'.
  ENDIF.

" ── START-OF-SELECTION ──────────────────────────────────────────
" Ana bloğunuz. Çalışma zamanı, doğrulama başarılı olduktan sonra bunu tetikler.
" Raporların neredeyse tamamı buradadır.
START-OF-SELECTION.
  SELECT * FROM sflight
    INTO TABLE @DATA(lt_flights)
    WHERE class = @p_class.

  " ... görüntüleme mantığı buraya gelir (bölüm 8.3 / 8.4)

" ── END-OF-SELECTION ────────────────────────────────────────────
" START-OF-SELECTION'dan sonra tetiklenir. Toplamlar ve özetler için kullanın.
END-OF-SELECTION.
  WRITE: / 'Dönen satır sayısı:', lines( lt_flights ).
```

> ⚠️ **C#/Python tuzağı:** Olaylar arasında atlama için `return` veya `exit()` yoktur. `AT SELECTION-SCREEN` sırasında `MESSAGE TYPE 'E'` çalıştırıldığında *çalışma durdurulur ve selection screen yeniden gösterilir* — `START-OF-SELECTION`'a yayılan bir exception fırlatmaz. Bunu, exception değil, yerleşik bir doğrulama kısa devresi olarak düşünün.

**Tam olay sırası** (çoğu rapor yalnızca vurgulananları kullanır):

| Olay | Ne zaman tetiklenir | Benzetme |
|---|---|---|
| `LOAD-OF-PROGRAM` | Program belleğe ilk yüklendiğinde | Static constructor |
| **`INITIALIZATION`** | Selection screen render edilmeden önce | Form `Load` + varsayılanlar |
| `AT SELECTION-SCREEN OUTPUT` | Selection screen her yeniden çizildiğinde | `Paint` / yeniden render |
| **`AT SELECTION-SCREEN`** | Execute'den sonra, ana mantık öncesi | Göndermede form doğrulaması |
| **`START-OF-SELECTION`** | Ana blok — asıl iş yeri | `Main()` / request handler |
| **`END-OF-SELECTION`** | Ana bloktan sonra | Yanıt sonlandırma |
| `TOP-OF-PAGE` | Her yeni çıktı sayfasından önce | Sayfa başlığı şablonu |
| `END-OF-PAGE` | Her çıktı sayfası dolduğunda | Sayfa alt bilgi şablonu |

> 🧭 **İş hayatında:** Bakımını üstleneceğiniz raporların %95'i yalnızca `INITIALIZATION`, `AT SELECTION-SCREEN` ve `START-OF-SELECTION` kullanır. Sayfa sonu olayları (`TOP-OF-PAGE`, `END-OF-PAGE`) klasik listelere (bölüm 8.3) aittir ve yeni kod olarak nadiren yazılır.

---

## 8.2 Selection Screen — Giriş Formunuz

### 1️⃣ Benzetme

**Selection screen**, SAP'ın yerleşik giriş formudur. ABAP çalışma zamanı, kodunuzu çalıştırmadan önce onu bir diyalog olarak render eder. Hangi alanları istediğinizi siz tanımlarsınız; UI, F4 değer yardımı, çeviri ve düzeni SAP otomatik olarak halleder.

Bu, bir MVC form modeline ya da bir CLI argüman ayrıştırıcısına en çok benzer — tek farkla: çalışma zamanı, HTML'i (SAP GUI ekranını) sizin tanımlamalarınızdan kendisi oluşturur.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — basit bir sorgu formu modeli
public class FlightSearchForm
{
    [Required]
    public string AirlineCode { get; set; }

    public DateTime? DepartureDateFrom { get; set; }
    public DateTime? DepartureDateTo   { get; set; }

    [Range(1, 9999)]
    public int? MaxRows { get; set; } = 100;
}
```

```python
# Python CLI karşılığı
parser = argparse.ArgumentParser()
parser.add_argument("--airline",  required=True)
parser.add_argument("--date-low", type=lambda s: datetime.strptime(s, "%Y-%m-%d"))
parser.add_argument("--date-high", type=lambda s: datetime.strptime(s, "%Y-%m-%d"))
parser.add_argument("--max-rows", type=int, default=100)
```

### 3️⃣ ABAP'taki karşılığı

```abap
REPORT zflight_search.

" ── PARAMETERS ──────────────────────────────────────────────────
" Tek değerli bir giriş alanı. Metin kutusu / CLI argümanı gibi.
PARAMETERS:
  p_carrid TYPE sflight-carrid OBLIGATORY,  " zorunlu; DDIC'ten F4 yardımı
  p_maxrow TYPE i DEFAULT 100.

" ── SELECT-OPTIONS ──────────────────────────────────────────────
" FROM / TO aralık çifti (alt değer, üst değer, işaret, seçenek).
" İki giriş alanı + çoklu değer açılır penceresi için bir düğme olarak render edilir.
SELECT-OPTIONS:
  s_fldate FOR sflight-fldate,              " tarih aralığı; OBLIGATORY gerekmez
  s_seats  FOR sflight-seatsocc.

" ── SELECTION-SCREEN BLOCK ──────────────────────────────────────
" Alanların etrafına başlıklı görsel bir çerçeve ekler.
SELECTION-SCREEN BEGIN OF BLOCK b1 WITH FRAME TITLE TEXT-001.
  PARAMETERS p_detail AS CHECKBOX DEFAULT ' '.
SELECTION-SCREEN END OF BLOCK b1.

START-OF-SELECTION.
  SELECT carrid connid fldate seatsocc seatsmax
    FROM sflight
    INTO TABLE @DATA(lt_fl)
    WHERE carrid    = @p_carrid
      AND fldate    IN @s_fldate       " IN = aralık tablosunu otomatik işler
      AND seatsocc  IN @s_seats
    UP TO @p_maxrow ROWS.
```

**`SELECT-OPTIONS` temel mekanikleri:**

Bir `SELECT-OPTIONS` alanı (yukarıdaki `s_fldate`) aslında bir **ranges tablosudur** — `SIGN`, `OPTION`, `LOW`, `HIGH` sütunlarına sahip bir tablodur. Open SQL'deki `IN` operatörü tüm bunu otomatik olarak işler.

```abap
" Aralıkları programatik olarak da oluşturabilirsiniz:
DATA lr_date TYPE RANGE OF sflight-fldate.
lr_date = VALUE #(
  ( sign = 'I' option = 'BT' low = '20240101' high = '20241231' )
  ( sign = 'E' option = 'EQ' low = '20240701' )  " 1 Temmuz'u hariç tut
).
SELECT ... WHERE fldate IN @lr_date.
```

> ⚠️ **C#/Python tuzağı:** `SELECT-OPTIONS` basit bir `WHERE field BETWEEN low AND high` değildir. Dahil etmeleri (`sign = 'I'`), hariç tutmaları (`sign = 'E'`), kalıpları (`option = 'CP'`, LIKE gibi) ve aynı anda birden fazla aralığı destekleyen çok satırlı bir aralık tablosudur. Bu güç, onu gerçek raporlarda her yerde görmemizin nedenidir.

> 🧭 **İş hayatında:** Bir kullanıcı "birden fazla tesis koduna göre filtrelemem ve ikisini hariç tutmam gerekiyor" dediğinde, yanıt `WERKS` üzerinde bir `SELECT-OPTIONS`'tır. Özel çoklu değer giriş mantığına başvurmayın — çalışma zamanı bunu zaten yapıyor.

---

## 8.3 Klasik WRITE Listeleri — Hâlâ Karşılaşacağınız Çıktı

### 1️⃣ Benzetme

Grid'ler ve HTML'den önce, SAP raporları sanal bir sayfaya yazdırıyordu — ekranda görünen bir nokta vuruşlu yazıcı çıktısı gibi. `WRITE` ifadesi, ABAP çıktısının `Console.Write`'ıdır. Sonuç, sabit sütunlar, sayfa sonları ve araç çubuğuyla birlikte kaydırılabilir bir "liste" olur.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — biçimlendirilmiş konsol çıktısı
Console.WriteLine($"{"Havayolu",-8} {"Uçuş",6} {"Tarih",-12} {"Koltuk",8}");
Console.WriteLine(new string('-', 40));
foreach (var f in flights)
    Console.WriteLine($"{f.Carrid,-8} {f.Connid,6} {f.Fldate,-12} {f.Seatsocc,8}");
```

```python
# Python — biçimlendirilmiş yazdırma
print(f"{'Havayolu':<8} {'Uçuş':>6} {'Tarih':<12} {'Koltuk':>8}")
print("-" * 40)
for f in flights:
    print(f"{f['carrid']:<8} {f['connid']:>6} {f['fldate']:<12} {f['seatsocc']:>8}")
```

### 3️⃣ ABAP'taki karşılığı

```abap
START-OF-SELECTION.
  SELECT carrid connid fldate seatsocc seatsmax
    FROM sflight
    INTO TABLE @DATA(lt_fl)
    WHERE carrid = @p_carrid.

  " ── Başlık satırı (TOP-OF-PAGE'de tetiklenir) ──────────────────
  " Ayrı tanımlanır, aşağıya bakın.

  LOOP AT lt_fl INTO DATA(ls_fl).
    WRITE: /  ls_fl-carrid,       " / = yeni satır
              ls_fl-connid,
              ls_fl-fldate,
          (8) ls_fl-seatsocc,     " (8) = sütun genişliği
              ls_fl-seatsmax.
    " Dolu uçuşları vurgula
    IF ls_fl-seatsocc >= ls_fl-seatsmax.
      FORMAT COLOR COL_NEGATIVE INTENSIFIED ON.
      WRITE: / '*** DOLU ***'.
      FORMAT COLOR OFF INTENSIFIED OFF.
    ENDIF.
  ENDLOOP.

TOP-OF-PAGE.
  WRITE: / 'Havayolu', 10 'Bağlantı', 18 'Tarih', 26 'Dolu', 36 'Max'.
  ULINE.   " yatay çizgi
```

**Yaygın WRITE değiştiricileri:**

| Söz dizimi | Etki | C# karşılığı |
|---|---|---|
| `WRITE: /` | Çıktıdan önce yeni satır | `Console.WriteLine` |
| `WRITE: (8) field` | Sabit sütun genişliği | `{field,8}` |
| `WRITE: 20 field` | 20. sütunda çıktı | Sekme durağı |
| `FORMAT COLOR COL_POSITIVE` | Yeşil arka plan | Konsol rengi |
| `ULINE` | Yatay çizgi | `new string('-', n)` |

> ⚠️ **C#/Python tuzağı:** Klasik listeler **salt okunur çıktıdır**. `WRITE` listesinde bir hücreyi düzenlenebilir yapamaz veya düğme ekleyemezsiniz. Etkileşimli çıktı için — sıralama, filtreleme, düzenleme — ALV'ye ihtiyacınız var (sonraki bölüm). Klasik listeler hâlâ varlığını sürdürüyor çünkü hızlıdır, yazdırılabilirdir ve her SAP kullanıcısı nasıl kullanacağını bilir.

> 🧭 **İş hayatında:** Her eski sistemde klasik `WRITE` raporlarına rastlayacaksınız. Muhtemelen yeni tane *yazmayacaksınız*, ancak "bir sütun ekle" veya "negatif stok için rengi değiştir" gibi isteklerle karşılaşacaksınız — bu yüzden söz dizimini bilmek hemen karşılık verir.

---

## 8.4 ALV — Her SAP Kullanıcısının Beklediği Grid

### 1️⃣ Benzetme

ALV (ABAP List Viewer), SAP'ın yerleşik `DataGridView` / `DataFrame` görüntüleme bileşenidir. Her SAP kullanıcısı sıralamayı, filtrelemeyi, toplamları, Excel'e aktarmayı ve sütun düzenini yapılandırmayı bilir — çünkü ALV tüm bunları bedava sunar. Sizin işiniz yalnızca internal tabloyu sağlamaktır; gerisini ALV halleder.

İki ALV API'si vardır:

| API | Yaş | Karar |
|---|---|---|
| `REUSE_ALV_GRID_DISPLAY` (function module) | Klasik, OO öncesi | Bunu *okuyacaksınız*; yeni yazmayın |
| `CL_SALV_TABLE` (OO factory) | Modern (7.0+) | Bunu yazın. Temiz, test edilebilir. |

Yeni kodlarda yalnızca `CL_SALV_TABLE` kullanacağız.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — WinForms formunda DataGridView
var grid = new DataGridView { DataSource = flightList };
grid.Columns["Carrid"].HeaderText = "Havayolu";
grid.Columns["Seatsocc"].DefaultCellStyle.Alignment = DataGridViewContentAlignment.MiddleRight;
form.Controls.Add(grid);
form.ShowDialog();
```

```python
# Python — pandas tabulate karşılığı
import pandas as pd
from tabulate import tabulate

df = pd.DataFrame(flights)
df.rename(columns={"carrid": "Havayolu", "seatsocc": "Dolu"}, inplace=True)
print(tabulate(df, headers="keys", tablefmt="grid", showindex=False))
```

### 3️⃣ ABAP'taki karşılığı — CL_SALV_TABLE temelleri

```abap
" Minimal CL_SALV_TABLE kullanım kalıbı
TRY.
    DATA lo_alv TYPE REF TO cl_salv_table.

    cl_salv_table=>factory(
      IMPORTING
        r_salv_table = lo_alv
      CHANGING
        t_table      = lt_flights    " internal tablonuz
    ).

    " İsteğe bağlı: sütunları yapılandır
    DATA(lo_cols) = lo_alv->get_columns( ).
    lo_cols->set_optimize( abap_true ).   " otomatik genişlik

    " Görüntüle — ekranı devralır
    lo_alv->display( ).

  CATCH cx_salv_msg INTO DATA(lx).
    MESSAGE lx->get_text( ) TYPE 'E'.
ENDTRY.
```

Hepsi bu. Üç adım: tablonuzu doldurun, `factory`'yi çağırın, `display`'i çağırın. Kullanıcı, ekstra UI kodu olmadan tam sıralama/filtreleme/Excel dışa aktarma özelliklerini elde eder.

---

## 8.5 Tam Çalışan ALV Raporu — SE38'e Yapıştırın ve Çalıştırın

Aşağıda, `SE38`'e kopyalayabileceğiniz, herhangi bir `Z` paketinde oluşturabileceğiniz ve `SFLIGHT` demo tablolarına sahip (her ABAP deneme/eğitim sisteminde standarttır) bir sistemde hemen çalıştırabileceğiniz eksiksiz, bağımsız bir ALV raporu bulunmaktadır.

### Önce C#/Python karşılığı

```csharp
// C# — "uçuşları getir ve bir grid yazdır"
public class FlightRow
{
    public string Airline    { get; init; }
    public string Connection { get; init; }
    public DateTime Date     { get; init; }
    public decimal Price     { get; init; }
    public string Currency   { get; init; }
    public int SeatsOccupied { get; init; }
    public int SeatsMax      { get; init; }
    public double LoadFactor { get; init; }
}

var rows = db.Flights
    .Where(f => f.CarrId == carrierInput && f.FlDate >= dateFrom)
    .OrderBy(f => f.FlDate)
    .Select(f => new FlightRow
    {
        Airline       = f.CarrId,
        Connection    = f.ConnId,
        Date          = f.FlDate,
        Price         = f.Price,
        Currency      = f.Currency,
        SeatsOccupied = f.SeatsOcc,
        SeatsMax      = f.SeatsMax,
        LoadFactor    = f.SeatsMax > 0 ? (double)f.SeatsOcc / f.SeatsMax : 0
    })
    .ToList();

DataGrid.DataSource = rows;  // framework grid'i render eder
```

```python
# Python karşılığı
import pandas as pd

df = pd.read_sql(
    "SELECT carrid, connid, fldate, price, currency, seatsocc, seatsmax "
    "FROM sflight WHERE carrid = %s AND fldate >= %s ORDER BY fldate",
    con, params=[carrier, date_from]
)
df["load_pct"] = (df["seatsocc"] / df["seatsmax"] * 100).round(1)
print(df.to_string(index=False))
```

### Tam ABAP raporu

```abap
*&---------------------------------------------------------------------*
*& Report  ZDEMO_FLIGHT_ALV
*& Yazar   : Kariyer Geçişi Alıştırmaları
*& Amaç    : Tam ALV raporu — SFLIGHT verisi, CL_SALV_TABLE, sütun yapılandırması
*&---------------------------------------------------------------------*
REPORT zdemo_flight_alv.

"─────────────────────────────────────────────────────────────────────
" 1. TÜR TANIMI — çıktı satırlarımızın şekli
"─────────────────────────────────────────────────────────────────────
TYPES: BEGIN OF ty_flight,
         carrid    TYPE sflight-carrid,
         connid    TYPE sflight-connid,
         fldate    TYPE sflight-fldate,
         price     TYPE sflight-price,
         currency  TYPE sflight-currency,
         seatsocc  TYPE sflight-seatsocc,
         seatsmax  TYPE sflight-seatsmax,
         load_pct  TYPE p LENGTH 5 DECIMALS 1,   " hesaplanan: % doluluk
       END OF ty_flight.

DATA gt_flights TYPE STANDARD TABLE OF ty_flight WITH EMPTY KEY.

"─────────────────────────────────────────────────────────────────────
" 2. SELECTION SCREEN
"─────────────────────────────────────────────────────────────────────
SELECTION-SCREEN BEGIN OF BLOCK b1 WITH FRAME TITLE TEXT-001.
  PARAMETERS:
    p_carrid TYPE sflight-carrid DEFAULT 'LH',
    p_maxrow TYPE i              DEFAULT 200.
  SELECT-OPTIONS:
    s_fldate FOR sflight-fldate.
SELECTION-SCREEN END OF BLOCK b1.

"─────────────────────────────────────────────────────────────────────
" 3. INITIALIZATION — varsayılan tarih aralığı ayarla (son 2 yıl)
"─────────────────────────────────────────────────────────────────────
INITIALIZATION.
  s_fldate-low  = cl_abap_context_info=>get_system_date( ) - 730.
  s_fldate-high = cl_abap_context_info=>get_system_date( ).
  APPEND s_fldate.

"─────────────────────────────────────────────────────────────────────
" 4. GİRİŞ DOĞRULAMA
"─────────────────────────────────────────────────────────────────────
AT SELECTION-SCREEN.
  IF p_carrid IS INITIAL.
    MESSAGE 'Bir havayolu kodu girin (örn. LH, AA, UA)' TYPE 'E'.
  ENDIF.
  IF p_maxrow <= 0 OR p_maxrow > 9999.
    MESSAGE 'Maksimum satır sayısı 1 ile 9999 arasında olmalıdır' TYPE 'E'.
  ENDIF.

"─────────────────────────────────────────────────────────────────────
" 5. ANA MANTIK
"─────────────────────────────────────────────────────────────────────
START-OF-SELECTION.

  " 5a. SFLIGHT'tan veri getir
  SELECT carrid, connid, fldate, price, currency, seatsocc, seatsmax
    FROM sflight
    INTO CORRESPONDING FIELDS OF TABLE @gt_flights
    WHERE carrid = @p_carrid
      AND fldate IN @s_fldate
    ORDER BY fldate ASCENDING
    UP TO @p_maxrow ROWS.

  IF sy-subrc <> 0.
    MESSAGE |{ p_carrid } havayolu için uçuş bulunamadı| TYPE 'I'.
    RETURN.
  ENDIF.

  " 5b. Doluluk oranı sütununu hesapla (DB'de saklanmıyor)
  LOOP AT gt_flights ASSIGNING FIELD-SYMBOL(<ls_fl>).
    IF <ls_fl>-seatsmax > 0.
      <ls_fl>-load_pct = ( <ls_fl>-seatsocc / <ls_fl>-seatsmax ) * 100.
    ENDIF.
  ENDLOOP.

  " 5c. Görüntüleme metoduna aktar
  lcl_display=>show( gt_flights ).

"─────────────────────────────────────────────────────────────────────
" 6. YEREL SINIF — görüntüleme mantığını ayrı tutar (iyi alışkanlık)
"─────────────────────────────────────────────────────────────────────
CLASS lcl_display DEFINITION.
  PUBLIC SECTION.
    CLASS-METHODS show
      IMPORTING it_data TYPE STANDARD TABLE.
ENDCLASS.

CLASS lcl_display IMPLEMENTATION.
  METHOD show.
    DATA lo_alv  TYPE REF TO cl_salv_table.
    DATA lo_cols TYPE REF TO cl_salv_columns_table.
    DATA lo_col  TYPE REF TO cl_salv_column_table.
    DATA lo_funs TYPE REF TO cl_salv_functions_list.
    DATA lo_disp TYPE REF TO cl_salv_display_settings.
    DATA lo_sort TYPE REF TO cl_salv_sorts.

    TRY.
        " ── ALV örneği oluştur ───────────────────────────────────
        cl_salv_table=>factory(
          IMPORTING r_salv_table = lo_alv
          CHANGING  t_table      = it_data
        ).

        " ── Araç çubuğu: tüm standart düğmeler (sırala, filtrele, Excel) ──
        lo_funs = lo_alv->get_functions( ).
        lo_funs->set_all( abap_true ).

        " ── Görüntüleme ayarları ─────────────────────────────────
        lo_disp = lo_alv->get_display_settings( ).
        lo_disp->set_list_header( 'Uçuş Doluluk Raporu' ).
        lo_disp->set_striped_pattern( abap_true ).  " zebra satırlar

        " ── Sütun yapılandırması ─────────────────────────────────
        lo_cols = lo_alv->get_columns( ).
        lo_cols->set_optimize( abap_true ).

        " Başlıkları yeniden adlandır (DDIC adları → insan tarafından okunabilir)
        lo_col ?= lo_cols->get_column( 'CARRID' ).
        lo_col->set_long_text( 'Havayolu' ).

        lo_col ?= lo_cols->get_column( 'CONNID' ).
        lo_col->set_long_text( 'Bağlantı' ).

        lo_col ?= lo_cols->get_column( 'FLDATE' ).
        lo_col->set_long_text( 'Uçuş Tarihi' ).

        lo_col ?= lo_cols->get_column( 'PRICE' ).
        lo_col->set_long_text( 'Bilet Fiyatı' ).

        lo_col ?= lo_cols->get_column( 'SEATSOCC' ).
        lo_col->set_long_text( 'Dolu Koltuklar' ).

        lo_col ?= lo_cols->get_column( 'SEATSMAX' ).
        lo_col->set_long_text( 'Toplam Koltuk' ).

        lo_col ?= lo_cols->get_column( 'LOAD_PCT' ).
        lo_col->set_long_text( 'Doluluk %' ).

        " ── Varsayılan sıralama: en yeni uçuşlar önce ───────────
        lo_sort = lo_alv->get_sorts( ).
        lo_sort->add_sort(
          columnname = 'FLDATE'
          order      = if_salv_c_sort_order=>descending
        ).

        " ── ÇALIŞTIR! ────────────────────────────────────────────
        lo_alv->display( ).

      CATCH cx_salv_msg cx_salv_not_found cx_salv_data_error INTO DATA(lx).
        MESSAGE lx->get_text( ) TYPE 'E'.
    ENDTRY.
  ENDMETHOD.
ENDCLASS.
```

> 💡 **`lcl_display` neden yerel sınıftır?** Sunum mantığını bir sınıf içinde tutmak (yerel bile olsa), görüntüleme metodunu bağımsız olarak test etmenizi ve değiştirmenizi sağlar. Alternatif — her şeyi `START-OF-SELECTION`'a tıkmak — tam olarak devraldığınız ve temizlemeniz istenen bakımı zor kod türüdür.

> 🧭 **İş hayatında:** Gerçek bir projede `lcl_display` kalıbı, tıklama/çift tıklama olayları için `IF_SALV_EVENTS_ACTIONS_TABLE`'ı implemente eden `ZCL_FLIGHT_REPORT` gibi tam bir global sınıfa dönüşür. Mülakatçılar "ALV satırında çift tıklamayı nasıl yönetirsiniz?" sorusunu çok sever — yanıt, o interface'i implemente etmek ve `lo_alv->get_event( )` üzerinde bir handler kaydetmektir.

**Raporu çalıştırma:**

1. `SE38`'i açın, program adını `ZDEMO_FLIGHT_ALV` olarak girin, **Oluştur**'a tıklayın.
2. Kodu yapıştırın, program tipini **Executable Program** olarak ayarlayın, `$TMP` paketi atayın.
3. `F8` (Execute) tuşuna basın. Selection screen belirir; tekrar **Execute**'a basın.
4. Sıfır ekstra UI kodu ile sıralama, filtreleme ve Excel dışa aktarma özellikli tam bir ALV grid elde edersiniz.

---

## 🧠 Özet

| Kavram | Nedir | Zihinsel modeliniz |
|---|---|---|
| `INITIALIZATION` | Ekran öncesi kurulum | Form `Load` / argüman varsayılanları |
| `AT SELECTION-SCREEN` | Gönderim sonrası doğrulama | Model doğrulama / argparse hatası |
| `START-OF-SELECTION` | Ana mantık | `Main()` / request handler |
| `PARAMETERS` | Tek giriş alanı | CLI argümanı / metin girişi |
| `SELECT-OPTIONS` | Aralık girişi (çoklu değer) | QueryString aralığı + çoklu seçim |
| `WRITE` | Klasik metin çıktısı | Sütun düzeniyle `Console.WriteLine` |
| `CL_SALV_TABLE` | Modern ALV grid | DataGridView / pandas `to_string` |

Rapor olay modeli, bir kez gördükten sonra sıralı ve basittir. Gerçek beceri, *kodunuzu hangi olaya koyacağınızı* bilmektir — ve neredeyse her şey için `START-OF-SELECTION`'ı varsayılan olarak kullanmak.

---

*[← İçindekiler](../content.md) | [← Önceki: Modern ABAP Söz Dizimi](07-modern-abap-syntax.md) | [Sonraki: Module Pool (Dialog) Programlama →](09-module-pool-programming.md)*
