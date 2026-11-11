# API: Python SDK

{% hint style="info" %}
**هل تبحث عن API الكاملة؟** هذه الصفحة عبارة عن دليل عملي. توجد كل فئة عامة، وطريقة، وتوقيع دقيق، ومثال قابل للنسخ واللصق في [مرجع SDK](reference/sdk-reference.md)، الذي تم تحسينه خصيصًا للمساعدات الذكية.**هل تعمل مع مساعد ذكاء اصطناعي؟** الصق رابط URL هذا في الدردشة حتى يتوفر لديه Chloros API الكامل والحديث لـ Chloros 1.3.0:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

كل صفحة من هذا الدليل متاحة بتنسيق ماركداون الخام تحت اسم الرابط المختصر بالاحرف الصغيرة + `.md`، والدليل بأكمله مفهرس في `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` على PyPI) يدير كل ما يمكن لتطبيق سطح المكتب القيام به من لغة Python: معالجة الصور دفعة واحدة، والتحكم المباشر في كاميرا LATTICE ومصفوفاتها، وجلسات مستشعر الضوء DAQ، وأتمتة المشاريع المحفوظة. وهي عبارة عن طبقة رقيقة فوق نفس البنية الخلفية المحلية التي تستخدمها واجهة المستخدم الرسومية (GUI) وواجهة سطر الأوامر (CLI) (HTTP على `127.0.0.1:5000`)، لذا فإن السلوك متطابق عبر جميع الأسطح الثلاثة.

## التثبيت

يأتي SDK مرفقًا مع حزمة سطح المكتب Chloros، التي توفر أيضًا البنية الخلفية للمعالجة وبيئات تشغيل الأجهزة، وتتطابق دائمًا مع إصدارها.

**الخطوة 1 — تثبيت Chloros.** Windows: قم بتشغيل أداة تثبيت سطح المكتب (المسار الافتراضي `C:\Program Files\Chloros\`) من صفحة [التنزيل](download.md). Linux: قم بتثبيت الحزمة `.deb` باستخدام `sudo apt install ./chloros_1.3.0_<arch>.deb` ([تثبيت Linux](linux/linux-installation.md)).**الخطوة 2 — التحقق من SDK** (Python 3.7+). عادةً لا يوجد شيء لتثبيته:

* **نظام Windows:** يقوم المثبت بتثبيت حزمة SDK المدمجة في إصدار Python الخاص بنظامك (إذا لم يكن Python مثبتًا، فسيتخطى هذه الخطوة).
* **Linux:** يقوم `.deb` بتثبيت SDK الخاص بنظام `python3`، في `/usr/lib/python3/dist-packages`. يُوصى الحزمة بمتطلباتها، `python3-numpy` و `python3-requests`: حيث توفرها الحزمة `apt install`، بينما لا توفرها الحزمة `dpkg -i`.
* **لا يرى virtualenv** حزم النظام؛ قم بتثبيت ملف wheel المرفق فيه: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` على نظام Linux، أو `chloros_sdk-*.whl` الموجود في المجلد `sdk` ضمن مجلد التثبيت في نظام Windows (`C:\Program Files\Chloros\sdk\` بشكل افتراضي).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

على جهاز لا يحتوي على حزمة Chloros، والذي يعمل SDK الخاص به على تشغيل الخلفية على جهاز آخر (انظر [ما الذي يحصل عليه التثبيت باستخدام pip فقط](#what-a-pip-only-install-gets))، قم بتثبيته من PyPI: `pip install chloros-sdk`. يتم تحديث PyPI عند إصدار كل نسخة، بحيث يتوافق مع أحدث إصدار مستقر.

**الخطوة 3 — قم بتسجيل الدخول مرة واحدة لكل جهاز:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

يتم تخزين بيانات الاعتماد مؤقتًا في `~/.chloros/` (في كلا النظامين). في نظام Windows، يمكنك تسجيل الدخول بشكل مماثل من خلال علامة التبويب «المستخدم» <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> في تطبيق سطح المكتب. يتطلب SDK خطة Chloros+ مدفوعة — انظر [متطلبات الترخيص](#license-requirement) أدناه.

| المتطلبات | التفاصيل |
| --- | --- |
| **تثبيت Chloros** | Windows: أداة التثبيت المكتبية؛ Linux: حزمة `.deb` (توفر الملف الثنائي للخلفية وSDK) |
| **لغة Python** | 3.7 أو أعلى (تم تطويره/اختباره على الإصدار 3.10) |
| **نظام التشغيل** | Windows 10/11 64 بت، أو Ubuntu 22.04 LTS أو أحدث، أو NVIDIA Jetson (JetPack 6) |
| **الترخيص** | حساب Chloros+ نشط، أي فئة مدفوعة (Copper أو أعلى) |

## النجاح في 60 ثانية

تقوم دعوة واحدة بإنشاء مشروع، واستيراد مجلد، وتهيئة المعالجة، وتشغيل مسار المعالجة — مع بدء تشغيل الخلفية تلقائيًا إذا لم تكن قيد التشغيل بالفعل:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(في نظام Linux، استخدم مسارات Linux: `/home/user/drone_images/flight001`. يعمل SDK بنفس الطريقة على كلا النظامين.)

هل تقوم بمعالجة مجلد لقطات LATTICE؟ استخدم الغلاف المتوافق مع LATTICE — فهو يطبق الإعدادات الافتراضية الصحيحة (بدون كشف هدف اللوحة، وإزالة التباين القياسية):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — التحكم الكامل في خط الأنابيب

لأي شيء يتجاوز سطرًا واحدًا، استخدم `ChlorosLocal`. فهو يستخدم خلفية تعمل بالفعل (تلك الخاصة بتطبيق سطح المكتب)، أو يبدأ تشغيل الخلفية المثبتة عند الاستخدام الأول (`auto_start_backend=True`؛ في نظام Windows من المجلد `backend` التابع لمجلد تثبيت Chloros، و`C:\Program Files\Chloros\backend\` بشكل افتراضي)، ويقوم بإنشاء المشاريع وتهيئتها، ومراقبة التقدم، وتقديم ملخص بعد انتهاء التشغيل.

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
احتفظ بـ `http://127.0.0.1:5000` الافتراضي بدلاً من استبداله بـ `localhost` — في نظام Windows، يتم تحويل `localhost` أولاً إلى `::1` ويستغرق ذلك حوالي ثانيتين لكل طلب يتم توجيهه إلى الخلفية التي تدعم IPv4 فقط.
{% endhint %}

استخدمه كمدير سياق. يؤدي الخروج من الكتلة إلى إنهاء استخدام هذه الحالة للخلفية، وليس الخلفية نفسها: فهي تظل قيد التشغيل للنص البرمجي التالي وتتوقف تلقائيًا بعد 30 دقيقة من عدم الاستخدام. `shutdown_backend()` يوقف هذه الحالة بمجرد بدء تشغيلها.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` يقبل الكلمات الرئيسية التالية: `debayer`، `vignette_correction`، `reflectance_calibration`، `indices`، `export_format`، `ppk`، `daq_log_path`، `input_level`، `radiometric_output`، `array_alignment`، `array_alignment_crop`، و`array_alignment_interpolation`، و`custom_settings`. القيم الرئيسية:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

تم توثيق أزرار التحكم الخاصة بـ LATTICE (`input_level`، و`radiometric_output`، ومجموعة `array_alignment*`) مع جداول قيمها الكاملة في [مرجع SDK](reference/sdk-reference.md#supported-values).

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

عند الانتهاء، يرفق `process()` ملخص معالجة الخلفية باسم `result["summary"]`. كل إدخال في `summary["hints"]` عبارة عن جملة كاملة تشرح أي أمر جدير بالملاحظة — على سبيل المثال، لماذا لم ينتج عن عملية تشغيل أي مخرجات — كما يتم إعادة إصدار كل تلميح كملف Python `UserWarning`، لذا فإن عمليات التشغيل الفارغة تُشخص ذاتيًا حتى لو لم تقم أبدًا بفحص القاموس:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**لا يتم إثارة استثناء `process()` عندما لا تنتج عملية التشغيل أي صور.** هذا هو المكان الوحيد الذي يختلف فيه كل من SDK وCLI عن عمد: يعامل `chloros-cli process` عبارة «تم طلب المنتجات، ولم يتم كتابة أي منها» على أنها فشل ويخرج بقيمة غير صفرية، في حين أن SDK يعود بشكل طبيعي ويبلغ عن الحالة من خلال `summary` / التلميحات. إذا كان من المفترض أن يتوقف مسار العمل الخاص بك عند عملية تشغيل فارغة، فتحقق من ذلك بنفسك — افحص `summary` (أو احسب عدد الملفات الموجودة في مجلد المشروع) بدلاً من الاعتماد على استثناء.
{% endhint %}

## Smart Connect — الأجهزة الحية

تقوم ثلاثة مساعدات بفتح جلسات مستمرة في تجمع الأجهزة الخاص بالخلفية — وهو نفس التجمع الذي تستخدمه واجهة سطر الأوامر (CLI) والنصوص البرمجية الأخرى وعلامات تبويب الكاميرا في واجهة المستخدم الرسومية (GUI)، بحيث تتعايش نصوص SDK معها دون التنافس على المنافذ التسلسلية أو عرض النطاق الترددي للشبكة. تعمل جميعها تلقائيًا على بدء تشغيل خلفية محلية إذا لم تكن هناك خلفية قيد التشغيل. لا تشكل لوحة DAQ في واجهة المستخدم الرسومية (GUI) جزءًا من المجموعة: أغلق مستشعر الضوء في واجهة المستخدم الرسومية (GUI) قبل فتحه من خلال برنامج نصي.

### كاميرا LATTICE واحدة — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**تقع المسارات النسبية في المكان الذي يتم فيه تشغيل البرنامج النصي الخاص بك (تم تغيير ذلك في الإصدار 1.3.0).** مع الخلفية المحلية، يكون المسار النسبي `output_dir` (بما في ذلك المسار الافتراضي `"output"`) — والمسارات الأخرى التي تفتحها الخلفية (`burst_dir`، `profile_path`، `save_path`، `mask_path`) — يتم تحليلها وفقًا لمجلد عمل البرنامج النصي الخاص بك، لذا فإن `cam.capture("output/")` يكتب إلى `<current folder>/output/`. في الإصدار 1.2.0، كانت الخلفية المثبتة تحلها في مجلد التثبيت الخاص بها وتفشل مع ظهور رسالة «الوصول مرفوض». أما بالنسبة للخلفية البعيدة، فيتم إرسال المسارات كما تمت كتابتها.
{% endhint %}

### المصفوفة المتزامنة — `connect_array`

`connect_array` هي نقطة الدخول الموصى بها لمنصات الكاميرات المتعددة. وهي تُشغّل نفس مسار الإعداد الذكي الذي تعمل به واجهة المستخدم الرسومية (GUI): تحليل الشبكة، والاختيار التلقائي لمستوى المزامنة، ومزامنة الوقت عبر بروتوكول PTP، واختيار تنسيق البكسل لكل كاميرا، وتهيئة التعريض التلقائي (AE)، وتجهيز مشغل GPIO. **الرقم التسلسلي الأول هو الرئيسي** (وهو الذي يطلق نبضة تشغيل الجهاز)؛ أما البقية فهي تابعة.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

أضف `smart=True` إلى أي عملية التقاط مصفوفة لانتظار استقرار التعريض التلقائي عبر جميع الكاميرات قبل التشغيل.

جديد في الإصدار 1.3.0، يوفر لك `ArraySession` أيضًا إطارات متزامنة مباشرةً، دون كتابة ملفات:

* يُرجع `arr.grab()` إطارًا واحدًا لكل كاميرا، جميعها من مشغل واحد، كمصفوفات numpy (`{serial: array}`)؛ `.group_id` هو المشغل المشترك، بينما يحدد `.errors` أي كاميرا تم استبعادها.
* ينتج `arr.frame_stream(fps=5.0)` مجموعة واحدة من هذا النوع لكل نبضة، من أجل المعالجة المباشرة في مسار المعالجة الخاص بك.
* يقوم `arr.continuous("output/")` بتشغيل الالتقاط المستمر الخاص بالخلفية (الوضع المستمر لواجهة المستخدم الرسومية) ويعيد `ContinuousHandle` مع `status()`، `wait()` و `stop()` — استخدمها بدلاً من حلقة الالتقاط عندما تحتاج إلى حفظ كل مشغل بمعدل عالٍ.

للاطلاع على أوضاع التقاط البيانات (فردي / مستمر / بفواصل زمنية / الأسرع)، وأجهزة التسجيل، والتقاط متتالي إلى فيديو، ومحاذاة المصفوفات، راجع [مرجع SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### مستشعر الضوء DAQ — `connect_daq_sensor`

بدون معلمات، يقوم `connect_daq_sensor()` بالكشف الذكي عن وسيلة النقل (الأولوية: إيثرنت → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

يحمل كل إطار قيمة `spectrum` المكونة من 135 نقطة (وحدة W/m²/nm عند المعايرة)، وعلامة `is_saturated`، و CIE `x`، `y`، و`z`. لتحديد مستشعر أو وسيلة نقل معينة، قم بتمرير تلميح صريح واحد:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

لاحظ أن ملفات تعريف تصحيح السعة (`cap_id`) **ليست** من عناصر التحكم في SDK — قم باختيارها عبر `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` بدلاً من ذلك.

### المشاريع المحفوظة — `open_project`

يحتفظ مشروع Chloros المحفوظ بالأجهزة المتصلة به (`cameras.json` + `sensors.json` إلى جانب `project.json`)، ويمكن لـ `chloros_sdk.open_project(path)` إعادة توصيل الكاميرات والمصفوفات ومستشعرات الضوء في آن واحد وتشغيل عمليات الالتقاط حسب اسم الجهاز. تُعاد توصيل مستشعرات الضوء عبر خادم Chloros الخلفي قيد التشغيل، مع الإعدادات المحفوظة وسعة التصوير الخاصة بها. انظر [أتمتة المشروع](reference/sdk-reference.md#project-automation--chlorosproject) في المرجع.

## ما الذي تحصل عليه التثبيتات التي تقتصر على pip

تحقق من علامات التوفر على مستوى الوحدة النمطية قبل استخدام أسطح الأجهزة:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

على مضيف يحتوي **فقط** على `pip install chloros-sdk` ولا يحتوي على حزمة سطح مكتب Chloros:

* **لا** تعمل `ChlorosLocal` و `process_folder` و `process_lattice_capture` — فهي تحتاج إلى الملف الثنائي للخلفية المضمن في أداة تثبيت سطح المكتب.
* أدوات المساعدة الخاصة بـ smart-connect (`connect_camera`، و`connect_array`، و`connect_daq_sensor`) هي عملاء HTTP بحتون، لذا فهي تعمل مع خادم خلفي موجود على جهاز آخر — لكن الخوادم الخلفية المرفقة ترتبط بـ loopback فقط، لذا يجب عليك إعادة توجيه المنفذ بنفسك (على سبيل المثال: `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) وتمرير `backend_url="http://127.0.0.1:5000"` مع `auto_start_backend=False`. انظر [وضع الخلفية البعيدة](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* تستورد فئات LATTICE المباشرة للأجهزة (`LatticeCamera`، `CameraPool`، …) قابلة للاستيراد، لكنها تحتاج إلى بيئة تشغيل Arena SDK من حزمة سطح المكتب — وبدونها، فإن `CAMERA_AVAILABLE` تساوي `False`.
* يتم تشغيل مستشعرات DAQ فقط من خلال واجهة خلفية (ممرّة) — `connect_daq_sensor()`، أو `proj.sensors` الخاص بالمشروع؛ حيث تفتح الواجهة الخلفية المستشعر على جهازها الخاص.

## المساعدون الذكيون — `chloros-mcp`

تتضمن حزمة تطوير البرامج (SDK) `chloros-mcp`، وهو خادم MCP يتيح لـ Claude Desktop وClaude Code وCursor وعملاء MCP الآخرين تشغيل Chloros: معالجة الصور، وتشغيل كاميرات LATTICE والمصفوفات ومستشعرات الضوء DAQ. تغطي أدواته وظائف حزمة تطوير البرامج (SDK) هذه. يتم تثبيته مع حزمة تطوير البرامج (SDK)، لذا فإن الإعداد في Claude Code يتم بسطر واحد:

```bash
claude mcp add chloros -- chloros-mcp
```

العملاء الآخرون: [استخدام Chloros مع المساعدين الذكيين](ai-assistants.md). الخيارات وجميع الأدوات: [خادم MCP (مساعدو الذكاء الاصطناعي)](reference/sdk-reference.md#mcp-server-ai-assistants).

## متطلبات الترخيص

يتطلب الوصول إلى SDK تسجيل دخول نشط إلى Chloros+ على أي فئة مدفوعة — **Copper أو أعلى**(Copper / Bronze / Silver / Gold)؛ لا تتيح الفئة المجانية «Iron» الوصول إلى SDK/CLI. يتم تطبيق هذا الشرط**من جانب الخادم**: يجب أن يتضمن كل طلب SDK جلسة نشطة وخطة مدفوعة، وإلا فإن الخادم الخلفي سيعرض الرمز `403` / `PLAN_UPGRADE_REQUIRED` (يتم رفعها كـ `ChlorosLicenseError` بواسطة `ChlorosLocal`، وكـ `ChlorosConnectError` بواسطة المساعدات `connect_*`). يحصل المتصل الذي تم تسجيل خروجه على `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) بدلاً من ذلك — تؤدي إعادة تشغيل `chloros-cli login` إلى حل الحالة الأولى ولكن ليس الثانية.

يعمل الاستخدام دون اتصال بالإنترنت خلال فترة السماح الخاصة بالخطة: يتم قراءة المستوى من ذاكرة التخزين المؤقت للتحقق من صحة الخادم (5 دقائق) أو ذاكرة التخزين المؤقت للترخيص الموقّع والمرتبط بالجهاز (30 يومًا للخطط الشهرية؛ حتى انتهاء صلاحية الاشتراك للخطط السنوية). عند انتهاء فترة السماح، تتحول الخطة إلى الإصدار المجاني ويتوقف الوصول إلى SDK حتى يتصل الجهاز بالخادم مرة واحدة. يظل `chloros-cli status` متاحًا في الفئة المجانية، لذا يظل السبب مرئيًا دائمًا. انظر [تسجيل الدخول إلى Chloros+](chloros+-login.md).

## الاستثناءات

التقط الفئة الأساسية للتعامل مع «أي خطأ في Chloros»:

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

جميع استثناءات خط الأنابيب (`ChlorosBackendError`، `ChlorosConnectionError`، `ChlorosLicenseError`، `ChlorosAuthenticationError`، `ChlorosConfigurationError`، `ChlorosProcessingError`) تنشأ من `ChlorosError`. هناك نقطة يجب الانتباه إليها: `ChlorosConnectError` — لا يُثار إلا بواسطة `connect_camera` / `connect_array` / `connect_daq_sensor` — تنشأ من `Exception` العادي، **وليس** من `ChlorosError`، لذا لن تلتقطها `except ChlorosError`. توجد التسلسل الهرمي الكامل في [مرجع SDK](reference/sdk-reference.md#exceptions).

## انظر أيضًا

* [مرجع SDK](reference/sdk-reference.md) — واجهة API الكاملة، المُحسّنة لمساعدات الذكاء الاصطناعي.
* [مرجع CLI](reference/cli-reference.md) — كل أمر فرعي في CLI يعكس استدعاءً في SDK.
* [استخدام Chloros مع المساعدات الذكية](ai-assistants.md) — إرشادات الإعداد والمطالبات لـ `chloros-mcp`.
* [التنزيل](download.md) — برامج التثبيت لنظامي التشغيل Windows وLinux.
