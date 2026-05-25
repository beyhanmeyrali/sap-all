# Kısım 11: ABAP Objects — OOP ve C#/Python Karşılaştırması

*Sınıf tasarımı içgüdülerinizin nihayet evinde hissettiği yer — ve birkaç güzel sürpriz.*

---

## ☕ Dürüst ilk izlenim

ABAP, nesne yönelimi gelmeden önce onlarca yıl boyunca prosedürler ve global verilerle idare etti. "ABAP Objects" 1990'ların sonunda geldi — ve bu gerçek bir OOP sistemiydi, sonradan yapıştırılmış bir eklenti değil. Class, interface, inheritance, exception, event: hepsi var.

Söz dizimi C#'tan daha sözcük yüklü, Python'dan daha açık. Hayatınızın geri kalanı boyunca `CLASS ... IMPLEMENTATION.`, `ENDCLASS.`, `METHOD ... ENDMETHOD.` yazacaksınız. Bir günün sonunda doğal gelir; bir haftanın sonunda kas belleği olur.

Başlamadan önce ihtiyacınız olan eşleştirme tablosu:

```text
   C# / Python kavramı                 ABAP karşılığı
   ─────────────────────────────     ────────────────────────────────────
   class MyClass { }                 CLASS zcl_my_class DEFINITION. ... ENDCLASS.
   public / protected / private      PUBLIC SECTION. / PROTECTED SECTION. / PRIVATE SECTION.
   : (inheritance)                   INHERITING FROM zcl_parent
   interface IFoo { }                INTERFACE zif_foo. ... ENDINTERFACE.
   abstract class                    ABSTRACT CLASS (DEFINITION içinde)
   sealed / final class              FINAL CLASS (DEFINITION içinde)
   override                          REDEFINITION (METHODS bildiriminde)
   static field / method             CLASS-DATA / CLASS-METHODS
   constructor                       CONSTRUCTOR (instance) / CLASS_CONSTRUCTOR (static)
   this.                             ME->
   throw new MyException(...)        RAISE EXCEPTION TYPE zcx_my_exception ...
   try / catch / finally             TRY / CATCH / CLEANUP
   event Action<T>                   EVENTS + RAISE EVENT / SET HANDLER
   new Foo()                         NEW zcl_foo( ) veya CREATE OBJECT
```

---

## 11.1 Class, Method, Attribute ve Görünürlük

### 1️⃣ Benzetme

Bir class, bir plan şemasıdır. ABAP'ta bu şemayı iki parçaya bölersiniz — bir `DEFINITION` (eski usul C++ düşünürsek başlık dosyası) ve bir `IMPLEMENTATION` (gövde). Her ikisi de aynı `.clas.abap` dosyasında yaşar ama net biçimde ayrılır.

Görünürlük, her üye için ayrı belirteçler yerine bölüm anahtar kelimeleriyle yönetilir. `PUBLIC SECTION.` yazarsınız, altındaki her şeyi public olarak listelersiniz; ardından `PRIVATE SECTION.` yazıp private olanları listelersiniz.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — basit bir hesap makinesi sınıfı
public class Calculator
{
    private double _lastResult;

    public double Add(double a, double b)
    {
        _lastResult = a + b;
        return _lastResult;
    }

    public double LastResult => _lastResult;
}
```

```python
# Python
class Calculator:
    def __init__(self):
        self._last_result = 0.0

    def add(self, a: float, b: float) -> float:
        self._last_result = a + b
        return self._last_result

    @property
    def last_result(self) -> float:
        return self._last_result
```

### 3️⃣ ABAP'taki karşılığı

```abap
"------------------------------------------------------------------
" ZCL_CALCULATOR — DEFINITION
"------------------------------------------------------------------
CLASS zcl_calculator DEFINITION
  PUBLIC          " tüm tüketicilere görünür
  FINAL           " alt sınıf oluşturulamaz (C# sealed gibi)
  CREATE PUBLIC.  " herkes örneklendirebilir

  PUBLIC SECTION.
    METHODS:
      add
        IMPORTING iv_a         TYPE decfloat34
                  iv_b         TYPE decfloat34
        RETURNING VALUE(rv_result) TYPE decfloat34,

      last_result
        RETURNING VALUE(rv_result) TYPE decfloat34.

  PRIVATE SECTION.
    DATA mv_last_result TYPE decfloat34.

ENDCLASS.

"------------------------------------------------------------------
" ZCL_CALCULATOR — IMPLEMENTATION
"------------------------------------------------------------------
CLASS zcl_calculator IMPLEMENTATION.

  METHOD add.
    mv_last_result = iv_a + iv_b.
    rv_result      = mv_last_result.
  ENDMETHOD.

  METHOD last_result.
    rv_result = mv_last_result.
  ENDMETHOD.

ENDCLASS.
```

> ⚠️ **C#/Python tuzağı:** ABAP'ta `RETURNING` parametresi tek bir değer döndürmenin yoludur — bir fonksiyonun dönüş değeri gibi davranır. `EXPORTING` veya `CHANGING` parametreli metodlar birden fazla değer döndürebilir (Kısım 12'de daha fazlası). `RETURNING` metodundan döndürmek için `return` anahtar kelimesi yoktur — parametre adına doğrudan atama yaparsınız (`rv_result = ...`).

**İsimlendirme kuralları** (tüm SAP ekibi bunları bekler):

| Önek | Anlamı |
|------|---------|
| `zcl_` | özel class |
| `zif_` | özel interface |
| `zcx_` | özel exception sınıfı |
| `iv_` | importing değer parametresi |
| `ev_` | exporting değer parametresi |
| `cv_` | changing değer parametresi |
| `rv_` | returning değer parametresi |
| `mv_` | member (instance) değişkeni |
| `gv_` | global değişken |
| `lv_` | yerel değişken |
| `lt_` | yerel tablo |
| `ls_` | yerel yapı (structure) |

> 🧭 **İş hayatında:** Kıdemli geliştiriciler değişken isimlerinizi mantığınızdan önce okur. Yanlış önek kullanmak (`lv_var` yerine `l_var`) "bu kişi ABAP'ı eski eğitimlerden öğrenmiş" sinyali verir. Yukarıdaki tabloyu kullanın. Clean ABAP stil rehberi (bölüm 11.6) bunu daha da pekiştirir.

---

## 11.2 Interface, Inheritance, Polimorfizm, ABSTRACT/FINAL, Redefinition

### 1️⃣ Benzetme

Interface ve inheritance tam olarak beklediğiniz gibi çalışır. ABAP'a özgü incelik: bir interface uyguladığınızda metot adları interface adıyla öneklenir; yani `area` yerine `zif_shape~area`. Tuhaf görünür; ad çakışmalarını ortadan kaldırır.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# — interface + inheritance
public interface IShape
{
    double Area();
    double Perimeter();
}

public abstract class Shape : IShape
{
    public abstract double Area();
    public abstract double Perimeter();
    public string Describe() => $"Area={Area():F2}, Perimeter={Perimeter():F2}";
}

public sealed class Circle : Shape
{
    private readonly double _radius;
    public Circle(double radius) { _radius = radius; }
    public override double Area()      => Math.PI * _radius * _radius;
    public override double Perimeter() => 2 * Math.PI * _radius;
}
```

```python
# Python
from abc import ABC, abstractmethod
import math

class IShape(ABC):
    @abstractmethod
    def area(self) -> float: ...
    @abstractmethod
    def perimeter(self) -> float: ...

class Circle(IShape):
    def __init__(self, radius: float):
        self._radius = radius
    def area(self)      -> float: return math.pi * self._radius ** 2
    def perimeter(self) -> float: return 2 * math.pi * self._radius
```

### 3️⃣ ABAP'taki karşılığı

```abap
"------------------------------------------------------------------
" ZIF_SHAPE — interface
"------------------------------------------------------------------
INTERFACE zif_shape PUBLIC.
  METHODS:
    area      RETURNING VALUE(rv_result) TYPE decfloat34,
    perimeter RETURNING VALUE(rv_result) TYPE decfloat34.
ENDINTERFACE.

"------------------------------------------------------------------
" ZCL_SHAPE — soyut temel sınıf
"------------------------------------------------------------------
CLASS zcl_shape DEFINITION
  PUBLIC
  ABSTRACT.          " doğrudan örneklenemez

  PUBLIC SECTION.
    INTERFACES zif_shape.

    METHODS:
      describe RETURNING VALUE(rv_text) TYPE string.

ENDCLASS.

CLASS zcl_shape IMPLEMENTATION.
  METHOD describe.
    rv_text = |Alan={ zif_shape~area( ) }, Çevre={ zif_shape~perimeter( ) }|.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------------
" ZCL_CIRCLE — somut alt sınıf
"------------------------------------------------------------------
CLASS zcl_circle DEFINITION
  PUBLIC
  FINAL
  INHERITING FROM zcl_shape.

  PUBLIC SECTION.
    METHODS:
      constructor IMPORTING iv_radius TYPE decfloat34,
      " REDEFINITION: miras alınan interface metodlarını ezdiğimizi ABAP'a bildiriyoruz:
      zif_shape~area      REDEFINITION,
      zif_shape~perimeter REDEFINITION.

  PRIVATE SECTION.
    DATA mv_radius TYPE decfloat34.

ENDCLASS.

CLASS zcl_circle IMPLEMENTATION.

  METHOD constructor.
    mv_radius = iv_radius.
  ENDMETHOD.

  METHOD zif_shape~area.
    CONSTANTS lc_pi TYPE decfloat34 VALUE '3.14159265358979323846'.
    rv_result = lc_pi * mv_radius * mv_radius.
  ENDMETHOD.

  METHOD zif_shape~perimeter.
    CONSTANTS lc_pi TYPE decfloat34 VALUE '3.14159265358979323846'.
    rv_result = 2 * lc_pi * mv_radius.
  ENDMETHOD.

ENDCLASS.
```

> ⚠️ **C#/Python tuzağı:** Interface adını metot adından ayıran tilde `~` (`zif_shape~area`), ABAP'ın bir metodun hangi interface'e ait olduğunu çözümleme biçimidir. Dışarıdan çağırırken interface'e tiplenmiş bir referans kullanıp `lo_shape->zif_shape~area( )` çağırabilirsiniz ya da — daha yaygın olarak — referans değişkeninizi `REF TO zif_shape` olarak tanımlayıp `lo_shape->area( )` şeklinde doğrudan çağırabilirsiniz.

**Polimorfizm uygulamada:**

```abap
DATA: lo_shape  TYPE REF TO zif_shape,
      lt_shapes TYPE TABLE OF REF TO zif_shape.

" Yarıçapı 5 olan bir daire
lo_shape = NEW zcl_circle( iv_radius = 5 ).
APPEND lo_shape TO lt_shapes.

" Listeyi polimorfik olarak dolaşın — tıpkı C# IShape[] / Python List[IShape] gibi
LOOP AT lt_shapes INTO lo_shape.
  WRITE: / lo_shape->area( ).
ENDLOOP.
```

---

## 11.3 Static ve Instance, Constructor'lar, ME->

### 1️⃣ Benzetme

`CLASS-DATA` ve `CLASS-METHODS` static üyelerdir — tüm instance'lar arasında paylaşılır; C#'taki `static` veya Python'daki sınıf düzeyi attribute'larla aynıdır. `ME->` ise `this.` / `self.` karşılığıdır.

ABAP'ın iki özel constructor'ı vardır:
- `CONSTRUCTOR` — instance constructor'ı (`NEW` / `CREATE OBJECT` ile çağrılır), parametre alır.
- `CLASS_CONSTRUCTOR` — static constructor (program yüklendiğinde bir kez çalışır, parametre almaz). C#'taki `static` constructor veya Python'daki `__init_subclass__`/modül düzeyi başlatma kodunun karşılığı.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C# static üyeler + constructor
public class InvoiceCounter
{
    private static int _total = 0;
    private readonly int _id;

    static InvoiceCounter() { Console.WriteLine("Sınıf yüklendi"); }

    public InvoiceCounter()
    {
        _total++;
        _id = _total;
    }

    public static int Total => _total;
    public int Id => _id;
}
```

### 3️⃣ ABAP'taki karşılığı

```abap
CLASS zcl_invoice_counter DEFINITION PUBLIC CREATE PUBLIC.

  PUBLIC SECTION.
    CLASS-METHODS:
      class_constructor,                          " static constructor
      get_total RETURNING VALUE(rv_total) TYPE i.

    METHODS:
      constructor,
      get_id RETURNING VALUE(rv_id) TYPE i.

  PRIVATE SECTION.
    CLASS-DATA gv_total TYPE i.   " tüm instance'lar arasında paylaşılır
    DATA       mv_id    TYPE i.   " instance'a özgü

ENDCLASS.

CLASS zcl_invoice_counter IMPLEMENTATION.

  METHOD class_constructor.
    " Sınıf program oturumuna ilk yüklendiğinde bir kez çalışır
    WRITE: / 'Sınıf yüklendi.'.
    gv_total = 0.
  ENDMETHOD.

  METHOD constructor.
    " ME-> 'this.' / 'self.' anlamına gelir
    gv_total  = gv_total + 1.
    ME->mv_id = gv_total.     " ME-> burada isteğe bağlı ama açık ve okunabilir
  ENDMETHOD.

  METHOD get_total.
    rv_total = gv_total.
  ENDMETHOD.

  METHOD get_id.
    rv_id = mv_id.
  ENDMETHOD.

ENDCLASS.

" Kullanım
DATA: lo1 TYPE REF TO zcl_invoice_counter,
      lo2 TYPE REF TO zcl_invoice_counter.

lo1 = NEW #( ).
lo2 = NEW #( ).

WRITE: / zcl_invoice_counter=>get_total( ).   " 2  (=> ile static çağrı)
WRITE: / lo1->get_id( ).                      " 1  (-> ile instance çağrısı)
WRITE: / lo2->get_id( ).                      " 2
```

> 💡 **`=>` ve `->` — bilmeniz gereken tek kural:**
> - `=>` bir **static/class** metodu çağırır veya bir **class-data** attribute'u okur.
> - `->` bir **instance** metodu çağırır veya bir **instance** attribute'u okur.
>
> C#'ta `ClassName.StaticMethod()` ile `instance.InstanceMethod()` arasındaki fark gibi, sadece farklı oklar.

---

## 11.4 Sınıf Tabanlı Exception'lar: cx_root, RAISE, TRY/CATCH/CLEANUP

### 1️⃣ Benzetme

ABAP'ın modern exception sistemi C# ile neredeyse bire bir örtüşür. Her exception, kök `CX_ROOT`'tan miras alan bir *sınıftır*. Hiyerarşinin her gün kullanacağınız üç önemli dalı vardır:

```
cx_root
  ├─ cx_static_check    (metot imzasında bildirilmeli — checked exception'lar)
  ├─ cx_dynamic_check   (bildirmeniz zorunlu değil ama yapabilirsiniz)
  └─ cx_no_check        (runtime hataları — bunları normalde yakalamayın)
```

> ⚠️ **C#/Python tuzağı:** `cx_static_check`, ABAP'ın Java checked exception'larına karşılık gelir — eğer birini `RAISE` ederseniz metot imzasının `RAISING` ile bunu bildirmesi gerekir. Biri fırlatan bir metodu çağırıp handle etmezseniz derleyici sizi uyarır. C#'ta böyle bir şey yok. Python'da kesinlikle yok.

### 2️⃣ Bunu zaten biliyorsun

```csharp
// C#
public class InsufficientFundsException : Exception
{
    public decimal Balance { get; }
    public InsufficientFundsException(decimal balance)
        : base($"Yetersiz bakiye. Bakiye: {balance}")
    { Balance = balance; }
}

public void Withdraw(decimal amount)
{
    if (amount > _balance)
        throw new InsufficientFundsException(_balance);
    _balance -= amount;
}

try
{
    account.Withdraw(1000);
}
catch (InsufficientFundsException ex)
{
    Console.WriteLine($"Hata: {ex.Message}, Bakiye {ex.Balance} idi");
}
finally
{
    Console.WriteLine("Her zaman çalışır — CLEANUP gibi");
}
```

```python
# Python
class InsufficientFundsError(Exception):
    def __init__(self, balance: float):
        super().__init__(f"Yetersiz bakiye. Bakiye: {balance}")
        self.balance = balance

try:
    account.withdraw(1000)
except InsufficientFundsError as e:
    print(f"Hata: {e}, bakiye {e.balance} idi")
finally:
    print("Her zaman çalışır")
```

### 3️⃣ ABAP'taki karşılığı

Önce exception sınıfınızı oluşturun — ADT'de paketinize sağ tıklayın → New → ABAP Class, şablon olarak "Exception Class" seçin. Otomatik olarak `cx_static_check`'ten miras alır (ya da `cx_dynamic_check` seçebilirsiniz).

```abap
"------------------------------------------------------------------
" ZCX_INSUFFICIENT_FUNDS — exception sınıfı (ADT/SE24 üzerinden oluşturulur)
" Otomatik olarak CX_STATIC_CHECK'ten miras alır
"------------------------------------------------------------------
CLASS zcx_insufficient_funds DEFINITION
  PUBLIC
  INHERITING FROM cx_static_check
  FINAL
  CREATE PUBLIC.

  PUBLIC SECTION.
    DATA mv_balance TYPE decfloat34 READ-ONLY.

    METHODS constructor
      IMPORTING iv_balance TYPE decfloat34
      RAISING   cx_static_check.  " exception constructor'ları için zorunlu kalıp

  PROTECTED SECTION.
ENDCLASS.

CLASS zcx_insufficient_funds IMPLEMENTATION.
  METHOD constructor.
    CALL METHOD super->constructor( ).   " üst constructor'ı çağır
    mv_balance = iv_balance.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------------
" Exception fırlatan bir sınıf
"------------------------------------------------------------------
CLASS zcl_account DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    METHODS:
      constructor IMPORTING iv_balance TYPE decfloat34,
      withdraw
        IMPORTING iv_amount TYPE decfloat34
        RAISING   zcx_insufficient_funds.   " imzada bildirilmeli!
  PRIVATE SECTION.
    DATA mv_balance TYPE decfloat34.
ENDCLASS.

CLASS zcl_account IMPLEMENTATION.
  METHOD constructor.
    mv_balance = iv_balance.
  ENDMETHOD.

  METHOD withdraw.
    IF iv_amount > mv_balance.
      RAISE EXCEPTION TYPE zcx_insufficient_funds
        EXPORTING iv_balance = mv_balance.
    ENDIF.
    mv_balance = mv_balance - iv_amount.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------------
" Çağıran
"------------------------------------------------------------------
DATA lo_account TYPE REF TO zcl_account.
lo_account = NEW #( iv_balance = 500 ).

TRY.
    lo_account->withdraw( iv_amount = 1000 ).
    WRITE: / 'Para çekme başarılı.'.

  CATCH zcx_insufficient_funds INTO DATA(lx_funds).
    WRITE: / |Hata: Yetersiz bakiye. Bakiye { lx_funds->mv_balance } idi|.

  CLEANUP.
    " CLEANUP, exception buradaki TRY bloğunda yakalanmayıp
    " yukarı doğru yayıldığında çalışır — kaynakları serbest bırakmak için kullanın
    " (C#'ta yeniden fırlatan senaryodaki finally gibi).
    " "Her zaman çalışır" semantiği için CLEANUP'ı kullanın.
    WRITE: / 'Cleanup bloğu çalıştı.'.
ENDTRY.
```

> 💡 **CLEANUP ve finally:** `CLEANUP` yalnızca exception mevcut `TRY` bloğunda yakalanmayıp çağrı yığınında daha yukarı yayıldığında çalışır. C#'taki `finally` gibi "her zaman çalışır" semantiği istiyorsanız, temizleme mantığınızı hem `CATCH` hem de `CLEANUP` bloklarına yazın — ya da bir yardımcı metotta yeniden düzenleyin.

---

## 11.5 Event'ler, FRIENDS ve Factory Deseni

### Event'ler

ABAP event'leri C# delegate/event'leri gibi çalışır. Bir sınıf `EVENTS` ile bir event bildirir. Başka bir sınıf (*handler*) static veya instance bir metot bildirir ve `SET HANDLER` ile kaydolur.

```abap
"------------------------------------------------------------------
" ZCL_SENSOR — eşik aşıldığında event fırlatır
"------------------------------------------------------------------
CLASS zcl_sensor DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    EVENTS threshold_exceeded
      EXPORTING VALUE(ev_reading) TYPE decfloat34.

    METHODS:
      constructor IMPORTING iv_threshold TYPE decfloat34,
      report_reading IMPORTING iv_value TYPE decfloat34.

  PRIVATE SECTION.
    DATA mv_threshold TYPE decfloat34.
ENDCLASS.

CLASS zcl_sensor IMPLEMENTATION.
  METHOD constructor.
    mv_threshold = iv_threshold.
  ENDMETHOD.

  METHOD report_reading.
    IF iv_value > mv_threshold.
      RAISE EVENT threshold_exceeded EXPORTING ev_reading = iv_value.
    ENDIF.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------------
" ZCL_ALERT_HANDLER — event'i dinler
"------------------------------------------------------------------
CLASS zcl_alert_handler DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    METHODS on_threshold_exceeded
      FOR EVENT threshold_exceeded OF zcl_sensor
      IMPORTING ev_reading.
ENDCLASS.

CLASS zcl_alert_handler IMPLEMENTATION.
  METHOD on_threshold_exceeded.
    WRITE: / |UYARI! { ev_reading } değeri eşiği aştı.|.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------------
" Birleştirin
"------------------------------------------------------------------
DATA lo_sensor  TYPE REF TO zcl_sensor.
DATA lo_handler TYPE REF TO zcl_alert_handler.

lo_sensor  = NEW #( iv_threshold = 100 ).
lo_handler = NEW #( ).

" Kaydol: lo_handler, lo_sensor'dan gelen event'leri dinler
SET HANDLER lo_handler->on_threshold_exceeded FOR lo_sensor.

lo_sensor->report_reading( 90 ).   " sessizlik
lo_sensor->report_reading( 115 ).  " UYARI!
```

### FRIENDS

`FRIENDS` kasıtlı olarak kapsüllemeyi kırar — adı geçen sınıf bir başkasının `PRIVATE SECTION`'ına erişebilir. Bunu dikkatli kullanın (genellikle birim test yardımcı sınıfları için):

```abap
CLASS zcl_my_service DEFINITION PUBLIC FINAL CREATE PUBLIC
  FRIENDS zcl_my_service_test.   " test sınıfı tam erişim kazanır

  PRIVATE SECTION.
    DATA mv_secret TYPE string.
ENDCLASS.
```

### NEW ve CREATE OBJECT ile Factory Deseni

`NEW`, modern (ABAP 7.4+) sözdizimidir — bunu kullanın. `CREATE OBJECT`, eski kod tabanlarında göreceğiniz klasik formdur.

```abap
" Modern — bunu kullanın
DATA(lo_calc) = NEW zcl_calculator( ).

" Klasik — eski kodda okuyacaksınız
DATA lo_calc2 TYPE REF TO zcl_calculator.
CREATE OBJECT lo_calc2.
```

Basit bir factory:

```abap
CLASS zcl_shape_factory DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    CLASS-METHODS create_shape
      IMPORTING iv_type         TYPE string
                iv_dimension    TYPE decfloat34
      RETURNING VALUE(ro_shape) TYPE REF TO zif_shape
      RAISING   cx_no_check.
ENDCLASS.

CLASS zcl_shape_factory IMPLEMENTATION.
  METHOD create_shape.
    CASE iv_type.
      WHEN 'CIRCLE'.
        ro_shape = NEW zcl_circle( iv_radius = iv_dimension ).
      WHEN OTHERS.
        RAISE EXCEPTION TYPE cx_no_check.
    ENDCASE.
  ENDMETHOD.
ENDCLASS.

" Kullanım — çağıran yalnızca interface'i bilir
DATA(lo_shape) = zcl_shape_factory=>create_shape(
  iv_type      = 'CIRCLE'
  iv_dimension = 7 ).
WRITE: / lo_shape->area( ).
```

---

## 11.6 Clean ABAP — Sizi İşe Aldıran Topluluk Stil Rehberi

SAP, **[Clean ABAP](https://github.com/SAP/styleguides/blob/main/clean-abap/CleanABAP.md)** stil rehberini yayımladı — bu, ABAP dünyasında Google stil rehberinin karşılığıdır ve ciddi ekiplerin kod inceleme kontrol listesi olarak kullandığı şeydir.

İlk günden bilmeniz gereken temel kurallar:

| Kural | Klasik (kaçının) | Clean (kullanın) |
|------|----------------|-------------|
| Fonksiyonel çağrıları tercih edin | `CALL METHOD lo->foo( ).` | `lo->foo( ).` |
| Satır içi bildirimleri tercih edin | `DATA lv_x TYPE i. lv_x = 1.` | `DATA(lv_x) = 1.` |
| `CREATE OBJECT` yerine `NEW` kullanın | `CREATE OBJECT lo TYPE zcl_foo.` | `DATA(lo) = NEW zcl_foo( ).` |
| EXPORTING yerine tek RETURNING | `EXPORTING ev_result = ...` | `RETURNING VALUE(rv_result)` |
| Kısa metot gövdeleri (~20 satır) | 400 satırlık TANRI metodu | Birçok küçük metot |
| Kendini belgeleyen isimler | `lv_x`, `lv_flag` | `lv_customer_count`, `lv_found` |
| Eski sözdizimi kullanmayın | `MOVE a TO b.`, `COMPUTE x = ...` | `b = a.`, `x = ...` |

> 🧭 **İş hayatında:** İlk projenizde yapılan bir kod incelemesinde kıdemli bir kişi bunları anında tarar. Clean ABAP öncesi eğitim almış inceleyiciler kendileri klasik stil yazıyor olabilir — ama *siz*, yeni işe giren biri olarak, modern temiz kod yazmanızı bekleyeceklerdir. İşin püf noktası bu: bazı iş arkadaşlarınızdan daha modern olmak zorundayız.

**Clean Core ile bağlantı:** Clean ABAP, SAP'ın S/4HANA için **Clean Core** girişimiyle örtüşür. Clean Core, *SAP standart kodunu değiştirme* ve *yalnızca yayımlanmış ABAP Cloud API'lerini kullan* anlamına gelir. Özel kodunuzu `PUBLIC` yayımlanmış sınıfları kullanan temiz, nesne yönelimli ABAP'ta yazarsanız, kodunuzun yükseltmelere karşı hayatta kalma ihtimali artar ve ABAP Test Cockpit (ATC) clean-core denetiminden geçer. Bunu Kısım 14'te daha ayrıntılı ele alıyoruz.

---

## 11.7 Çalışma Örneği: ZCL_INVOICE_VALIDATOR

Finans modülü özelleştirmesinde gerçekten kullanabileceğiniz gerçekçi bir sınıfta her şeyi bir araya getirelim.

```abap
"------------------------------------------------------------------
" ZCX_VALIDATION_ERROR — exception sınıfı
"------------------------------------------------------------------
CLASS zcx_validation_error DEFINITION
  PUBLIC INHERITING FROM cx_static_check FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    DATA mv_message TYPE string READ-ONLY.
    METHODS constructor
      IMPORTING iv_message TYPE string
      RAISING   cx_static_check.
ENDCLASS.
CLASS zcx_validation_error IMPLEMENTATION.
  METHOD constructor.
    CALL METHOD super->constructor( ).
    mv_message = iv_message.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------------
" ZIF_VALIDATOR — interface
"------------------------------------------------------------------
INTERFACE zif_validator PUBLIC.
  METHODS validate
    IMPORTING is_invoice TYPE any    " üretimde gerçek bir yapı tipi kullanın
    RAISING   zcx_validation_error.
ENDINTERFACE.

"------------------------------------------------------------------
" ZCL_INVOICE_VALIDATOR — somut uygulama
"------------------------------------------------------------------
CLASS zcl_invoice_validator DEFINITION
  PUBLIC FINAL CREATE PUBLIC.

  PUBLIC SECTION.
    INTERFACES zif_validator.

    " Başarılı doğrulama sonrası fırlatılan event
    EVENTS validation_passed
      EXPORTING VALUE(ev_invoice_id) TYPE vbeln.  " vbeln = SD belge numarası tipi

    CLASS-METHODS get_instance
      RETURNING VALUE(ro_instance) TYPE REF TO zcl_invoice_validator.

  PRIVATE SECTION.
    CLASS-DATA go_instance TYPE REF TO zcl_invoice_validator.  " singleton

    METHODS:
      check_amount
        IMPORTING iv_amount TYPE wrbtr           " wrbtr = döviz tutarı DDIC tipi
        RAISING   zcx_validation_error,
      check_company_code
        IMPORTING iv_bukrs TYPE bukrs            " bukrs = şirket kodu DDIC tipi
        RAISING   zcx_validation_error.

ENDCLASS.

CLASS zcl_invoice_validator IMPLEMENTATION.

  " Singleton factory metodu
  METHOD get_instance.
    IF go_instance IS NOT BOUND.
      go_instance = NEW #( ).
    ENDIF.
    ro_instance = go_instance.
  ENDMETHOD.

  " Ana doğrulama — interface metodunu uygular
  METHOD zif_validator~validate.
    " Gerçek kodda is_invoice tipli bir yapı olurdu
    " Örnek amaçlı private kontrol metodlarını çağırıyoruz:
    TRY.
        check_amount(       iv_amount = '100.00' ).
        check_company_code( iv_bukrs  = '1000'   ).

        " Doğrulama başarılı — event fırlat
        RAISE EVENT validation_passed
          EXPORTING ev_invoice_id = '0090000123'.

      CATCH zcx_validation_error.
        RAISE.   " çağırana yeniden fırlat
    ENDTRY.
  ENDMETHOD.

  METHOD check_amount.
    IF iv_amount <= 0.
      RAISE EXCEPTION TYPE zcx_validation_error
        EXPORTING iv_message = 'Fatura tutarı pozitif olmalıdır'.
    ENDIF.
  ENDMETHOD.

  METHOD check_company_code.
    " Gerçek uygulama: T001'den SELECT
    IF iv_bukrs IS INITIAL.
      RAISE EXCEPTION TYPE zcx_validation_error
        EXPORTING iv_message = 'Şirket kodu zorunludur'.
    ENDIF.
  ENDMETHOD.

ENDCLASS.

"------------------------------------------------------------------
" Örnek çağıran (örneğin bir raporda veya başka bir sınıfta)
"------------------------------------------------------------------
DATA lo_validator TYPE REF TO zcl_invoice_validator.
lo_validator = zcl_invoice_validator=>get_instance( ).

TRY.
    lo_validator->zif_validator~validate( is_invoice = VALUE #( ) ).
    WRITE: / 'Fatura başarıyla doğrulandı.'.
  CATCH zcx_validation_error INTO DATA(lx_err).
    WRITE: / |Doğrulama başarısız: { lx_err->mv_message }|.
ENDTRY.
```

> 🧭 **İş hayatında:** Klasik GUI'de sınıflara göz atmak ve düzenlemek için `SE24` işlemini kullanırsınız. ADT'de (Eclipse) bir sınıfa `Ctrl+Shift+A` tuşuna basıp sınıf adını yazarak gidebilirsiniz. Her iki durumda da DEFINITION/IMPLEMENTATION ayrımı görünürdür. "Bu BAdI nerede uygulanmış?" veya "bu doğrulayıcıyı hata ayıkla" denildiğinde, `SE24` başlangıç noktanızdır.

---

## 🧠 Özet

- ABAP OOP gerçek bir OOP'tur — class, interface, inheritance, polimorfizm, exception, event.
- Görünürlük üye başına değil **bölüm** bazındadır (`PUBLIC SECTION.` / `PROTECTED SECTION.` / `PRIVATE SECTION.`).
- **`INHERITING FROM`** = `: BaseClass`, **`REDEFINITION`** = `override`, **`ABSTRACT`** / **`FINAL`** doğrudan eşleşir.
- Static üyeler: `CLASS-DATA` + `CLASS-METHODS`; `=>` ile çağrılır. Instance üyeler: `DATA` + `METHODS`; `->` ile çağrılır. `ME->` = `this.`/`self.`
- Exception sınıfları `cx_static_check` (checked), `cx_dynamic_check` veya `cx_no_check`'ten miras alır. `TRY/CATCH/CLEANUP`, `try/catch/finally`'nin yerini alır — ama `CLEANUP` ≠ `finally`.
- Event'ler: `EVENTS` + `RAISE EVENT` + `SET HANDLER`. Kasıtlı kapsülleme kırma için `FRIENDS`.
- **Clean ABAP**: `NEW`, satır içi bildirimler, fonksiyonel çağrılar, kısa odaklı metodlar kullanın. Stil rehberini öğrenin — ilk hafta kopya kağıdınızdır.

---

*[← İçindekiler](../content.md) | [← Önceki: Smartforms & Adobe Forms](10-smartforms-and-adobe-forms.md) | [Sonraki: Function Module, RFC & BAPI →](12-function-modules-rfc-bapi.md)*
