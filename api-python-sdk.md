# API : Python SDK

{% hint style="info" %}
**هل تبحث عن API الكامل؟** هذه الصفحة عبارة عن دليل عملي. توجد كل فئة عامة، وطريقة، وتوقيع دقيق، ومثال قابل للنسخ واللصق في [مرجع SDK](reference/sdk-reference.md)، وهو مُحسّن لمساعدات الذكاء الاصطناعي.**هل تعمل مع مساعد ذكاء اصطناعي؟** الصق هذا URL في الدردشة حتى يتوفر لديه Chloros 1.2.0 API الكامل والمحدث:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

كل صفحة من هذا الدليل متاحة بتنسيق ماركداون الخام تحت اسم الرابط المختصر بالاحرف الصغيرة + `.md`، والدليل بأكمله مفهرس في `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` على PyPI) تتحكم في كل ما يمكن لتطبيق سطح المكتب القيام به بدءًا من Python: المعالجة الدفعية للصور، والتحكم المباشر في كاميرا LATTICE ومصفوفاتها، وجلسات مستشعر الضوء DAQ، وأتمتة المشاريع المحفوظة. وهي عبارة عن طبقة رقيقة فوق نفس البنية الخلفية المحلية التي تستخدمها واجهة المستخدم الرسومية وCLI (HTTP على `127.0.0.1:5000`)، لذا فإن السلوك متطابق عبر الأسطح الثلاثة جميعها.

## التثبيت

يتكون التثبيت من خطوتين: أولًا حزمة سطح المكتب Chloros (التي توفر البنية الخلفية للمعالجة وبيئات تشغيل الأجهزة)، ثم حزمة Python.

**الخطوة 1 — تثبيت Chloros.** Windows: قم بتشغيل أداة التثبيت الخاصة بسطح المكتب (المسار الافتراضي `C:\Program Files\MAPIR\Chloros\`) من صفحة [التنزيل](download.md). Linux: قم بتثبيت حزمة `.deb` ([تثبيت Linux](linux/linux-installation.md)).**الخطوة 2 — تثبيت SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

قد لا تحتاج حتى إلى pip: فكل أداة تثبيت تأتي مع عجلة SDK مطابقة. تقوم أداة التثبيت Windows بتثبيتها تلقائيًا في نظامك Python؛ أما برنامج التثبيت Linux `.deb` فيضعه في `/usr/lib/chloros/sdk/` ويطبع الأمر الدقيق `pip install --user`. يتم تحديث PyPI عند إصدار كل نسخة، لذا فإن `pip install chloros-sdk` يتوافق مع أحدث إصدار مستقر.

**الخطوة 3 — تسجيل الدخول مرة واحدة لكل جهاز:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

يتم تخزين بيانات الاعتماد مؤقتًا في `~/.chloros/` (في كلا النظامين). في Windows، يمكنك تسجيل الدخول بشكل مماثل من خلال علامة التبويب «<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">» (بيانات الاعتماد) في تطبيق سطح المكتب. يتطلب SDK خطة Chloros+ مدفوعة — انظر [متطلبات الترخيص](#license-requirement) أدناه.

| المتطلبات | التفاصيل |
| --- | --- |
| **تثبيت Chloros** | Windows: أداة التثبيت لسطح المكتب؛ Linux: حزمة `.deb` (توفر الملف الثنائي للخلفية) |
| **Python** | 3.7 أو أعلى (تم تطويره/اختباره على الإصدار 3.10) |
| **نظام التشغيل** | Windows 10/11 64 بت، أو Ubuntu 22.04 LTS أو أحدث، أو NVIDIA Jetson (JetPack 6) |
| **الترخيص** | تسجيل دخول Chloros+ نشط، أي فئة مدفوعة (Copper أو أعلى) |

## الفوز في 60 ثانية

مكالمة واحدة تنشئ مشروعًا، وتستورد مجلدًا، وتُهيئ المعالجة، وتشغّل مسار المعالجة — مع بدء تشغيل الخلفية تلقائيًّا إذا لم تكن قيد التشغيل بالفعل:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(في Linux، استخدم مسارات Linux: `/home/user/drone_images/flight001`. يعمل SDK بنفس الطريقة على كلا النظامين.)

هل تقوم بمعالجة مجلد التقاط LATTICE؟ استخدم الغلاف المتوافق مع LATTICE — فهو يطبق الإعدادات الافتراضية الصحيحة (بدون اكتشاف هدف اللوحة، وإزالة التباين القياسية):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — التحكم الكامل في مسار المعالجة

لأي شيء يتجاوز سطرًا واحدًا، استخدم `ChlorosLocal`. فهو يقوم بتشغيل الخلفية عند الاستخدام الأول (`auto_start_backend=True`)، وينشئ المشاريع ويقوم بتكوينها، ويراقب التقدم، ويعرض ملخصًا بعد انتهاء التشغيل.

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

{% hint style="info" %}
احتفظ بـ `http://127.0.0.1:5000` الافتراضي بدلاً من استبداله بـ `localhost` — في Windows، يتم تحويل `localhost` إلى `::1` أولاً ويستغرق حوالي ثانيتين لكل طلب على الخلفية التي تدعم IPv4 فقط.
{% endhint %}

استخدمه كمدير سياق لضمان التنظيف:

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

يقبل `configure()` الكلمات الرئيسية التالية: `debayer`، `vignette_correction`، `reflectance_calibration`، `indices`، `export_format`، `ppk`، `daq_log_path`، `input_level`، `radiometric_output`، `array_alignment`، `array_alignment_crop`، `array_alignment_interpolation`، و `custom_settings`. القيم الرئيسية:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

المقابض الخاصة بـ LATTICE (`input_level`، و`radiometric_output`، ومجموعة `array_alignment*`) موثقة مع جداول قيمها الكاملة في [مرجع SDK](reference/sdk-reference.md#supported-values).

### مراقبة التقدم

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### قراءة ملخص ما بعد التشغيل — واكتشاف عمليات التشغيل الفارغة

عند الانتهاء، يرفق `process()` ملخص معالجة الخلفية كملف `result["summary"]`. كل إدخال في `summary["hints"]` عبارة عن جملة كاملة تشرح أي أمر جدير بالملاحظة — على سبيل المثال، لماذا لم ينتج عن عملية التشغيل أي مخرجات — كما يتم إعادة إصدار كل تلميح على شكل Python `UserWarning`، لذا فإن عمليات التشغيل الفارغة تُشخص نفسها بنفسها حتى لو لم تقم أبدًا بفحص القاموس:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**لا يتم إصدار `process()` عندما لا تنتج عملية تشغيل أي صور.** هذا هو المكان الوحيد الذي يختلف فيه SDK و CLI عن عمد: تتعامل `chloros-cli process` مع عبارة &quot;تم طلب النتائج، ولم يتم كتابة أي منها&quot; على أنها فشل وتخرج بقيمة غير صفرية، في حين أن SDK تعود بشكل طبيعي وتبلغ عن الحالة من خلال `summary` / التلميحات. إذا كان من المفترض أن يتوقف مسار العمل الخاص بك عند تشغيل فارغ، فتحقق من ذلك بنفسك — افحص `summary` (أو احسب عدد الملفات الموجودة في مجلد المشروع) بدلاً من الاعتماد على استثناء.
{% endhint %}

## Smart Connect — الأجهزة الحية

تقوم ثلاثة أدوات مساعدة بفتح جلسات مستمرة في تجمع الأجهزة الخاص بالخلفية — وهو نفس التجمع الذي تستخدمه واجهة المستخدم الرسومية (GUI)، لذا تتعايش نصوص SDK مع تطبيق سطح المكتب دون التنافس على المنافذ التسلسلية أو عرض النطاق الترددي للشبكة. تعمل جميعها على بدء تشغيل الخلفية المحلية تلقائيًا إذا لم تكن أي منها قيد التشغيل.

### كاميرا LATTICE واحدة — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### مصفوفة متزامنة — `connect_array`

`connect_array` هي نقطة الدخول الموصى بها لمجموعات الكاميرات المتعددة. وهي تُشغّل نفس مسار الإعداد الذكي الذي تعمل به واجهة المستخدم الرسومية (GUI): تحليل الشبكة، والاختيار التلقائي لطبقة المزامنة، ومزامنة الوقت عبر بروتوكول PTP، واختيار تنسيق البكسل لكل كاميرا، وتهيئة التعريض التلقائي (AE)، وتفعيل مشغل GPIO. **المنفذ التسلسلي الأول هو المنفذ الرئيسي** (وهو الذي يطلق نبضة تشغيل الجهاز)؛ أما البقية فهي تابعة.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

أضف `smart=True` إلى أي عملية التقاط مصفوفة لانتظار استقرار التعريض التلقائي عبر جميع الكاميرات قبل التشغيل. للاطلاع على أوضاع الالتقاط (فردي / مستمر / بفواصل زمنية / الأسرع)، والمسجلات، والتصوير المتتالي إلى فيديو، ومحاذاة المصفوفة، راجع [مرجع SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### مستشعر الضوء DAQ — `connect_daq_sensor`

في حالة عدم وجود معلمات، يقوم `connect_daq_sensor()` بالكشف الذكي عن وسيلة النقل (الأولوية: إيثرنت → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

يحمل كل إطار قيمة `spectrum` المكونة من 135 نقطة (W/m²/nm عند المعايرة)، وعلامة `is_saturated`، و CIE `x`، `y`، و`z`. لتحديد مستشعر أو وسيلة نقل معينة — وهو الخيار الموثوق به على المضيفات ذات واجهات الشبكة المتعددة، حيث قد يفوت الاكتشاف التلقائي لشبكة إيثرنت جهاز DAQ-E سليمًا في المحاولة الأولى — قم بتمرير تلميح صريح واحد:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

لاحظ أن ملفات تعريف تصحيح السعة (`cap_id`) **ليست** عنصر تحكم من نوع SDK — اخترها بدلاً من ذلك عبر `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` بدلاً من ذلك.

### المشاريع المحفوظة — `open_project`

يحتفظ المشروع المحفوظ Chloros بالأجهزة المتصلة به (`cameras.json` + `sensors.json` إلى جانب `project.json`)، ويمكن لـ `chloros_sdk.open_project(path)` إعادة توصيل كل شيء دفعة واحدة والتقاط البيانات حسب اسم الجهاز. انظر [أتمتة المشاريع](reference/sdk-reference.md#project-automation--chlorosproject) في المرجع.

## ما تحصل عليه التثبيتات التي تعتمد على pip فقط

تحقق من علامات التوفر على مستوى الوحدة النمطية قبل استخدام أسطح الأجهزة:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

على مضيف يحتوي **فقط** على `pip install chloros-sdk` ولا يحتوي على حزمة سطح المكتب Chloros:

* **لا** تعمل `ChlorosLocal` و `process_folder` و `process_lattice_capture` — فهي تحتاج إلى الملف الثنائي الخلفي المضمن في برنامج تثبيت سطح المكتب.
* أدوات المساعدة في الاتصال الذكي (`connect_camera`، و`connect_array`، `connect_daq_sensor`) هي عملاء HTTP بحتون، لذا فهي تعمل مع خادم خلفي على جهاز آخر — لكن الخوادم الخلفية المرفقة ترتبط بـ loopback فقط، لذا يجب عليك إعادة توجيه المنفذ بنفسك (على سبيل المثال `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) وتمرير `backend_url="http://127.0.0.1:5000"` مع `auto_start_backend=False`. انظر [وضع الخلفية البعيدة](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* فئات LATTICE المباشرة للأجهزة (`LatticeCamera`، `CameraPool`، …) قابلة للاستيراد، لكنها تحتاج إلى بيئة تشغيل Arena SDK الموجودة في حزمة سطح المكتب — فبدونها، تصبح `CAMERA_AVAILABLE` هي `False`.
* تأتي `daq_sdk` (فئات DAQ المباشرة) مع تثبيت سطح المكتب، وليس مع حزمة PyPI، لذلك فإن `DAQ_AVAILABLE` يساوي `False` على مضيف يعمل بـ pip فقط — قم بتشغيل مستشعرات DAQ عبر `connect_daq_sensor()` مقابل خادم خلفي (ممرر عبر نفق) بدلاً من ذلك.

## متطلبات الترخيص

يتطلب الوصول إلى SDK وجود حساب Chloros+ نشط على أي فئة مدفوعة — **Copper أو أعلى**(Copper / Bronze / Silver / Gold)؛ لا تتيح الفئة المجانية Iron الوصول إلى SDK/CLI. يتم التطبيق**من جانب الخادم**: يجب أن يتضمن كل طلب SDK جلسة نشطة وخطة مدفوعة، وإلا فإن الخلفية ترجع `403` / `PLAN_UPGRADE_REQUIRED` (يتم إثارة هذا الخطأ كـ `ChlorosLicenseError` بواسطة `ChlorosLocal`، وكـ `ChlorosConnectError` بواسطة المساعدات `connect_*`). يحصل المتصل الذي تم تسجيل خروجه على `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) بدلاً من ذلك — تؤدي إعادة تشغيل `chloros-cli login` إلى حل الحالة الأولى ولكن ليس الثانية.

يعمل الاستخدام دون اتصال بالإنترنت خلال فترة السماح الخاصة بالخطة: يتم قراءة الفئة من ذاكرة التخزين المؤقت للتحقق من صحة الخادم (5 دقائق) أو ذاكرة التخزين المؤقت للترخيص الموقع والمرتبط بالجهاز (30 يومًا للخطط الشهرية؛ حتى انتهاء صلاحية الاشتراك للخطط السنوية). عند انتهاء فترة السماح، تتحول الخطة إلى النسخة المجانية ويتوقف الوصول إلى SDK حتى يتصل الجهاز بالخادم مرة واحدة. يظل `chloros-cli status` متاحًا في المستوى المجاني، لذا يظل السبب مرئيًا دائمًا. انظر [Chloros+ تسجيل الدخول](chloros+-login.md).

## الاستثناءات

استخدم الفئة الأساسية للتعامل مع «أي خطأ حدث في Chloros»:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

جميع استثناءات مسار المعالجة (`ChlorosBackendError`، `ChlorosConnectionError`، `ChlorosLicenseError`، `ChlorosAuthenticationError`، `ChlorosConfigurationError`، `ChlorosProcessingError`) تنشأ من `ChlorosError`. هناك نقطة يجب الانتباه إليها: `ChlorosConnectError` — لا يُثار إلا بواسطة `connect_camera` / `connect_array` / `connect_daq_sensor` — مشتق من `Exception` العادي، **وليس** من `ChlorosError`، لذا لن يلتقطه `except ChlorosError`. يوجد التسلسل الهرمي الكامل في [مرجع SDK](reference/sdk-reference.md#exceptions).

## انظر أيضًا

* [مرجع SDK](reference/sdk-reference.md) — السطح الكامل لـ API، المُحسَّن لمساعدات الذكاء الاصطناعي.
* [مرجع CLI](reference/cli-reference.md) — كل أمر فرعي من أوامر CLI يعكس استدعاءً لـ SDK.
* [التنزيل](download.md) — برامج التثبيت لـ Windows و Linux.
