# Chapter 11: ABAP Objects — OOP vs C#/Python

*Where your class-design instincts finally feel at home — with a few delightful surprises.*

---

## ☕ The honest first take

ABAP had procedures and global data for decades before object-orientation arrived. "ABAP Objects" landed in the late 1990s — and it was a real OOP system, not a bolt-on. Classes, interfaces, inheritance, exceptions, events: it's all there.

The syntax is wordier than C# and more explicit than Python. You will type `CLASS ... IMPLEMENTATION.` and `ENDCLASS.` and `METHOD ... ENDMETHOD.` for the rest of your life. After a day it feels natural; after a week it's muscle memory.

Here is the mapping you need before anything else:

```text
   C# / Python concept               ABAP equivalent
   ─────────────────────────────     ────────────────────────────────────
   class MyClass { }                 CLASS zcl_my_class DEFINITION. ... ENDCLASS.
   public / protected / private      PUBLIC SECTION. / PROTECTED SECTION. / PRIVATE SECTION.
   : (inheritance)                   INHERITING FROM zcl_parent
   interface IFoo { }                INTERFACE zif_foo. ... ENDINTERFACE.
   abstract class                    ABSTRACT CLASS (in DEFINITION)
   sealed / final class              FINAL CLASS (in DEFINITION)
   override                          REDEFINITION (in METHODS declaration)
   static field / method             CLASS-DATA / CLASS-METHODS
   constructor                       CONSTRUCTOR (instance) / CLASS_CONSTRUCTOR (static)
   this.                             ME->
   throw new MyException(...)        RAISE EXCEPTION TYPE zcx_my_exception ...
   try / catch / finally             TRY / CATCH / CLEANUP
   event Action<T>                   EVENTS + RAISE EVENT / SET HANDLER
   new Foo()                         NEW zcl_foo( ) or CREATE OBJECT
```

---

## 11.1 Classes, Methods, Attributes, Visibility

### 1️⃣ The analogy

A class is a blueprint. In ABAP you split the blueprint into two parts — a `DEFINITION` (the header file, if you're thinking old-school C++) and an `IMPLEMENTATION` (the body). Both live in the same `.clas.abap` file but are clearly separated.

Visibility is handled with section keywords rather than per-member modifiers. You write `PUBLIC SECTION.` once, list everything public under it, then `PRIVATE SECTION.` and list everything private.

### 2️⃣ You already know this

```csharp
// C# — a simple calculator class
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

### 3️⃣ The ABAP way

```abap
"------------------------------------------------------------------
" ZCL_CALCULATOR — DEFINITION
"------------------------------------------------------------------
CLASS zcl_calculator DEFINITION
  PUBLIC          " visible to all consumers
  FINAL           " cannot be subclassed (like C# sealed)
  CREATE PUBLIC.  " anyone can instantiate it

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

> ⚠️ **C#/Python gotcha:** In ABAP a `RETURNING` parameter is how you return a single value — it acts like a function return. Methods with `EXPORTING` or `CHANGING` parameters can return multiple values (more on that in Chapter 12). There's no `return` keyword for returning from a `RETURNING` method — you just assign to the parameter name (`rv_result = ...`).

**Naming conventions** (the whole SAP team will expect these):

| Prefix | Meaning |
|--------|---------|
| `zcl_` | custom class |
| `zif_` | custom interface |
| `zcx_` | custom exception class |
| `iv_` | importing value parameter |
| `ev_` | exporting value parameter |
| `cv_` | changing value parameter |
| `rv_` | returning value parameter |
| `mv_` | member (instance) variable |
| `gv_` | global variable |
| `lv_` | local variable |
| `lt_` | local table |
| `ls_` | local structure |

> 🧭 **On the job:** Senior developers will read your variable names before your logic. Using wrong prefixes (`l_var` instead of `lv_var`) signals "this person learned ABAP from old tutorials." Use the table above. Clean ABAP style guide (section 11.6) enforces this further.

---

## 11.2 Interfaces, Inheritance, Polymorphism, ABSTRACT/FINAL, Redefinition

### 1️⃣ The analogy

Interfaces and inheritance work exactly as you expect. The ABAP-specific wrinkle: when you implement an interface, the method names are prefixed with the interface name, so `zif_shape~area` instead of just `area`. It looks weird; it eliminates name collisions.

### 2️⃣ You already know this

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

### 3️⃣ The ABAP way

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
" ZCL_SHAPE — abstract base class
"------------------------------------------------------------------
CLASS zcl_shape DEFINITION
  PUBLIC
  ABSTRACT.          " cannot be instantiated directly

  PUBLIC SECTION.
    INTERFACES zif_shape.

    METHODS:
      describe RETURNING VALUE(rv_text) TYPE string.

ENDCLASS.

CLASS zcl_shape IMPLEMENTATION.
  METHOD describe.
    rv_text = |Area={ zif_shape~area( ) }, Perimeter={ zif_shape~perimeter( ) }|.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------------
" ZCL_CIRCLE — concrete subclass
"------------------------------------------------------------------
CLASS zcl_circle DEFINITION
  PUBLIC
  FINAL
  INHERITING FROM zcl_shape.

  PUBLIC SECTION.
    METHODS:
      constructor IMPORTING iv_radius TYPE decfloat34,
      " REDEFINITION tells ABAP we're overriding inherited interface methods:
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

> ⚠️ **C#/Python gotcha:** The tilde `~` separating interface name from method name (`zif_shape~area`) is ABAP's way of resolving which interface a method belongs to. When you call it from outside, you can use a reference typed to the interface and call `lo_shape->zif_shape~area( )`, or — more commonly — type your reference variable as `REF TO zif_shape` and call `lo_shape->area( )` directly.

**Polymorphism in action:**

```abap
DATA: lo_shape  TYPE REF TO zif_shape,
      lt_shapes TYPE TABLE OF REF TO zif_shape.

" A circle with radius 5
lo_shape = NEW zcl_circle( iv_radius = 5 ).
APPEND lo_shape TO lt_shapes.

" Walk the list polymorphically — just like C# IShape[] / Python List[IShape]
LOOP AT lt_shapes INTO lo_shape.
  WRITE: / lo_shape->area( ).
ENDLOOP.
```

---

## 11.3 Static vs Instance, Constructors, ME->

### 1️⃣ The analogy

`CLASS-DATA` and `CLASS-METHODS` are the static members — shared across all instances, same as `static` in C# or class-level attributes in Python. `ME->` is `this.` / `self.`.

ABAP has two special constructors:
- `CONSTRUCTOR` — instance constructor (called on `NEW` / `CREATE OBJECT`), receives parameters.
- `CLASS_CONSTRUCTOR` — static constructor (called once per program load, no parameters). Equivalent to a C# `static` constructor or Python `__init_subclass__`/module-level init code.

### 2️⃣ You already know this

```csharp
// C# static members + constructor
public class InvoiceCounter
{
    private static int _total = 0;
    private readonly int _id;

    static InvoiceCounter() { Console.WriteLine("Class loaded"); }

    public InvoiceCounter()
    {
        _total++;
        _id = _total;
    }

    public static int Total => _total;
    public int Id => _id;
}
```

### 3️⃣ The ABAP way

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
    CLASS-DATA gv_total TYPE i.   " shared across all instances
    DATA       mv_id    TYPE i.   " per-instance

ENDCLASS.

CLASS zcl_invoice_counter IMPLEMENTATION.

  METHOD class_constructor.
    " Runs once when the class is first loaded into the program session
    WRITE: / 'Class loaded.'.
    gv_total = 0.
  ENDMETHOD.

  METHOD constructor.
    " ME-> is 'this.' / 'self.'
    gv_total  = gv_total + 1.
    ME->mv_id = gv_total.     " ME-> optional here but explicit and readable
  ENDMETHOD.

  METHOD get_total.
    rv_total = gv_total.
  ENDMETHOD.

  METHOD get_id.
    rv_id = mv_id.
  ENDMETHOD.

ENDCLASS.

" Usage
DATA: lo1 TYPE REF TO zcl_invoice_counter,
      lo2 TYPE REF TO zcl_invoice_counter.

lo1 = NEW #( ).
lo2 = NEW #( ).

WRITE: / zcl_invoice_counter=>get_total( ).   " 2  (static call via =>)
WRITE: / lo1->get_id( ).                      " 1  (instance call via ->)
WRITE: / lo2->get_id( ).                      " 2
```

> 💡 **`=>` vs `->` — the one rule you must know:**
> - `=>` calls a **static/class** method or reads a **class-data** attribute.
> - `->` calls an **instance** method or reads an **instance** attribute.
>
> It's like `ClassName.StaticMethod()` vs `instance.InstanceMethod()` in C#, just with different arrows.

---

## 11.4 Class-Based Exceptions: cx_root, RAISE, TRY/CATCH/CLEANUP

### 1️⃣ The analogy

ABAP's modern exception system maps almost perfectly to C#. Every exception is a *class* that inherits from the root `CX_ROOT`. The hierarchy has three important branches you'll use every day:

```
cx_root
  ├─ cx_static_check    (must be declared in method signature — checked exceptions)
  ├─ cx_dynamic_check   (don't have to declare, but can)
  └─ cx_no_check        (runtime errors — don't catch these normally)
```

> ⚠️ **C#/Python gotcha:** `cx_static_check` is ABAP's version of Java checked exceptions — if you `RAISE` one, the method signature must declare it with `RAISING`. If you call a method that raises one and don't handle it, the compiler warns you. C# doesn't have this. Python definitely doesn't.

### 2️⃣ You already know this

```csharp
// C#
public class InsufficientFundsException : Exception
{
    public decimal Balance { get; }
    public InsufficientFundsException(decimal balance)
        : base($"Insufficient funds. Balance: {balance}")
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
    Console.WriteLine($"Error: {ex.Message}, Balance was {ex.Balance}");
}
finally
{
    Console.WriteLine("Always runs — like CLEANUP");
}
```

```python
# Python
class InsufficientFundsError(Exception):
    def __init__(self, balance: float):
        super().__init__(f"Insufficient funds. Balance: {balance}")
        self.balance = balance

try:
    account.withdraw(1000)
except InsufficientFundsError as e:
    print(f"Error: {e}, balance was {e.balance}")
finally:
    print("Always runs")
```

### 3️⃣ The ABAP way

First, create your exception class — in ADT, right-click your package → New → ABAP Class, choose "Exception Class" as the template. It will automatically inherit from `cx_static_check` (or you can pick `cx_dynamic_check`).

```abap
"------------------------------------------------------------------
" ZCX_INSUFFICIENT_FUNDS — exception class (created via ADT/SE24)
" Inherits from CX_STATIC_CHECK automatically
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
      RAISING   cx_static_check.  " required boilerplate for exception constructors

  PROTECTED SECTION.
ENDCLASS.

CLASS zcx_insufficient_funds IMPLEMENTATION.
  METHOD constructor.
    CALL METHOD super->constructor( ).   " call parent constructor
    mv_balance = iv_balance.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------------
" A class that raises the exception
"------------------------------------------------------------------
CLASS zcl_account DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    METHODS:
      constructor IMPORTING iv_balance TYPE decfloat34,
      withdraw
        IMPORTING iv_amount TYPE decfloat34
        RAISING   zcx_insufficient_funds.   " declared in signature!
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
" Caller
"------------------------------------------------------------------
DATA lo_account TYPE REF TO zcl_account.
lo_account = NEW #( iv_balance = 500 ).

TRY.
    lo_account->withdraw( iv_amount = 1000 ).
    WRITE: / 'Withdrawal successful.'.

  CATCH zcx_insufficient_funds INTO DATA(lx_funds).
    WRITE: / |Error: Insufficient funds. Balance was { lx_funds->mv_balance }|.

  CLEANUP.
    " CLEANUP runs when an exception is NOT caught here but
    " propagates upward — use it to release resources (like C# finally
    " in a re-throw scenario). For "always runs" semantics, use CLEANUP.
    WRITE: / 'Cleanup block ran.'.
ENDTRY.
```

> 💡 **CLEANUP vs finally:** `CLEANUP` only runs when the exception is *not* caught in the current `TRY` block and is propagating further up the call stack. If you want "always runs" semantics like C# `finally`, write your cleanup logic in *both* the `CATCH` and `CLEANUP` blocks — or refactor into a helper method.

---

## 11.5 Events, FRIENDS, and the Factory Pattern

### Events

ABAP events work like C# delegates/events. A class declares an event with `EVENTS`. Another class (the *handler*) declares a static or instance method and registers with `SET HANDLER`.

```abap
"------------------------------------------------------------------
" ZCL_SENSOR — raises an event when threshold exceeded
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
" ZCL_ALERT_HANDLER — listens to the event
"------------------------------------------------------------------
CLASS zcl_alert_handler DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    METHODS on_threshold_exceeded
      FOR EVENT threshold_exceeded OF zcl_sensor
      IMPORTING ev_reading.
ENDCLASS.

CLASS zcl_alert_handler IMPLEMENTATION.
  METHOD on_threshold_exceeded.
    WRITE: / |ALERT! Reading { ev_reading } exceeded threshold.|.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------------
" Wire it together
"------------------------------------------------------------------
DATA lo_sensor  TYPE REF TO zcl_sensor.
DATA lo_handler TYPE REF TO zcl_alert_handler.

lo_sensor  = NEW #( iv_threshold = 100 ).
lo_handler = NEW #( ).

" Register: lo_handler listens to events from lo_sensor
SET HANDLER lo_handler->on_threshold_exceeded FOR lo_sensor.

lo_sensor->report_reading( 90 ).   " silence
lo_sensor->report_reading( 115 ).  " ALERT!
```

### FRIENDS

`FRIENDS` breaks encapsulation deliberately — a named class can access `PRIVATE SECTION` of another. Use it sparingly (usually for unit test helper classes):

```abap
CLASS zcl_my_service DEFINITION PUBLIC FINAL CREATE PUBLIC
  FRIENDS zcl_my_service_test.   " test class gets full access

  PRIVATE SECTION.
    DATA mv_secret TYPE string.
ENDCLASS.
```

### Factory Pattern with NEW vs CREATE OBJECT

`NEW` is the modern (ABAP 7.4+) syntax — use it. `CREATE OBJECT` is the classic form you'll see in older codebases.

```abap
" Modern — use this
DATA(lo_calc) = NEW zcl_calculator( ).

" Classic — you'll read this in legacy code
DATA lo_calc2 TYPE REF TO zcl_calculator.
CREATE OBJECT lo_calc2.
```

A simple factory:

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

" Usage — caller only knows about the interface
DATA(lo_shape) = zcl_shape_factory=>create_shape(
  iv_type      = 'CIRCLE'
  iv_dimension = 7 ).
WRITE: / lo_shape->area( ).
```

---

## 11.6 Clean ABAP — The Community Style Guide That Gets You Hired

SAP published the **[Clean ABAP](https://github.com/SAP/styleguides/blob/main/clean-abap/CleanABAP.md)** style guide — it's the ABAP world's equivalent of the Google style guide and it's what serious teams use as their code-review checklist.

Key rules you should know on day one:

| Rule | Classic (avoid) | Clean (use) |
|------|----------------|-------------|
| Prefer functional calls | `CALL METHOD lo->foo( ).` | `lo->foo( ).` |
| Prefer inline declarations | `DATA lv_x TYPE i. lv_x = 1.` | `DATA(lv_x) = 1.` |
| Prefer `NEW` over `CREATE OBJECT` | `CREATE OBJECT lo TYPE zcl_foo.` | `DATA(lo) = NEW zcl_foo( ).` |
| Single RETURNING over EXPORTING | `EXPORTING ev_result = ...` | `RETURNING VALUE(rv_result)` |
| Short method bodies (~20 lines) | 400-line GOD method | Many small methods |
| Self-documenting names | `lv_x`, `lv_flag` | `lv_customer_count`, `lv_found` |
| No obsolete syntax | `MOVE a TO b.`, `COMPUTE x = ...` | `b = a.`, `x = ...` |

> 🧭 **On the job:** In a code review on your first project, a senior will scan for these instantly. Reviewers who trained before Clean ABAP may themselves write classic style — but they will expect *you*, as a new hire, to write modern clean code. This is the curve-ball: you have to be more modern than some of your colleagues.

**Connection to Clean Core:** Clean ABAP overlaps with SAP's **Clean Core** initiative for S/4HANA. Clean Core means *don't modify SAP standard code* and *use only released ABAP Cloud APIs*. If you write your custom code in clean, object-oriented ABAP using `PUBLIC` released classes, your code is more likely to survive upgrades and pass an ABAP Test Cockpit (ATC) clean-core check. We cover this more in Chapter 14.

---

## 11.7 Worked Example: ZCL_INVOICE_VALIDATOR

Let's pull everything together in a realistic class you could actually use in a finance module customization.

```abap
"------------------------------------------------------------------
" ZCX_VALIDATION_ERROR — exception class
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
    IMPORTING is_invoice TYPE any    " use a real structure type in production
    RAISING   zcx_validation_error.
ENDINTERFACE.

"------------------------------------------------------------------
" ZCL_INVOICE_VALIDATOR — concrete implementation
"------------------------------------------------------------------
CLASS zcl_invoice_validator DEFINITION
  PUBLIC FINAL CREATE PUBLIC.

  PUBLIC SECTION.
    INTERFACES zif_validator.

    " Event raised after successful validation
    EVENTS validation_passed
      EXPORTING VALUE(ev_invoice_id) TYPE vbeln.  " vbeln = SD document number type

    CLASS-METHODS get_instance
      RETURNING VALUE(ro_instance) TYPE REF TO zcl_invoice_validator.

  PRIVATE SECTION.
    CLASS-DATA go_instance TYPE REF TO zcl_invoice_validator.  " singleton

    METHODS:
      check_amount
        IMPORTING iv_amount TYPE wrbtr           " wrbtr = currency amount DDIC type
        RAISING   zcx_validation_error,
      check_company_code
        IMPORTING iv_bukrs TYPE bukrs            " bukrs = company code DDIC type
        RAISING   zcx_validation_error.

ENDCLASS.

CLASS zcl_invoice_validator IMPLEMENTATION.

  " Singleton factory method
  METHOD get_instance.
    IF go_instance IS NOT BOUND.
      go_instance = NEW #( ).
    ENDIF.
    ro_instance = go_instance.
  ENDMETHOD.

  " Main validation — implements the interface method
  METHOD zif_validator~validate.
    " In real code, is_invoice would be a typed structure
    " For illustration we call the private check methods:
    TRY.
        check_amount(       iv_amount = '100.00' ).
        check_company_code( iv_bukrs  = '1000'   ).

        " Validation passed — raise event
        RAISE EVENT validation_passed
          EXPORTING ev_invoice_id = '0090000123'.

      CATCH zcx_validation_error.
        RAISE.   " re-raise to caller
    ENDTRY.
  ENDMETHOD.

  METHOD check_amount.
    IF iv_amount <= 0.
      RAISE EXCEPTION TYPE zcx_validation_error
        EXPORTING iv_message = 'Invoice amount must be positive'.
    ENDIF.
  ENDMETHOD.

  METHOD check_company_code.
    " Real implementation: SELECT from T001
    IF iv_bukrs IS INITIAL.
      RAISE EXCEPTION TYPE zcx_validation_error
        EXPORTING iv_message = 'Company code is required'.
    ENDIF.
  ENDMETHOD.

ENDCLASS.

"------------------------------------------------------------------
" Example caller (e.g., in a report or another class)
"------------------------------------------------------------------
DATA lo_validator TYPE REF TO zcl_invoice_validator.
lo_validator = zcl_invoice_validator=>get_instance( ).

TRY.
    lo_validator->zif_validator~validate( is_invoice = VALUE #( ) ).
    WRITE: / 'Invoice validated successfully.'.
  CATCH zcx_validation_error INTO DATA(lx_err).
    WRITE: / |Validation failed: { lx_err->mv_message }|.
ENDTRY.
```

> 🧭 **On the job:** You'll encounter the `SE24` transaction to browse and edit classes in the classic GUI. In ADT (Eclipse), you navigate to a class by pressing `Ctrl+Shift+A` and typing the class name. Either way, the DEFINITION/IMPLEMENTATION split is visible. When you're asked "find where this BAdI is implemented" or "debug this validator," `SE24` is your starting point.

---

## 🧠 Recap

- ABAP OOP is real OOP — classes, interfaces, inheritance, polymorphism, exceptions, events.
- Visibility is by **section** (`PUBLIC SECTION.` / `PROTECTED SECTION.` / `PRIVATE SECTION.`), not per-member.
- **`INHERITING FROM`** = `: BaseClass`, **`REDEFINITION`** = `override`, **`ABSTRACT`** / **`FINAL`** map directly.
- Static members: `CLASS-DATA` + `CLASS-METHODS`; call with `=>`. Instance members: `DATA` + `METHODS`; call with `->`. `ME->` = `this.`/`self.`
- Exception classes inherit from `cx_static_check` (checked), `cx_dynamic_check`, or `cx_no_check`. `TRY/CATCH/CLEANUP` replaces `try/catch/finally` — but `CLEANUP` ≠ `finally`.
- Events: `EVENTS` + `RAISE EVENT` + `SET HANDLER`. `FRIENDS` for deliberate encapsulation breaks.
- **Clean ABAP**: use `NEW`, inline declarations, functional calls, short focused methods. Learn the style guide — it's your first-week cheat code.

---

*[← Contents](../content.md) | [← Previous: Smartforms & Adobe Forms](10-smartforms-and-adobe-forms.md) | [Next: Function Modules, RFC & BAPI →](12-function-modules-rfc-bapi.md)*
