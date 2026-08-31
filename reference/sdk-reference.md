# Chloros Python SDK مرجع

**الإصدار:**

1.2.0**تاريخ الإنشاء:**2026-07-29 19:19 ·**تاريخ المراجعة:** 2026-08-30**الحزمة:** `chloros-sdk` (PyPI)**الجمهور المستهدف:** مُحسَّن للاستخدام مع نماذج اللغة الكبيرة (LLM)؛ قابل للقراءة البشرية.**النطاق:** كل فئة ودالة ومساعد عام يُقدمه `import chloros_sdk`، مع أمثلة قابلة للنسخ واللصق تغطي معالجة الصور، والتحكم في كاميرا واحدة، والمصفوفات المتزامنة، وأجهزة استشعار DAQ، وأتمتة المشاريع.

إذا كنت بحاجة إلى النقاط البارزة فقط، فانتقل إلى:
- [التثبيت والبدء السريع](#installation)
- [Smart-Connect لمصفوفات LATTICE](#smart-connect-for-lattice-cameras)
- [جلسات مستشعرات DAQ](#daq-sensor-sessions)
- [أتمتة المشاريع](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## البنية في 60 ثانية

SDK عبارة عن طبقة رقيقة من Python فوق الخلفية Chloros (نفس خادم Flask الذي تستخدمه واجهة المستخدم الرسومية لسطح المكتب وCLI). للأتمتة، قم باستيراد `chloros_sdk` واستدعاء الطرق عالية المستوى؛ في الخلفية، يتحول كل استدعاء إلى طلب HTTP إلى الخلفية المحلية على المنفذ 5000 — `http://127.0.0.1:5000/api/...` (عمداً وليس `localhost`، الذي يُحل أولًا إلى `::1` على Windows ويستغرق حوالي ثانيتين لكل طلب مقابل نظام خلفي يعمل بـ IPv4 فقط). يمتلك النظام الخلفي مجموعة الأجهزة — الكاميرات، وأجهزة استشعار DAQ، وملفات تعريف المحاذاة، ومخازن الإطارات — لذا يمكن لبرامجSDKالتعايش مع واجهة المستخدم الرسومية (GUI) دون التنافس على المنافذ التسلسلية أو عرض النطاق الترددي لبطاقة الشبكة.

هناك ثلاث واجهات ستستخدمها:

1. **`ChlorosLocal` + وظائف مجانية** (`process_folder`، `process_lattice_capture`) — مسار معالجة الصور. قم بتشغيل مجلد كامل عبر عمليات المعايرة / إزالة التباين اللوني / تصدير الفهرس من خلال استدعاء واحد لـ Python.
2. **مقابض التوصيل الذكي** (`connect_camera`، `connect_array`، `connect_daq_sensor`) — فتح جلسة خلفية دائمة للأجهزة العاملة. نفس تدفق «smart-prep» الذي تستخدمه واجهة المستخدم الرسومية: فحص الشبكة، والاختيار التلقائي للمستوى، وPTP، وتهيئة AE، وتكوين مشغل GPIO.
3. **`ChlorosProject` / `open_project`** — تحميل مشروع محفوظ (مجلد يحتوي على `cameras.json` + `sensors.json` + `project.json`)، وتوصيل كل شيء دفعة واحدة، والتحكم في عمليات التقاط البيانات باستخدام معالجات مسماة.

تقوم السطحان 1 و2 **ببدء تشغيل الخلفية المحلية تلقائيًا** إذا لم تكن هناك خلفية مستمعة بالفعل (نفس الملف الثنائي المرفق الذي تقوم واجهة المستخدم الرسومية/ CLI بتشغيله) — لذا يعمل البرنامج النصي البسيط من شل جديد دون الحاجة إلى بدء تشغيل الخلفية أولاً. قم بتمرير `auto_start_backend=False` لإلغاء الاشتراك (على سبيل المثال عند الإشارة إلى خلفية بعيدة، والتي لا يتم إنشاءها أبدًا). انظر [التشغيل التلقائي للخلفية](#backend-auto-start). يتصرف جهاز Surface 3 بشكل مختلف: لا يقبل `open_project()` أي معلمة `auto_start_backend`، ولا يقوم `connect_all()` أبدًا بتشغيل الخلفية — فهو يختبر `http://127.0.0.1:5000` مرة واحدة، وإذا لم يرد أي شيء، فإنه يعود بصمت إلى التحكم المباشر (بدون خلفية) التحكم في الجهاز (بدون خلفية). فقط `proj.process()` و`stream(..., overlays=True)` يقومان بإنشاء `ChlorosLocal()` بشكل مؤجل (والذي يقوم بالتشغيل التلقائي).

جميعها تخضع للتوثيق: قم بتشغيل `chloros-cli login` مرة واحدة على الجهاز، أو قم بتسجيل الدخول عبر واجهة المستخدم الرسومية لسطح المكتب. تؤدي استدعاءات SDK بدون جلسة عمل صالحة إلى إثارة استثناء `ChlorosAuthenticationError`.

المتطلبات:
- Python 3.7+ (حسب ما هو مذكور في الحزمة؛ تم تطويرها واختبارها على الإصدار 3.10)
- تثبيت Chloros Desktop محليًا (يأتي الملف الثنائي للخلفية مضمّنًا داخل برنامج التثبيت)
- تسجيل دخول نشط إلى Chloros+. الحد الأدنى لـ SDK / CLI هو فئة **Copper**أو أعلى (Copper / Bronze / Silver / Gold)؛ أما فئة**Iron**المجانية فلا تتمتع بالوصول إلى SDK / CLI. ويتم تطبيق هذا**من جانب الخادم**: يجب أن يتضمن كل طلب يحمل العلامة SDK / CLI جلسة حية وخطة مدفوعة، وإلا فإن الخلفية ترجع الرمز `403` مع `error_code: PLAN_UPGRADE_REQUIRED` (يظهر على شكل `ChlorosLicenseError` بواسطة `ChlorosLocal`، وكـ `ChlorosConnectError` بواسطة المساعدات `connect_*`). أما المتصل الذي تم تسجيل خروجه فيحصل على `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) بدلاً من ذلك — وهما مختلفان لأن إعادة تشغيل `chloros-cli login` يُصلح الأول ولا يمكنه إصلاح الثاني.
- يُدعم الاستخدام دون اتصال بالإنترنت خلال فترة السماح الخاصة بالخطة: يتم قراءة المستوى من ذاكرة التخزين المؤقت للتحقق من صحة الخادم (5 دقائق) أو ذاكرة التخزين المؤقت للترخيص الموقع والمرتبط بالجهاز (30 يومًا للخطط الشهرية، وحتى انتهاء صلاحية الاشتراك للخطط السنوية). وعند انتهاء فترة السماح هذه، تتحول الخطة إلى الخطة المجانية ويتوقف الوصول إلى SDK / CLI يتم إيقاف الوصول حتى يتمكن الجهاز من الوصول إلى الخادم مرة واحدة. يظل `chloros-cli status` (`GET /api/license-status`) قابلاً للوصول في المستوى المجاني، لذا فإن السبب واضح — فهو المسار الوحيد SDK / CLI المعفى من بوابة المستوى.
- Windows 10/11 64 بت، **Ubuntu 22.04 LTS أو أحدث**، أو Jetson (JetPack 6).**لا** يتم دعم Ubuntu 20.04: تعتمد تبعيات `.deb` على ما ترتبط به الخلفية، بما في ذلك `libc6 (>= 2.34)`، ويأتي نظام Focal مزودًا بـ glibc 2.31.

---

## التثبيت

يُعد Python SDK طبقة رقيقة Python فوق الخلفية Chloros. لكل ما يتجاوز بضعة سير عمل يقتصر على جمع البيانات (DAQ)، ستحتاج إلى **تثبيت حزمة سطح المكتبChloros** محليًّا (Windows أو Linux `.deb`) — فهي التي توفر الملف الثنائي للخلفية، وبيئة التشغيل Arena SDK لكاميرات LATTICE، وحزم المعايرة.

أحدث التنزيلات: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### الخطوة 1 — تثبيت حزمة منصة Chloros

#### Windows (.exe)

1. قم بتنزيل `Chloros-Setup-x.y.z.exe` من صفحة التنزيل.
2. قم بتشغيل برنامج التثبيت واتبع إرشادات المعالج. مسار التثبيت الافتراضي هو `C:\Program Files\MAPIR\Chloros\`.
3. قم بتشغيل Chloros مرة واحدة على الأقل وقم بتسجيل الدخول باستخدام حسابك على Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### الخطوة 2 — تثبيت Python SDK

**يأتي مُثبِّت Chloros مزودًا بعجلة (SDK) مطابقة.** كل مُثبِّت Windows وملف .deb Linux يضع ملف `chloros_sdk-X.Y.Z-py3-none-any.whl` على القرص يطابق تمامًا إصدار واجهة المستخدم الرسومية (GUI) / CLI / الخلفية (backend). لا داعي للبحث في PyPI للحفاظ على التزامن.

#### Windows

يقوم المثبت بتشغيل `pip install` تلقائيًا على ملف wheel المرفق باستخدام مشغل Python الخاص بنظامك (يفضل استخدام مشغل `py.exe`، ويُستخدم `python -m pip` كخيار بديل). لا يتطلب الأمر أي إجراء — يعمل `import chloros_sdk` في بيئة Python الخاصة بك بعد التثبيت الناجح. إذا لم يكن هناك ملف Python على الجهاز، فإن المثبت يتخطى هذه الخطوة بصمت ويستمر كل من واجهة المستخدم الرسومية (GUI) وCLI في العمل.

#### Linux (.deb)

يضع ملف .deb العجلة في `/usr/lib/chloros/sdk/`. يقوم `postinst` بطباعة الأمر بالضبط — ترفض توزيعات PEP 668 الكتابة إلى pip بشكل عام بشكل افتراضي، لذا لا نقوم بالتثبيت التلقائي:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

بالنسبة لعمليات نشر Jetson المعزولة عن الشبكة، تتم هذه العملية بشكل غير متصل بالإنترنت تمامًا — حيث توجد حزمة wheel بالفعل على القرص.

#### PyPI العام

بالنسبة للمضيفات التي تعمل بـ pip فقط (غير المثبت عليها حزمة سطح المكتب Chloros؛ سير العمل الذي يعتمد على الخلفية البعيدة أو DAQ فقط):

```bash
pip install chloros-sdk
```

يتم تحديث PyPI في إصدارات المثبت الخاصة بالإصدارات النهائية، بحيث يتطابق ملف wheel المنشور مع أحدث إصدار مستقر. أما إصدارات التطوير (مثل `1.1.4.dev1`) فلا يتم توزيعها إلا عبر ملف wheel المضمن في المثبت.

#### التحقق

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **يلزم اشتراك في Chloros+.** تتطلب جميع استدعاءات SDK تسجيل دخول نشط إلى Chloros+. قم بتشغيل `chloros-cli login user@example.com 'YourPassword'` مرة واحدة لكل جهاز؛ حيث يتم تخزين بيانات الاعتماد مؤقتًا في `~/.chloros/`.

### هل أحتاج إلى حزمة سطح المكتب؟

حزمة pip وحدها **ليست** كافية لمعظم سير العمل. إليك ما تحتاجه كل واجهة من واجهات SDK:

| واجهة SDK | هل تحتاج إلى حزمة سطح المكتب؟ | لماذا |
| --- | --- | --- |
| `ChlorosLocal`، `process_folder`، `process_lattice_capture` | **نعم** | يبدأ تشغيل الملف الثنائي للخلفية تلقائيًا على `/usr/lib/chloros/chloros-backend` (Linux) أو `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`، `connect_array`، `connect_daq_sensor`، `analyze_array_network`، `list_*`، `discover_*` | **نعم**(محلي)**/ لا**(عن بُعد) | عملاء HTTP خالصون عبر الخلفية. الخلفية المحلية → يلزم وجود حزمة سطح المكتب. الخلفية البعيدة → `backend_url=`**عبر نفق** (انظر وضع الخلفية البعيدة — الخلفيات المرفقة تربط حلقة الارتداد فقط). |
| `ChlorosProject` / `open_project` | **نعم** | يدير المشاريع المحفوظة عبر الخلفية. |
| فئات LATTICE المباشرة (`LatticeCamera`، `CameraPool`، `Calibration`، `DLS`، …) | **نعم** | تحتاج إلى بيئة التشغيل الأصلية لـ Arena SDK المضمنة في حزمة سطح المكتب. وإلا فإن `CAMERA_AVAILABLE` تساوي `False` عند الاستيراد. |
| فئات DAQ المباشرة (`DAQUSensor`، `DAQMSensor`، `DAQESensor`، `SensorFleet`، `discover_all`) | **لا** | Python خالص عبر pyserial/bleak/zeroconf. يمكن لبيئة تعتمد على pip فقط تشغيل أجهزة DAQ من البداية إلى النهاية. |

### وضع الخلفية البعيدة (مضيف يعمل بـ pip فقط، عبر نفق)

> **لا يمكن الوصول إلى الخلفية المرفقة عبر شبكة LAN.** إصدارات
> الإنتاج تربط وضع loopback فقط (كلا عائلتي loopback) وترفض رفضًا قاطعًا
> الوضع الوحيد غير loopback (`CHLOROS_CLOUD_MODE`)، لذا
> **لا يمكن لـ `backend_url="http://<lan-ip>:5000"` العمل مع
> Chloros** — هذا النمط لم يعمل أبدًا إلا مع الخلفية المصدر/التطويرية
> (source/dev). لتشغيل خلفية على جهاز آخر، قم بتوجيه منفذ loopback الخاص بها
> بنفسك ووجه SDK إلى النفق:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

يمكن للمضيفات التي لا تحتوي على شاشة / CI / الروبوتية الاحتفاظ بجهاز واحد مزود بتثبيت سطح المكتب الكامل باعتباره «خادم Chloros» و`pip install chloros-sdk` في كل مكان آخر — لكن النقل بينهما يتم عبر النفق المذكور أعلاه الذي يرتبه المستخدم، وليس عبر اتصال LAN مباشر URL.

> **قيود معروفة — لا يدعم `ChlorosLocal` pip فقط.** يقوم `ChlorosLocal(backend_url=BACKEND)` حاليًا بحل ملف ثنائي للخلفية المحلية في منشئه *قبل* فحص URL، ويُصدر استثناء `ChlorosBackendError` («لم يتم العثور على الخلفية Chloros…») في حالة عدم تثبيت أي حزمة سطح مكتب — حتى مع وجود خلفية بعيدة يمكن الوصول إليها. فقط واجهة الاتصال الذكي المذكورة أعلاه (`connect_camera` / `connect_array` / `connect_daq_sensor`، بالإضافة إلى `analyze_array_network` والمساعدات `list_*` / `discover_*`) تعمل من مضيف يعمل بنظام pip فقط.

### سير العمل المخصص لـ DAQ فقط (مضيف يعمل بـ pip فقط)

إذا كنت تحتاج فقط إلى مستشعرات DAQ ولا تستخدم كاميرات LATTICE أو معالجة الصور، فإن حزمة pip تكون مكتفية بذاتها:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

لا حاجة إلى خلفية (backend) أو ملف .deb أو تسجيل الدخول عبر Chloros+ للعمل المباشر مع أجهزة DAQ.

---

## البدء السريع

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## فهرس المستوى الأعلى لـ API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## معالجة الصور — `ChlorosLocal`

فئة خط الأنابيب الرئيسية. تقوم بتشغيل الخلفية عند الاستخدام الأول، وإنشاء/تكوين المشاريع، ومراقبة التقدم، وإرجاع ملخصات ما بعد التشغيل.

### منشئ

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

### الطرق

| الطريقة | الوصف |
| --- | --- |
| `create_project(project_name, camera=None)` | إنشاء مشروع جديد (اختياريًا باستخدام قالب كاميرا مثل `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | استيراد صور بتنسيقات RAW/TIF/JPG/DNG **وتسجيلات مستشعر الضوء `.daq`**. تُرجع `count` (الصور) و `scan_count` (التسجيلات). تُصدر تحذيرًا فقط إذا لم يحتوي المجلد على أي منهما. |
| `export_light_sensor(daq=True, csv=True)` | كتابة ملفات `.daq` + `.csv` المعايرة لكل تسجيل لمستشعر الضوء في المشروع، في ملف `<project>/Light Sensor/`. انظر [تسجيلات مستشعر الضوء](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | ضبط أزرار المعالجة. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | تشغيل مسار المعالجة. يُرجع `{"status": "complete", "async": False}`، بالإضافة إلى مفتاح `summary` عندما توفره الخلفية — انظر [ملخص ما بعد التشغيل والتلميحات](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | فحص حالة الخلفية. |
| `logout()` | مسح بيانات الاعتماد المخزنة مؤقتًا. |
| `shutdown_backend()` | إنهاء تشغيل الخلفية (إذا تم بدء التشغيل باستخدام SDK). |
| `discover_cameras()` | اكتشاف كاميرات LATTICE **عبر الخلفية الخاصة بهذه المثيل** (`/api/camera/discover`). تُرجع قائمة من القواميس (`serial`، `model`، `ip`، …) — بنفس الشكل الذي تراه واجهة المستخدم الرسومية/CLI. قائمة فارغة إذا لم يتم العثور على أي كاميرات أو إذا كانت الخلفية غير قابلة للوصول. |
| `camera_capture(output_dir, format="tiff", **settings)` | التقاط إطار واحد**عبر الخلفية**(يتم تشغيله تلقائيًا بواسطة هذا المقبض) بحيث يحصل على نفس الإعدادات مثل واجهة المستخدم الرسومية/ CLI (الافتراضي 12 بت، إعادة استخدام المجمع، وبيانات تعريف التوقيت المضمنة). حدد الهدف باستخدام `serial=` أو `device_index=`؛ وقم بتمرير `exposure`/`gain`/`pixel_format`/`preset` كـ `**settings`. يُرجع قاموس البيانات الوصفية القديم (`filepath`، `width`، `height`، `pixel_format`، `exposure_time`، `gain`، `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | إنتاج إطارات معاينة مركبة بالتراكب من كاميرا مجمعة — عميل MJPEG خفيف عبر مسار `/api/camera/<serial>/stream-annotated` الخاص بالخلفية (الزيبرا / الشبكة / التقاطع / الرسم البياني / الذروة / النقطة المرسومة من جانب الخادم). ينتج `decode=True` مصفوفات BGR؛ بينما ينتج `False` بايتات «JPEG» الخام. يمكن الوصول إليها أيضًا لكل مشروع باسم `ChlorosProject.stream(overlays=True)`. |

استخدم كمدير سياق لضمان التنظيف:

```python
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

### تسجيلات مستشعر الضوء — `.daq` + `.csv` المعايرين

يمكن تسجيل جهاز DAQ-U / DAQ-M / DAQ-E **بدون** حزمة المعايرة الخاصة به. وهذا هو
ما تفعله المسجلات العامة [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) بشكل افتراضي: فهي تسجل قراءات المستشعر الأولية وتضع علامة على
الملف بحيث يقوم موقع Chloros بجلب معايرة المصنع الخاصة بذلك المستشعر **عن طريق الرقم التسلسلي** — ذاكرة التخزين المؤقتة المحلية
أولًا، ثم سحابة MAPIR — وتطبقها عند الاستيراد.

يقوم Chloros بإعادة كتابة النتيجة في شكل منتجين لكل تسجيل، تحت
`<project>/Light Sensor/`:

| المنتج | ما هو |
| --- | --- |
| `<name>_calibrated.daq` | الأرشيف القابل لإعادة المعالجة — بنفس مخطط التسجيل المباشر، مع الإشارة الآن إلى الحزمة التي أنتجته. إعادة استيراده **لا** إعادة معايرته للمرة الثانية. |
| `<name>_calibrated.csv` | الإشعاع الطيفي بوحدة W/m²/nm على شبكة الأطوال الموجية الخاصة بالمستشعر، صف واحد لكل قراءة، بالإضافة إلى أعمدة قياس الضوء (الطاقة الإجمالية، اللوكس الضوئي/الظلامي، PPFD وتقسيمه إلى الأزرق/الأخضر/الأحمر، طول الموجة الذروي). |
| `<name>_raw.daq` / `<name>_raw.csv` | **أجهزة الاستشعار غير المجمعة فقط (DAQ-A).** قراءات الطيف الأولية لجهاز الاستشعار — *وليس* الإشعاع. انظر أدناه. |

يقوم `process()` بتنفيذ هذا التصدير كإحدى مراحله. **لا** يتطلب صورًا:
فمستشعر الضوء الذي يُحمل بمفرده يمثل سير عمل من الدرجة الأولى، ومثل هذا المشروع لا يحتوي على أي
صور بحكم تصميمه.

**يتم تصدير تسجيلات DAQ-A كعدد خام.** تعود عائلة DAQ-A إلى ما قبل نظام الحزم
الخاص بكل رقم تسلسلي ولا تحتوي على حزمة يجب جلبها — بل يتم معايرتها ميدانيًا مقابل
هدف انعكاس، ولهذا السبب لم تحتج أبدًا إلى حزمة. يتم تصدير هذه التسجيلات
تحت جذع `_raw` بدلاً من `_calibrated`: اسم ملف مختلف بدلاً من علامة
داخل الملف، لأن المعلومة يجب أن تبقى سليمة عند إرسالها عبر البريد الإلكتروني كاسم عاري. ويُظهر رأس
الملف `.csv` إلى `raw spectral sensor counts (NOT irradiance)` ويحذر من أن
القيم قابلة للمقارنة **داخل** الملف — وهو بالضبط ما تستخدمه المعايرة القائمة على الهدف
— وليس عبر أجهزة الاستشعار. الأعمدة الضوئية المعتمدة على الطاقة (الطاقة الإجمالية،
اللوكس الفوتوبي/السكوتوبي، PPFD) تظهر **NULL** بدلاً من أن يتم حسابها من القيم المجمعة.

أما أجهزة DAQ-U / DAQ-M / DAQ-E التي تعذر استرداد حزمة البيانات الخاصة بها، فلا تزال **يتم تخطيها**،
وليس كتابة البيانات الخام: ففي هذه الحالة تكون الحزمة موجودة ويُعد «إعادة الاتصال وإعادة المعالجة» نصيحة حقيقية.

التسجيلات القديمة **v1.01 / v1.02** (تقوم وحدة DAQ-A-SD بكتابتها) لا تحتوي على حقبة لكل قراءة،
بل تحتوي فقط على وقت كتابة الملف. ولا يزال برنامج مطابقة الصورة↔الضوء المتدفق إلى الأسفل يرفضها — فمطابقة
إطار مع وقت الكتابة سيكون خطأً غير مرئي — لكن أداة التصدير تقرأها، و
CSV تطبع `clock=daq_created_on` بحيث يحدد المنتج الساعة التي يعمل عليها.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

يتم الإبلاغ عن التسجيل الذي يتعذر استرداد حزمة المعايرة الخاصة به (في وضع عدم الاتصال، أو مستشعر بدون
معايرة في الملف) تحت الرمز `skipped` **مع ذكر السبب**. ولا يتم
كتابته أبدًا كملف «معاير» يحتوي على القيم الأولية — اتصل بالإنترنت و
أعد التشغيل، وستكتمل عملية التصدير.

### استدعاءات التقدم

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### ملخص ما بعد التشغيل والإرشادات

عند الانتهاء، يقوم `process()` باسترداد `GET /api/processing-summary` وإرفاق النص الأساسي كملف `result["summary"]`. تتم عملية الاسترداد على أساس «أفضل جهد ممكن» ولا تعيق أبدًا العودة الناجحة — إذا كان الملخص غير متاح، فإن `process()` يعود إلى الشكل البسيط `{"status": "complete", "async": False}`. كل إدخال في `summary["hints"]` — جمل كاملة مع الإجراء التصحيحي المقترح، على سبيل المثال لماذا أسفرت عملية التشغيل عن ناتج صفر — يُعاد إصداره أيضًا كـ Python `UserWarning`، لذا فإن عمليات التشغيل ذات الناتج الصفر تُشخص نفسها بنفسها حتى لو لم تفحص القاموس أبدًا:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` هو النصف القابل للقراءة آليًّا:

| المفتاح | ما يُحسب |
| --- | --- |
| `models` | مجموعات الكاميرات في الجولة. |
| `images_in_groups` | الصور المصدر عبر تلك المجموعات. |
| `targets_found` | أهداف الانعكاس المكتشفة. |
| `images_calibrated` | الصور التي تمت معايرتها خلال الجولة. |
| `exported_files` | **ملفات منتجات الصور التي أنشأتها الجولة.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | تسجيلات مستشعر الضوء، تم عدها بشكل منفصل عن قصد — فهي تأتي من مرحلة مختلفة وتوجد في عمليات التشغيل التي لا تحتوي على صور على الإطلاق، لذا فإن دمجها سيجعل عملية التشغيل التي تعتمد على DAQ فقط تبدو وكأنها تصدر. |

إلى جانبها: `summary["output_dirs"]` (كل دليل تمت الكتابة إليه)،
`summary["light_sensor_export"]`، `summary["stopped"]` (تكون قيمتها «صحيح» عندما يقطع المستخدم
التشغيل، بحيث لا تُعتبر عمليات العد الجزئية بمثابة تشغيل مكتمل لم يُنتج العدد المطلوب)، و
`summary["groups"]` (التفصيل حسب المجموعة).

يتم تسجيل `exported_files` بواسطة خط الأنابيب **أثناء الكتابة**، وليس من خلال المسح
لكائنات الصور الخاصة بالمشروع بعد ذلك. تقوم استراتيجيات التشغيل المتوازي ووحدة معالجة الرسومات (GPU) بإنشاء كائنات الصور الخاصة بها
(في العمليات الفرعية للعامل بالنسبة لمسارات وحدة معالجة الرسومات)، لذا كان المسح القديم يُبلغ عن
`0 file(s) written` لكل عملية من هذا النوع ثم أصدرت تلميح «صفر تصدير» — في عمليات
كان كل شيء فيها يعمل بشكل صحيح. إذا قمت بكتابة برنامج نصي بناءً على هذا الرقم، فإن عملية التشغيل المتوازي السليمة الآن
تُبلغ عن عدد غيرصفر.

تتخطى تقارير تخطي مستشعر الضوء السبب الذي حدده القارئ فعليًّا لكل ملف — مثل
مخطط غير قابل للقراءة، أو حزمة مفقودة، أو خطأ في الكتابة — **بعد إزالة التكرارات**، لذا فإن العشرين ملفًا
التي تم تخطيها لسبب واحد تُقرأ كسبب واحد بدلاً من عشرين تكرارًا له.

> **لا يتم إثارة `process()` عندما لا ينتج التشغيل أي صور.** هذا هو المكان الوحيد الذي يختلف فيه SDK و
> CLI عن عمد: `chloros-cli process` يعامل &quot;تم طلب منتجات، ولم يتم
> كتابة أي منها&quot; على أنه فشل ويخرج بقيمة غير صفرية، في حين أن SDK يعود بشكل طبيعي ويبلغ عن
> الحالة من خلال `summary` / hints. إذا كان من المفترض أن يتوقف مسار العمل الخاص بك عند عملية تشغيل فارغة، فتحقق من ذلك
> بنفسك — افحص `summary` (أو احسب عدد الملفات الموجودة في مجلد المشروع) بدلاً من الاعتماد على
> عدم وجود استثناء. الأسباب المعتادة هي مجلد إدخال لم يتم التعرف عليه على أنه
> التقاط، وتخطي المنتجات باعتبارها غير قابلة للتطبيق على الكاميرات الموجودة (على سبيل المثال، الإشعاع من الكاميرات التي تعمل بنظام &quot;RGB&quot; فقط
>).

### وظائف ملائمة

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### القيم المدعومة

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### المخرجات الإشعاعية (خط أنابيب LATTICE متعدد الأطياف)

مستوى التصدير متعدد الأطياف (M3C/M3M) لخط أنابيب LATTICE التابع لـ `process` — `reflectance` (الافتراضي)، أو `radiance`، أو `sensor-response`، أو `all` (كل وضع قابل للتطبيق لكل صورة) — يُربط بإعداد معالجة **&quot;الإخراج الإشعاعي&quot;** الخاص بالمشروع. ويحتوي `configure()` على كلمة رئيسية مخصصة له:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

لا تزال طريقة الهروب المتقدمة — كتابة مفتاح المشروع `"Radiometric output"` عبر `custom_settings` — تعمل، ولكن تذكر أنها تستبدل كتلة الإعدادات بأكملها (انظر التحذير أدناه):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (الافتراضي) يقسم إشعاع الكاميرا على **تدفق DAQ المتجه لأسفل المتطابق مع الطابع الزمني**، والذي يتم تحديده تلقائيًا من `.daq` (DAQ-U/M/E) المسجل**أو `.csv` الأصلي لـ DAQ-M**الموجود بجانب الصور؛ ويتم**جلب أي حزمة معايرة خاصة بالكاميرا أو DAQ مفقودة محليًّا تلقائيًّا من AWS** عند الاستخدام الأول. يعرض CLI ذلك على شكل خيارات تبديل للمنتج حسب النوع في `chloros-cli process`: `--radiance`/`--no-radiance`، `--reflectance`/`--no-reflectance`، `--debayered`، `--preview`.

> `custom_settings` **يحل محل** كتلة الإعدادات المحسوبة بأكملها (وهو يتجاوز الكلمات الرئيسية الأخرى لـ `configure()` والتحقق من الصحة حسب التصميم). عند استخدامه، قم بتضمين كل مفتاح `Project Settings` يهمك، كما في المثال أعلاه.

---

## Smart-Connect لكاميرات LATTICE

جلسات خلفية دائمة للأجهزة الحية. نفس نقاط النهاية التي تستخدمها واجهة المستخدم الرسومية (GUI)، لذا يكون السلوك متطابقًا عبر SDK / CLI / واجهة المستخدم الرسومية (GUI).

### كاميرا واحدة — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### التوقيع الخاص بـ `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### طرق `CameraSession`

| الطريقة | الوصف |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | قراءة عقد GenICam؛ تُرجع `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | كتابة العقد بالاسم المألوف (`exposure_time`، `gain`، `pixel_format`، `width`، `height`، `target_brightness`، `ae_damping`، `ae_upper_limit`، `trigger_mode`، `trigger_source`، …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | التقاط **إطار واحد**. تُرجع قائمة مكونة من عنصر واحد من قواميس بيانات الإطار. (تمت إزالة التقاط المتتالي/متعدد الإطارات — استدعِ `capture()` في حلقة إذا كنت بحاجة إلى سلسلة.) |
| `disconnect()` | التحرير من المجمع. لا تؤدي أي عملية إذا كنا مرتبطين بجلسة مفتوحة بالفعل. |

ضوابط التصدير في `capture()` (نفس نموذج المصفوفة + واجهة المستخدم الرسومية):

- `processing` / `levels` — `processing="all"` يحفظ كل أنواع التصدير القابلة للتطبيق؛ `levels=["raw","radiance"]` يحفظ فقط تلك (يتجاوز إعدادات `processing`). قم بحذف كليهما لاستخدام الإعداد الافتراضي للخلفية.
- `force_daq=True` — حفظ قراءة DAQ/DLS المعينة كملف جانبي `.daq` حتى في حالة التقاط البيانات الخام فقط، بحيث يمكن إعادة معالجة الإطار لاحقًا إلى انعكاسية/مؤشر انكسار. لا يتم تنفيذ أي عملية إذا لم يكن DAQ مرتبطًا.

### المصفوفة المتزامنة — `ArraySession` (Smart-Prep)

`connect_array` هو **نقطة الدخول الموصى بها** لإعدادات الكاميرات المتعددة. وهو يقوم بتشغيل تدفق Smart-Prep الكامل لواجهة المستخدم الرسومية (GUI) في الخلفية:

1. **تحليل الشبكة** (`/api/camera/array/recommend`) — يحدد أكبر حجم للإطار الذي يتناسب مع مستوى sim-emit دون فقدان أي إطارات.
2. **الاختيار التلقائي للمستوى** — `sim-capture-sim-emit` إذا كان خط الاتصال قادرًا على تحمله؛ وإلا فإن `sim-capture-ftd-stagger` أو `slip-emit-and-capture`.
3. **التقليص التلقائي**— يقوم بتقليص حجم الإطار / زيادة تجميع الإطارات بصمت عندما لا يستطيع خط الاتصال تحمل الدقة المطلوبة.**شبكة الأمان هذه لا تغطي الإفراط التراكمي في الاشتراك**: لا يمكن حل مشكلة وجود عدد كبير جدًا من الكاميرات بالنسبة لخط الاتصال عن طريق تقليص الإطارات — انظر [الإفراط في الاشتراك](#over-subscription-the-per-cam-floor).
4. **تم تمكين PTP** افتراضيًا — الطوابع الزمنية عبر الكاميرات قابلة للمقارنة بدقة تصل إلى أجزاء من الميكروثانية.
5. **الاختيار التلقائي لتنسيق البكسل لكل كاميرا** — الكاميرات ذات التكوين «RGB» → `BayerRG8`، الكاميرات متعددة الأطياف → `BayerRG12`.
6. **تحديد القيمة الأولية للتعرض التلقائي (AE)** — التقاط لقطات لحالة التعريض التلقائي (AE) الحالية لكل كاميرا حتى لا تؤدي عملية الاتصال إلى إعادة ضبط التعريض أثناء التشغيل.
7. **تكوين مشغل GPIO** — يقوم `connect_array` بتجهيز كل كاميرا (`TriggerMode=On`، `TriggerSource=Line2`) بحيث تدفع نبضة الوحدة الرئيسية الوحدات التابعة عبر كابل M8. هذه خطوة خاصة بالمصفوفات فقط: يتم تم فتحها باستخدام `LatticeCamera` تعمل بشكل حر بدلاً من ذلك.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### توقيع `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

قيم `force_tier`:
- `"sim-capture-sim-emit"` — تزامن حقيقي (تلتقط جميع الكاميرات الصور عند نفس حافة الساعة).
- `"sim-capture-ftd-stagger"` — تداخل مرن في المجال الزمني (تُرسل الكاميرات في أوقات متباعدة قليلاً بحيث يتم تسلسل الحزم على خط الاتصال).
- `"slip-emit-and-capture"` — التقاط متسلسل لكل كاميرا (بدون مزامنة زمنية؛ الخيار الوحيد عندما لا يتناسب حجم الإطار مع التزامن).

`wire_ceiling_mbps` يتجاوز **ميزانية السلك المستدامة للمضيف** بالميغابايت في الثانية — الرقم
الوحيد الذي يعتمد عليه تخصيص المصفوفة بأكملها. اتركه على `None` لاستخدام القيمة المكتشفة تلقائيًا
القيمة المكتشفة تلقائيًا. قم بخفضها عندما تبلغ المصفوفة عن إطارات تالفة وفقًا لـ GVSP: فالقيمة التلقائية مشتقة
من معدل الارتباط المعلن لبطاقة NIC، والذي يبالغ في تقدير محولات USB ومسارات PCIe الرفيعة و
البنى المشتركة المزدحمة — ويظهر هذا التقدير الزائد على شكل إطارات تالفة بدلاً من
وصلة بطيئة بشكل واضح. يتم حفظ القيمة في كتلة التقاط المصفوفة الخاصة بالمشروع، لذا فإن
إعادة فتحها أو استخدام `connect_array` لاحقًا يعيدها مثل أي إعداد آخر للمصفوفة.
انظر [صحة المصفوفة](#array-health--which-subsystem-is-losing-frames).

#### الاكتتاب الزائد (الحد الأدنى لكل كاميرا)

تقوم تقنية Sim-emit بتخصيص حصة لكل كاميرا من ميزانية السلك الآمنة من التصادم، بحد أدنى **8 ميغابايت/ثانية لكل كاميرا**(`per_cam_floor_bps`). بمجرد أن يتجاوز `N × floor` الحد الأقصى الآمن من التصادم، تقوم المصفوفة**بالتسجيل الزائد على السلك**— ووضع الفشل هو فقدان حزم GVSP، وليس انخفاض معدل الإطارات — ولا يوجد حل يتعلق بحجم الإطار:**يقلل التجميع ومنطقة الاهتمام (ROI) من عدد البايتات لكل إطار، وليس عدد البايتات المُوزَّعة في الثانية**التي تقارنها عملية الفحص الإجمالية. الحدود القصوى العملية للدقة الكاملة على مضيف 1 جيجابت إيثرنت:**6 كاميرات بـ 1500 MTU، 9 منها بإطارات جامبو** (يُبلغ `max_cams_collision_safe` في استجابة التحليل عن الحد الأقصى للخط الخاص بك). الحلول: عدد أقل من الكاميرات، أو استخدام إطارات جامبو من طرف إلى طرف، أو بطاقة شبكة أسرع.

- تحتوي استجابات `analyze_array_network()` و `/api/camera/array/connect` على `oversubscribed` و `aggregate_demand_bps` و `collision_safe_ceiling_bps` و `max_cams_collision_safe` و `per_cam_floor_bps`. وعندما تكون قيمة `oversubscribed` صحيحة، يقوم الإسقاط **بإعادة تعيين حقول الإطارات في الثانية (fps) إلى الصفر** (`achievable_fps_max` / `fps_bright` / `fps_dark`) بدلاً من الإبلاغ عن معدل مضلل يُظهر أن النظام يعمل ببطء.
- يقبل `POST /api/camera/array/connect` يقبل معلمة نصية `pin_resolution` (**HTTP فقط — وليست معلمة kwarg من نوع SDK**؛ ولا يكشف `connect_array` عنها). يؤدي التثبيت إلى إزالة شبكة الأمان الخاصة بالتنازل التدريجي في التجميع، لذا فإن أي اتصال مفرط الاشتراك مع تعيين `pin_resolution` يتم**رفضه بشكل قاطع** مع ظهور خطأ يحدد كل الحلول الممكنة. وبدون التثبيت، يستمر الاتصال في عملية التخفيض التدريجي ولكنه يحذر من أن التقليص لا يمكنه مسح المجموع.
- مخرج طوارئ للعمل التجريبي: اضبط `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` في بيئة الخلفية لتخفيف الرفض إلى تحذير صاخب — يمكنك الاتصال على أي حال وقبول فقدان الحزم.

#### حالة المصفوفة — أي نظام فرعي يفقد الإطارات

يحمل `GET /api/camera/array/<array_id>/capability` كتلة `health` نشطة على
مصفوفة متصلة، يتم إعادة تقييمها في نافذة متجددة **مدتها 10 ثوانٍ**. وهي تقسم فقدان الإطارات
إلى سببين يتطلبان إصلاحات متعاكسة، بدلاً من معدل «غير مكتمل» واحد لا
يحدد أيًّا منهما:

| الحقل | المعنى | النظام الفرعي |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (لكل مسلسل) | **وصل الإطار وكان تالفًا من الناحية الهيكلية**— فقدان حزمة GVSP. |**الشبكة**: سعة السلك، وتيرة الإرسال، حلقة استقبال بطاقة الشبكة، وحدة MTU |
| `never_arrived_rate_pct` (لكل مسلسل) | **لم يصل الإطار على الإطلاق**— لم يتم تشغيل الكاميرا، أو لم يخرج منها أي شيء. |**المشغل / المزامنة**: كابل M8، `line=`، `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | أسوأ معدل للكاميرا لكل منها. | — |
| `per_cam_rate_pct` | المعدل الإجمالي لعدم اكتمال التسجيل لكل كاميرا (كلا السببين معًا). | — |
| `stable_for_seconds` | المدة التي بقيت فيها كل كاميرا دون 0.01٪. | — |

إلى جانب `health`، يُشير السجل نفسه إلى عدد المرات التي يتوقف فيها التخصيص بالكامل:

| الحقل | ما |
| --- | --- |
| `wire_ceiling_mbps` | ميزانية النطاق الترددي المستدامة السارية للمضيف، ميغابايت/ثانية. |
| `wire_ceiling_source` | مصدر هذا الرقم، بالكلمات — على سبيل`USB-capped 200 MB/s (was theoretical 1062; …)` أو `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` عندما يقوم `wire_ceiling_mbps=` قام بتعيينه. |
| `nic_is_usb` | `true` لمحول إيثرنت USB. |

لا يوجد غلاف SDK لهذه النقطة النهائية — اقرأها مباشرةً:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**قراءتها:** قيمة `gvsp_corrupt_rate_pct` غير صفرية مع وجود `never_arrived_rate_pct` عند 0 تعني
أن التشغيل ومزامنة الكابل مثاليان وأن 100% من الفقدان يقع على مسار الشبكة — قم بخفض
`wire_ceiling_mbps` وأعد الاتصال. أما النمط العكسي فيشير إلى كابل المزامنة أو
خط التشغيل بدلاً من ذلك.

> **`target_fps` ليس العامل المسبب للإطارات التالفة.** يتم كتابة وتيرة GevSCPD مرة واحدة عند
> الاتصال، لذا فإن خفض معدل التشغيل يغير دورة العمل وليس
> معدل اندفاع الإرسال المتزامن. لم ينتج عن خفض الطلب المقاس بمقدار 5× أي تحسن، بينما
> خفض السقف الأقصى للسلك من 240 إلى 200 ميغابايت/ثانية خفض نسبة الإطارات التالفة في نفس الجهاز من 10.4% إلى
> 0.00%.

> **التقلص التلقائي في منتصف التدفق غير متاح في البرنامج الثابت TRI032S.** لا يمكن للمصفوفة قيد التشغيل
> إصلاح هذا بنفسها؛ افصلها وأعد توصيلها حتى يعيد محدد وقت الاتصال التخطيط وفقًا
> للسقف الجديد.

**يتم تحديد سقف محول إيثرنت USB بـ 200 ميغابايت/ثانية** بواسطة أداة الفحص بغض النظر عن
الملصق المرفق به: جدول الكفاءة الذي يحول معدل الارتباط إلى رقم مستدام مشتق من
PCIe، وتعلن بطاقة شبكة USB عن معدل ارتباط إيثرنت الخاص بها بينما تكون مقيدة بـ
ناقل USB وبرنامج التشغيل الخاص به. الحد الأقصى هو قيمة مطلقة، وليس نسبة — محول USB 1 جيجابت إيثرنت
يستمد ~80 ميجابايت/ثانية ولا يتأثر بذلك.

#### طرق `ArraySession`

| الطريقة | الوصف |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | مجموعة التقاط واحدة متزامنة. تُرجع `CaptureResult` (قائمة بقواميس الإطارات + `.skipped`). عناصر التحكم في التصدير أدناه. |
| `capture(..., smart=True)` | **التصوير الذكي** — ينتظر استقرار AE عبر جميع الكاميرات، ثم يبدأ التصوير. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | أسرع التقاط: الخام فقط + قراءة DAQ المخصصة (+ الفهرس المدمج المجاني). يعكس زر «أسرع التقاط» في واجهة المستخدم الرسومية. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | مرة واحدة / مستمر / بفواصل زمنية في حلقة واحدة محدودة. تُرجع `list[CaptureResult]`.**يتطلب `count` و/أو `duration_s`** حتى يتم إنهاؤه (لا يتوفر زر Ctrl+C في &quot;SDK&quot;). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | بدء تسجيل عرض المؤشر المدمج المباشر إلى فيديو/GIF → `RecorderHandle`. مسجل مركب واحد لكل مصفوفة. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | بدء سلسلة لقطات باير الخام عالية معدل الإطارات في الثانية → `RecorderHandle`. إعادة المعالجة دون اتصال بالإنترنت باستخدام `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | إعادة معالجة سلسلة لقطات «راو» المحفوظة في وضع غير متصل بالإنترنت لتحويلها إلى مقطع (مقاطع) فيديو مُعاير. يتم التعليق حتى الانتهاء (`wait=True`) ثم إرجاع `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | استقصاء مهمة إنشاء غير متصلة بالإنترنت: `{running, result, error, burst_dir}`. |
| `disconnect()` | تحرير المصفوفة بأكملها. |

`capture()`ضوابط تصدير  (نفس نقطة النهاية التي تستخدمها واجهة المستخدم الرسومية/ CLI):

- `processing` / `levels` — `processing="all"` (أو `levels=["raw","radiance",…]`) يحفظ كل نوع تصدير قابل للتطبيق لكل كاميرا؛ قيمة `processing` واحدة تحفظ هذا المستوى فقط.
- `aligned=True` — تحويل كل تصدير غير خام لكل عضو إلى [ملف تعريف المحاذاة](#array-alignment) للمصفوفة (مسجل بشكل مشترك)؛ بينما يبقى التصدير الخام غير محولولكنها تحمل التحويل في البيانات الوصفية. يتم الرجوع إلى الوضع غير المُحاذي (مع ظهور تحذير في `alignment` الخاص بالنتيجة) إذا لم يكن للمصفوفة ملف تعريف.
- `render_index=False` — تخطي تراكب مؤشر الغطاء النباتي لكل كاميرا؛ يقوم الإعداد الافتراضي يعرضه حيثما تم تكوينه.
- `force_daq=True` — حفظ قراءة DAQ/DLS المخصصة كملف جانبي `.daq` حتى في حالة عدم احتياج أي مستوى مختار لها.

**ضغط TIFF (مفتاح HTTP-only):** لا يرسل `ArraySession.capture()` مفتاح `compression`، لذا يتم تطبيق الإعداد الافتراضي للخلفية — يقرأ `POST /api/camera/array/capture` معلمة نصية `compression`، `"deflate"` بشكل افتراضي (zlib L1 بدون فقدان البيانات + متنبئ أفقي، ~4.1 ميغابايت لكل إطار بدقة كاملة). يكتب `"none"` البيانات غير المضغوطة (~6.3 ميغابايت/إطار) بـ**سرعة كتابة أسرع بحوالي 5 أضعاف** — وكلاهما خاليان من الفقدان ويتم قراءتهما بشكل متطابق عند الاستيراد. لا تكشف وظيفة «SDK» عن أي معلمة kwarg لها؛ المخرج البديل هو `chloros-cli lattice array-capture --compression none` أو HTTP الخام. كما تحتفظ DEFLATE بـ GIL الخاص بـ «Python»، لذا لا تتم عمليات الكتابة المضغوطة بالتوازي عبر خيوط الكتابة لكل كاميرا — يتطلب التقاط مستمر بدقة كاملة لـ 8كاميرا بدقة كاملة بمعدل المستشعر تتطلب `compression: "none"`. التفاصيل: [CLI مرجع → array-capture](cli-reference.md).**تجاوزات التصدير لكل عضو (HTTP فقط):**تقبل نفس نقطة النهاية أيضًا `exclude_serials` (قائمة — استبعاد العناصر من المجموعة المحفوظة؛ لا يزال المصفوف يعمل كمجموعة واحدة متزامنة ويتم إرجاع العناصر المستبعدة في `excluded`)، و`serial_levels` (`{serial: [level tokens]}` تجاوزات على مستوى كل كاميرا)، و `serial_index` (`{serial: bool}` تجاوزات تراكب الفهرس لكل كاميرا). هذه معلمات نصية متطابقة مع واجهة المستخدم الرسومية و**ليست معلمات kwargs لـ SDK بعد**؛ أما العناصر الغائبة عن الخرائط فتعود إلى `levels` / `render_index` على مستوى المصفوفة.

##### فحص الكاميرات التي تم تخطيها — `CaptureResult.skipped`

تُرجع `ArraySession.capture()` قيمة `CaptureResult`، وهي فئة فرعية من `list`: قم بتكرارها، وفهرستها، واستخدام `len()` عليها — كل الأنماط الموجودة تظل تعمل. يمكن للكود الجديد فحص سمة `.skipped` لمعرفة الكاميرات التي تم استبعادها ولماذا. الحالة الأكثر شيوعًا هي الكاميرات RGB في مصفوفة مرشحات مختلطة عند طلب `processing="radiance"` أو `"reflectance"` — حيث إن الإشعاع لكل باير لا معنى له بالنسبة لمستشعر النطاق العريض، لذا يتخطى النظام الخلفي تلك الكاميرات بدلاً من إنتاج بيانات غير منطقية.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

تتبع رموز الأسباب النمط `<level>-not-applicable-to-rgb-cam` (إدخال واحد لكل مستوى تم تخطيه، يحمل كل منها `level`). حالات التخطي الخاصة بالانعكاس هي `reflectance-skipped-no-fresh-dls` (لا توجد قراءة حديثة للضوء المتدفق لأسفل متاحة)، و`reflectance-skipped-bound-daq-unavailable (…)` (تعذر الوصول إلى وحدة DAQ المرتبطة)، و`dls-uncalibrated-band-<nm>` — حيث يقع النطاق في الغالب خارج النطاق المعاير إشعاعيًّا لمستشعر الضوء في وحدة DAQ (~374–974 نانومتر)، لذا يتم رفض التقسيم المطلق للانعكاسية القائم على وحدة DAQ ويتم تخفيض الإطار بشكل ملحوظ إلى استجابة المستشعر. من بين وحدات التخزين (SKUs) المتوفرة، لا يُشغّلها سوى طراز F988؛ والمسار المدعوم لتلك الكاميرا هو سير عمل لوحة الانعكاسية.

مستويات `processing`:

| المستوى | المخرجات |
| --- | --- |
| `"raw"` | باير أحادي القناة (الكاميرات أحادية اللون: النطاق الفردي) مباشرة من المستشعر. |
| `"debayered"` *(الإعداد الافتراضي لـ SDK)* | BGR ثلاثي القنوات عبر إزالة التباين الثنائي الخطي (الكاميرات أحادية اللون: مقياس رمادي أحادي القناة). |
| `"radiance"` | float32 W/م²/سر/نانومتر عبر السلسلة الإشعاعية الكاملة. متعدد الأطياف فقط — يتم تخطي الكاميرات RGB. |
| `"reflectance"` | uint16 0..32768 (جاهز لـ Pix4D)؛ يتطلب إقرانًا مباشرًا مع DAQ للحصول على مرجع مطلق. متعدد الأطياف فقط. |
| `"display"` | سلسلة كاملة تتطابق مع معاينة واجهة المستخدم الرسومية (CCM + WB + غاما وفقًا لملف تعريف الكاميرا). |
| `"all"` | **ملف واحد لكل مستوى قابل للتطبيق** لكل كاميرا (مطابقًا للإعداد الافتراضي لـ «Capture All» / CLI في واجهة المستخدم الرسومية). ثم يحتوي `CaptureResult` الذي يتم إرجاعه على قاموس إطار واحد لكل `(cam, level)`، مع وجود المستوى في كل قاموس؛ أما المستويات غير القابلة للتطبيق فتظهر في `.skipped`. يتم حفظ قراءة DAQ المستخدمة لأي إطار انعكاس كملف جانبي `.daq`. |

> **ملاحظة — يختلف الإعداد الافتراضي عن CLI.** يتم تعيين `ArraySession.capture()` افتراضيًّا إلى `processing="debayered"`؛ ويكون الإعداد الافتراضي للأمر `chloros-cli lattice array-capture` هو `processing="all"`. قم بتمرير `processing="all"` بشكل صريح من SDK لتعكس عملية الحفظ متعددة المستويات في CLI /GUI.

### أوضاع الالتقاط والمسجلات

يعكس سطح المصفوفة لوحة الالتقاط في واجهة المستخدم الرسومية (GUI): أوضاع الغالق الفردي / المستمر / الفاصل الزمني / الأسرع، بالإضافة إلى مسجلين (فيديو مركب مباشر وسلسلة لقطات خام → إعادة معالجة في وضع غير متصل).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**هي حلقة «مستمر/بفواصل زمنية» الخاصة بـ «SDK». ونظرًا لعدم وجود `Ctrl+C` لقطعها من البرنامج النصي،**يجب** تمرير `count` و/أو `duration_s` (تتوقف عند الوصول إلى أي منهما). يتم قياس `interval_s` من بداية كل تمريرة (بما يتوافق مع واجهة المستخدم الرسومية). تمر المعلمات المتبقية مباشرةً إلى `capture()`.
- **`record`** هو *من فئة المراقبة*: فهو يلتقط المركب ذي الفهرس المدمج المباشر كما يُعرض، لذا يجب أن يكون الدفق المدمج مفتوحًا حتى تصل الإطارات. مسجل مركب واحد لكل مصفوفة (يُطلق استثناءً إذا كان هناك مسجل قيد التشغيل بالفعل).
- **`burst` → `build_video`** هو *من فئة التحليل*: `burst` يكتب الإطارات الأولية + بيان لكل إطار + واحد `.daq` لكل قراءة DLS مميزة تحت `<output>/bursts/<base>/` بمعدل كامل لحلقة الالتقاط (بدون تسلسل، وبدون exiftool، وبدون عرض مباشر). يقوم `build_video` بمطابقة توقيت كل إطار مع أقرب `.daq` وإعادة تشغيل سلسلة الإشعاع/الانعكاس/المعامل الضوئي في مسار الاستيراد. `products` هي قائمة بـ `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (الافتراضي: المؤشر المدمج). كما يقوم `burst().stop()` تلقائيًا بتحديد أفضلبناء مؤشر مدمج بأفضل جهد ممكن، ويتم إرجاعه كـ `build_job` في نتيجة التوقف.

#### `RecorderHandle`

يتم إرجاعه بواسطة `ArraySession.record()` و`ArraySession.burst()`. استخدمه كمدير سياق للتوقف التلقائي عند الخروج من النطاق، أو قم بتشغيله يدويًّا.

| العضو | الوصف |
| --- | --- |
| `job_id` | معرّف المهمة الخلفية (sلسلة). |
| `kind` | `"composite"` (من `record`) أو `"raw"` (من `burst`). |
| `start_stats` | القاموس الذي يتم إرجاعه بواسطة استدعاء `start`. |
| `result` | `None` أثناء التشغيل؛ قاموس نتائج الإيقاف النهائي بمجرد التوقف. |
| `stats(timeout=10.0)` | إحصائيات المهمة الحية (الإطارات المكتوبة، معدل الإطارات في الثانية الفعلي، الوقت المنقضي). |
| `stop(timeout=60.0)` | إيقاف المسجل؛ يُرجع النتيجة النهائية ويخزنها في ذاكرة التخزين المؤقت. متكرر (الاستدعاء الثاني يُرجع النتيجة المخزنة في ذاكرة التخزين المؤقت). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### الارتباط بمصفوفة متصلة بالفعل — `attach_array`

إذا كانت المصفوفة قيد التشغيل بالفعل (تم فتحها بواسطة واجهة المستخدم الرسومية، أو استدعت جلسة سابقة من SDK الدالة `connect_array`)، استخدم `attach_array` للحصول على مقبض لها بدلاً من إعادة الاتصال. تظهر دائمًا رسالة الخطأ التالية في `connect_array` &quot;الكاميرا موجودة<sn> بالفعل في المصفوفة<id>&quot; في تلك الحالة، لأن إرسال طلب POST لـ `/array/connect` لعضو فيالمجمع ليس عملية متجانسة؛ حيث يقرأ `attach_array` `/api/camera/array/list` ويقوم بالمطابقة إما عبر array_id أو serials.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

النمط: يجب أن تحاول البرامج النصية SDK التي تعمل بشكل مشترك مع واجهة المستخدم الرسومية لسطح المكتب استخدام `attach_array` أولاً، واللجوء إلى `connect_array` إذا لم يكن هناك مصفوفة في المجموعة بعد.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **هام — يؤدي الخروج من context-manager إلى قطع الاتصال.**يقوم `ArraySession.disconnect()` دائمًا بإرسال طلب POST إلى `/array/disconnect`؛ لا يوجد حارس «مرفق غير مملوك» كما هو الحال مع `CameraSession` / `DAQSensorSession`. إذا كنت تتشارك مع واجهة المستخدم الرسومية (GUI) ولا تريد تفكيك المصفوفة عند الخروج من النطاق،**فلا تستخدم كتلة `with`** — احتفظ بالمقبض في متغير عادي وتخطى `disconnect()` الصريح:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### أداة مساعدة لتحليل الشبكة

مفيدة قبل فتح المصفوفة — تتنبأ بما إذا كانت الإعدادات المقترحة ستناسبك أم لا:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` هو أحد `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (وإلا `error`). `auto_capped_fps` يعني أن الدقة المطلوبة تتناسب مع حلقة RX فقط عند معدل تشغيل محدد — احتفظ بالدقة وقم بتمرير `target_fps=result["recommended"]["recommended_target_fps"]` إلى `connect_array` (انظر [المثال 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**كيفية قراءة الإسقاط** (نفس نموذج لوحة إعدادات المصفوفة في واجهة المستخدم الرسومية):

- **يتم جمع «السلسلة» (`frame_bytes_total`) لكل كاميرا على حدة وفقًا لتنسيق البكسل الفعلي لكل كاميرا.**الكاميرات الأحادية**M3M**تبث Mono12 (2 بايت/بكسل) بغض النظر عن القيمة التي تمررها لـ `pixel_format`، لذا فإن الإطار كامل الدقة المكون من 4 كاميرات يبلغ حجمه**حوالي 25 ميغابايت** مع ثلاث كاميرات أحادية، وليس ~12.6 ميغابايت كما يفترض في حالة استخدام 8 بت بالكامل. تقوم الخلفية بتحديد تنسيق كل كاميرا بناءً على طرازها.
- **القدرة الاستيعابية (`burst_fits_nic_ring`) تراعي معدل التفريغ**، وليس بين «الاندفاع الكامل» و«الحلقة»: حيث يتناسب «الإرسال المتزامن» عندما يفرغ المضيف حلقة الاستقبال (RX ring) أسرع من سرعة ملئها بالكاميرات. المضيف بسرعة 10G + كاميرات بسرعة 1 GbE**يسمح** بالدقة الكاملة حتى عندما يتجاوز الاندفاع سعة الحلقة؛ بينما يقوم المضيف 1 جيجابت إيثرنت بحظرها (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` هو سقف متحفظ للاسترداد التسلسلي** — `max(readout+emit, N×emit)` مع تقييد الإرسال لكل كاميرا بحد أقصى يوازي سرعة وصلة الكاميرا 1 جيجابت في الثانية، بغض النظر عن التعرض. على سبيل المثال، ~2.8 إطار في الثانية لمصفوفة مكونة من 4 كاميرات بدقة كاملة و12 بت (يتطابق مع القيمة المقاسة في وقت التشغيل ~2.7–3.0). النموذج الكامل: [CLI مرجع → نموذج الإطارات في الثانية (fps) للصفوف ونمط الاندفاع](cli-reference.md#array-fps--burst-model).
- **الإفراط في الاشتراك (`oversubscribed: true`) يعني أن الحد الأدنى N × لكل كاميرا يتجاوز الحد الأقصى الآمن من التصادم** — حقول الإطارات في الثانية (`achievable_fps_max` / `fps_bright` / `fps_dark`) تقرأ القيمة 0، ولا يمكن حل المشكلة عن طريق التخفيض التلقائي/التجميع (فهي تقلل عدد البايتات لكل إطار، وليس عدد البايتات الموزعة في الثانية). وتتمثل الحلول في تقليل عدد الكاميرات، أو استخدام إطارات جامبو، أو بطاقة شبكة أسرع؛ ويُبلغ `max_cams_collision_safe` عن الحد الأقصى (6 كاميرات بدقة كاملة على شبكة 1 جيجابت إيثرنت بمعدل 1500 MTU، و9 كاميرات مع إطارات جامبو). كما تتضمن الاستجابة أيضًا `aggregate_demand_bps` و`collision_safe_ceiling_bps`، و`per_cam_floor_bps` (8 ميغابايت/ثانية). انظر [الاشتراك الزائد](#over-subscription-the-per-cam-floor).

### الاكتشاف والإدراج

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

تقوم مصفوفات LATTICE بتشغيل AE بشكل مستمر في الخلفية بمجرد اتصالها، لكن المشهد الذي تم توجيهه حديثًا يستغرق لحظة حتى يتقارب. **Smart-capture** هي ميزة مدمجة مريحة: فهي تستطلع تعريض كل كاميرا، وتنتظر حتى تستقر المصفوفة عبر النافذة، ثم تطلق عملية الالتقاط. وهي تعادل واجهة المستخدم الرسومية: حيث يستدعي زر &quot;الالتقاط الذكي&quot; في تطبيق سطح المكتب يستدعي نفس نقطة النهاية الخلفية.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

عند التشغيل عبر `ChlorosProject` (القسم التالي) تحصل على المزيد من خيارات الضبط:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

تكون سياسة التعريض التلقائي الذكي (smart-AE) متحفظة بشكل افتراضي. قم بتشديد الإعدادات في `exposure_tolerance_pct` للأعمال الإشعاعية التي تتطلب دقة عالية؛ وقم بتخفيفها للمشاهد سريعة التغير حيث تريد فقط «قربًا كافيًا».

---

## جلسات مستشعرات DAQ

مجموعة خلفية دائمة لمستشعرات الطيف (DAQ-U عبر USB، DAQ-M عبر BLE، DAQ-E عبر إيثرنت). تعكس سطح الكاميرا: الكشف الذكي، إعادة استخدام المجموعة، التوصيل المتماثل.

### الكشف الذكي (بدون تكوين)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

الأولوية: إيثرنت → BLE → USB. قم بتمرير أي تلميح صريح واحد لتثبيت وسيلة النقل.

### النقل المثبت

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### طرق `DAQSensorSession`

| الطريقة | الوصف |
| --- | --- |
| `status(timeout=10.0)` | ملخص إدخال المجموعة (حالة البث/التسجيل، نطاق الطول الموجي، معرف التجزئة للمعايرة، وقت التكامل، frame_avg، حالة AE). |
| `latest(n=1, timeout=10.0)` | إرجاع ما يصل إلى N منإطارات الطيف الأحدث. |
| `stream_start()` / `stream_stop()` | استئناف / إيقاف البث مؤقتًا (يظل المقبض مفتوحًا). |
| `record_start(output_dir=None, device_name=None)` | بدء تسجيل ملف .daq. إرجاعمسار الملف. يُرفض في حالة DAQ-U/M بدون حزمة معايرة AWS (يُستثنى DAQ-E مستثنى). |
| `record_stop()` | إيقاف التسجيل. يُرجع `{path, rows}`. |
| `disconnect()` | التحرير من المجموعة. لا تؤدي أي عملية للمقابض المرفقة غير المملوكة . |

> **ملفات تعريف تصحيح الحد الأقصى (`cap_id`) ليست من عناصر التحكم في «SDK».** لا تكشف `connect_daq_sensor()` / `DAQSensorSession` عن أي معلمة `cap_id` أو طريقة `set_cap`. حدد ملف تعريف تصحيح سقف الأسطول عبر CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) أو مسارات HTTP الخاصة بـ `/api/daq` في الخلفية (تقبل `/api/daq/connect` و `/api/daq/<id>/cap-id` المسار `cap_id`).

### الاكتشاف — العثور على عنوان للاتصال به

يقوم `discover_daq_sensors()` بمسح USB / BLE / ETH بحثًا عن أجهزة الاستشعار التي *يمكنك* فتحها. وهو النظير الخاص بـ DAQ لـ `discover_lattice_cameras()`، والطريقة الوحيدة للحصول على **عنوان MAC BLE الخاص بـ DAQ-M** — حيث يمتلك DAQ-E اسم مضيف ويمتلك DAQ-U منفذ COM، لكن عنوان MAC لا يظهر مطبوعًا على الجهاز ولا يرد في قائمة نظام التشغيل.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| الحقل | الوصف |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | منفذ COM / عنوان MAC لـ BLE / اسم المضيف — يتم تمريره إلى `connect_daq_sensor` كـ `port=` / `mac=` / `eth_host=`. |
| `display` | تسمية قابلة للقراءة البشرية. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`، أو `None` بالنسبة لمنفذ لا يستطيع المسح التعرف عليه (لا يمكن تمييز محولات USB التسلسلية بدون جهاز استشعار، لذلك يتم عرض العناصر المجهولة بدلاً من إخفائها). |
| `extra` | تفاصيل كل وسيلة نقل (الاسم المعلن لـ BLE، الشركة المصنعة لـ USB، عنوان IP/برنامج DAQ-E/…). يتم حذف القيم الفارغة. |

| المعلمة | القيمة الافتراضية | الوصف |
| --- | --- | --- |
| `transports` | الثلاثة جميعًا | التسلسل (أو سلسلة csv) الذي يحدد نطاق الفحص. من المفيد تمريره عندما تعرف ما تريد — حيث يُعد BLE هو الجزء البطيء. |
| `scan_timeout` | 5 | نافذة المسح لكل وسيلة نقل بالثواني؛ تحدد الخلفية النطاق بين 1 و20. |
| `timeout` | 60.0 | الحد الأقصى لـ «HTTP» للمكالمة بأكملها (كما هو الحال في أماكن أخرى في SDK). |
| `auto_start_backend` | `True` | إنشاء خلفية محلية إذا لم يكن هناك أي منها قيد التشغيل. لا يتم إنشاء خلفية محلية أبدًا لـ `backend_url` عن بُعد. |

> **لا تظهر المستشعرات المفتوحة بالفعل في المجموعة.** تتوقف الأجهزة الطرفية BLE المتصلة عن الإعلان ولا يمكن فحص منفذ COM المفتوح، لذا تدرج عملية الاكتشاف ما هو *متاحًا للاتصال*. من المتوقع الحصول على نتيجة فارغة مباشرة بعد توصيل جهاز ما — استخدم `list_daq_sensors()` لما لديك بالفعل. يتم تخطي وسائل النقل التي يتعذر إجراء المسح لها (عدم تثبيت bleak / zeroconf) بدلاً من إثارة استثناء، لذا لا يزال الجهاز الذي لا يحتوي على بلوتوث يحصل على إجابات USB و ETH.

### القائمة

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### التشارك مع واجهة المستخدم الرسومية (GUI) / CLI

إذا كان واجهة المستخدم الرسومية (GUI) تحتوي بالفعل على مستشعر مفتوح، فإن استدعاء `connect_daq_sensor(port="COM3")` من Python يُرجع مقبضًا مُعلمًا بـ `already_connected=True`. عندئذٍ يصبح `disconnect()` الخاص بالجلسة عملية لا تؤدي إلى أي نتيجة، لذا فإن البرنامج النصي SDK الخاص بكإزالة المستشعر من واجهة المستخدم الرسومية عند الخروج من نطاق العمل.

### فئات الأجهزة المباشرة (بدون خلفية)

يتم إعادة تصدير `daq_sdk` بواسطة `chloros_sdk`، لذا يمكنك أيضًا تشغيل المستشعرات من البداية إلى النهاية أثناء العملية دون الحاجة إلى الخلفية:

> **التوافر:**يتم توزيع `daq_sdk` مع تثبيت سطح المكتب Chloros،**وليس** مع حزمة PyPI — يوفر لك `pip install chloros-sdk` `lattice_sdk` ولكنه يترك `chloros_sdk.DAQ_AVAILABLE == False`. تحقق من هذا العلامة قبل استخدام هذه الفئات؛ على مضيف يعمل بـ pip فقط، قم بتشغيل المستشعر عبر [`connect_daq_sensor()`](#daq-sensor-sessions)، والذي لا يحتاج إلى مكتبات نقل محلية.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

يفضل استخدام مسار الاتصال الذكي (`connect_daq_sensor`) عندما تريد الملكية المشتركة مع واجهة المستخدم الرسومية؛ استخدم الفئات المباشرة للبرامج النصية التي تعمل بدون واجهة مستخدم والتي تمتلك المستشعر حصريًا.

---

## أتمتة المشروع — `ChlorosProject`

مشروع «Chloros» المحفوظ هو مجلد يحتوي على `cameras.json` + `sensors.json` + `project.json`. يقوم `open_project` يقوم بتحميل ملف البيان، و`connect_all` يقوم بتشغيل كل جهاز محفوظ عبر الإنترنت بإعداداته المحفوظة — وهي نفس حالة الأجهزة التي تنتجها واجهة المستخدم الرسومية.

### مثال بسيط

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

أو كمدير سياق:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### طرق `ChlorosProject`

| الطريقة | الوصف |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | اكتشاف + توصيل كل جهاز محفوظ. تُرجع تقرير توصيل لكل فئة. تستخدم خلفية قيد التشغيل عندما تكون هناك خلفية تستمع على `127.0.0.1:5000`؛ وإلا فإنها تعود بصمت إلى التحكم المباشر (بدون خلفية) في الجهاز عبر `lattice_sdk` — ولا تقوم أبدًا بإنشاء خلفية. |
| `disconnect_all()` | إنهاء كل شيء. |
| `capture_all(output_dir=".")` | إطار واحد من كل كاميرا + مصفوفة + طيف من كل مستشعر. |
| `stream(camera, overlays=False, fps=10.0)` | مولد ينتج إطارات BGR إطارات 2 من كاميرا (أو مصفوفة) محددة. `overlays=False` عبارة عن حلقة التقاط مباشرة لـ `lattice_sdk` (تنتج المصفوفات قواميس `{serial: frame}`). يتم توجيه `overlays=True` عبر `ChlorosLocal.camera_stream()` →`/api/camera/<serial>/stream-annotated`، مع تمرير كتلة `ui.overlay` المحفوظة الخاصة بالكاميرا كمعلمات استعلام. يتطلب وضع الخلفية و**كاميرا مستقلة**: تثير الكاميرا في الوضع المباشر خطأ `RuntimeError` (لا يمكن للخلفية التقاط كاميرا تمتلكها هذه العملية) وتثير المصفوفة خطأ `NotImplementedError` (تراكبات مركبة لكل كاميرا — بث عنصر حسب الاسم). المكافئ أحادي الخطوة: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | تشغيل عملية المحاذاة على كل مصفوفة متصلة حاليًا. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | تشغيل مسار المعايرة/الفهرسة على صور المشروع (يغلف `ChlorosLocal.process`؛ هذه الأربعة هي **المعلمات الوحيدة** المقبولة — `indices=` وما يليها تثير استثناء `TypeError`؛ تعيين الفهارس عبر `ChlorosLocal.configure()`). تقوم بإنشاء `ChlorosLocal()`، والذي يقوم بتشغيل الخلفية تلقائيًا. |

السمات:
- `proj.cameras` — `Dict[str, CameraHandle]` مرتبة حسب الاسم ورقم التسلسل.
- `proj.arrays` — `Dict[str, ArrayHandle]` مرتبة حسب الاسم و array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` مرتبة حسب الاسم و slot_id.
- `proj.config` — `project.json["config"]` قاموس.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**مستويات المعالجة.** `capture()`، و`grab()`، و`frame_stream()` جميعها تأخذ نفس الرمز `processing`
، وسلسلة السلسلة تراكمية — كل مستوى يُشغّل كل ما يعلوه:

| المستوى | المخرجات | ملاحظات |
| --- | --- | --- |
| `raw` | باير أحادي القناة، أصلي للمستشعر | بدون إزالة التمويه. التراكبات غير متوفرة في هذا المستوى. |
| `debayered` | 3 قنوات BGR (**افتراضي**) | إزالة التمويه الثنائي الخطي. المستوى الوحيد الذي يعمل بدون وضع الخلفية. |
| `radiance` | float32، واط/م²/سترلين/نانومتر | سلسلة إشعاعية كاملة: إزالة التمويه + فصل الألوان 3×3 (متعدد الأطياف) + DSNU + تعويض المجال المسطح + مقياس NIST، مع استبعاد التعرض × الكسب بحيث تكون القيم مطلقة. |
| `reflectance` | uint16، 32768 = 1.0 | الإشعاع مقسومًا على الإشعاع الساقط (ρ = π·L/E). يتطلب قراءة DLS/DAQ — انظر الملاحظة أدناه. |
| `display` | 8 بت شبيه بـ sRGB | عرض مكافئ لواجهة المستخدم الرسومية: CCM + توازن اللون الأبيض + جاما عبر ملف تعريف الألوان النشط للكاميرا. |

أي شيء بخلاف `debayered` يتطلب وضع الخلفية؛ الكاميرا ذات الوضع المباشر ترفع
`NotImplementedError`. يحتاج `reflectance` إلى قراءة صالحة للانعكاس — نقطة نهاية الإطار تسحب
DAQ المجمّع إلى فتحة DLS الخاصة بالكاميرا تلقائيًا، ولكن في حالة عدم وجود DAQ مرتبط، ترفض السلسلة
مخرج الانعكاس وتقوم بختم التخفيض بصدق في البيانات الوصفية المرتجعة بدلاً من
إرجاع منتج أقل جودة بصمت.

> **مقياس DN للانعكاس — لا تقم بترميزه بشكل ثابت.** يستخدم انعكاس LATTICE `32768` = ρ 1.0 ويقوم بتسجيل
> XMP `Chloros:PixelScale=32768`؛ بينما يستخدم انعكاس Survey3 المعادلة `65535` = ρ 1.0 ولا يحمل أي
> علامات `Chloros:*`. اقرأ العلامة واقسم عليها. وهي مُعرَّفة في مجال uint16، لذا تظل
> `32768` لكل تنسيق يعيد تحجيم القيم (TIFF بـ 16 بت، PNG /JPG بـ 8 بت، النسبة المئوية بـ 32 بت) — قم بتطبيع
> نوع البيانات المخزن وإعادته إلى uint16 أولاً (×257 من 8 بت، ×65535 من float). الاستثناء الوحيد:
> يتم *اقتطاع* التقاط المصدر ذي 8 بت المكتوب على أنه 8 بت TIFF، ولا يتم إعادة تحديده، لذا لا يوجد مقياس يصفه
> — Chloros يتجاهل `PixelScale` ومجموعة MicaSense بالكامل في تلك الحالة. تعامل مع العلامة المفقودة
> في ملف انعكاس LATTICE على أنه «لا يوجد مقياس صالح»، وليس كقيمة افتراضية.

> **يتم نقل EXIF إلى التصدير.** يقوم `process()` بنسخ كتلة GPS الخاصة بالالتقاط المصدر
> **و ExifIFD الخاص بها** إلى كل منتج، لذا فإن عمليات التصدير تحمل `FocalLength`، `FNumber`،
> `ExposureTime`، `ISO`، `DateTimeOriginal` و `CameraSerialNumber` بالإضافة إلى
> الإحداثيات الجغرافية. `FocalLength` هو ما يستخدمه Pix4D لحساب مسافة العينة الأرضية — وبدونه
> تعود عملية إعادة البناء إلى مقياس خاطئ تمامًا (في حالة تم قياسها، تحول موقع مساحته 411 مترًا
> إلى موقع مساحته 47.8 كيلومترًا). تم تعمد عدم استخدام `-all:all` في النسخة: حيث إن العلامات الهيكلية لـ IFD0 تعطل
> إخراج LATTICE، وتم استبعاد `ExifImageWidth`/`Height` لأنهما يصفان مصدر
> بدلاً من الصورة النقطية المصدرة.

العلامات الفرعية لمرحلة الالتقاط (تنطبق على المستويات الإشعاعية — `radiance`، `reflectance`، `display`):

| العلامة | القيمة الافتراضية | المعنى |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + مجال مسطح + فصل 3x3 + مقياس إشعاعي NIST. |
| `apply_white_balance` | `True` | WB LUT. يدعم DLS عندما يكون جهاز DAQ مرتبطًا بالكاميرا. |
| `apply_index` | `False` | تقييم مؤشر الغطاء النباتي. |
| `index_expression` | `None` | صيغة التجاوز. غير فارغ → تمكين المؤشر تلقائيًا. |
| `annotated` | `False` | تراكب زخارف واجهة المستخدم الرسومية (الزيبرا/الشبكة/الذروة). غير متاح لـ `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **نوع القيمة المرجعة هو `CapturePathMap`، وليس `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` هو `Dict[str, Union[str, List[str]]]`: مستوى واحد
> يمنح `processing` كل تسلسل مسارًا واحدًا، بينما يمنحه المستوى المتعدد (`"all"`، أو
> قائمة `levels` صريحة) تمنحه **القائمة المرتبة** لكل منتج تم حفظه لتلك
> الكاميرا. ويصل المركب المباشر المدمج، في حالة وجود بث مباشر، تحت مفتاح
> `"combined"` بدلاً من أن يصل تحت رقم تسلسلي. الكود الذي يفترض وجود `str` يتعطل عند
> شكل القائمة دون أن يعترض أي مدقق أنواع — فقد ذكرت التعليقات التوضيحية أن `Dict[str, str]`
> لفترة بعد إصدار نموذج القائمة، وهذا هو سبب وجود الاسم المستعار. قم بالتوحيد
> عندما تريد النموذج المسطح:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### محاذاة المصفوفة

يكشف `ArrayHandle` سطح المحاذاة بالكامل. تكون ملفات التعريف مخصصة للجلسة فقط بشكل افتراضي — استدعِ `export_alignment()` صراحةً للاحتفاظ بها.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### محاذاة وقت الاتصال

يمكن لـ `connect_all(align=...)` إجراء المحاذاة التلقائية لكل مصفوفة عند الاتصال:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

يتم الرجوع إلى `project.json["config"]["auto_align_on_connect"]` في حالة عدم التحديد.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## الأجهزة المباشرة (بدون خلفية)

عندما تريد عدم الاعتماد مطلقًا على الخلفية (CI، الروبوتات عديمة الرأس، المدمجة)، قم باستيراد `lattice_sdk` و `daq_sdk` مباشرةً — كلاهما مُصدَّرمن قبل `chloros_sdk`. ملاحظة بشأن `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` موجود في حزمة PyPI (ولكنه يحتاج إلى وجود بيئة تشغيل Arena SDK)، بينما يتم توفير `daq_sdk` فقط مع تثبيت سطح المكتب.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### الإعدادات المسبقة والمشغل

ثلاثة من الإعدادات المسبقة الأربعة تعمل في وضع **التشغيل الحر**: حيث تقوم الكاميرا بالتعريض الضوئي بشكل مستمر وتقوم
`capture()` يعرض الإطار التالي. `triggered` هو الاستثناء — فهو يُجهز
الكاميرا لحدث حافة جهاز في السطر 2، لذا لا تلتقط أي شيء حتى وصوله.

| الإعداد المسبق | المشغل | استخدمه عندما |
| --- | --- | --- |
| `default` | التشغيل الحر | الاستخدام العام |
| `high_speed` | التشغيل الحر | 8 بت، بحد أقصى 60 إطارًا في الثانية، تعريض قصير |
| `high_quality` | التشغيل الحرالتشغيل | 12 بت، بلا حد أقصى لإطارات في الثانية — الخيار المعتاد للصور الثابتة |
| `triggered` | **مُجهز، الخط 2** | الكاميرا موصولة بكابل مزامنة M8 ويقوم شيء آخر بتشغيلها |

إذا اخترت `triggered` (أو قمت بضبط `trigger_mode="On"` بنفسك) دون وجود أي شيء
يقوم بتشغيل الخط 2، فسيتم تجاوز مهلة كل `capture()` — وهذا صحيح، لأنك طلبت
الكاميرا بالانتظار. يوضح خطأ «SDK» هذا الأمر عند حدوثه؛ انظر
[SC_ERR_TIMEOUT أثناء الالتقاط](#direct-hardware-backend-free).

> **ملاحظة — رسائل &quot;GVSP probe&quot; / `SC_ERR_TIMEOUT -1011` عند الاتصال ليست أخطاء.**&gt; عند الاتصال، يحاول SDK التفاوض بشأن**الإطارات الضخمة** (حزم GVSP بحجم 9000 بايت) لتحقيق معدل نقل بيانات أعلى. على وصلة بطاقة شبكة مباشرة من نقطة إلى نقطة (مثل عنوان `169.254.x.x` محلي للوصلة)، لا تستطيع الشبكة عادةً نقل الإطارات الضخمة، لذا تنتهي مهلة هذا الاختبار وتسجل سطورًا مثل:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> هذا هو **الخيار البديل المصمم**: يعود جهاز**SDK** تلقائيًا إلى الحزم القياسية بحجم 1500 بايت وتستمر الكاميرا في الاتصال بشكل طبيعي (السطور `[chunk-enable …]` التالية هي جزء من تسلسل الاتصال العادي). لا يزال التقاط الصور يعمل.
>
> يمكنك تخطي هذا الاختبار، لكن **إنه ليس مجرد كاتم صوت للسجلات— بل إنه يعطل الإطارات الضخمة.** تستجيب الكاميرا لطلبات ping التي تحمل علامة «Don&#x27;t-Fragment» فقط حتى 1500 بايت بغض النظر عن جودة شبكتك، لذا لا يمكن لاختبار ping وحده اكتشاف الإطارات الضخمة أبدًا؛ وهذا الاختبار هو الشيء الوحيد القادر على ذلك. قم بتعطيله وستعمل الكاميرا بحزم قياسية سعة 1500 بايت إلى الأبد، على أي شبكة:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> لا يستحق الأمر العناء إلا على شبكة *تعلم* أنها لا تدعم الإطارات الضخمة، حيث يوفر ذلك حوالي ثانية واحدة من وقت الاتصال لكل كاميرا. نظرًا لأنها مقايضة حقيقية وليست شكلية، فإن «SDK» (أداة التحكم في الأداء) تشير إلى ذلك الآن عند استخدامها:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **اتركها كما هي ما لم يكن لديك سبب.** إذا تُركت هذه الخيار مُفعَّلًا، فإن كل عملية اتصال تعيد قياس الشبكة المتوفرة لديك فعليًّا: قم بتوصيلها بمحول يدعم حزم «جامبو»، وستقوم عملية الاتصال التالية بتحديد حزم «جامبو» تلقائيًّا، دون الحاجة إلى أي تكوين أو إعادة تشغيل.
>
> إذا كنت *ترغب* في الحصول على سرعة نقل البيانات &quot;جامبو&quot;، فقم بتمكين &quot;جامبو&quot; من طرف إلى طرف (MTU بطاقة الشبكة 9000 + محول يدعمها)، أو قم بتثبيته باستخدام `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` عندما تعلم أن الوصلة تدعمه — على الرغم من أننا نفضل استخدام `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` لكل أمر بدلاً من ضبطه بشكل دائم، لأن الحجم المثبت يتخطى عملية الفحص ويتوقف عن التكيف مع الشبكة الموجودة أمامه. **كل** جهاز في المسار يجب أن يمرر حزم «جامبو» — بما في ذلك أي مقسم أو محقن PoE، وهو السبب المعتاد الذي يجعل الإعداد القادر على التعامل مع حزم «جامبو» غير قادر على نقلها.

> **`SC_ERR_TIMEOUT -1011` أثناء `capture()` / `grab*()` يمثل مشكلة مختلفة — فهذا خطأ حقيقي.**&gt; الملاحظة أعلاه تتعلق فقط بـ `-1011` الذي تم تسجيله بواسطة**اختبار وقت الاتصال**. نفس الخطأ الذي ظهر من**التقاط** يعني أن الكاميرا متصلة بشكل جيد ولكنها لا ترسل أي صور:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> الدليل الدامغ هو أن قناة *التحكم* في الكاميرا سليمة — عملية الاكتشاف تعمل، والإعدادات وعمليات الكتابة `[chunk-enable …]` تنجح جميعها — في حين أن *كل* إطار يتجاوز مهلة الانتظار.
>
> **السبب المعتاد هو أن الكاميرا مُهيأة للتشغيل بواسطة مشغل مادي.** مع `trigger_mode="On"` و `trigger_source="Line2"`، لا ترسل الكاميرا أي شيء على الإطلاق حتى وصول حافة كهربائية على كابل التزامن M8. إذا لم يكن هناك كابل يحرك هذا الخط، فإن كل عملية التقاط تنتظر إلى الأبد. الكاميرا ليست معطلة والشبكة تعمل بشكل جيد — فهي تقوم بالضبط بما طُلب منها.
>
> الإعدادات المسبقة `CameraSettings()` و `default` / `high_speed` / `high_quality` تعمل في وضع التشغيل الحر، وعملية التقاط تنتهي بفترة انتظار أثناء التجهيز تشرح السبب بنفسها بدلاً من طباعة `-1011` فقط. `PRESETS["triggered"]` يُشغّل Line2، حسب التصميم.
>
> لإجبار أي كاميرا على التشغيل الحر:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> إذا استمرت مهلة الانتظار في الانقضاء مع `trigger_mode="Off"`، فهذا يعني أن الكاميرا لا ترسل البيانات فعليًّا — أرسل إلينا السجل و `ip link show`.

#### ملفات تعريف الألوان (المعاينة المباشرة لـ RGB) — `set_color_profile`

يختار `LatticeCamera.set_color_profile(profile, custom_cct_k=None)` ملف تعريف لون العرض لل**المعاينة المباشرة** على كاميرات RGB (تتجاهل الكاميرات متعددة الأطياف هذا الإعداد):

| الملف | المعنى |
| --- | --- |
| `raw` | تجاوز سلسلة القياس الإشعاعي بالكامل. |
| `linear` | DSNU + flat + WB، بدون CCM، بدون جاما. |
| `natural` | خطي + CCM مقاس + جاما sRGB، مع اللمسة النهائية البسيطة فقط (تنعيم اللون + التشبع) — الإعداد الافتراضي الواقعي. |
| `enhanced` | `natural` بالإضافة إلى التشطيب الكامل بتقنية Hub-Parity (إزالة التهيج اللوني، والحيوية، والتباين المحلي CLAHE). مظهر أكثر ثراءً بتكلفة **تبلغ تقريبًا ضعف تكلفة المعالجة لكل إطار**، وبالتالي معدل إطارات مباشر أقل. |
| `custom_temp` | `natural` لكن WB مثبت على `custom_cct_k` كلفن (يتم تجاهل DLS؛ ويتم تثبيته بين 2000–10000 كلفن من جانب الخلفية). |

الملف الشخصي هو **معاينة مباشرة فقط** : تحصل اللقطات المحفوظة دائمًا على اللمسة النهائية الغنية الكاملة بغض النظر عن الملف الشخصي المحدد، لذا فإن اختيار `natural` لاستعادة وقت الإطار لا يقلل من جودة ما يتم حفظه على القرص. يؤدي الملف الشخصي غير المعروف إلى رفع `ValueError`؛ وعندما تكون الخلفية chloros متاحة، يتم إرسال التغيير إليها عبر POST أيضًا بحيث يعكس الإطار التالي في المعاينة هذا التغيير (يحصل مستخدمو SDK المباشرون الذين ليس لديهم خلفية على تغيير الإعدادات أيضًا).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### الكاميرات أحادية اللون (M3M) و`Calibration`

الكاميرا أحادية اللون **M3M** (`M3M-<lens>-F<wavelength>`) هي كاميرا أحادية النطاق: مستوى واحد من التدرج الرمادي، بدون فسيفساء باير، وبدون مصفوفة تداخل طيفي. يتعرف `Calibration` عليها ويكشف عن علامة `is_mono`. لا يزال الانعكاس ينطبق كخريطة إشعاعية لكل نطاق (عملية الفصل هي المصفوفة الهوية)، لكن العمليات الحسابية متعددة النطاقات على كاميرا واحدة تُنتج نتائج صحيحة بدلاً من إرجاع نتائج غير منطقية:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

لبناء مؤشر نباتي من أجهزة أحادية، قم بدمج عدة كاميرات M3M ذات أطوال موجية مختلفة في مكدس متعدد النطاقات مُحاذي (انظر [محاذاة المصفوفة](#array-alignment)) واحسب المؤشر عبر تلك المجموعة بدلاً من كاميرا واحدة.

الوضع المباشر لـ DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **المفاتيح المقبولة لـ `apply_sensor_settings`**— بالضبط `integration_time_ms`، `frame_avg`، `ae_enabled`، `sunshine_diffuser_installed` (DAQ-E؛ تم إهمالها لصالح `cap_id`)، و`filter_model` (DAQ-M)، و`cap_id` (جميع أنواع DAQ؛ `None`/`""`/`"none"` = مستشعر عاري، بدون تصحيح السعة). المفاتيح غير المعروفة**يتم تجاهلها بصمت** — على سبيل المثال، `{"integration_time": 64}` لا تؤدي أي وظيفة (يجب أن تكون `integration_time_ms`). تُرجع `{"applied": [...], "errors": {...}}` ولا تثير أي استثناءات أبدًا.

`chloros_sdk` يعيد تصدير السطح الأساسي المستخدم أعلاه فقط. يضيف السجل العام الكامل لـ `daq_sdk` API (22 اسمًا) ما يلي — قم باستيرادها من `daq_sdk` مباشرةً:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## الاستثناءات

التقط الفئة الأساسية لمعالجة «أي خطأ حدث في Chloros»:

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

> يتم تصدير `ChlorosAuthenticationError` و `ChlorosConfigurationError` في المستوى الأعلى جنبًا إلى جنب مع البقية؛ كما يمكن استيرادهما من `chloros_sdk.exceptions` كما هو موضح.

التسلسل الهرمي:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## أمثلة شاملة

### 1. معالجة مجلد باستخدام شريط تقدم مخصص

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. مصفوفة LATTICE الحية → الانعكاس + مرجع DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. حملة التقاط مدفوعة بالمشروع

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. دفق الإطارات من كاميرات متعددة → مسار NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. نصوص برمجية التقاط مباشرة على الأجهزة (بدون خلفية)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. اختبار القدرات قبل توصيل مصفوفة من 4 كاميرات

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. ما يعادل وصفة الالتقاط (Python خالص)

تحتوي لغة DSL الخاصة بوصفات «CLI» على ما يعادلها مباشرةً في «Python»:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## التشغيل التلقائي للخلفية

نقاط الدخول الخاصة بـ smart-connect — `connect_camera`، و`connect_array`، `connect_daq_sensor`، و `discover_lattice_cameras` — هي عملاء رفيعة (HTTP) تفترض أن الخلفية تستمع على `127.0.0.1:5000` (القيمة الافتراضية لـ smart-connectURL). عندما تكون واجهة المستخدم الرسومية (GUI) أو CLI قيد التشغيل بالفعل، فإن أحدها يعمل. من خلال البرنامج النصي الأساسي، قد لا يكون هناك — لذا فإن هذه الوظائف **تقوم بتشغيل الملف الثنائي للخلفية المضمّن تلقائيًا** (بدون نافذة، بنفس الطريقة التي يعمل بها `ChlorosLocal`) قبل أول استدعاء لها، ثم تنتظر حتى `backend_startup_timeout` حتى يتم تشغيله.

القواعد:

- **يتم إنشاء URL محلي فقط.** يُعتبر `backend_url` الذي يشير إلى `localhost` / `127.0.0.1` / `[::1]` مؤهلاً؛ أما أي يُفترض أنه جهاز شخص آخر ولن يتم إنشاؤه أبدًا.
- **تُترك الخلفية قيد التشغيل لإعادة الاستخدام** (مثل CLI) — لا يحدث إيقاف تشغيل ضمني عند خروج البرنامج النصي. تؤدي إعادة تشغيل البرنامج النصي إلى إعادة استخدام الخلفية النشطة.
- **يمكنك إلغاء الاشتراك باستخدام `auto_start_backend=False`** في أي من هذه الاستدعاءات (على سبيل المثال، عندما تشير إلى خلفية بعيدة، أو تدير دورة حياة الخلفية بنفسك).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

إذا تعذر تحديد موقع الملف الثنائي المضمّن أو تشغيله، فإن استدعاء HTTP اللاحق يثير استثناءً قابلًا للتنفيذ و**مراعيًّا للمنصة** `ChlorosConnectError` بدلاً من تتبع بسيط لرفض الاتصال — في Windows، يوجهك إلى تطبيق سطح المكتب أو إلى أمر `chloros-cli`؛ أما على Linux (بدون واجهة مستخدم رسومية)، فيوجهك إلى أمر `chloros-cli` أو إلى `.deb`.

---

## البيئة والرؤوس

يقوم SDK بتمييز كل استدعاء للخلفية HTTP بـ `X-Chloros-Client: sdk`. تطبق الخلفية قواعد الترخيص SDK / CLI (يتطلب تسجيل الدخول **و** خطة مدفوعة Chloros+) بدلاً من مسار المستوى المجاني لواجهة المستخدم الرسومية. يتم تعيين هذا تلقائيًا عند الاستيراد — لاتحتاج إلى القيام بأي شيء.

يتم التعرف على `http://localhost` و `http://127.0.0.1` باعتبارهما الخادم الخلفي المحلي. ولا يتم تغيير المكالمات الموجهة إلى المضيفات الأخرى (مثل خدمة التحليلات الخاصة بك).

تجاوز الخلفية URL عن طريق تمرير `backend_url=` (أو `api_url=` على `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(لا يصل `backend_url` غير المرتبط بـ loopback إلا إلى الخلفية source/dev — فالخلفيات المرفقة تربط loopback فقط؛ انظر «وضع الخلفية البعيدة» (Remote-Backend Mode) لمعرفة نمط النفق.)

---

## الإصدارات والتوافق

- يتم عرض إصدار SDK على أنه `chloros_sdk.__version__`.
- يربط SDK سلوك الدبابيس بإصدار الخلفية المرفقة. عادةً ما ينجح الجمع بين إصدار أقدم من SDK مع خلفية أحدث (نقاط نهاية متوافقة مع الإصدارات الأحدث)، لكن الجمع بين إصدار أحدث من SDK مع خلفية أقدم قد يؤدي إلى ظهور أخطاء `404` على نقاط النهاية الجديدة — قم بترقية تطبيق سطح المكتب ليتوافق مع ذلك.
- واجهة-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) ونقطة نهاية تحليل الشبكة تعيد مخططات JSON مستقرة؛ الحقول الجديدة إضافية.

---

## إرشادات استكشاف الأخطاء وإصلاحها

- **`ChlorosAuthenticationError: Login required`** → قم بتشغيل `chloros-cli login EMAIL PASSWORD` مرة واحدة على هذا الجهاز، أو قم بتسجيل الدخول عبر تطبيق سطح المكتب Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → تقوم مكالمات «الاتصال الذكي» (smart-connect) بتشغيل الخلفية المحلية تلقائيًا، لذا لا تظهر هذه الرسالة إلا عندما يتعذر العثور على الملف الثنائي المضمّن أو(مثل مضيف يعمل بـ pip فقط بدون حزمة سطح المكتب). الرسالة تتكيف مع النظام الأساسي: على Windows افتح تطبيق سطح المكتب أو قم بتشغيل أي أمر `chloros-cli`؛ على Linux قم بتشغيل أمر `chloros-cli` (لا توجد واجهة مستخدم رسومية) أو قم بتثبيت `.deb`. بالنسبة للخلفية البعيدة، قم بتمرير `backend_url=` (و `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** عند الاستيراد → فشل تحميل `lattice_sdk` (عادةً ما يكون ذلك بسبب عدم تثبيت مكتبات DLL الخاصة بوقت تشغيل Arena SDK). لا يزال السطح غير الخاص بالكاميرا يعمل.
- **تُرجع Array connect دقة أقل من الدقة الأصلية**→ تقوم ميزة «التحضير الذكي» (smart-prep) في الخلفية بتقليص حجم الإطار تلقائيًا ليتناسب مع السلك. استخدم `analyze_array_network()` لمعرفة السبب، ثم قم إما بترقية الرابط، أو قبول التقليص، أو قم بتمرير `force_tier="slip-emit-and-capture"` للالتقاط التسلسلي. شبكة الأمان الخاصة بالتقليص**لا** تغطي الاكتتاب الزائد الإجمالي (`oversubscribed: true`، حقول fps 0): لا يمكن معالجة وجود عدد كبير جدًا من الكاميرات على القناة عن طريق تجميع الصور/منطقة الاهتمام (ROI) — قم بتقليل عدد الكاميرات، أو تمكين الإطارات الضخمة، أو الانتقال إلى بطاقة شبكة أسرع (انظر [الاشتراك الزائد](#over-subscription-the-per-cam-floor)).
- **يُبلغ `analyze_array_network()` أن حلقة الاستقبال (RX ring) لبطاقة الشبكة صغيرة جدًا (~0.26 ميغابايت) / بوابات التوصيل مع ظهور رسالة &quot;FRAMES WILL DROP&quot;** → حلقة الاستقبال في بطاقة NIC المضيفة في إعدادها الافتراضي (غالبًا ما تُعاد ضبطها إلى 32 بعد تحديث برنامج تشغيل بطاقة NIC). في محول Realtek USB 10GbE، اضبط `ReceiveBufferLen=256` و `PendingReceives=64` (مرتفع)، ثم أعد تشغيل الخادم الخلفي حتى يعيد قراءة الحلقة. الإجراء الكامل: [CLI مرجع → إعداد بطاقة NIC المضيفة وضبطها](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **توقف المضيف عند إعادة التشغيل/الإغلاق، ثم ظهور أخطاء WMI `Invalid class` لاحقًا / عدم تمكين بطاقة NIC** → برنامج تشغيل USB 10GbE قديم يتسبب في ظهور الخطأ `DRIVER_POWER_STATE_FAILURE` (شاشة الموت الزرقاء BSOD `0x9F`). قم بتحديث برنامج تشغيل المهايئ إلى الإصدار الحالي (≥ 2026) وأعد تطبيق إعدادات حلقة الاستقبال. انظر [مرجع CLI → إعداد المضيف وضبط بطاقة الشبكة](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **رفض الانعكاس** → يجب ربط وحدة DAQ نشطة بالكاميرا (أو المصفوفة) للحصول على انعكاس بمقياس مطلق. قم بالربط إما عبر واجهة المستخدم الرسومية أو استخدم `processing="radiance"` (W/m²/sr/nm) الذي لا يتطلب مستشعرًا مقترنًا.
- **يستغرق التقاط `smart=True` وقتًا أطول من المتوقع** → يعتمد تقارب AE على ديناميكيات المشهد؛ قم بتشديد `exposure_tolerance_pct` أو تقصير `stability_window_s` إذا كنت تريد مشغلًا أسرع (أقل استقرارًا).

---

## انظر أيضًا

- [مرجع CLI](cli-reference.md) — كل أمر فرعي في CLI يعكس استدعاءً لـ SDK.
- [دليل مستشعرات DAQ](../daq/README.md) — قواعد التوصيل والمعايرة والتسجيل الخاصة بالمستشعرات.
- الوثائق عبر الإنترنت: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
