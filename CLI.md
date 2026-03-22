# CLI : سطر الأوامر

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>يوفر **Chloros CLI** وصولاً قوياً عبر سطر الأوامر إلى محرك معالجة الصور Chloros، مما يتيح الأتمتة وكتابة البرامج النصية والتشغيل بدون واجهة مستخدم لسير عمل التصوير الخاص بك.

### الميزات الرئيسية

* 🚀 **الأتمتة** - معالجة دفعية برمجية لمجموعات بيانات متعددة
* 🔗 **التكامل** - دمج في سير العمل والخطوط الإنتاجية الحالية
* 💻 **التشغيل بدون واجهة** - التشغيل بدون واجهة المستخدم الرسومية
* 🌍 **متعدد اللغات** - دعم 38 لغة
* ⚡ **المعالجة المتوازية** - [التكيف الحسابي الديناميكي](processing-architecture/dynamic-compute-adaptation.md) يقوم بالتحسين تلقائيًا وفقًا لأجهزتك

### المتطلبات

| المتطلب          | التفاصيل                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **نظام التشغيل** | Windows 10/11 (64 بت)، Linux x86_64 (amd64)، Linux arm64 (NVIDIA Jetson JetPack 6) |
| **الترخيص**          | Chloros+ ([يلزم اشتراك مدفوع](https://cloud.mapir.camera/pricing)) |
| **الذاكرة**           | 8 جيجابايت من ذاكرة الوصول العشوائي (RAM) كحد أدنى (يوصى بـ 16 جيجابايت)                                  |
| **الإنترنت**         | مطلوب لتنشيط الترخيص                                     |
| **مساحة القرص**       | تختلف حسب حجم المشروع                                              |

{% hint style="warning" %}
**متطلبات الترخيص**: يتطلب CLI اشتراكًا مدفوعًا في Chloros+. لا تتضمن الخطط القياسية (المجانية) إمكانية الوصول إلى CLI. تفضل بزيارة [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) للترقية.
{% endhint %}

## البدء السريع

### التثبيت

#### Windows

يتم تضمين CLI تلقائيًا مع أداة تثبيت Chloros:

1. قم بتنزيل وتشغيل **Chloros Installer.exe**

2. أكمل معالج التثبيت
3. تم تثبيت CLI في: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
يقوم المثبت تلقائيًا بإضافة `chloros-cli` إلى مسار PATH في نظامك. أعد تشغيل جهازك بعد التثبيت.
{% endhint %}

#### Linux

قم بتثبيت حزمة `.deb` الخاصة بهيكلية نظامك:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

للحصول على إعدادات Linux التفصيلية، انظر [تثبيت Linux](linux/linux-installation.md).

### الإعداد الأولي

قبل استخدام CLI، قم بتنشيط ترخيص Chloros+ الخاص بك:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### الاستخدام الأساسي

معالجة مجلد بالإعدادات الافتراضية:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## مرجع الأوامر

### الصيغة العامة

```
chloros-cli [global-options] <command> [command-options]
```

***

## الأوامر

### `process` - معالجة الصور

معالجة الصور في مجلد باستخدام المعايرة.

**الصيغة:**

```bash
chloros-cli process <input-folder> [options]
```

**أمثلة:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### خيارات أمر المعالجة

| الخيار                | النوع    | الافتراضي        | الوصف                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | المسار    | _مطلوب_     | المجلد الذي يحتوي على صور متعددة الأطياف بتنسيقات RAW/JPG                                         |
| `-o, --output`        | المسار    | نفس الإدخال  | مجلد الإخراج للصور المعالجة                                                     |
| `-n, --project-name`  | سلسلة  | يتم إنشاؤها تلقائيًا | اسم المشروع المخصص                                                                    |
| `--vignette`          | علامة    | ممكّن        | تمكين تصحيح التظليل                                                             |
| `--no-vignette`       | علامة    | -              | تعطيل تصحيح التظليل                                                            |
| `--reflectance`       | علامة    | ممكّن        | تمكين معايرة الانعكاس                                                         |
| `--no-reflectance`    | علامة    | -              | تعطيل معايرة الانعكاس                                                        |
| `--ppk`               | علامة    | معطل       | تطبيق تصحيحات PPK من بيانات مستشعر الضوء .daq                                      |
| `--format`            | اختيار  | TIFF (16 بت)  | تنسيق الإخراج: `TIFF (16-bit)`، `TIFF (32-bit, Percent)`، `PNG (8-bit)`، `JPG (8-bit)` |
| `--min-target-size`   | عدد صحيح | تلقائي           | الحد الأدنى لحجم الهدف بالبكسل لاكتشاف لوحة المعايرة                          |
| `--target-clustering` | عدد صحيح | تلقائي           | عتبة تجميع الأهداف (0-100)                                                    |
| `--debayer`           | اختيار  | `standard`     | طريقة إزالة التباين: `standard` أو `texture-aware` (Chloros+ فقط)                          |
| `--target`، `--targets` | علامة  | معطل       | البحث فقط عن أهداف المعايرة في مجلد فرعي &quot;target&quot; أو &quot;targets&quot; (يسرع المعالجة) |
| `--indices`           | قائمة    | لا شيء           | مؤشرات الغطاء النباتي المطلوب حسابها (على سبيل المثال، `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | سلسلة  | لا شيء           | قفل التعرض لطراز الكاميرا (السن 1)                                                 |
| `--exposure-pin-2`    | سلسلة  | لا شيء           | قفل التعرض لطراز الكاميرا (السن 2)                                                 |
| `--recal-interval`    | عدد صحيح | تلقائي           | فاصل إعادة المعايرة بالثواني                                                      |
| `--timezone-offset`   | عدد صحيح | 0              | فارق التوقيت بالساعات                                                               |

***

### `login` - مصادقة الحساب

قم بتسجيل الدخول باستخدام بيانات اعتماد Chloros+ لتمكين معالجة CLI.

**الصيغة:**

```bash
chloros-cli login <email> <password>
```

**مثال:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**الأحرف الخاصة**: استخدم علامات الاقتباس المفردة حول كلمات المرور التي تحتوي على أحرف مثل `$` أو `!` أو مسافات.
{% endhint %}

**الناتج:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - مسح بيانات الاعتماد

مسح بيانات الاعتماد المخزنة وتسجيل الخروج من حسابك.

**الصيغة:**

```bash
chloros-cli logout
```

**مثال:**

```bash
chloros-cli logout
```

**الناتج:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**مستخدمو SDK**: يوفر Python SDK أيضًا طريقة برمجية `logout()` لمسح بيانات الاعتماد داخل نصوص Python البرمجية. انظر [وثائق Python SDK](api-python-sdk.md#logout) للحصول على التفاصيل.
{% endhint %}

***

### `status` - التحقق من حالة الترخيص

عرض حالة الترخيص والمصادقة الحالية.

**الصيغة:**

```bash
chloros-cli status
```

**مثال:**

```bash
chloros-cli status
```

**الناتج:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - التحقق من تقدم التصدير

مراقبة تقدم التصدير في الخيط 4 أثناء المعالجة أو بعدها.

**الصيغة:**

```bash
chloros-cli export-status
```

**مثال:**

```bash
chloros-cli export-status
```

**حالة الاستخدام:** استدعاء هذا الأمر أثناء تشغيل المعالجة للتحقق من تقدم التصدير.***

### `language` - إدارة لغة الواجهة

عرض أو تغيير لغة واجهة CLI.

**الصيغة:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**أمثلة:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### اللغات المدعومة (38 لغة إجمالاً)

| الرمز    | اللغة              | الاسم الأصلي      |
| ------- | --------------------- | ---------------- |
| `en`    | الإنجليزية               | English          |
| `es`    | الإسبانية               | Español          |
| `pt`    | البرتغالية            | Português        |
| `fr`    | الفرنسية                | Français         |
| `de`    | الألمانية                | Deutsch          |
| `it`    | الإيطالية               | Italiano         |
| `ja`    | اليابانية              | 日本語              |
| `ko`    | الكورية                | 한국어              |
| `zh`    | الصينية (المبسطة)  | 简体中文             |
| `zh-TW` | الصينية (التقليدية) | 繁體中文             |
| `ru`    | الروسية               | Русский          |
| `nl`    | الهولندية                | Nederlands       |
| `ar`    | العربية                | العربية          |
| `pl`    | البولندية                | Polski           |
| `tr`    | التركية               | Türkçe           |
| `hi`    | الهندية                 | हिंदी            |
| `id`    | الإندونيسية            | Bahasa Indonesia |
| `vi`    | الفيتنامية            | Tiếng Việt       |
| `th`    | التايلاندية                  | ไทย              |
| `sv`    | السويدية               | Svenska          |
| `da`    | الدنماركية                | Dansk            |
| `no`    | النرويجية             | Norsk            |
| `fi`    | الفنلندية               | Suomi            |
| `el`    | اليونانية                 | Ελληνικά         |
| `cs`    | التشيكية                 | Čeština          |
| `hu`    | المجرية             | Magyar           |
| `ro`    | الرومانية              | Română           |
| `uk`    | الأوكرانية             | Українська       |
| `pt-BR` | البرتغالية البرازيلية  | Português Brasileiro |
| `zh-HK` | الكانتونية             | 粵語             |
| `ms`    | الماليزية                 | Bahasa Melayu    |
| `sk`    | السلوفاكية                | Slovenčina       |
| `bg`    | البلغارية             | Български        |
| `hr`    | الكرواتية              | Hrvatski         |
| `lt`    | الليتوانية            | Lietuvių         |
| `lv`    | اللاتفية               | Latviešu         |
| `et`    | الإستونية              | Eesti            |
| `sl`    | السلوفينية             | Slovenščina      |

{% hint style="success" %}
**الاستمرار التلقائي**: يتم حفظ تفضيلات اللغة الخاصة بك في `~/.chloros/cli_language.json` وتستمر عبر جميع الجلسات.
{% endhint %}

***

### `set-project-folder` - تعيين مجلد المشروع الافتراضي

تغيير موقع مجلد المشروع الافتراضي (مشترك مع واجهة المستخدم الرسومية في Windows).

**الصيغة:**

```bash
chloros-cli set-project-folder <folder-path>
```

**أمثلة:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - إظهار مجلد المشروع

عرض موقع مجلد المشروع الافتراضي الحالي.

**الصيغة:**

```bash
chloros-cli get-project-folder
```

**مثال:**

```bash
chloros-cli get-project-folder
```

**الناتج:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - إعادة التعيين إلى الإعدادات الافتراضية

إعادة تعيين مجلد المشروع إلى الموقع الافتراضي.

**الصيغة:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - تشغيل تشخيصات النظام

تشغيل 7 فحوصات تشخيصية للتحقق من تكوين النظام.

**الصيغة:**

```bash
chloros-cli selftest
```

**التشخيصات التي يتم إجراؤها:**

1. فحص الإصدار
2. توفر المنفذ (5000)
3. بدء تشغيل الخلفية
4. اختبار الاتصال API
5. معلومات النظام واكتشاف وحدة معالجة الرسومات (GPU)
6. التحقق من نماذج مزيل الضوضاء
7. فحص توفر CUDA

{% hint style="info" %}
**مفيد في استكشاف الأخطاء وإصلاحها**: قم بتشغيل `selftest` بعد التثبيت للتحقق من تكوين نظامك بشكل صحيح، خاصةً على Linux/Jetson حيث قد يحتاج إعداد وحدة معالجة الرسومات (GPU) و CUDA إلى التحقق.
{% endhint %}

***

### `update` - البحث عن التحديثات (Linux فقط)

ابحث عن تحديثات CLI وقم بتثبيتها على أنظمة Linux.

**الصيغة:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| الخيار    | الوصف                        |
| --------- | ---------------------------------- |
| `--check` | البحث عن التحديثات فقط، دون تثبيتها |

{% hint style="info" %}
هذا الأمر متاح على Linux فقط. في Windows، يتم توفير التحديثات من خلال المثبت.
{% endhint %}

***

## الخيارات العامة

تنطبق هذه الخيارات على جميع الأوامر:

| الخيار            | النوع    | الافتراضي       | الوصف                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | المسار    | يتم الكشف عنه تلقائيًا | مسار الملف القابل للتنفيذ للخلفية                       |
| `--port`          | عدد صحيح | 5000          | رقم منفذ الخلفية API                          |
| `--restart`       | علامة    | -             | فرض إعادة تشغيل الخلفية (إنهاء العمليات الحالية) |
| `--version`       | علامة    | -             | عرض معلومات الإصدار والخروج                |
| `--help`          | علامة    | -             | عرض معلومات المساعدة والخروج                   |

{% hint style="info" %}
**الكشف التلقائي عن الخلفية**: يتم الكشف التلقائي عن مسار `--backend-exe` حسب النظام الأساسي:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (يدوي)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**مثال مع الخيارات العامة:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## دليل إعدادات المعالجة

### المعالجة المتوازية والتكيف الحسابي الديناميكي

يتضمن Chloros 1.1.0 [التكيف الحسابي الديناميكي](processing-architecture/dynamic-compute-adaptation.md) — حيث **يكتشف محرك المعالجة أجهزتك تلقائيًا** ويختار الإستراتيجية المثلى:

| النظام الأساسي | الإستراتيجية | العمال | خط الأنابيب | ملاحظات |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | فعالة في استخدام الذاكرة، متسلسلة |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | معالجة GPU متزامنة |
| **كمبيوتر مكتبي مزود بوحدة معالجة رسومات (GPU) سعة 8 جيجابايت** | `GPU_SINGLE` | 3 | `tiled_gpu` | أداء جيد للكمبيوتر المكتبي |
| **كمبيوتر مكتبي مزود بوحدة معالجة رسومات (GPU) بسعة 12 جيجابايت أو أكثر** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | أداء مثالي للكمبيوتر المكتبي |
| **نظام يعمل بمعالج فقط** | `CPU_PARALLEL` | النوى - 1 | `cpu_fallback` | لا حاجة لوحدة معالجة رسومات |

{% hint style="success" %}
**لا حاجة للتكوين اليدوي!** يقوم Chloros بالكشف التلقائي عن وحدة المعالجة المركزية (CPU) ووحدة معالجة الرسومات (GPU) وذاكرة الوصول العشوائي (RAM) وأجهزة استشعار الحرارة (في Jetson)، ثم يقوم تلقائيًا بتكوين مسار المعالجة الأمثل.
{% endhint %}

### طرق إزالة التباين

| الطريقة | علامة CLI | الجودة | السرعة | الترخيص |
| --- | --- | --- | --- | --- |
| **قياسي (سريع، جودة متوسطة)** | `--debayer standard` | جيد | سريع | مجاني / Chloros+ |
| **مع مراعاة النسيج (بطيئة، جودة عالية)** | `--debayer texture-aware` | عالية | بطيئة | Chloros+ فقط |

طريقة إزالة التباين الافتراضية هي **قياسي**. تستخدم طريقة**مراعاة النسيج** نموذج إزالة الضوضاء بالذكاء الاصطناعي/التعلم الآلي للحصول على مخرجات بأعلى جودة، ولكنها تتطلب ترخيص Chloros+ ووحدة معالجة رسومات NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### تصحيح التظليل

**وظيفته:** يصحح انخفاض الإضاءة عند حواف الصورة (الزوايا الداكنة الشائعة في صور الكاميرا).

* **ممكّن افتراضيًا** - يجب على معظم المستخدمين إبقاء هذه الميزة ممكّنة
* استخدم `--no-vignette` لتعطيلها

{% hint style="success" %}
**التوصية**: قم دائمًا بتمكين تصحيح التظليل لضمان سطوع موحد عبر الإطار.
{% endhint %}

### معايرة الانعكاس

يحول قيم المستشعر الأولية إلى نسب انعكاس موحدة باستخدام لوحات المعايرة.

* **ممكّن افتراضيًا** - ضروري لتحليل الغطاء النباتي
* يتطلب لوحات هدف المعايرة في الصور
* استخدم `--no-reflectance` لتعطيل

{% hint style="info" %}
**المتطلبات**: تأكد من أن لوحات المعايرة معرضة بشكل صحيح ومرئية في صورك لتحويل الانعكاس بدقة.
{% endhint %}

### تصحيحات PPK

**وظيفتها:** يطبق تصحيحات حركية معالجة لاحقًا باستخدام بيانات سجل DAQ-A-SD لتحسين دقة GPS.

* **معطل افتراضيًا**
* استخدم `--ppk` للتفعيل
* يتطلب ملفات .daq في مجلد المشروع من مستشعر الضوء DAQ-A-SD MAPIR.

### تنسيقات الإخراج

<table><thead><tr><th width="197">التنسيق</th><th width="130.20001220703125">عمق البت</th><th width="116.5999755859375">حجم الملف</th><th>الأفضل لـ</th></tr></thead><tbody><tr><td><strong>TIFF (16 بت)</strong> ⭐</td><td>عدد صحيح 16 بت</td><td>كبير</td><td>تحليل نظم المعلومات الجغرافية، التصوير المساحي (موصى به)</td></tr><tr><td><strong>TIFF (32 بت، نسبة مئوية)</strong></td><td>عدد عائم 32 بت</td><td>كبير جدًا</td><td>التحليل العلمي، البحث</td></tr><tr><td><strong>PNG (8 بت)</strong></td><td>عدد صحيح 8 بت</td><td>متوسط</td><td>الفحص البصري، المشاركة عبر الويب</td></tr><tr><td><strong>JPG (8 بت)</strong></td><td>عدد صحيح 8 بت</td><td>صغير</td><td>معاينة سريعة، إخراج مضغوط</td></tr></tbody></table>***

## الأتمتة والبرمجة النصية

### المعالجة الدفعية في PowerShell (Windows)

معالجة مجلدات مجموعات البيانات المتعددة تلقائيًا على Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### نص برمجي دفعي لـ Windows (Windows)

حلقة بسيطة للمعالجة الدفعية على Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### معالجة الدُفعات باستخدام Bash (Linux)

معالجة مجلدات مجموعات البيانات المتعددة على Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### نص برمجي أتمتة Python (متعدد المنصات)

أتمتة متقدمة مع معالجة الأخطاء (تعمل على Windows و Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## سير عمل المعالجة

### سير العمل القياسي

1. **الإدخال**: مجلد يحتوي على أزواج من الصور بتنسيقات RAW/JPG
2. **الاكتشاف**: يقوم CLI بالمسح التلقائي للبحث عن ملفات الصور المدعومة
3. **المعالجة**: يتكيف الوضع المتوازي مع نوى وحدة المعالجة المركزية (CPU) لديك (Chloros+)
4. **الإخراج**: ينشئ مجلدات فرعية لطرازات الكاميرات تحتوي على الصور المعالجة

### مثال على بنية الإخراج

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### تقديرات وقت المعالجة

أوقات المعالجة النموذجية لـ 100 صورة (12 ميجابكسل لكل منها):

| النظام الأساسي | الوضع | الوقت المقدر | ملاحظات |
| --- | --- | --- | --- |
| **كمبيوتر مكتبي مزود بوحدة معالجة رسومات (GPU) بسعة 12 جيجابايت أو أكثر** | `GPU_PARALLEL` | 5-10 دقائق | الخيار الأسرع |
| **كمبيوتر مكتبي مزود بوحدة معالجة رسومات (GPU) سعة 8 جيجابايت** | `GPU_SINGLE` | 10-15 دقيقة | أداء جيد |
| **Jetson Orin NX سعة 16 جيجابايت** | `GPU_PARALLEL` | 15-25 دقيقة | حوسبة طرفية |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 30-60 دقيقة | محدودة الذاكرة |
| **CPU فقط** | `CPU_PARALLEL` | 20-40 دقيقة | لا تتطلب GPU |

{% hint style="info" %}
**نصيحة حول الأداء**: يختلف وقت المعالجة بناءً على عدد الصور والدقة وطريقة إزالة التباين والأجهزة. تستغرق طريقة إزالة التباين التي تراعي النسيج وقتًا أطول بكثير من الطريقة القياسية. انظر [التكيف الديناميكي للحوسبة](processing-architecture/dynamic-compute-adaptation.md) للحصول على التفاصيل.
{% endhint %}

***

## استكشاف الأخطاء وإصلاحها

### لم يتم العثور على CLI

**خطأ Windows:**

```
'chloros-cli' is not recognized as an internal or external command
```

**حلول Windows:**

1. تحقق من موقع التثبيت:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. استخدم المسار الكامل إذا لم يكن موجودًا في PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. أضف إلى PATH يدويًا:
   * افتح خصائص النظام → متغيرات البيئة
   * قم بتحرير متغير PATH
   * أضف: `C:\Program Files\Chloros\resources\cli`
   * أعد تشغيل المحطة الطرفية

**Linux خطأ:**

```
chloros-cli: command not found
```

**Linux الحلول:**

1. تحقق من التثبيت:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. أعد تحميل شل:

```bash
source ~/.bashrc
```

3. تحقق من الأذونات:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### فشل بدء تشغيل الخلفية**خطأ:**

```

Backend failed to start within 30 seconds
```

**الحلول:**

1. تحقق مما إذا كان الخادم الخلفي قيد التشغيل بالفعل (أغلقه أولاً)
2. تحقق من أن جدار الحماية لا يحجب الاتصال (Windows) أو تحقق من توفر المنفذ (Linux: `lsof -i :5000`)
3. جرب منفذًا مختلفًا:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. فرض إعادة تشغيل الخلفية:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. في Linux، تحقق من وجود الملف القابل للتنفيذ للخلفية:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### مشكلات الترخيص / المصادقة**خطأ:**

```

Chloros+ license required for CLI access
```

**الحلول:**

1. تحقق من أن لديك اشتراكًا نشطًا في Chloros+
2. قم بتسجيل الدخول باستخدام بيانات اعتمادك:

```bash
chloros-cli login user@example.com 'password'
```

3. تحقق من حالة الترخيص:

```bash
chloros-cli status
```

4. اتصل بالدعم الفني: info@mapir.camera

***

### لم يتم العثور على صور**خطأ:**

```

No images found in the specified folder
```

**الحلول:**

1. تحقق من أن المجلد يحتوي على التنسيقات المدعومة (.RAW، .TIF، .JPG)
2. تحقق من صحة مسار المجلد (استخدم علامات الاقتباس للمسارات التي تحتوي على مسافات)
3. تأكد من أن لديك أذونات قراءة للمجلد
4. تحقق من صحة امتدادات الملفات

***

### توقف المعالجة أو تعطلها**الحلول:**

1. تحقق من مساحة القرص المتاحة (تأكد من كفايتها للإخراج)
2. أغلق التطبيقات الأخرى لتحرير الذاكرة
3. قلل عدد الصور (قم بالمعالجة على دفعات)

***

### المنفذ قيد الاستخدام بالفعل**الخطأ:**

```

Port 5000 is already in use
```

**الحلول:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## الأسئلة الشائعة

### س: هل أحتاج إلى ترخيص لـ CLI؟

**ج:**نعم! يتطلب CLI**ترخيص Chloros+** مدفوعًا.

* ❌ الخطة القياسية (المجانية): CLI معطل
* ✅ خطط Chloros+ (المدفوعة): CLI ممكّن بالكامل

اشترك في: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### س: هل يمكنني استخدام CLI على خادم بدون واجهة مستخدم رسومية (GUI)؟**ج:** نعم! يعمل CLI بدون واجهة مستخدم تمامًا. هذه هي حالة الاستخدام الأساسية في Linux.**خادم Windows:**
* خادم Windows 2016 أو أحدث
* Visual C++ Redistributable مثبت

**خادم Linux:**
* Ubuntu 20.04+ / Debian 11+ (amd64) أو JetPack 6 (arm64)
* التثبيت عبر حزمة `.deb`

**كلا النظامين:**
* ذاكرة وصول عشوائي (RAM) بسعة 8 جيجابايت كحد أدنى (يوصى بـ 16 جيجابايت)
* تنشيط الترخيص لمرة واحدة: `chloros-cli login user@example.com 'password'`

***

### س: أين يتم حفظ الصور المعالجة؟**ج:**بشكل افتراضي، يتم حفظ الصور المعالجة في**نفس المجلد الذي تم إدخالها منه** في مجلدات فرعية لطراز الكاميرا (على سبيل المثال، `Survey3N_RGN/`).

استخدم خيار `-o` لتحديد مجلد إخراج مختلف:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### س: هل يمكنني معالجة عدة مجلدات في وقت واحد؟**ج:** ليس مباشرةً في أمر واحد، ولكن يمكنك استخدام البرمجة النصية لمعالجة المجلدات بالتسلسل. راجع قسم [الأتمتة والبرمجة النصية](CLI.md#automation--scripting).***

### س: كيف أحفظ مخرجات CLI في ملف سجل؟**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**الدُفعات:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### س: ماذا يحدث إذا ضغطت على Ctrl+C أثناء المعالجة؟**ج:** سيقوم CLI بما يلي:

1. إيقاف المعالجة بشكل سلس
2. إيقاف تشغيل الخلفية
3. الخروج برمز 130

قد تبقى الصور التي تمت معالجتها جزئيًا في مجلد الإخراج.

***

### س: هل يمكنني أتمتة معالجة CLI؟**ج:** بالطبع! تم تصميم CLI للأتمتة. انظر [الأتمتة والبرمجة النصية](CLI.md#automation--scripting) لأمثلة PowerShell (Windows) و Batch (Windows) Bash (Linux)، وPython (متعدد المنصات).***

### س: كيف يمكنني التحقق من إصدار CLI؟**ج:**

```bash
chloros-cli --version
```

**الناتج:**

```

Chloros CLI 1.1.0
```

***

## الحصول على المساعدة

### مساعدة سطر الأوامر

اعرض معلومات المساعدة مباشرةً في CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### قنوات الدعم

* **البريد الإلكتروني**: info@mapir.camera
* **الموقع الإلكتروني**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **الأسعار**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## أمثلة كاملة

### المثال 1: المعالجة الأساسية

المعالجة بالإعدادات الافتراضية (التدرج اللوني، الانعكاس):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### المثال 2: مخرجات علمية عالية الجودة

32 بت عائم TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### المثال 3: معالجة سريعة للمعاينة

8 بت PNG بدون معايرة للمراجعة السريعة:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### المثال 4: المعالجة المصححة بواسطة PPK

تطبيق تصحيحات PPK مع الانعكاس:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### المثال 5: موقع إخراج مخصص

المعالجة إلى موقع مختلف بتنسيق محدد:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### المثال 6: سير عمل المصادقة

سير عمل المصادقة الكامل (نفسه على جميع المنصات):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### المثال 7: استخدام متعدد اللغات

تغيير لغة الواجهة (نفسه على جميع المنصات):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
