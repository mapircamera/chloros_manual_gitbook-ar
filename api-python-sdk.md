# API : Python SDK

**Chloros Python SDK** يوفر وصولاً برمجياً إلى محرك معالجة الصور Chloros، مما يتيح الأتمتة وسير العمل المخصص والتكامل السلس مع تطبيقات Python وخطوط البحث الخاصة بك.

### الميزات الرئيسية

* 🐍 **Python الأصلي** - لغة API نظيفة ومكتوبة بلغة Python لمعالجة الصور
* 🔧 **وصول كامل إلى API** - تحكم كامل في معالجة Chloros
* 🚀 **الأتمتة** - إنشاء سير عمل مخصص لمعالجة الدُفعات
* 🔗 **التكامل** - تضمين Chloros في تطبيقات Python الحالية
* 📊 **جاهز للبحث** - مثالي لمسارات التحليل العلمي
* ⚡ **المعالجة المتوازية** - قابل للتوسع وفقًا لعدد نوى وحدة المعالجة المركزية (Chloros+)

### المتطلبات

| المتطلبات          | التفاصيل                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros مثبت** | Windows: أداة تثبيت سطح المكتب؛ Linux: حزمة `.deb`                  |
| **الترخيص**          | Chloros+ ([يلزم وجود خطة مدفوعة](https://cloud.mapir.camera/pricing)) |
| **نظام التشغيل** | Windows 10/11 (64 بت)، Linux x86_64 (amd64)، Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 أو أعلى                                                |
| **الذاكرة**           | 8 جيجابايت من ذاكرة الوصول العشوائي (RAM) كحد أدنى (يوصى بـ 16 جيجابايت)                                  |
| **الإنترنت**         | مطلوب لتنشيط الترخيص                                     |

{% hint style="warning" %}
**متطلبات الترخيص**: يتطلب Python SDK اشتراكًا مدفوعًا في Chloros+ للوصول إلى API. لا تتضمن الخطط القياسية (المجانية) إمكانية الوصول إلى API/SDK. تفضل بزيارة [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) للترقية.
{% endhint %}

## البدء السريع

### التثبيت

التثبيت عبر pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**الإعداد الأولي**: قبل استخدام SDK، قم بتنشيط ترخيص Chloros+ عن طريق فتح Chloros، Chloros (المتصفح) أو Chloros CLI وتسجيل الدخول باستخدام بيانات اعتمادك. لا يلزم القيام بذلك سوى مرة واحدة. في Linux (بدون واجهة مستخدم رسومية)، استخدم: `chloros-cli login user@example.com 'password'`
{% endhint %}

### الاستخدام الأساسي

معالجة مجلد ببضع أسطر فقط:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**مسارات عبر الأنظمة الأساسية**: تستخدم أمثلة الكود في هذه الصفحة مسارات بنمط Windows (على سبيل المثال، `C:\\DroneImages\\Flight001`). في Linux، استخدم مسارات Linux بدلاً من ذلك (على سبيل المثال، `/home/user/drone_images/flight001` أو `~/drone_images/flight001`). يعمل SDK بشكل متطابق على كلا النظامين.
{% endhint %}

### التحكم الكامل

لسير العمل المتقدم:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## دليل التثبيت

### المتطلبات الأساسية

قبل تثبيت SDK، تأكد من توفر ما يلي:

1. **تثبيت Chloros** — Windows: أداة التثبيت لسطح المكتب ([تنزيل](download.md))؛ Linux: حزمة `.deb` ([تثبيت Linux](linux/linux-installation.md))
2. **Python 3.7+** مثبت ([python.org](https://www.python.org))
3. **ترخيص Chloros+ نشط** ([ترقية](https://cloud.mapir.camera/pricing))

### التثبيت عبر pip

**التثبيت القياسي:**

```bash
pip install chloros-sdk
```

**مع دعم مراقبة التقدم:**

```bash
pip install chloros-sdk[progress]
```

**التثبيت الخاص بالتطوير:**

```bash
pip install chloros-sdk[dev]
```

### التحقق من التثبيت

اختبر أن SDK مثبت بشكل صحيح:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## الإعداد الأولي

### تنشيط الترخيص

يستخدم SDK نفس الترخيص المستخدم في Chloros و Chloros (المتصفح) و Chloros و CLI. قم بالتفعيل مرة واحدة عبر واجهة المستخدم الرسومية أو CLI:**Windows:**افتح**Chloros أو Chloros (المتصفح)** وقم بتسجيل الدخول في علامة التبويب &quot;المستخدم&quot; <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> ، أو استخدم CLI.**Linux:** استخدم CLI (لا تتوفر واجهة المستخدم الرسومية):

```bash
chloros-cli login user@example.com 'your_password'
```

يتم تخزين الترخيص مؤقتًا محليًا ويستمر عبر عمليات إعادة التشغيل.

{% hint style="success" %}
**الإعداد لمرة واحدة**: بعد تسجيل الدخول عبر واجهة المستخدم الرسومية أو CLI، يستخدم SDK الترخيص المخزن مؤقتًا تلقائيًا. لا حاجة لمصادقة إضافية!
{% endhint %}

{% hint style="info" %}
**تسجيل الخروج**: يمكن لمستخدمي SDK مسح بيانات الاعتماد المخزنة مؤقتًا برمجيًا باستخدام طريقة `logout()`. انظر [طريقة logout()](#logout) في مرجع API.
{% endhint %}

### اختبار الاتصال

تحقق من أن SDK يمكنه الاتصال بـ Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## مرجع API

### فئة ChlorosLocal

الفئة الرئيسية لمعالجة الصور المحلية لـ Chloros.

#### المنشئ

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**المعلمات:**

| المعلمة                 | النوع | القيمة الافتراضية                   | الوصف                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL للخلفية المحلية Chloros          |
| `auto_start_backend`      | bool | `True`                    | بدء تشغيل الخلفية تلقائيًا إذا لزم الأمر |
| `backend_exe`             | str  | `None` (الكشف التلقائي)      | مسار الملف القابل للتنفيذ للخلفية            |
| `timeout`                 | int  | `30`                      | مهلة الطلب بالثواني            |
| `backend_startup_timeout` | int  | `60`                      | مهلة بدء تشغيل الخلفية (بالثواني) |

**أمثلة:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**الكشف التلقائي عبر الأنظمة الأساسية**: يحاول SDK تلقائيًا مسار الخلفية الصحيح لنظامك الأساسي:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (يدوي)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### الطرق

#### `create_project(project_name, camera=None)`

إنشاء مشروع Chloros جديد.

**المعلمات:**

| المعلمة      | النوع | مطلوب | الوصف                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | نعم      | اسم المشروع                                     |
| `camera`       | str  | لا       | قالب الكاميرا (على سبيل المثال، &quot;Survey3N\_RGN&quot;، &quot;Survey3W\_OCN&quot;) |

**النتائج:** `dict` - استجابة إنشاء المشروع**مثال:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

استيراد الصور من مجلد.

**المعلمات:**

| المعلمة     | النوع     | مطلوب | الوصف                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | نعم      | مسار المجلد الذي يحتوي على الصور         |
| `recursive`   | bool     | لا       | البحث في المجلدات الفرعية (الافتراضي: False) |

**تُرجع:** `dict` - نتائج الاستيراد مع عدد الملفات**مثال:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

تكوين إعدادات المعالجة.

**المعلمات:**

| المعلمة                 | النوع | الافتراضي                 | الوصف                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;قياسي (سريع، جودة متوسطة)&quot; | طريقة إزالة التمويه            |
| `vignette_correction`     | bool | `True`                  | تمكين تصحيح التظليل      |
| `reflectance_calibration` | bool | `True`                  | تمكين معايرة الانعكاس  |
| `indices`                 | قائمة | `None`                  | مؤشرات الغطاء النباتي المطلوب حسابها |
| `export_format`           | سلسلة  | &quot;TIFF (16 بت)&quot;         | تنسيق الإخراج                   |
| `ppk`                     | bool | `False`                 | تمكين تصحيحات PPK          |
| `custom_settings`         | dict | `None`                  | إعدادات مخصصة متقدمة        |

**تنسيقات التصدير:**

* `"TIFF (16-bit)"` - موصى به لنظام المعلومات الجغرافية/التصوير المساحي
* `"TIFF (32-bit, Percent)"` - التحليل العلمي
* `"PNG (8-bit)"` - الفحص البصري
* `"JPG (8-bit)"` - إخراج مضغوط

**المؤشرات المتاحة:**NDVI، NDRE، GNDVI، OSAVI، CIG، EVI، SAVI، MSAVI، MTVI2، والمزيد.**مثال:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

معالجة صور المشروع.

**المعلمات:**

| المعلمة           | النوع     | الافتراضي      | الوصف                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | وضع المعالجة: &quot;parallel&quot; أو &quot;serial&quot;   |
| `wait`              | bool     | `True`       | انتظار الانتهاء                       |
| `progress_callback` | callable | `None`       | دالة استدعاء التقدم (progress, msg) |
| `poll_interval`     | float    | `2.0`        | فاصل الاستقصاء للتقدم (بالثواني)   |

**تُرجع:** `dict` - نتائج المعالجة

{% hint style="warning" %}
**الوضع المتوازي**: يتطلب ترخيص Chloros+. يتكيف تلقائيًا مع نوى وحدة المعالجة المركزية (CPU) لديك (حتى 16 عامل).
{% endhint %}

**مثال:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

الحصول على تكوين المشروع الحالي.

**النتائج:** `dict` - تكوين المشروع الحالي**مثال:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

الحصول على معلومات حالة الخلفية بما في ذلك تقدم المعالجة لكل مؤشر ترابط.

**النتائج:** `dict` - حالة الخلفية بالهيكل التالي:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**مثال:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

إيقاف تشغيل الخلفية (إذا تم تشغيلها بواسطة SDK).

**مثال:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

مسح بيانات الاعتماد المخزنة مؤقتًا من النظام المحلي.

**الوصف:**

تسجيل الخروج برمجيًا عن طريق إزالة بيانات اعتماد المصادقة المخزنة مؤقتًا. وهذا مفيد في:
* التبديل بين حسابات Chloros+ المختلفة
* مسح بيانات الاعتماد في البيئات الآلية
* أغراض أمنية (على سبيل المثال، إزالة بيانات الاعتماد قبل إلغاء التثبيت)

**القيم المرجعة:** `dict` - نتيجة عملية تسجيل الخروج**مثال:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**مطلوب إعادة المصادقة**: بعد استدعاء `logout()`، يجب عليك تسجيل الدخول مرة أخرى عبر Chloros أو Chloros (المتصفح) أو Chloros أو CLI قبل استخدام SDK.
{% endhint %}

***

### وظائف الملاءمة

#### `process_folder(folder_path, **options)`

وظيفة ملاءمة من سطر واحد لمعالجة مجلد.

**المعلمات:**

| المعلمة                 | النوع     | الافتراضي         | الوصف                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | مطلوب        | مسار المجلد الذي يحتوي على الصور     |
| `project_name`            | str      | يتم إنشاؤه تلقائيًا  | اسم المشروع                   |
| `camera`                  | str      | `None`          | قالب الكاميرا                |
| `indices`                 | قائمة     | `["NDVI"]`      | المؤشرات المطلوب حسابها           |
| `vignette_correction`     | منطقية     | `True`          | تمكين تصحيح التظليل     |
| `reflectance_calibration` | منطقية     | `True`          | تمكين معايرة الانعكاس |
| `export_format`           | سلسلة      | &quot;TIFF (16 بت)&quot; | تنسيق الإخراج                  |
| `mode`                    | str      | `"parallel"`    | وضع المعالجة                |
| `progress_callback`       | callable | `None`          | استدعاء التقدم              |

**النتائج:** `dict` - نتائج المعالجة**مثال:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## دعم مدير السياق

يدعم SDK مديري السياق للتنظيف التلقائي:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## أمثلة كاملة

{% hint style="info" %}
**مستخدمو Linux**: تستخدم جميع الأمثلة أدناه مسارات Windows. استبدل مسارات `C:\\...` بمسارات Linux الخاصة بك (على سبيل المثال، `/home/user/...` أو `~/...`). جميع وظائف SDK متطابقة عبر الأنظمة الأساسية.
{% endhint %}

### المثال 1: المعالجة الأساسية

معالجة مجلد بالإعدادات الافتراضية:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### المثال 2: سير العمل المخصص

تحكم كامل في مسار المعالجة:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### المثال 3: المعالجة المجمعة لمجلدات متعددة

معالجة مجموعات بيانات رحلات متعددة:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### المثال 4: تكامل مسار البحث

دمج Chloros مع تحليل البيانات:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### المثال 5: مراقبة التقدم المخصصة

تتبع التقدم المتقدم باستخدام التسجيل:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### المثال 6: معالجة الأخطاء

معالجة أخطاء قوية للاستخدام في الإنتاج:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### المثال 7: إدارة الحساب وتسجيل الخروج

إدارة بيانات الاعتماد برمجياً:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### المثال 8: أداة سطر الأوامر

إنشاء أداة CLI مخصصة باستخدام SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**الاستخدام:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## معالجة الاستثناءات

يوفر SDK فئات استثناءات محددة لأنواع مختلفة من الأخطاء:

### التسلسل الهرمي للاستثناءات

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### أمثلة على الاستثناءات

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## موضوعات متقدمة

### تكوين الخلفية المخصص

استخدم موقعًا أو تكوينًا مخصصًا للخلفية:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### المعالجة غير المعطلة

ابدأ المعالجة وتابع المهام الأخرى:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### إدارة الذاكرة

بالنسبة لمجموعات البيانات الكبيرة، قم بالمعالجة على دفعات:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## استكشاف الأخطاء وإصلاحها

### الخلفية لا تبدأ

**المشكلة:** SDK يفشل في بدء تشغيل الخلفية**الحلول:**

1. تحقق من تثبيت Chloros:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. تحقق من جدار الحماية (Windows) أو توفر المنفذ (Linux: `lsof -i :5000`)
3. جرب مسار الخلفية اليدوي:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### لم يتم الكشف عن الترخيص**المشكلة:** SDK يحذر من فقدان الترخيص**الحلول:**

1. افتح Chloros أو Chloros (المتصفح) أو Chloros أو CLI وقم بتسجيل الدخول.
2. تحقق من تخزين الترخيص مؤقتًا:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. في حالة مواجهة مشكلات في بيانات الاعتماد، امسح بيانات الاعتماد المخزنة مؤقتًا وأعد تسجيل الدخول:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. اتصل بالدعم الفني: info@mapir.camera

***

### أخطاء الاستيراد**المشكلة:** `ModuleNotFoundError: No module named 'chloros_sdk'`**الحلول:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### انتهاء مهلة المعالجة**المشكلة:** انتهاء مهلة المعالجة**الحلول:**

1. زيادة مهلة الانتظار:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. معالجة دفعات أصغر
3. التحقق من مساحة القرص المتاحة
4. مراقبة موارد النظام

***

### المنفذ قيد الاستخدام بالفعل**المشكلة:** منفذ الخلفية 5000 مشغول**الحلول:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

أو ابحث عن العملية المتعارضة وأغلقها:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## نصائح لتحسين الأداء

### تحسين سرعة المعالجة

1. **استخدام الوضع المتوازي** (يتطلب Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **تقليل دقة الإخراج** (إذا كان ذلك مقبولاً)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **تعطيل الفهارس غير الضرورية**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **المعالجة على SSD** (وليس HDD)***

### تحسين الذاكرة

بالنسبة لمجموعات البيانات الكبيرة:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### المعالجة في الخلفية

تحرير Python لمهام أخرى:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## أمثلة على التكامل

### تكامل Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## الأسئلة الشائعة

### س: هل يتطلب SDK اتصالاً بالإنترنت؟

**ج:** فقط لتفعيل الترخيص الأولي. بعد تسجيل الدخول عبر Chloros أو Chloros (المتصفح) أو Chloros CLI، يتم تخزين الترخيص مؤقتًا محليًا ويعمل دون اتصال بالإنترنت لمدة 30 يومًا.***

### س: هل يمكنني استخدام SDK على خادم بدون واجهة مستخدم رسومية (GUI)؟**ج:** نعم! يعمل SDK بدون واجهة مستخدم على كل من خادمي Windows و Linux.**Linux (موصى به للاستخدام بدون واجهة):**
* التثبيت عبر حزمة `.deb`
* تنشيط الترخيص: `chloros-cli login user@example.com 'password'`

**خادم Windows:**
* خادم Windows 2016 أو أحدث
* تثبيت Chloros (لمرة واحدة)
* الترخيص مفعل عبر CLI أو على أي جهاز

***

### س: ما الفرق بين Desktop و CLI و SDK؟

| الميزة         | واجهة المستخدم الرسومية لـ Desktop | سطر الأوامر لـ CLI | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **الواجهة**   | النقر بالماوس | الأوامر          | Python API  |
| **الأفضل لـ**    | العمل المرئي | البرمجة النصية        | التكامل |
| **الأتمتة**  | محدودة     | جيدة             | ممتازة   |
| **المرونة** | أساسية       | جيدة             | قصوى     |
| **الترخيص**     | Chloros+    | Chloros+         | Chloros+    |***

### س: هل يمكنني توزيع التطبيقات التي تم إنشاؤها باستخدام SDK؟**ج:** يمكن دمج كود SDK في تطبيقاتك، ولكن:

* يحتاج المستخدمون النهائيون إلى تثبيت Chloros
* يحتاج المستخدمون النهائيون إلى تراخيص Chloros+ نشطة
* يتطلب التوزيع التجاري ترخيص OEM

اتصل بـ info@mapir.camera للاستفسارات المتعلقة بـ OEM.

***

### س: كيف أقوم بتحديث SDK؟

```bash
pip install --upgrade chloros-sdk
```

***

### س: أين يتم حفظ الصور المعالجة؟

بشكل افتراضي، في مسار المشروع:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### س: هل يمكنني معالجة الصور من نصوص Python التي تعمل وفقًا لجدول زمني؟**ج:** نعم! استخدم جدولة نظام التشغيل مع نصوص Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** قم بالجدولة عبر &quot;جدولة المهام&quot; للتشغيل يوميًا.**Linux:** قم بالجدولة عبر cron:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### س: هل يدعم SDK async/await؟**ج:** الإصدار الحالي متزامن. للحصول على سلوك غير متزامن، استخدم `wait=False` أو قم بالتشغيل في مؤشر ترابط منفصل:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### س: كيف يمكنني التبديل بين حسابات Chloros+ المختلفة؟**ج:** استخدم طريقة `logout()` لمسح بيانات الاعتماد المخزنة مؤقتًا، ثم أعد تسجيل الدخول باستخدام الحساب الجديد:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

بعد تسجيل الخروج، قم بالمصادقة باستخدام الحساب الجديد عبر واجهة المستخدم الرسومية (GUI) أو المتصفح أو CLI قبل استخدام SDK مرة أخرى.

***

## الحصول على المساعدة

### الوثائق

* **مرجع API**: هذه الصفحة

### قنوات الدعم

* **البريد الإلكتروني**: info@mapir.camera
* **الموقع الإلكتروني**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **الأسعار**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### أمثلة على الكود

جميع الأمثلة المدرجة هنا تم اختبارها وهي جاهزة للاستخدام. انسخها وقم بتكييفها لتناسب حالتك.

***

## الترخيص**برنامج مملوك** - حقوق النشر (c) 2025 MAPIR Inc.

يتطلب SDK اشتراكًا نشطًا في Chloros+. يُحظر الاستخدام أو التوزيع أو التعديل غير المصرح به.
