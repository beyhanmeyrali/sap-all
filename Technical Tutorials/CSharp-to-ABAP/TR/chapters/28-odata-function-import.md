# Kısım 28: Function Import'lar

*CRUD yetmediğinde — "Onayla" gibi bir iş operasyonunu düzgün bir OData aksiyonu olarak nasıl açarsınız.*

---

## 28.1 Sorun: CRUD her şeyi karşılamıyor ☕

REST taraftarları size her şeyin bir kaynak olduğunu ve kaynaklara yapılan her şeyin GET/POST/PUT/DELETE ile ifade edilebileceğini söyler. Teoride güzel. 40 yıllık SAP iş mantığı üzerine kurulu bir ERP sisteminde gerçeklik daha karmaşıktır.

"Satış Siparişini Onayla" operasyonunu düşünün. Hangi kaynağı değiştiriyorsunuz? Sipariş başlığını mı? Bir bakıma — ama *onay* bir *iş olayıdır*; yan etkileri vardır: iş akışını tetikleyebilir, muhasebe belgesi oluşturabilir, durum alanını güncelleyebilir ve e-posta gönderebilir. Tüm bunları `PATCH /SalesOrderHeaderSet('1001')` isteğine `{ "Status": "B" }` ile sıkıştırmak bir yalandır — iş niyetini gizliyorsunuz ve SAP standart mantığını devre dışı bırakıyorsunuz.

OData'nın "RPC tarzı bir aksiyona ihtiyacım var" sorusuna cevabı **Function Import**'tur.

### Üç adımlı zihinsel model

| Benzetme | Ne olduğu |
|---|---|
| Bir formda "Onayla" yazan düğme | Veri alanı değil — bir *komut* |
| REST API'de `/orders/1001/approve` adresine POST | Gövdesi ve dönüş değeri olan adlandırılmış bir operasyon |
| OData'da Function Import | Aynı şey — metadata'da tanımlı, URL ile çağrılabilir adlandırılmış aksiyon |

> 💡 OData v4 bunları "Actions" (durum değiştiren) ve "Functions" (salt okunur) olarak yeniden adlandırmıştır. OData v2 / SEGW'de, farklı OData sürümlerinden bağımsız olarak hepsine **Function Import** denir. İş hayatında her iki terimi de duyacaksınız — aynı kavramın farklı OData sürümlerindeki adı olduğunu bilin.

---

## 28.2 Bunu zaten biliyorsun

### C# — Web API aksiyon metodu

```csharp
// C# — ASP.NET Core Web API
// Bu standart bir REST kaynak operasyonu DEĞİLDİR — bir komuttur
[HttpPost("{orderId}/approve")]
public async Task<IActionResult> ApproveSalesOrder(
    string orderId,
    [FromBody] ApproveRequest request)
{
    var result = await _orderService.Approve(orderId, request.ApprovedBy, request.Note);
    if (!result.Success)
        return BadRequest(result.ErrorMessage);

    return Ok(new ApproveResult
    {
        OrderId    = orderId,
        NewStatus  = result.NewStatus,
        ApprovedAt = result.ApprovedAt
    });
}

public record ApproveRequest(string ApprovedBy, string Note);
public record ApproveResult(string OrderId, string NewStatus, DateTime ApprovedAt);
```

Çağıran taraf:
```http
POST /api/orders/0000001001/approve
Content-Type: application/json

{ "ApprovedBy": "JDOE", "Note": "Kredi kontrolünden sonra onaylandı." }
```

### Python — FastAPI POST endpoint'i

```python
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter()

class ApproveRequest(BaseModel):
    approved_by: str
    note: str

class ApproveResult(BaseModel):
    order_id:    str
    new_status:  str
    approved_at: str

@router.post("/orders/{order_id}/approve", response_model=ApproveResult)
async def approve_sales_order(order_id: str, body: ApproveRequest):
    result = await order_service.approve(order_id, body.approved_by, body.note)
    return ApproveResult(
        order_id    = order_id,
        new_status  = result.new_status,
        approved_at = result.approved_at.isoformat()
    )
```

Şimdi bunu SAP/OData sürümüne eşleyelim.

---

## 28.3 SEGW'de Function Import tanımlama 🛠️

### Adım 1 — Function Import oluşturma

SEGW'de **Function Imports → Create** üzerine sağ tıklayın (veya araç çubuğunu kullanın).

| Alan | Değer |
|---|---|
| Function Import name | `ApproveSalesOrder` |
| HTTP Method | `POST` ← durum değiştiren operasyonlar MUTLAKA POST olmalıdır |
| Return type | Entity type → `SalesOrderHeader` |
| Return cardinality | `0..1` (güncellenmiş başlığı veya hiçbir şey döndürür) |

> ⚠️ **C#/Python tuzağı:** Durum değiştiren OData v2 Function Import'ları **mutlaka** HTTP POST kullanmalıdır. Yan etkisi olan bir function import'u GET olarak ayarlarsanız, GET yanıtlarını önbelleğe alan istemciler (bazı HTTP proxy'leri gibi) sisteminizi bozar. GET = salt okunur. POST = aksiyon. Bu kural tartışılmazdır.

### Adım 2 — Parametre ekleme

Function Import editörünün **Parameters** sekmesinde:

| Parametre adı | Tür | Mod | Nullable |
|---|---|---|---|
| `OrderId` | `Edm.String` | In | No |
| `ApprovedBy` | `Edm.String` | In | No |
| `Note` | `Edm.String` | In | Yes |

### Adım 3 — Generate

Generate'e basın. SEGW, `ZSALESORDER_SRV_MPC`'yi günceller ve `ZSALESORDER_SRV_DPC_EXT`'teki stub'ı oluşturur/günceller. Metod stub'ını arayın:

```abap
METHOD execute_action.
  " TODO: implement
ENDMETHOD.
```

SEGW tarafından oluşturulan metadata artık şunları içerir:

```xml
<FunctionImport
  Name="ApproveSalesOrder"
  ReturnType="ZSALESORDER_SRV.SalesOrderHeader"
  m:HttpMethod="POST">
  <Parameter Name="OrderId"    Type="Edm.String" Mode="In" />
  <Parameter Name="ApprovedBy" Type="Edm.String" Mode="In" />
  <Parameter Name="Note"       Type="Edm.String" Mode="In" Nullable="true" />
</FunctionImport>
```

---

## 28.4 DPC_EXT'te EXECUTE_ACTION'ı yeniden tanımlama 🔁

Tek bir servisteki tüm function import'lar **bir** metodu paylaşır: `execute_action`. Doğru işleyiciye yönlendirmek için `iv_action_name` üzerinde dallanırsınız — bir dağıtıcı gibi.

```abap
CLASS zsalesorder_srv_dpc_ext DEFINITION
  INHERITING FROM zsalesorder_srv_dpc
  FINAL
  CREATE PUBLIC.

PUBLIC SECTION.
  METHODS execute_action REDEFINITION.

PRIVATE SECTION.
  METHODS approve_sales_order
    IMPORTING
      it_parameter TYPE /iwbep/t_mgw_name_value_pair
      io_tech_request_context TYPE REF TO /iwbep/if_mgw_req_context
    CHANGING
      co_entity TYPE REF TO data
    RAISING
      /iwbep/cx_mgw_busi_exception
      /iwbep/cx_mgw_tech_exception.

ENDCLASS.

CLASS zsalesorder_srv_dpc_ext IMPLEMENTATION.

  "=========================================================================
  " EXECUTE_ACTION — bu servisteki tüm Function Import'lar için yönlendirici
  "=========================================================================
  METHOD execute_action.

    CASE iv_action_name.
      WHEN 'ApproveSalesOrder'.
        approve_sales_order(
          EXPORTING
            it_parameter            = it_parameter
            io_tech_request_context = io_tech_request_context
          CHANGING
            co_entity               = co_entity ).

      " Buraya WHEN dalları olarak daha fazla function import ekleyin
      WHEN 'RecalculatePrices'.
        " ... ilerideki implementasyon ...

      WHEN OTHERS.
        " Bilinmeyen aksiyon — framework'ün 404 tarzı hata fırlatmasına izin ver
        SUPER->execute_action(
          EXPORTING iv_action_name            = iv_action_name
                    it_parameter              = it_parameter
                    io_tech_request_context   = io_tech_request_context
          CHANGING  co_entity                 = co_entity ).
    ENDCASE.

  ENDMETHOD.

  "=========================================================================
  " PRIVATE — ApproveSalesOrder implementasyonu
  "=========================================================================
  METHOD approve_sales_order.

    " --- 1. Gelen parametreleri oku ----------------------------------------
    DATA lv_order_id    TYPE vbeln_va.
    DATA lv_approved_by TYPE string.
    DATA lv_note        TYPE string.

    LOOP AT it_parameter INTO DATA(ls_param).
      CASE ls_param-name.
        WHEN 'OrderId'.    lv_order_id    = ls_param-value.
        WHEN 'ApprovedBy'. lv_approved_by = ls_param-value.
        WHEN 'Note'.       lv_note        = ls_param-value.
      ENDCASE.
    ENDLOOP.

    " --- 2. Doğrula --------------------------------------------------------
    IF lv_order_id IS INITIAL.
      RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
        EXPORTING
          textid  = /iwbep/cx_mgw_busi_exception=>business_error
          message = 'OrderId gereklidir'.
    ENDIF.

    " Siparişin var olup olmadığını kontrol et
    SELECT SINGLE gbstk FROM vbak INTO @DATA(lv_current_status)
      WHERE vbeln = @lv_order_id.

    IF sy-subrc <> 0.
      RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
        EXPORTING
          textid = /iwbep/cx_mgw_busi_exception=>entity_not_found.
    ENDIF.

    IF lv_current_status = 'C'.   " 'C' = Tamamlandı
      RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
        EXPORTING
          textid  = /iwbep/cx_mgw_busi_exception=>business_error
          message = 'Sipariş zaten tamamlandı — onaylanamaz'.
    ENDIF.

    " --- 3. İş mantığı — SAP standart BAPI çağrısı ------------------------
    "     Gerçek projede BAPI_SALESORDER_CHANGE veya özel bir FM çağırırsınız.
    "     Burada deseni direkt DB güncellemesiyle gösteriyoruz (yalnızca illüstrasyon).
    "     Üretim kodu değişim belgeleri için her zaman BAPI'lerden geçer.
    DATA lt_return TYPE TABLE OF bapiret2.

    " BAPI çağrı deseni simülasyonu:
    "   CALL FUNCTION 'Z_APPROVE_SALES_ORDER'
    "     EXPORTING iv_vbeln       = lv_order_id
    "               iv_approved_by = lv_approved_by
    "               iv_note        = lv_note
    "     TABLES    return         = lt_return.
    "
    "   LOOP AT lt_return INTO DATA(ls_ret) WHERE type CA 'EAX'.
    "     RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
    "       EXPORTING message = ls_ret-message.
    "   ENDLOOP.
    "
    "   COMMIT WORK AND WAIT.

    " Demo için — durum alanını direkt güncelle
    UPDATE vbak SET gbstk = 'B'
      WHERE vbeln = lv_order_id.

    IF sy-subrc <> 0.
      RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception
        EXPORTING
          textid  = /iwbep/cx_mgw_busi_exception=>business_error
          message = 'Sipariş durumu güncellenemedi'.
    ENDIF.

    COMMIT WORK AND WAIT.

    " --- 4. Yanıtta döndürmek için güncellenmiş başlığı oku ---------------
    SELECT SINGLE *
      FROM vbak
      INTO @DATA(ls_vbak)
      WHERE vbeln = @lv_order_id.

    " OData entity type'ına eşle
    DATA(ls_entity) = VALUE zcl_zsalesorder_srv_mpc=>ts_salesorderheader(
      order_id   = ls_vbak-vbeln
      customer   = ls_vbak-kunnr
      order_date = ls_vbak-audat
      net_amount = ls_vbak-netwr
      currency   = ls_vbak-waerk
      status     = ls_vbak-gbstk   " Artık 'B' (onaylandı)
    ).

    " --- 5. Sonucu çıktı referansına yaz -----------------------------------
    "     co_entity, REF TO data türündedir — yapımızı ona atarız
    GET REFERENCE OF ls_entity INTO co_entity.

  ENDMETHOD.

ENDCLASS.
```

> ⚠️ **C#/Python tuzağı:** `co_entity` çıktı parametresi `REF TO data` türündedir — genel bir işaretçi. Doğru türe dönüştürmezsiniz; bunun yerine tiplenmiş yapınızı yerel bir değişkene koyar ve `GET REFERENCE OF ls_entity INTO co_entity` yaparsınız. Framework, metadata'dan ne türü beklediğini bilir ve dahili olarak dönüştürür. Türler eşleşmezse çalışma zamanında dump alırsınız — SEGW tarafından oluşturulan `ts_salesorderheader` türü tam olarak eşleşmelidir.

> ⚠️ **C#/Python tuzağı:** C# aksiyon metodunda `return BadRequest(message)` yaptığınızın aksine ABAP'ta hata mesajları exception'lar tarafından taşınır. `RAISE EXCEPTION TYPE /iwbep/cx_mgw_busi_exception` ile `message = '...'`, istemciye HTTP 400/422 tarzı bir hata göndermenin yoludur. Framework bunu otomatik olarak bir OData hata yanıtına dönüştürür.

---

## 28.5 /IWFND/GW_CLIENT'ta test ve istek formatı 🎯

### /IWFND/GW_CLIENT'ta

1. `/IWFND/GW_CLIENT`'ı açın.
2. **Service** = `ZSALESORDER_SRV` olarak ayarlayın.
3. **HTTP Method** = `POST` olarak ayarlayın.
4. **Request URI**:

```
/sap/opu/odata/sap/ZSALESORDER_SRV/ApproveSalesOrder?OrderId='0000001001'&ApprovedBy='JDOE'&Note='Kredi kontrolünden sonra onaylandı'
```

POST function import'larda parametreler sorgu dizesine gider (gövdeye değil). İstek gövdesi boştur.

5. **Execute** (F8)'e basın. Güncellenmiş başlık entity'siyle HTTP 200 almalısınız.

### Alternatif — Postman / curl

```http
POST /sap/opu/odata/sap/ZSALESORDER_SRV/ApproveSalesOrder?OrderId='0000001001'&ApprovedBy='JDOE'&Note='Onaylandı'
Authorization: Basic <base64>
Accept: application/json
X-CSRF-Token: <alınan token>
Content-Length: 0
```

```json
{
  "d": {
    "__metadata": { "type": "ZSALESORDER_SRV.SalesOrderHeader" },
    "OrderId":    "0000001001",
    "Customer":   "0000001000",
    "OrderDate":  "/Date(1716508800000)/",
    "NetAmount":  "2429.99",
    "Currency":   "USD",
    "Status":     "B"
  }
}
```

`"Status": "B"` — sipariş onaylanmıştır.

### Hata yanıtı (iş doğrulama hatası)

```json
{
  "error": {
    "code":    "SY/530",
    "message": {
      "lang":  "en",
      "value": "Sipariş zaten tamamlandı — onaylanamaz"
    }
  }
}
```

> 🧭 **İş hayatında:** POST tabanlı function import'ları çağırmadan önce neredeyse her zaman bir CSRF token almanız gerekir. Desen şöyledir: önce `GET /sap/opu/odata/sap/<SRV>/$metadata` isteğini `X-CSRF-Token: Fetch` başlığıyla yapın — yanıt `X-CSRF-Token: <token>` ile döner. Ardından bu token değerini POST isteğinde kullanın. Fiori bunu otomatik olarak yapar; Postman / test kodu yapmaz.

---

## 🧠 Özet

- **Function Import**'lar, CRUD'a temiz bir şekilde uymayan adlandırılmış operasyonları (komutları) açmak için OData v2 yoludur.
- SEGW'de tanımlayın: ad, HTTP metodu (durum değişiklikleri için POST), dönüş türü, parametreler.
- Tüm function import'lar tek bir `EXECUTE_ACTION` metodunu paylaşır. `CASE` ifadesiyle `iv_action_name` üzerinde dallanın.
- Parametreleri `it_parameter`'dan okuyun (ad/değer çiftleri). İş hataları için `/iwbep/cx_mgw_busi_exception` fırlatın.
- Sonuçları `GET REFERENCE OF` aracılığıyla yapınızı `co_entity`'ye atayarak döndürün.
- Yan etkili function import'lar için her zaman HTTP POST kullanın — GET yalnızca okumalar içindir.
- Dış istemcilerden yapılan tüm GET dışı çağrılar için CSRF token gereklidir.

*[← İçindekiler](../content.md) | [← Önceki: Header + Item Verisi](27-odata-header-and-item.md) | [Sonraki: CREATE_DEEP_ENTITY →](29-odata-create-deep-entity.md)*
