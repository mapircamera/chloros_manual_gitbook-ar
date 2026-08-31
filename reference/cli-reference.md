# Chloros CLI مرجع

**الإصدار:**

1.2.0**تاريخ الإنشاء:**2026-07-29 19:19 ·**تاريخ المراجعة:** 2026-08-30**الجمهور المستهدف:** مُحسّن للاستخدام من قِبل نماذج اللغة الكبيرة (LLM)؛ قابل للقراءة البشرية.**النطاق:** كل أمر فرعي موجه للمستخدم في `chloros-cli`، مع الخيارات والأمثلة القابلة للنسخ واللصق.

هذا المستند هو المرجع الكامل لأداة سطر الأوامر `chloros-cli` المرفقة مع MAPIR Chloros. وقد صُمم ليكون شاملاً عن قصد بحيث يمكن لـ LLM (أو الإنسان) يمكنه تكوين أي سير عمل مدعوم من القوائم أدناه دون الحاجة إلى فحص شفرة المصدر.

إذا كنت بحاجة إلى النقاط البارزة فقط، فانتقل إلى:
- [بدء سريع في خمس دقائق](#five-minute-quickstart)
- [سير عمل LATTICE Camera First-Connect](#lattice-camera-first-connect-workflow)
- [سير عمل DAQ Sensor First-Connect](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [أوضاع الالتقاط والمسجلات وإعادة المعالجة دون اتصال](#capture-modes-recorders--offline-reprocess)

---

## القواعد

- تبدأ جميع الأوامر ببادئة `chloros-cli`. في Windows، يكون الملف الثنائي هو `chloros-cli.exe`؛ أما في Linux /Jetson، فهو `chloros-cli`.
- تُعرض الوسيطات الاختيارية على النحو التالي: `--flag`. أما الوسيطات الموضعية الإلزامية فتُعرض بدون أقواس.
- في حالة وجود قيمة افتراضية، يؤدي حذف العلامة إلى استخدام تلك القيمة.
- يُعد «CLI» عميلًا خفيفًا من نوع «HTTP» يعمل عبر الخادم الخلفي «Chloros» (خادم Flask على `127.0.0.1:5000`). يتم تشغيل الخلفية تلقائيًا بواسطة معظم الأوامر. يوجه `CHLOROS_BACKEND_URL=<url>` مجموعات الأوامر **`lattice`**و**`project`**و**`daq pool-*`** إلى الخلفية البعيدة — الأوامر الأساسية (`process`، `login`، `logout`، `status`، `export-status`، `time-sync`، `selftest`) تثبت عمدًا `http://127.0.0.1:<port>` وتتجاهلها (القيمة الثابتة لـ IPv4 تتجنب عقوبة «Windows» التي تبلغ `localhost`→`::1` عقوبة تبلغ حوالي 2 ثانية لكل طلب). انظر [متغيرات البيئة](#environment-variables).
- يلزم تسجيل الدخول بحساب Chloros+ لجميع استدعاءات SDK / CLI (قم بتشغيل `chloros-cli login` مرة واحدة لكل جهاز؛ يتم تخزينه مؤقتًا في `~/.chloros/`).
- تستخدم الأمثلة مسارات Linux؛ في Windows استبدل `/home/user/...` بـ `C:/Users/.../...`.

---

## ملخص المستوى الأعلى

```
chloros-cli [global options] COMMAND [command options]
```

### الخيارات العامة

| العلامة | الوصف |
| --- | --- |
| `--backend-exe PATH` | تجاوز الملف التنفيذي للخلفية الذي تم اكتشافه تلقائيًا. |
| `--port N` | منفذ الخلفية HTTP (الافتراضي: `5000`). |
| `-v, --verbose` | تمكين الإخراج التفصيلي. |
| `--restart` | إعادة تشغيل الخلفية قسريًّا (إيقاف أي عمليات `backend_server.py` قيد التشغيل). |
| `--version` | عرض الإصدار (`Chloros CLI 1.2.0`). |
| `--help` | عرض المساعدة العامة. |

### فهرس الأوامر

| الأمر | الغرض |
| --- | --- |
| [`process`](#chloros-cli-process) | معالجة مجلد من عمليات التقاط Survey3 أو LATTICE من البداية إلى النهاية. |
| [`login`](#chloros-cli-login) | مصادقة هذا الجهاز باستخدام حساب Chloros+. |
| [`logout`](#chloros-cli-logout) | مسح بيانات الاعتماد المخزنة مؤقتًا. |
| [`status`](#chloros-cli-status) | عرض حالة الترخيص / المصادقة الحالية. |
| [`export-status`](#chloros-cli-export-status) | عرض تقدم عملية التصدير المباشر لـ Thread-4 أثناء تشغيل `process`. |
| [`language`](#chloros-cli-language) | تعيين أو عرض قائمة بلغات العرض في CLI (يتم دعم 38 لغة). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | مجلد المشروع الافتراضي (مشترك مع واجهة المستخدم الرسومية). |
| [`update`](#chloros-cli-update) | البحث عن تحديثات CLI وتثبيتها (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | تشخيص النظام + اختبارات التشغيل الأولية. |
| [`time-sync`](#chloros-cli-time-sync) | حالة/التحكم في «جراندماستر» PTP. |
| [`lattice`](#chloros-cli-lattice) | التحكم في كاميرا LATTICE والتصوير (أكثر من 45 أمرًا فرعيًا). |
| [`daq`](#chloros-cli-daq) | التحكم في مستشعرات الطيف DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | فتح وتشغيل مشروع محفوظ من Chloros (الكاميرات + أجهزة DAQ). |

---

## التثبيت

يأتي `chloros-cli` مضمنًا في أداة تثبيت سطح المكتب Chloros على كل منصة مدعومة — ولا يوجد تنزيل منفصل لـ CLI. يؤدي تثبيت حزمة النظام الأساسي إلى إضافة `chloros-cli` إلى `PATH` الخاص بك جنبًا إلى جنب مع تطبيق سطح المكتب والملف الثنائي للخلفية الذي يشغله.

أحدث التنزيلات: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> يتضمن المثبت أيضًا نصوص برمجية مريحة لتشغيل التطبيق (`Chloros_CLI.bat` / `Chloros_CLI.ps1`، `Launch_CLI.*`، `chloros-cli.sh`) تفتحCLI؛ وقد تم تناولها في [CLI دليل المستخدم](../CLI.md) ولن يتم تكرارها هنا.

### Windows (.exe)

1. قم بتنزيل أداة التثبيت Windows من صفحة التنزيل.
2. قم بتشغيل `Chloros-Setup-x.y.z.exe` واتبع إرشادات المعالج. مسار التثبيت الافتراضي هو `C:\Program Files\Chloros\` (يتم وضع ملف CLI في `C:\Program Files\Chloros\cli\`، والذي يضيفه برنامج التثبيت إلى متغير PATH).
3. افتح محطة طرفية جديدة (`cmd.exe`، أو PowerShell، أو محطة طرفية Windows) حتى يتم التقاط الإصدار المحدث `PATH`.

```powershell
chloros-cli --version
```

يقوم المثبت تلقائيًا بإضافة `chloros-cli.exe` إلى نظامك `PATH` و ويضم وقت تشغيل Arena SDK اللازم لكاميرات LATTICE.

### Linux amd64 (.deb)

لأجهزة العمل x86_64 التي تعمل بنظام Ubuntu 22.04 LTS أو أحدث / القائمة على Debian.

> **Ubuntu 20.04 غير مدعوم.** قائمة تبعيات الحزمة مستمدة من
> ما يرتبط به الخلفية فعليًا، وهذا يشمل `libc6 (>= 2.34)`؛
> حيث توفر focal glibc 2.31. `apt` ترفض التثبيت بدلاً من السماح بفشله في
> وقت التشغيل.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

يقوم ملف .deb بتثبيت:
- `chloros-cli` إلى `/usr/bin/chloros-cli`
- الخلفية المُجمَّعة إلى `/usr/lib/chloros/chloros-backend`
- وقت تشغيل Arena SDK (لكاميرات LATTICE)
- نماذج إزالة الضوضاء، وحزم المعايرة، وتكوين قناة التحديث

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

نفس تخطيط ملف .deb الخاص بـ amd64، مع إصدار CUDA مُعدّل خصيصًا لأجهزة Jetson Orin / Orin NX / Orin Nano.

### المصادقة مرة واحدة لكل جهاز

تتطلب كل منصة تسجيل دخول لمرة واحدة عبر Chloros+ قبل أن تعمل استدعاءات SDK / CLI:

```bash
chloros-cli login user@example.com 'YourPassword'
```

يتم تخزين بيانات الاعتماد مؤقتًا في `~/.chloros/user_session.json`.

### التحقق من التثبيت

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **يلزم الاشتراك في Chloros+.**يتطلب CLI خطة Chloros+ نشطة.**Copper**هي الفئة Chloros+ المبدئية — كل فئة مدفوعة Chloros+ تتمتع بإمكانية الوصول إلى CLI / SDK؛ فقط الفئة المجانية**Iron** لا تتمتع بذلك. (خريطة معرّفات الخطط: `0`=Iron/free، `1`=Copper، `2`=Bronze، `3`=Silver، `4`=Gold.) قم بالترقية على [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> يتم فرض هذا الحد الأدنى من قبل الخلفية (backend)، وليس فقط من خلال CLI: يتم رفض أي طلب SDK / CLI يُرفض الطلب الذي يحمل العلامة -flagged دون خطة مدفوعة برمز الخطأ `403 PLAN_UPGRADE_REQUIRED`، سواء أتى من `chloros-cli`، أو من Python SDK، أو من عميل HTTP مخصص. أما المتصل الذي تم تسجيل خروجه فيحصل على الرمز `401 AUTH_REQUIRED` بدلاً من ذلك. يعمل الوصول في وضع عدم الاتصال خلال فترة السماح الخاصة بالخطة (30 يومًا شهريًّا، حتى تاريخ انتهاء الصلاحية للخطة السنوية) ويتوقف عند انتهاء تلك الفترة؛ ويستمر `chloros-cli status` في العمل حتى يظل السبب مرئيًّا (وهو المسار الوحيد SDK / CLI المعفى من بوابة المستويات — `GET /api/license-status`).

---

## البدء السريع في خمس دقائق

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

معالجة مجلد من الصور عبر مسار Chloros الكامل (الكشف عن الهدف → المعايرة → التظليل → الانعكاس → تصدير المؤشر).

### ملخص

```
chloros-cli process INPUT [OPTIONS]
```

### المعلمات الموضعية

| المعلمة | الوصف |
| --- | --- |
| `INPUT` | المسار إلى مجلد الإدخال الذي يحتوي على ملفات `.raw + .jpg` (Survey3)، أو `.tif/.tiff` (LATTICE)، أو ملفات `.dng`. |

### الخيارات الشائعة

| العلامة | القيمة الافتراضية | الوصف |
| --- | --- | --- |
| `-o, --output PATH` | مجلد جديد مزود بختم زمني ضمن مسار المشروع الافتراضي (`~/Chloros Projects` ما لم يتم تكوينه) | مجلد المشروع المراد إنشاؤه أو إعادة استخدامه. إذا كان المجلد يحتوي بالفعل على ملف `project.json`، فسيتم إنشاء يتم إنشاء مجلد تابع له باسم `_1`/`_2` بدلاً من الكتابة فوقه. |
| `-n, --project-name NAME` | تلقائي (طابع زمني) | اسم المشروع. |
| `--debayer {standard,texture-aware}` | `standard` | يستخدم `texture-aware` أداة إزالة التبخير العصبية (Chloros+ لإزالة التباين العصبي؛ أبطأ ولكن بجودة أعلى. |
| `--vignette / --no-vignette` | `--vignette` | تصحيح التظليل. |
| `--reflectance / --no-reflectance` | `--reflectance` | معايرة الانعكاس (تستخدم هدف اللوحة إن وجد، ومعايرة NIST لكل سلسلة لـ LATTICE). بالنسبة لـ LATTICE متعدد الأطياف، يعمل هذا أيضًا كمفتاح تبديل **منتج** الانعكاس — انظر [مفاتيح التبديل للتصدير لكل منتج](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | off | تطبيق تصحيحات PPK GNSS من ملفات sidecar. |
| `--exposure-pin-1 MODEL` | off | تثبيت نموذج &quot;pin-1&quot; لجهاز الكاميرا المزدوجة Survey3. |
| `--exposure-pin-2 MODEL` | معطل | تثبيت نموذج «pin-2». |
| `--recal-interval SECONDS` | 0 | فرض إعادة تشغيل حسابات المعايرة كل N ثانية من وقت الالتقاط. |
| `--timezone-offset HOURS` | محلي | تجاوز إزاحة المنطقة الزمنية المدمجة في بيانات التعريف الناتجة. |
| `--format FORMAT` | `TIFF (16-bit)` | أحد الخيارات التالية: `TIFF (16-bit)`، `TIFF (32-bit, Percent)`، `PNG (8-bit)`، أو `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | لا شيء | مؤشرات الغطاء النباتي (`NDVI`، `NDRE`، `GNDVI`، `EVI`، `SAVI`، `OSAVI`، `CIG`، …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | فرض نقطة دخول مسار المعالجة لملفات LATTICE TIFF (لا يتأثر ملف Survey3.raw). وكذلك «منفذ الهروب» الذي يسمح بمعالجة لقطة **بدون ملف raw** — انظر [كيف يبدو مجلد اللقطات](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | قيد التشغيل | إخراج الناتج الخطي بعد إزالة التباين (`Debayered_Images`). انظر [مفاتيح التبديل للتصدير لكل منتج](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | قيد التشغيل | إخراج معاينة العرض (`Preview_Images`): RGB = توازن اللون الأبيض (مصدر الإضاءة DAQ عند توفره، وإلا فإن يكون &quot;العالم الرمادي&quot;) + جاما؛ multispec = تمدد الألوان الزائفة. |
| `--radiance / --no-radiance` | قيد التشغيل | إخراج الإشعاع بنوع float32 (`Radiance_Images`، W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | مرجع لمنتج انعكاس LATTICE: `auto` = الهدف داخل الإطار الذي اجتاز مرحلة ضمان الجودة (QA) هو المرجع المطلق، مع استخدام قيمة DAQ للانعكاس الهابط (ρ = π·L/E) كخيار بديل؛ `target` = صارم (بدون استبدال بـ DAQ)؛ `daq` = مرجعية DAQ. انظر [مفاتيح التبديل للتصدير لكل منتج](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | لا شيء | دليل**المقاسة** لكل وحدة (`<serial>.csv`)؛ يتم الرجوع إلى أطياف T3/T4P الاسمية في حالة الفشل. |
| `--array-alignment / --no-array-alignment` | مشغّل | مصفوفات LATTICE: تطبيق محاذاة الوحدة إلى الوحدة الموسومة في ملف XMP الخاص بـ `Chloros:Alignment*` لكل التقاط على كل منتج معالج (إزالة التباين / المعاينة / الإشعاع / الانعكاس / المؤشر). لا يتم تنفيذ أي عملية للصور التي لا تحتوي على العلامات. |
| `--array-alignment-crop / --no-array-alignment-crop` | crop | قص | قص الصادرات المُحاذاة إلى منطقة التداخل المشتركة للمصفوفة بحيث تشترك جميع الوحدات في مساحة واحدة؛ ويحتفظ `--no-…` باللوحة الكاملة للمستشعر (ملء أسود خارج المصدر). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | إعادة أخذ العينات لتصحيح انحراف المحاذاة. `nearest` يحافظ على قيم DN المصدر الدقيقة (بدون خلط القيم الإشعاعية بين البكسلات). |

### خيارات الكشف عن الهدف

| العلامة | الوصف |
| --- | --- |
| `--min-target-size PIXELS` | الحد الأدنى لحجم لوحة الهدف (بكسل) للكاشف. |
| `--target-clustering 0-100` | حساسية التجميع. |
| `--target / --targets` | معاملة المجلد المدخل على أنه لوحة الهدف فقط (تخطي الكشف الاستطلاعي). |

### أمثلة

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### مفاتيح التبديل للتصدير لكل منتج (LATTICE متعدد الأطياف)

تتفرع معالجة LATTICE إلى **كل منتج قابل للتطبيق في مسار واحد**. أزرار التبديل الأربعة لكل نوع — `--debayered`، `--preview`، `--radiance`، `--reflectance` — تكون جميعها**مشغلة افتراضيًا**؛ استخدم النموذج `--no-<type>` لإلغاء تشغيل أحدها. لا تصدر الكاميرات الرئيسية من طراز RGB سوى الصور بعد إزالة التباين اللوني + المعاينة (بدون إشعاع/انعكاس لكل نطاق)، لذا فإن `--radiance`/`--reflectance` لا تؤدي أي وظيفة بالنسبة لها. يتم تجاهل أزرار التبديل الخاصة بـ Survey3 `.raw` (التي تتبع مسار الانعكاس/الهدف القياسي). *(تم **إزالة**علامة `--radiometric-output {reflectance,radiance,sensor-response}`**تمت إزالته** واستبداله بهذه المفاتيح؛ ولم يعد هناك مستوى `sensor-response` بعد الآن.)*

| المنتج | المخرجات | هل يلزم تيار DAQ الهابط؟ |
| --- | --- | --- |
| `--debayered` | إزالة التداخل الخطي (`Debayered_Images`). | لا. |
| `--preview` | معاينة العرض (`Preview_Images`): RGB = WB + غاما؛ multispec = تمدد الألوان الزائفة. | لا. |
| `--radiance` | float32 واط/م²/سترلين/نانومتر من السلسلة الإشعاعية الكاملة (`Radiance_Images`). | رقم |
| `--reflectance` | uint16 معامل الانعكاس ρ (`32768` = 1.0)، جاهز للاستخدام مع Pix4D. | **نعم**، ما لم يتم تثبيته بواسطة هدف داخل الإطار اجتاز اختبار ضمان الجودة (انظر أدناه). |

يختار `--reflectance-source` مرجع الانعكاسية:**`auto`**(الافتراضي) يجعل الهدف الموجود داخل الإطار الذي اجتاز اختبار ضمان الجودة**مرجعًا مطلقًا**— حيث يتم تقييم سلاسل الخطوط التجريبية المثبتة بالهدف تقييمًا متقاطعًا على اللوحات المحتجزة ويتم تطبيق النتيجة الفائزة المقاسة — مع الرجوع إلى تقسيم DAQ النزولي (ρ = π·L/E) في حالة عدم وجود هدف أو فشل ضمان الجودة؛**`target`**صارم (لا يوجد استبدال DAQ)؛**`daq`**يختار السلوك الذي يعتمد على DAQ. تأتي هندسة الهدف (ArUco / منطقة الاهتمام الثابتة / الشريط) من تكوين هدف المشروع؛ يحتفظ `--target-reflectance-dir DIR` بمسح****** (`<serial>.csv`) يتم البحث عنها بواسطة الرقم التسلسلي/QR لوحدة الهدف، مع استخدام أطياف T3/T4P الاسمية كخيار بديل.

يقوم مسار الانعكاس في DAQ بحل **الانعكاس الهابط المطابق للطابع الزمني**تلقائيًا من**`.daq`**(DAQ-U/M/E)**أو ملف `.csv` الأصلي الخاص بـ DAQ-M**الأصلي لـ DAQ-M الموجود بجانب الصور. إذا لم يتم تخزين حزمة المعايرة الخاصة بكل كاميرا أو بـ DAQ محليًّا، فإن خط الإنتاج**يستردها تلقائيًّا من AWS** عند الاستخدام الأول (يتطلب اتصالاً بالإنترنت مرة واحدة؛ ويتم تخزينها مؤقتًا تحت `~/.chloros/`).

#### قراءة وحدات البكسل الانعكاسية (Pix4D / Metashape / البرامج النصية الخاصة بك)

يتم تخزين الانعكاسية كرقم صحيح DN، و**القيمة DN التي تعني ρ = 1.0 تعتمد على الكاميرا المصدر**:

| المصدر | ρ = 1.0 هو | كيفية التحديد |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (هامش حتى ρ 2.0) | XMP `Chloros:PixelScale=32768` مطبوع على الملف. |
| Survey3 | `65535` (مقصوص عند ρ 1.0) | لا توجد علامات XMP `Chloros:*` — وهذا الغياب *هو* الإشارة. |

**اقرأ `Chloros:PixelScale` واقسم عليه** بدلاً من افتراض ثابت. العلامة مُعرَّفة في مجال uint16، لذا تظل `32768` عبر تنسيقات الإخراج التي تعيد القياس — `TIFF (16-bit)`، `PNG (8-bit)`، `JPG (8-bit)` و`TIFF (32-bit, Percent)` كلها ذاتية الوصف (قم أولاً بتطبيع نوع البيانات المخزّن إلى uint16: ×257 من 8 بت، ×65535 من العائمة).

> **هناك حالة واحدة لا تحتوي على مقياس، حسب التصميم.** عندما يتم كتابة التقاط مصدر 8 بت (BayerRG8) كـ 8 بت TIFF، فإن خط المعالجة *يقتطع* القيم إلى 0..255 بدلاً من إعادة تحجيمها، لذلك يتم تسوية كل قيمة أعلى من ρ≈0.008 إلى 255 ولا يصف الملف أي مقياس. يتجاهل ملف Chloros عمدًا كل من التوبولين `Chloros:PixelScale` و`MicaSense:RadiometricCalibration` الموجودين هناك، ويسجل السبب. **إذا كانت العلامة غائبة في ملف انعكاس LATTICE، فلا تفترض وجود مقياس — أعد التصدير بدقة 16 بت أو 32 بت** بدلاً من تقسيم وحدات البكسل التي لم تكن قابلة للتقسيم أبدًا.

#### نقل بيانات EXIF إلى التصدير

يُنسخ `process` **GPS و ExifIFD** إلى كل منتج، لذا فإن
التصدير يحمل `FocalLength`، `FNumber`، `ExposureTime`، `ISO` و `DateTimeOriginal` و
`CameraSerialNumber` جنبًا إلى جنب مع الإحداثيات الجغرافية.

**`FocalLength` ليس اختياريًا في التصوير المساحي.** يقوم Pix4D بحساب مسافة العينة الأرضية بناءً على
البعد البؤري مضافًا إليه الارتفاع؛ وفي حالة عدم وجود العلامة، فإنه يعود إلى مقياس خاطئ تمامًا. في إحدى
رحلات الطيران فوق بستان برتقال التي شملت 49 لقطة، حولت العلامة المفقودة موقعًا مساحته 411 م × 160 م إلى موقع مُعاد بناؤه
مساحته 47.8 كم × 13 كم — صورة أورثوغرافية بدقة 455 ميجابكسل تتكون في معظمها من «بيانات غير متوفرة»، مما أدى إلى اعتبارها مشكلة في التبليط و
مشكلة في ملف BigTIFF قبل أن يتحقق أي شخص من GSD. إذا ظهرت صورتك الأورثوغرافية بمقياس غير معقول،
فقم بتشغيل `exiftool -FocalLength` على المنتج المُصدَّر أولاً.

النسخة **ليست** عن قصد `-all:all`: فالعلامات الهيكلية لـ IFD0 تعطل إخراج LATTICE عند
نسخها، وتم استبعاد `ExifImageWidth` / `ExifImageHeight` لأنهما يصفان
التقاط *المصدر* — وإلا فإن أي تصدير تم تغيير حجمه سيحمل أبعادًا
تتعارض مع الصورة النقطية الخاصة به. يتم كتابة XMP مباشرةً بدلاً من نسخه، لأن ExifTool
يتجاهل علامات XMP ذات الاستدعاء نفسه عند نسخ كتلة XMP (مما سيؤدي إلى حذف علامات المعايرة MAPIR
).

### مكان حفظ المخرجات

تُكتب المنتجات **ضمن مجلد المشروع، مجمعة حسب الكاميرا ثم حسب تنسيق الملف**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

مجلد الكاميرا هو `LATT-<sensor>-<lens>-F<filter>` بالنسبة لـ LATTICE (مطابقًا لـ EXIF
`Model` الخاص باللقطة) و `<model>_<filter>` لـ Survey3 — حيث تحتفظ الكاميرتان اللتان تشتركان في المستشعر والمرشح ولكنهما تختلفان
في العدسة بشجرتين منفصلتين، لأن التظليل ومجال الرؤية والتشويه تختلف. أما مجلد التنسيق
فيتبع `--format`: `tiff16`، `tiff8`، `png8`، `jpg8`، أو `tiff32` لـ
`TIFF (32-bit, Percent)`.

> **يحتفظ كل منتج مُصدَّر باسم الملف الأصلي.** تصدير Radiance لملف
> `capture_…_raw.tif` لا يزال يُسمى `capture_…_raw.tif` — إنه موجود فقط في
> `tiff32/Radiance_Images/`. **المجلد هو الذي يحدد المنتج، وليس اسم الملف**، لذا فإن البحث
> عن `*radiance*.tif` لا يعطي أي نتيجة؛ قم بالمطابقة على الدليل بدلاً من ذلك.

### تسجيلات مستشعر الضوء — تمت معايرتها `.daq` + `.csv`

يتعامل `process` أيضًا مع تسجيلات `.daq` الموجودة في مجلد الإدخال الخاص بك، وهو **لا**
يحتاج إلى أي صور للقيام بذلك: جهاز DAQ-U / DAQ-M / DAQ-E الذي تم تشغيله بمفرده يُعد
جهاز التقاط كامل، والمجلد الذي يحتوي فقط على ملفات `.daq` يُعتبر مدخلاً صالحًا.

يمكن تسجيل جهاز DAQ **بدون** معايرته — وهذا ما تقوم به أجهزة التسجيل العامة
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) : فهي تسجل قراءات المستشعر الأولية وتضع طابعًا على الملف بحيث
يسترد موقع Chloros معايرة المصنع الخاصة بذلك المستشعر **حسب الرقم التسلسلي** (الذاكرة المؤقتة المحلية أولًا،
ثم سحابة MAPIR) ويطبقها. ويقوم `process` بكتابة النتيجة مرة أخرى:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

يحتوي ملف `.csv` على صف واحد لكل قراءة: الطابع الزمني بتوقيت UTC، ووقت التكامل، والطاقة الإجمالية،
وحدة اللوكس الضوئية/الظلامية، PPFD (وتقسيمها إلى الأزرق/الأخضر/الأحمر)، الطول الموجي الأقصى، ثم
الطيف الكامل على شبكة الأطوال الموجية الخاصة بالمستشعر. يقوم `.daq` بإعادة الاستيراد دون الحاجة إلى
المعايرة مرة ثانية.

في حالة النجاح، يُبلغ التشغيل عن `Light-sensor products written: N (calibrated .daq + .csv)`.
تصف العبارة الواردة بين قوسين ما تمت كتابته فعليًّا، لذا تقرأ
`(RAW COUNTS — this sensor has no calibration bundle)` للمستشعر الذي لا يحتوي على حزمة و
`(N calibrated, M raw counts)` للمجلد الذي يحتوي على كليهما. تستمد عناوين
عناوين `[DAQ-EXPORT]` و `[RUN-SUMMARY]` الخاصة بالخلفية تستمد صياغتها بنفس الطريقة — لا يمكن لأي من
الثلاثة وصف التصدير الخام بأنه معاير.

تسجيل DAQ-U / DAQ-M / DAQ-E الذي يتعذر الحصول على حزمة المعايرة الخاصة به— إما لأنك
غير متصل بالإنترنت، أو لأن ملف معايرة ذلك المستشعر غير موجود — يتم **تخطيه مع ذكر السبب** في
سطر `[DAQ-EXPORT]`، ولا يتم كتابته أبدًا كملف «معاير» يحتوي على القيم الأولية.
اتصل بالإنترنت وأعد التشغيل. والسبب هو الذي حدده القارئ فعليًا
لذلك الملف (مخطط غير قابل للقراءة، عدم وجود حزمة، خطأ في الكتابة)، ويُدرج ملخص التشغيل
أسبابًا **متميزة** — حيث يتم تجاهل عشرين ملفًا لسبب واحد يُقرأ كسبب
واحد، وليس عشرين تكرارًا له.

#### تصدير تسجيلات DAQ-A كعدد خام

تسبق عائلة **DAQ-A** نظام الحزم لكل رقم تسلسلي ولا تحتوي على حزمة معايرة
لجلبها — بل تُعاير في الميدان مقابل هدف انعكاس، وهذا هو
السبب في أنها لم تحتج إليها أبدًا. أدى رفض تلك التسجيلات إلى عدم وجود طريقة لإخراج
أرقامها على الإطلاق، لذا يتم تصديرها تحت **اسم مختلف**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

اسم ملف مختلف بدلاً من علامة داخل الملف، لأن الادعاء يجب أن يبقى
عند إرساله عبر البريد الإلكتروني كاسم عاري. يذكر رأس الملف `.csv`
`raw spectral sensor counts (NOT irradiance)` ويحذر من أن القيم قابلة للمقارنة
**داخل** الملف — وهو بالضبط الغرض الذي تستخدمه المعايرة القائمة على الهدف — و
ليس عبر أجهزة الاستشعار. يتم كتابة الأعمدة الضوئية المعتمدة على الطاقة (الطاقة الإجمالية، واللوكس الضوئي و
اللوكس السكوتوبي، وPPFD) بقيمة **NULL** بدلاً من دمجها من القيم المعدودة، ويشير ملخص التشغيل
التشغيل تشير إلى `RAW COUNTS`، لذا لا يمكن قراءة ما تم «تصديره» في السجل على أنه إشعاع.

تسجيلات الإصدارات القديمة **v1.01 / v1.02** (يتم تسجيلها بواسطة DAQ-A-SD) لا تحتوي على حقبة لكل قراءة،
بل تحتوي فقط على وقت كتابة الملف. لا يزال مُطابق الصورة↔الإشعاع الهابط يرفضها — حيث إن مطابقة
إطار مع وقت الكتابة ستكون خاطئة بشكل غير مرئي — لكن أداة التصدير تقرأها، و
يقوم CSV بطباعة `clock=daq_created_on`، لذا يحدد المنتج الساعة التي يعمل عليها.

### ملاحظات

- يكتشف `process` تلقائيًا ما إذا كان المجلد الخاص بك من نوع «Survey3» أو «LATTICE» أو مختلطًا.
- يتم بث التقدم عبر «Server-Sent Events؛ ويُظهر CLI التقدم المباشر لكل مؤشر ترابط (الكشف، التحليل، المعالجة، التصدير).
- بالنسبة لـ Linux /Jetson، يقوم CLI بفحص ذاكرة التبديل وقد يصدر تحذيرًا قبل معالجة المجلدات الكبيرة. كما يقوم برنامج إزالة التباين (debayer) المراعي للنسيج بتطبيق حد أقصى لتردد وحدة معالجة الرسومات (GPU) تلقائيًا على أجهزة Jetson منخفضة الطاقة (Nano، Orin Nano).
- في حالة النجاح، يُبلغ التشغيل عن عدد منتجات الصور التي تم كتابتها (`Image products written: N`).

#### يفشل التشغيل الذي لا يكتب أي صور

إذا طلبت منتجات ولم يكتب التشغيل **أي شيء** — فقط `project.json` و
`calibration_data.json` — فإن `process` يعتبر ذلك فشلًا: حيث يُعرض
`Processing finished but wrote no image products.` و**يخرج بقيمة غير صفرية**، لذا يمكن للبرنامج النصي
اكتشاف ذلك. تذكر الرسالة اسم مجلد المشروع والأسباب المعتادة:

- لم يتم التعرف على مجلد الإدخال كمجلد التقاط (تحقق من التخطيط و`--input-level`)، أو
- تم تخطي كل منتج مطلوب باعتباره غير قابل للتطبيق على تلك الكاميرات (على سبيل المثال، طلب
  الإشعاع/الانعكاس من كاميرات RGB -only فقط).

أعدإعادة التشغيل باستخدام `--verbose` وتحقق من سجل الخلفية بحثًا عن الأسطر `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`،
التي تفسر حالات التخطي لكل كاميرا والتي لا تصل بخلاف ذلك إلى مخرجات CLI.

لا يزال التشغيل المتعمد الذي يعتمد على البيانات الوصفية فقط — مع إيقاف تشغيل جميع المنتجات وعدم استخدام `--indices` — يمثل
**نجاحًا**، لأن إخراج صورة فارغة هو النتيجة الصحيحة في هذه الحالة.

وكذلك **التشغيل باستخدام مستشعر الضوء فقط**: لا يحتوي مجلد تسجيلات `.daq` على صور لتصديرها
بحكم التعريف، ويتم تقييم التشغيل بناءً على ملفات `.daq` / `.csv` المعايرة التي كتبها بدلاً من ذلك.

---

## `chloros-cli login`

قم بمصادقة هذا الجهاز باستخدام حساب سحابي Chloros+. يتم تخزين بيانات الاعتماد بشكل آمن في ذاكرة التخزين المؤقت في `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### أمثلة

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (حذف أجزاء من كلمة المرور أو تكرارها). عند ظهور الرمز 401، يقوم CLI تلقائيًا بإعادة المحاولة مع إضافة `$$`إضافة، ثم بنصف كلمة المرور بعد إزالة التكرار؛ وإذا نجحت المحاولة، فسيتم تسجيل دخولك وعرض صيغة الفواصل المفردة الصحيحة لاستخدامها في المرة القادمة.

> **الاستخدام بدون واجهة مستخدم/عبر البرامج النصية: عدم وجود جلسة مخزنة مؤقتًا يعني ظهور موجه تفاعلي، وليس فشلًا سريعًا.** أي أمر يُنشئ خلفية (`process`، `status`، `export-status`، `time-sync`، …) يتم تشغيله بدون ترخيص/جلسة مخزنة مؤقتًا، فإنه ينتقل إلى موجه تفاعلي لـ `Email:` / `Password:` على stdin قبل المتابعة. وبالتالي، فإن المهمة غير المراقبة التي لا تحتوي على جلسة مخزنة مؤقتًا ستتوقف في انتظار الإدخال — قم بتشغيل `chloros-cli login EMAIL PASSWORD` مرة واحدة لكل جهاز قبل جدولة العمل بدون واجهة مستخدم.

---

## `chloros-cli logout`

يمسح الجلسة المخزنة مؤقتًا ويفرض تسجيل دخول جديد عند الاستدعاء التالي.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

يعرض فئة الترخيص الحالية (Iron/Copper/Bronze/Silver/Gold)، والمستخدم المصادق عليه، وعدد عمليات ربط الأجهزة.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

استقصاء تقدم التصدير المباشر لـ Thread-4. يمكن استدعاء هذه الأوامر بأمان **أثناء** تشغيل `process` من شل آخر.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

تعيين لغة العرض لـ «CLI» (يتم دعم 38 لغة، بما في ذلك CJK وRTL وIndic). يتم التراجع بسلاسة إلى اللغة الإنجليزية على وحدات التحكم القديمة التي لا تستطيع عرض النص البرمجي.

```
chloros-cli language [LANG_CODE] [--list]
```

### أمثلة

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## أوامر مجلد المشروع

تتولى هذه الأوامر إدارة موقع مجلد المشروع الافتراضي (المشترك مع واجهة المستخدم الرسومية).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Jetson فقط. يتحقق من `version_url` من `/etc/chloros/update.conf` ويعرض تنزيل وتثبيت `.deb` المطابق.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

على Linux /Jetson، يقوم CLI أيضًا **بفحص التحديثات تلقائيًا عند كل بدء تشغيل** (دون حجب، ولا يؤخر الأمر أبدًا): فهو يقرأ `/etc/chloros/update.conf`، ويخزن النتيجة مؤقتًا لمدة ساعة في `~/.chloros/update_cache.json`، ويطبع `Update available: vX.Y.Z / Run: chloros-cli update` عند وجود إصدار أحدث. يتم تخطي العملية بصمت في حالة حدوث أي خطأ أو عند وجود Windows.

---

## `chloros-cli selftest`

يُجري اختبارًا أوليًا من 7 خطوات: الإصدار، وتوافر المنفذ، وبدء تشغيل الخلفية، و`/api/test`، و`/api/system-info` (GPU/CUDA/PyTorch)، وجود نموذج مزيل الضوضاء، جاهزية CUDA + مزيل الضوضاء.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

حالة «الجراندماستر» PTP والتحكم فيه. يقوم مضيف «Chloros» بتشغيل «الجراندماستر» PTP؛ وتعمل كاميرات LATTICE ووحدات DAQ-E كأجهزة تابعة له من أجل الطوابع الزمنية عبر الأجهزة.

| الأمر الفرعي | الوصف |
| --- | --- |
| `status` | عرض حالة «الجراندماستر»، وأولويات BMCA، وهوية الساعة. |
| `peers` | سرد الأجهزة التابعة التي تم رصدها عبر Delay_Req (الكاميرات + مستشعرات DAQ-E). |
| `cameras` | حالة PTP لكل كاميرا (`PtpStatus`، `PtpOffsetFromMaster`، `PtpMeanPathDelay`). |
| `restart` | إعادة تشغيل عملية «الجراندماستر». |
| `set-priority --priority1 N --priority2 N` | تجاوز أولويات BMCA. |

### أمثلة

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

التحكم في كاميرا LATTICE. تمر كل أمر فرعي عبر الخلفية Chloros؛ تمتلك الخلفية مجموعة الكاميرات، لذا فإن استدعاءات CLI اللاحقة تعيد استخدام نفس المقبض المفتوح.

### الخيارات الشائعة (مشتركة بين معظم الأوامر الفرعية)

| العلامة | الوصف |
| --- | --- |
| `-d, --device N` | فهرس الكاميرا (الافتراضي: 0). |
| `-s, --serial SN` | رقم تسلسلي محدد؛ يتجاوز الإعداد `--device`. |
| `--serials SN1,SN2,…` | أرقام تسلسلية مفصولة بفواصل لعملية. |
| `--all` | التشغيل على كل كاميرا تم اكتشافها. |
| `--exposure US` | وقت التعريض بالمايكروثانية. |
| `--gain DB` | الكسب بالديسيبل. |
| `--pixel-format FMT` | على سبيل المثال: `BayerRG8`، `BayerRG12`. |
| `--width N` / `--height N` | أبعاد الصورة. |
| `--preset {default,high_quality,high_speed,triggered}` | تطبيق إعدادات محددة مسبقًا. جميع الإعدادات تعمل في وضع التشغيل الحر باستثناء `triggered`، الذي يجهز الكاميرا لاستقبال إشارة حافة من الأجهزة على الخط 2 — وفي حالة عدم وجود ما يحرك هذا الخط، ستظل الكاميرا في حالة انتظار إلى الأبد بدلاً من التقاط الصورة. |
| `-o, --output DIR` | دليل الإخراج (الافتراضي: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | حجم حزمة GVSP. يقوم `auto` بتشغيل اختبارات ICMP+GVSP؛ `jumbo` = 9000؛ `standard` = 1500. |

### سير عمل الاتصال الأول بكاميرا LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### مرجع الأوامر الفرعية

#### الاكتشاف والمعلومات

| الأمر الفرعي | الغرض |
| --- | --- |
| `lattice info` | سرد الكاميرات المتصلة (الشركة المصنعة، الطراز، الرقم التسلسلي، عنوان IP، MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | تحليل نظام المضيف لتحديد التكوين الأمثل للكاميرا. يتخطى `--no-discover` عملية اكتشاف الكاميرا (أسرع، تحليل يعتمد على بطاقة الشبكة فقط). |
| `lattice network [--fix] [--estimate] [--cameras N]` | التحقق من إعدادات بطاقة الشبكة (NIC) وتصحيحها؛ تقدير عرض النطاق الترددي/معدل الإطارات في الثانية (FPS). |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | قدرات الشبكة للخلفية ذات المخطط المستقر + توصية بالمصفوفة (تُرجع `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). يحافظ `auto_capped_fps` على الدقة المطلوبة ولكنه يحدد الحد الأقصى لمعدل الإطارات في الثانية المستهدف — اقرأ `recommended.recommended_target_fps` وقم بتمريره كهدف الاتصال؛ تعامل معه على أنه نجاح، وليس خطأ. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | تحليل «ماذا لو» دون فتح الكاميرات. **`--n-active` هو العدد الإجمالي للكاميرات المتصلة بالشبكة، وليس فقط تلك الموجودة في هذه المصفوفة**— قم برفعه عندما تقوم الكاميرات المستقلة بالبث في وقت واحد، أو عندما يتم حساب ميزانية الشبكة بناءً على طلب يقلل من عددها (الافتراضي: `len(--models)`). يُطبع دائمًا القيمة الإجمالية `Wire budget:` (المتطلبات بالميغابايت في الثانية مقابل الحد الأقصى الآمن للتصادم) وخطوط `Max cameras:`، ويضع علامة `** OVER-SUBSCRIBED**` عندما تتجاوز المصفوفة سعة الشبكة — انظر [نموذج الإطارات في الثانية (fps) والاندفاع (burst) للمصفوفة](#array-fps--burst-model). |
| `lattice gpu` | عرض حالة وحدة معالجة الرسومات (GPU). |
| `lattice firmware [--update] [--force] [-y\|--yes]` | فحص أو تحديث البرامج الثابتة للكاميرا. يتم تثبيت التحديد المحلي `.fwa`: يتم عرض الملف الموجود في `firmware/<MODEL_PREFIX>/` المطابق لـ `MIN_FIRMWARE_VERSION` الخاص بالإصدار يتم تحميله عند وجوده (الإصدار الأعلى فقط كخيار احتياطي)، لذا فإن صورة المورد الأحدث الموجودة على القرص تظل غير نشطة حتى يتم تحديث هذا التثبيت — تصل الإصدارات الأحدث عمدًا إلى الوحدات عبر قائمة AWS الموقعة، وهو الخيار المفضل عندما تكون الإصدارات أحدث. |
| `lattice presets [--apply NAME]` | عرض قائمة الإعدادات المسبقة للكاميرا أو تطبيقها. |
| `lattice status` | حالة الكاميرا المباشرة. |

#### الالتقاط

| الأمر الفرعي | الغرض |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | إطار واحد. **يحفظ كل أنواع التصدير افتراضيًا** (`--processing all`)؛ انظر [مستويات تصدير الالتقاط](#capture-export-levels-the-all-default). يحفظ `--levels` مجموعة فرعية محددة (يتجاوز `--processing`)؛ ويقوم `--force-daq` بكتابة قراءة DAQ المعينة كملف `.daq` حتى في حالة التقاط البيانات الخام فقط. `--jpeg-quality` = JPEG جودة 1–100 (الافتراضي 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | البث إلى القرص حتى الضغط على Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | معاينة MJPEG حية عبر المتصفح. `--ae-damping` يضبط تخميد التعريض التلقائي(0.4–100). |

#### ضبط المستشعر

| الأمر الفرعي | الغرض |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | قراءة/كتابة أي عقدة GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | التعريض الضوئي والتعرض التلقائي. |
| `lattice gain [--auto] [--off] [--set DB]` | الكسب والكسب التلقائي. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | منطقة الاهتمام (ROI) في المستشعر وتجميع البكسلات. |
| `lattice format [--set FMT] [--list]` | تنسيق البكسل. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | مشغل الأجهزة/البرامج. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (عدم وجود علامات = توازن اللون الأبيض لمرة واحدة) | عمليات توازن اللون الأبيض. كاميراتRGB/Bayer فقط؛ عملية غير فعالة (يتم تخطيها) في كاميرات M3M أحادية اللون. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | مسار ألوان العرض «RGB». `natural` (الافتراضي) هو المعالجة المباشرة الرخيصة؛ `enhanced` يضيف إزالة الحواف + الحيوية + التباين المحلي CLAHE للحصول على نظرة التوازن الكاملةبتكلفة تشبه ضعف تكلفة المعالجة لكل إطار، لذا فإن معدل الإطارات **المباشر** المنخفض — حيث تحصل اللقطات المحفوظة دائمًا على المعالجة الكاملة في كلتا الحالتين. RGB /كاميرات باير فقط؛ يتم تخطيها في كاميرات M3M أحادية اللون. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | عرض التشبع/التباين (كاميرات مرشح RGB). يتم تخطيها في كاميرات M3M أحادية اللون. |
| `lattice filter [--set NAME] [--list]` | تعيين طراز مرشح الكاميرا (`RGN-IMX265`، `OCN`، `NGB`، …). |
| `lattice power [--sleep]` | فحص نقاط الطاقة/الحرارة؛ من أجلالتلاعب بحالة السكون منخفضة الطاقة. |

#### المعايرة وأجهزة الاستشعار

| الأمر الفرعي | الغرض |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | المعايرة باستخدام هدف انعكاس. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | أوامر مستشعر الضوء الساقط المدمج. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | تطبيق تصحيح التظليل على الصور الموجودة. |

#### الكاميرات المتعددة (الجلسات المؤقتة)

| الأمر الفرعي | الغرض |
| --- | --- |
| `lattice multi-info` | سرد جميع الكاميرات ذات أدوار المزامنة. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | إطار واحد متزامن من كل كاميرا. يحفظ **جميع أنواع التصدير افتراضيًا**عند توصيل مصفوفة دائمة؛ يتم**إزالة التباين** فقط في حالة اللجوء المؤقت بدون مصفوفة (قم بتشغيل `array-connect` أولاً للباقي). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | بث الإطارات المتزامنة (مؤقت). |
| `lattice multi-test [--count N]` | اختبار توقيت مزامنة GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | الكشف التلقائي عن توصيلات GPIO الرئيسية/التابعة. |

#### المحاذاة

| الأمر الفرعي | الغرض |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — بالإضافة إلى أزرار الكاشف/المطابق `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`، وأزرار RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`، والإطارات `[--averaging mean\|median\|inlier_weighted]`، والقيود الهندسية `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`، والقيود المكانية `[--roi X0,Y0,X1,Y1] [--mask PATH]`، والتجاوزات لكل تابع `[--per-cam-override SN:KEY=VALUE]` (قابلة للتكرار) | حساب ملف تعريف المحاذاة من الكاميرات الحية. `--prefilter` يُعيَّن افتراضيًّا إلى `gradient` (خريطة الحواف؛ تتطابق مع أداة محاذاة واجهة المستخدم الرسومية/المصفوفة — تبقى الحواف عبر النطاقات الطيفية). يُظهر `--matcher flann` نتائج جيدة عند وجود ما يزيد عن ~5000 سمة؛ ويُعد `--averaging median` قويًا في مواجهة لقطة واحدة سيئة، بينما يقوم `inlier_weighted` بالترجيح حسب عدد التطابقات؛ `--lock-scale` يُسقط على أقرب دوران (بدون مقياس)، `--lock-axis` يُصفِّر مكون انتقال واحد؛ `--mask` ينطبق على كل كاميرا (استخدم `--per-cam-override` لإعدادات كل كاميرا على حدة، على سبيل المثال: `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` يرفض حفظ معايرة يتجاوز متوسط الجذر التربيعي (RMS) لإعادة الإسقاط فيها الحد المسموح به. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | التقاط إطار واحد متعدد النطاقات مُحاذي. يتم تعيين `--bit-depth` افتراضيًا لمطابقة الكاميرا؛ ويحتفظ `--no-crop` بالإطار الكامل (مع ملء الفراغات باللون الأسود)؛ و`--interpolation` (الافتراضي `linear`) و`--border-mode`/`--border-value` (الافتراضي `constant`/0) تتحكم في انحراف وحدة المعالجة المركزية (CPU) — ويظل مسار وحدة معالجة الرسومات (GPU) ثنائي الخطوط بغض النظر عن ذلك. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | إطارات متعددة النطاقات متوافقة مع الدفق (نفس أزرار التحكم في الانحراف مثل `align-apply`). |
| `lattice align-info --profile PATH [--json]` | عرض تفاصيل الملف الشخصي. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | تغيير ترتيب الطبقات. |

#### الفهرس / حسابات الغطاء النباتي

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

مجموعة العلامات الكاملة: `--input PATH | --live --profile PATH`، `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`، `--channel SYM=BAND` (قابل للتكرار)، `--capture-level raw|debayered|radiance|reflectance|unknown` (تجاوز مستوى الالتقاط المسجل في المصدر TIFF؛ الافتراضي: يُقرأ من بيانات تعريف TIFF)، `--output PATH`، `--output-format all|raw|tif|colorized|lut|png`، `--gradient NAME|JSON`، `--vmin/--vmax/--percentile LO,HI`، `--bg-mode clip|transparent|indexColor|backgroundColor`، `--colorize`، `--list-presets`، `--list-gradients`. مع `--live`، تنطبق أيضًا مقابض تحريف المحاذاة: `--save-multiband`، `--gpu/--no-gpu`، `--no-crop`، `--bit-depth 8|12|16`، `--vignette`، `--interpolation nearest|linear|cubic|lanczos`، `--border-mode constant|replicate|reflect|wrap`، `--border-value N`.

> **رموز `--channel` حساسة لحالة الأحرف.** يجب أن يتطابق جانب الرمز تمامًا مع أسماء قنوات الإعدادات المسبقة (تستخدم الإعدادات المسبقة الأحرف الصغيرة، على سبيل المثال: NDVI = `red`،`nir` — تحقق من `--list-presets`)، ويجب أن يتطابق جانب النطاق مع اسم نطاق في المجموعة المُحاذاة (أو أن يكون مؤشر نطاق يبدأ من 0 في الوضع غير المتصل بالإنترنت). يعمل `--channel red=Red_660 --channel nir=NIR_850`؛ بينما يفشل `--channel RED=660` مع ظهور خطأ `channel_map missing entries`.

#### الاتصالات المستمرة (Smart-Prep، تدفق مكافئ لواجهة المستخدم الرسومية)

تحافظ هذه الأوامر على الكاميرات مفتوحة في مجموعة الخلفية عبر عمليات استدعاء CLI.

| الأمر الفرعي | الغرض |
| --- | --- |
| `lattice cam-connect [--serial SN]` | إضافة كاميرا واحدة إلى المجموعة (كاميرا واحدة، بدون مصفوفة). |
| `lattice cam-disconnect [--serial SN] [--all]` | تحرير. |
| `lattice cam-list` | سرد الكاميرات الموجودة في المجموعة. |
| **`lattice array-connect`**|**توصيل مصفوفة متزامنة دائمة (نقطة الدخول الموصى بها).** يقوم بتشغيل تدفق الإعداد الذكي الكامل لواجهة المستخدم الرسومية. |
| `lattice array-disconnect [--array-id ID] [--all]` | تحرير مصفوفة. |
| `lattice array-list` | سرد المصفوفات المتصلة. |
| `lattice array-status [--array-id ID]` | معدل الإطارات في الثانية (fps) للبث المباشر، PTP، آخر خطأ. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | لقطة واحدة متزامنة من المصفوفة الحية — فردية / مستمرة / بفواصل زمنية / الأسرع. **القيمة الافتراضية هي `all`** (ملف واحد لكل نوع تصدير قابل للتطبيق لكل كاميرا). يتم الإبلاغ عن الكاميرات التي تم تخطيها (على سبيل المثال، RGB المستبعدة من الإشعاع/الانعكاس) يتم الإبلاغ عنها بواسطة `Skipped: SN:<serial> (<reason>)`؛ ويتم حفظ قراءة DAQ المستخدمة للانعكاس جنبًا إلى جنب مع `DAQ: <path>` والإبلاغ عنها بواسطته. انظر [أوضاع الالتقاط، المسجلات وإعادة المعالجة دون اتصال](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | تسجيل عرض المؤشر المدمج المباشر إلى فيديو/GIF (درجة المراقبة؛ يتطلب فتح الدفق المدمج). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | سلسلة لقطات باير الخام بمعدل إطارات عالي (مستوى التحليل؛ إعادة المعالجة دون اتصال بالإنترنت). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | إعادة معالجة سلسلة لقطات «راو» محفوظة لتحويلها إلى مقطع (مقاطع) فيديو معايرة. |

##### خيارات `array-connect`

| العلامة | القيمة الافتراضية | الوصف |
| --- | --- | --- |
| `--serials SN1,SN2,…` | الاكتشاف التلقائي لجميع كاميرات LATTICE (يتطلب وجود كاميرتين أو أكثر) | الرقم التسلسلي الأول هو الكاميرا الرئيسية (MASTER). عند حذفه، يتم تقييد عملية الاكتشاف لطرازات LATTICE (`TRI032*`) ويتم توصيلها جميعًا. |
| `--line {Line0,Line2,Line3}` | `Line2` | خط مزامنة GPIO. |
| `--target-fps F` | تلقائي | معدل إطلاق المشغل الرئيسي. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | تلقائي | تجاوز أداة اختيار المستوى. |
| `--wire-ceiling-mbps MB_PER_S` | تم الكشف التلقائي | **ميزانية السلك المستدامة للمضيف**ميزانية السلك المستدامة للمضيف، بالميغابايت في الثانية — الرقم الذي يعتمد عليه تخصيص المصفوفة بأكملها.** قم بتخفيضه عندما تبلغ المصفوفة عن إطارات تالفة بـ GVSP: يتم اشتقاق القيمة التلقائية من معدل الارتباط المعلن لبطاقة NIC، والذي يبالغ في تقدير محولات USB ومسارات PCIe الرفيعة والبنى المشتركة المزدحمة. يتم الاحتفاظ به في كتلة التقاط المصفوفة الخاصة بالمشروع ، لذا فإن إعادة فتح / CLI / SDK وإعادة الاتصال تعيدها. انظر [صحة المصفوفة](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | تلقائي | تجميع البيانات على مستوى الأجهزة. |
| `--no-recommend` | معطل | تخطي خطوة تحليل الشبكة. |
| `--no-ptp` | off | تعطيل PTP (وبالتالي **لا** يمكن مقارنة الطوابع الزمنية بين الكاميرات). |

### Smart-AE / Smart-Capture

تقوم مصفوفات LATTICE بتشغيل الضبط التلقائي للضوء (AE) بشكل مستمر في الخلفية بمجرد توصيلها، لكن المشهد الذي تم توجيهه حديثًا يستغرق لحظة حتى يتحقق التوافق. `array-capture --smart` هو **الحل المريح الجاهز**: فهو ينتظر حتى يستقر الضبط التلقائي للضوء (AE) عبر كل كاميرا في المصفوفة، ثم يقوم بتشغيل عملية الالتقاط. استخدمه عند تغيير المشاهد في منتصف الجلسة.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

سياسة الاستقرار متحفظة بشكل افتراضي: مهلة 5 ثوانٍ، نافذة استقرار 1.5 ثانية، تفاوت في التعريض المسموح به ±5٪. قم بضبطها عبر «SDK» (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) إذا كنت بحاجة إلى سلوك مختلف عن الأتمتة.

### مستويات تصدير اللقطات (الافتراضي هو `all`)

اعتبارًا من هذا الإصدار، `lattice capture` و `lattice multi-capture`، و`lattice array-capture` **يتم تعيينها افتراضيًا إلى `--processing all`** — ملف واحد محفوظ لكل نوع تصدير ينطبق على كل كاميرا، بما يتوافق مع سلوك «التقاط الكل» في واجهة المستخدم الرسومية. المستويات هي:

| المستوى | المخرجات | ينطبق على |
| --- | --- | --- |
| `raw` | باير أحادي القناة (الكاميرات أحادية اللون: النطاق الفردي) مباشرةً من المستشعر. | جميع الكاميرات. |
| `debayered` | إزالة التمويه BGR بثلاث قنوات (الكاميرات أحادية اللون: مقياس رمادي بقناة واحدة). | جميع الكاميرات. |
| `radiance` | float32 واط/م²/ستيرلاند/نانومتر عبر السلسلة الإشعاعية الكاملة. | متعدد الأطياف (M3C/M3M) فقط — **يتم تخطيه في الكاميرات المزودة بمرشح RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1.0)، جاهز للاستخدام مع Pix4D. | متعدد الأطياف فقط، و**فقط عند ربط وحدة DAQ + معايرة الكاميرا**؛ وإلا يتم تخطيه. |
| `preview` / `display` | سلسلة معاينة كاملة عبر واجهة المستخدم الرسومية (CCM + WB + جاما وفقًا لملف تعريف الكاميرا). `lattice capture` يسمي هذا `preview`؛ يستخدم كل من `array-capture` و`multi-capture` `display`. | جميع الكاميرات. |

قم بتمرير مستوى واحد لحفظ هذا المستوى فقط (`--processing debayered`). وعند طلب `all`، يتم تخطي المستويات التي لا تنطبق على كاميرا معينة (ويتم الإبلاغ عنها)، وليس كخطأ — فالكاميرا غير المتصلة أو غير المعايرة لا تزال تحصل على `raw` / `debayered` / `preview`.

بالنسبة لأي إطار انعكاس، تُكتب قراءة DAQ الساقطة التي تُستخدم فعليًّا تُسجَّل في ملف **`.daq`** مرفق بجوار الصورة (حتى يمكن إعادة معالجة الالتقاط لاحقًا) ويتم الإبلاغ عنها في سطر `DAQ:`.

### كيف يبدو مجلد اللقطات

يتم وضع كل نوع تصدير في **مجلد فرعي خاص به** تحت `-o`، وبالتالي لا يخلط التقاط متعدد المستويات بين الأنواع أبدًا:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` هو الطابع الزمني لالتقاط الصورة و`<serial>` هو الرقم التسلسلي للكاميرا، لذا فإن المجموعة المتزامنة الواحدة تشترك في
الطابع الزمني عبر الكاميرات. **لاحظ التباين الوحيد:** يتم تخزين المستوى `display` في مجلد
يسمى `preview/` بينما تحتفظ الملفات نفسها بـ `_display` في الاسم — يختلف المجلد واللاحقة
لتلك المستوى فقط. تعود المستويات غير المعروفة إلى مجلد يحمل اسمها، وإذا تعذر إنشاء المجلد الفرعي
، يتم كتابة الملف إلى جذر الإخراج بدلاً من ضياعه.

**إعادة معالجة مجلد التسجيلات:**قم بتوجيه `chloros-cli process` إلى**جذر الملتقطات**
(`output/`). عادةً ما يستورد `process` المجلد الذي تحدده فقط، ولكن عندما لا يحتوي ذلك المجلد على
صور ويحتوي على مجلدات فرعية، فإنه يتنقل تلقائيًا إلى المجلدات الفرعية — وبالتالي يتم التقاط المجلدات الفرعية من نفس مستوى المجلد الجذر و
المجلد الجذر `.daq` دفعة واحدة. يتم استيراد كل مستوى من مستويات الالتقاط كصورة واحدة مع
توفر المستويات الأخرى كأوضاع، بدلاً من صورة واحدة لكل مستوى.

كما أن تسمية **مجلد فرعي من المستوى** مباشرةً (على سبيل المثال `output/raw/`) يعمل أيضًا. يؤدي القيام بذلك إلى ترك المجلد الجذري
`.daq` خلفك، لذا انسخ أو أشر إلى قراءة DAQ جنبًا إلى جنب عند إعادة اشتقاق
من `raw/` — وإلا فلن يكون هناك ما يمكن مطابقة الطابع الزمني معه.

**تبدأ المعالجة دائمًا من `raw`.** ضمن كل عملية التقاط، يُعد الإطار الخام مصدر خط المعالجة؛
`debayered` و`radiance` و`reflectance` و`preview` تظهر كأوضاع قابلة للعرض ولكن لا يتم إرجاعها أبدًا
عبر خط الأنابيب. إعادة معالجة منتج مشتق ستؤدي إلى إعادة تطبيق التظليل (vignette) وCCM و
حسابات الإشعاع التي تم دمجها بالفعل في وحدات البكسل الخاصة به، لذا فإن «Chloros» يرفض ذلك بدلاً من
إجراء معالجة مزدوجة. هناك نتيجتان جديرتان بالذكر:

- لا تتم معالجة عمليات العرض `index/` و `composite/` **أبدًا**. فهي مخرجات وليست لقطات —
  لا يحتوي عرض LUT لـ «NDVI» على تفسير ذي مغزى للإشعاع.
- مجلد اللقطات الذي يتم تصديره **بدون** `raw` (على سبيل المثال `array-capture --processing reflectance`) لا
  يحتوي على مصدر شرعي في مسار المعالجة. يتم استيراد هذه اللقطات وعرضها بشكل طبيعي، لكن `process` يتخطاها
  ويشير إلى ذلك:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  إذا كنت بحاجة فعلاً إلى تمرير منتج مشتق — مثل جلسة مركزية تم التقاطها مع
  تشغيل `demosaic`، أو مجلد قديم — فإن `--input-level {raw,debayered,processed}` يفرض الإدخال
  وتتجاوز التخطي. هذا العلامة هي مخرج الطوارئ المتعمد؛ `auto` (الافتراضي)
  لا يعالج أبدًا التقاطًا لا يحتوي على بيانات خام.

### عمليات التقاط تم تخطيها في مصفوفات المرشحات المختلطة

عند مزج كاميرات «RGB» وكاميرات متعددة الأطياف في مصفوفة واحدة، يقوم `array-capture --processing radiance` (أو `reflectance`) بحفظ الإطارات متعددة الأطياف و**تخطي** كاميرات «RGB» — حيث إن الإشعاع لكل باير لا معنى له بالنسبة لمستشعر النطاق العريض. يقوم CLI يُظهر كل ملف تم حفظه (مع مستوى التصدير الخاص به)، وكل مرة يتم فيها كتابة `.daq`، وكل عملية تخطي بشكل صريح، لذا فإن عدد الملفاتمفاجئًا:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

تتبع رموز أسباب التخطي النمط `<level>-not-applicable-to-rgb-cam`. يمكن أيضًا تخطي الانعكاسية باستخدام `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`، وبـ `dls-uncalibrated-band-<nm>` عندما يقع النطاق في الغالب خارج النطاق المعاير إشعاعيًا لمستشعر الضوء DAQ (~374–974 نانومتر) — ومن بين وحدات التخزين المتوفرة في السوق (SKU) فقط F988، الذي يدعم مسار سير العمل الخاص بلوحة الانعكاس.

استخدم `--processing debayered` (أو `display`) لتضمين كل كاميرا بغض النظر عن نوع المرشح، أو الإعداد الافتراضي `all` للحصول على كل مستوى قابل للتطبيق لكل كاميرا في لقطة واحدة.

---

## أوضاع الالتقاط والمسجلات وإعادة المعالجة دون اتصال بالإنترنت

تعمل جميع هذه الأوضاع على **مصفوفة ثابتة** (قم بتشغيل `array-connect` أولاً). وهي تعكس لوحة الالتقاط في واجهة المستخدم الرسومية.

### أوضاع `array-capture`

`array-capture` هو أمر واحد يتضمن أربعة أوضاع للغالق بالإضافة إلى مجموعة من خيارات التبديل للتصدير:

| الوضع | العلامة | السلوك |
| --- | --- | --- |
| **مفرد** *(افتراضي)* | (لا شيء) | مجموعة التقاط واحدة متزامنة، ثم الخروج. |
| **مستمر** | `--continuous` | مرور متتالي حتى `Ctrl+C` أو `--count N` أو `--duration S`. |
| **فاصل زمني** | `--interval S` | دورة واحدة كل `S` ثانية (تُقاس من بداية كل دورة)، بنفس الحدود. |
| **الأسرع** | `--fastest` | البيانات الأولية فقط + قراءة DAQ المخصصة + المركب ذو المؤشر المدمج؛ يتخطى حساب الانعكاس/الانعكاس/العرض حتى يتم عرض الإطار بسرعة. ينطوي على `--processing raw --force-daq`. أعد معالجة `.daq` المحفوظ إلى منتجات معايرة لاحقًا. |

أزرار التبديل للتصدير (يمكن دمجها مع أي وضع؛ جميعها تشترك في واجهة المستخدم الرسومية/نقطة النهاية SDK):

| العلامة | التأثير |
| --- | --- |
| `--processing LEVEL` | مستوى تصدير واحد، أو `all` (الافتراضي). |
| `--levels L1,L2,…` | مجموعة فرعية صريحة من أنواع التصدير (على سبيل المثال `raw,radiance,reflectance`)؛ **يتجاوز `--processing`**. |
| `--aligned` / `--no-aligned` | تحويل التصدير غير الخام لكل عنصر إلى [ملف تعريف محاذاة](#) (مسجلة بشكل مشترك). تظل البيانات الخام غير محولة ولكنها تحمل التحويل في البيانات الوصفية. يتم الرجوع إلى حالة عدم المحاذاة (مع تحذير) إذا لم يكن للمصفوفة ملف تعريف. |
| `--index` / `--no-index` | حفظ / تخطي تراكب مؤشر الغطاء النباتي لكل كاميرا حيثما تم تكوينه. الافتراضي: عرضه. |
| `--force-daq` | حفظ قراءة DAQ/DLS كملف جانبي `.daq` حتى في حالة عدم احتياج أي مستوى مختار لها (مثل التقاط البيانات الخام فقط)، بحيث يمكن إعادة معالجة الإطارات إلى انعكاسية/مؤشر خارج الخط. |
| `--smart` | الانتظار حتى يستقر التعريض التلقائي (AE) عبر جميع الكاميرات قبل بدء التقاط الصورة (انظر [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | ضغط البكسلاتTIFF. `deflate` (الافتراضي) = zlib L1 بدون فقدان + متنبئ أفقي، ~4.1 ميغابايت لكل إطار بدقة كاملة؛ `none` = غير مضغوط، أسرع بحوالي 5× في الكتابة بمعدل ~6.3 ميغابايت لكل إطار — يُستخدم لتحقيق أقصى معدل مستدام عندما يسمح القرص بذلك. كلاهما بدون فقدان البيانات ويتم قراءتهما بشكل متطابق عند الاستيراد. |

> **كتابة أحاديةTIFF + نموذج المعدل المستمر.**تُكتب اللقطات في**مرور واحد**لملف tifffile يحمل البكسلات + XMP + IFD0 الصانع/الطراز (تم القياس علىالدقة الكاملة Mono12: 36 مللي ثانية مضغوطًا / 6.5 مللي ثانية غير مضغوط، مقابل ~148 مللي ثانية للطريقة القديمة المتمثلة في الكتابة ثم إعادة الكتابة باستخدام ExifTool)؛ أما العمل الوحيد المتبقي لـ ExifTool (تنقيح IFD الفرعي لـ EXIF) فيتم تنفيذه على عامل خلفية غير متزامن، ويكون الإطار مكتملًا وجاهزًا للاستيراد حتى حتى لو لم يتم تشغيله أبدًا. لاحظ أن ضغط DEFLATE يحتفظ بـ GIL الخاص بـ Python، لذا فإن عمليات الكتابة المضغوطة**لا**تتم بالتوازي عبر خيوط الكتابة الخاصة بكل كاميرا — يتطلب التقاط مستمر بدقة كاملة لـ 8 كاميرات بمعدل المستشعر (~10.4 إطارًا في الثانية) وجود `--compression none`**و** قرص من فئة NVMe (~500 ميغابايت/ثانية من الكتابة المستمرة). يتم عرض نفس الخيار باسم `compression` في `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — فيديو/GIF بفهرس مدمج (مستوى المراقبة)

يسجل كل ما **يعرضه العرض المباشر بفهرس مدمج** إلى `.avi` (واختياريًا إلى `.gif`). ونظرًا لأنه يستمد البيانات من المركب المباشر، يجب أن يكون الدفق المدمج مفتوحًا (على سبيل المثال، يتم معاينة المصفوفة في واجهة المستخدم الرسومية) حتى يتم تسجيل الإطارات. يقوم باستقصاء التقدم كل ثانيتين ويتوقف عند `--duration` أو `Ctrl+C` أو عندما ينهي المسجل العمل تلقائيًا.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| العلامة | الافتراضي | الوصف |
| --- | --- | --- |
| `--array-id ID` | المصفوفة فقط | المصفوفة المستهدفة (يُحذف إذا كانت هناك مصفوفة واحدة فقط متصلة). |
| `-o, --output DIR` | `output` | دليل الإخراج (الخلفية-محلية). |
| `--fps F` | `10` | معدل إطارات التسجيل. |
| `--duration S` | حتى Ctrl+C | التوقف التلقائي بعد `S` ثانية. |
| `--gif` | معطل | كتابة ملف GIF متحرك أيضًا. |
| `--gif-only` | معطل | كتابة ملف GIF فقط (بدون `.avi`). |

### `array-burst` — سلسلة لقطات عالية الإطارات في الثانية بتنسيق باير الخام (مستوى التحليل)

يقرأ مخزن المجموعة المتزامنة لحلقة الالتقاط مباشرةً — **بدون سلسلة معايرة، وبدون exiftool، ولا حاجة إلى العرض المباشر** — لذا تعمل بمعدل التقاط الكاميرا الكامل. تكتب إطارات RAW + بيان لكل إطار + ملف `.daq` واحد لكل قراءة DLS مميزة تحت `<output>/bursts/<base>/`. أعد المعالجة في وضع عدم الاتصال (الأمر التالي)، أو مرر `--build` لتنفيذها فور التوقف.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| العلامة | الافتراضي | الوصف |
| --- | --- | --- |
| `--array-id ID` | المصفوفة فقط | المصفوفة المستهدفة. |
| `-o, --output DIR` | `output` | دليل الإخراج (يتم تخزين الدفعات في `<DIR>/bursts/<base>/`). |
| `--duration S` | حتى Ctrl+C | التوقف التلقائي بعد `S` ثانية. |
| `--max-frames N` | غير محدود | التوقف التلقائي بعد إطارات `N` من الإطارات الأولية. |
| `--build` | معطل | بعد التوقف، إعادة معالجة السلسلة على الفور (مثل `array-build-video`). |
| `--products …` | `combined:index` | مع `--build`: تحديد مقاطع الفيديو المراد إنشاؤها (انظر أدناه). |
| `--fps F` | `10` | مع `--build`: معدل الإطارات في الثانية (fps) للفيديو الناتج. |
| `--save-tiffs` | off | مع `--build`: يتم أيضًا حفظ ملفات TIFF المعايرة لكل إطار. |
| `--gif` | إيقاف | مع `--build`: يتم أيضًا كتابة ملفات GIF متحركة. |

### `array-build-video` — إعادة معالجة سلسلة لقطات محفوظة في وضع غير متصل

مطابقة توقيت كل إطار خام مع أقرب قراءة محفوظة لـ `.daq` وتمريره عبر **نفس سلسلة الإشعاع / الانعكاس / المؤشر مثل مسار الاستيراد**، مما يؤدي إلى إنتاج مقطع فيديو واحد أو أكثر.

`--products` عبارة عن قائمة مفصولة بفواصل من عناصر `kind:level`، حيث `kind` ∈ `per_cam` | `combined` و `level` ∈ `radiance` | `reflectance` | `index`. ويُعيَّن `level` (بدون `kind:`) يُعيَّن افتراضيًّا إلى `per_cam`. القيمة الافتراضية هي `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| العلامة | القيمة الافتراضية | الوصف |
| --- | --- | --- |
| `--burst-dir DIR` | (مطلوب) | مسار مجلد التسلسلات (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | قائمة `kind:level`، كما هو مذكور أعلاه. |
| `--fps F` | `10` | معدل الإطارات في الثانية (fps) للفيديو الناتج. |
| `--save-tiffs` | off | حفظ ملفات TIFF المعايرة لكل إطار جنبًا إلى جنب مع مقاطع الفيديو. |
| `--gif` | off | كتابة ملفات GIF متحركة أيضًا. |

> **اختر المسجل المناسب.** `array-record` هو *مستوى المراقبة* — فهو يلتقط الصورة المركبة الحية كما تظهر على الشاشة ويحتاج إلى أن يكون البث مفتوحًا. `array-burst` → `array-build-video` هو *من فئة التحليل* — حيث يحفظ البيانات الأولية للمستشعر بالمعدل الكامل ويعيد بناء مقاطع فيديو الإشعاع/الانعكاس/المعامل المعايرة لاحقًا، دون الحاجة إلى عرض مباشر.

### كاميرات أحادية النطاق (M3M)

تعد سلسلة **M3M**النظير الأحادي لـ Bayer**M3C**: مرشح تداخل واحد ضيق النطاق لكل كاميرا (`M3M-<lens>-F<wavelength>`، على سبيل المثال `M3M-L87-F685`)، وبالتالي يقدم المستشعر**نطاقًا واحدًا بتدرج الرمادي** بدون فسيفساء باير. لا يوجد ما يحتاج إلى إزالة الفسيفساء، ولا تداخل بين القنوات يحتاج إلى فصل، ولا توازن أبيض يحتاج إلى ضبط — فمسار معالجة الألوان الكامل من RGB -display ببساطة لا ينطبق هنا.

ما يعنيه ذلك في كاميرا CLI:

- **`lattice white-balance`، `lattice color-profile`، `lattice color`**تكتشف الكاميرا أحادية اللون وتقوم**يتخطونها برسالة من سطر واحد** بدلاً من فرض إعدادات لا معنى لها. ولا تزال هذه الأوامر تعمل بشكل طبيعي مع كاميرا RGB /Bayer M3C في نفس الجلسة.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** لا تزال تعمل — فالإشعاع والانعكاس هما خريطتان إشعاعيتان *لكل نطاق* ومحددتان بشكل مثالي لنطاق واحد. تحمل الإطارات أحادية اللون مصفوفة استجابة المستشعر **المتطابقة** (بدون فصل 3×3)، لذا يمر المستوى عبر حسابات المعايرة دون تغيير.
- **لا يمكن لكاميرا أحادية واحدة إنتاج مؤشر نباتي.**NDVI / NDRE / إلخ. تحتاج إلى نطاقين على الأقل (على سبيل المثال Red + NIR). للحصول على مؤشر من أجهزة أحادية، قم بتوجيه**عدة** كاميرات M3M على أطوال موجية مختلفة، وقم بمحاذاة الصور في مكدس متعدد النطاقات، ثم احسب المؤشر *من ذلك*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` يجب أن تتطابق الرموز **بالضبط** مع أسماء القنوات المحددة مسبقًا (مع مراعاة)؛ الرموز في NDVI مكتوبة بأحرف صغيرة `red`,`nir` — انظر `--list-presets`)، ويشير اسم جانب النطاق إلى نطاق في المجموعة المُحاذاة (يقبل الوضع غير المتصل أيضًا ، على سبيل المثال `--channel red=0 --channel nir=1`).

المعرف المميز عبر المكدس هو الرمز `M3M` في سلسلة النموذج (لا يظهر أبدًا في سلسلة `M3C`)، ويظهر في واجهة المستخدم الرسومية/SDK على أنه `is_mono`.

---

## إعداد بطاقة شبكة المضيف (NIC) وضبطها (مصفوفات LATTICE)

تقوم كاميرات LATTICE ببث GVSP عبر محول إيثرنت المضيف، لذا في المصفوفات متعددة الكاميرات، يكون **برنامج تشغيل**المحول و**حجم حلقة الاستقبال** مهمين بقدر أهمية معدل الارتباط. تظهر الإعدادات الخاطئة على شكل `FRAMES WILL DROP` / بوابة `Reduce ROI to enable` في لوحة إعدادات المصفوفة (وفي `lattice network-analysis` / `analyze_array_network()` الخاص بـSDK)، حتى عندما تكون الكاميرات نفسها سليمة.

### محولات USB 10GbE — Realtek RTL8157 (&quot;Realtek USB 10GbE Family Controller&quot;)

| العنصر | القيمة المطلوبة | سبب أهميتها |
| --- | --- | --- |
| **إصدار برنامج التشغيل**|**≥ v10.67 (يناير 2026)**، INF `rtump64x64sta.inf` | برنامج التشغيل القديم**2016**(الإصدار 10.65، `rtump64x64.inf`) يعالج بشكل خاطئ إيقاف التشغيل وفحوصات الأخطاء مع**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**عند إيقاف التشغيل/إعادة التشغيل/وضع السكون. يتوقف الانتقال (تجاوز مهلة ~5 دقائق)، ويضطر المستخدم إلى إيقاف التشغيل قسريًّا، وتؤدي عمليات إيقاف التشغيل غير السليمة المتكررة**إلى إتلاف مستودع WMI**(تبدأ PowerShell/الأدوات في التعطل مع `Invalid class`) وتؤدي**إلى تعطل مكدس USB** عند التشغيل التالي (لا يتم تمكين بطاقة الشبكة؛ وتتوقف محركات أقراص USB عن التعداد). قم بالتحديث من realtek.com (أو مورد جهاز الدونجل) قبل الاعتماد على عمليات إعادة التشغيل النظيفة. |
| **مخازن الاستقبال**— الكلمة الرئيسية `ReceiveBufferLen` |**256**(الحد الأقصى للبرنامج التشغيلي) | حلقة الاستقبال (RX) لبطاقة الشبكة. الإعداد الافتراضي للبرنامج التشغيلي البالغ**32**لا يترك سوى ~0.26 ميغابايت من الحلقات القابلة للاستخدام — وهو حجم صغير جدًّا بالنسبة لعمليات الإرسال المتتالية من كاميرات متعددة — لذا تُبلغ لوحة المصفوفة عن الخطأ `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` وتمنع الاتصال. عند**256**، تكون الحلقة كبيرة (**حوالي 13.5 ميغابايت مقاسة على مضيف 10 جيجابت في الثانية في المختبر**)، مما يمنح خط أنابيب الاستقبال (RX) مساحة حقيقية لسلسلة عمليات GVSP متعددة الكاميرات. (يتم تحديد ما إذا كان تكوين معين *يقوم بالاتصال* فعليًا يتم تحديده من خلال فحصين — فحص القبول **المراعي للتصريف**وفحص**الإفراط التجميعي في الاشتراك** — وليس من خلال مقارنة أولية بين التصوير المتتالي والحلقة؛ انظر [نموذج معدل الإطارات في الثانية والتصوير المتتالي للمصفوفة](#array-fps--burst-model).) |
| **URBs الاستقبال**— الكلمة الرئيسية `PendingReceives` |**64** (الحد الأقصى) | كتل طلبات USB قيد النقل؛ يتم زيادتها جنبًا إلى جنب مع مخازن الاستقبال المؤقتة لاستيعاب الاندفاعات. |
| **الإطار الضخم** — الكلمة الرئيسية `*JumboPacket` | **9014** | مطلوب لحزم GVSP بحجم 9000 بايت (عدد الحزم لكل إطار أقل بـ 6 أضعاف مقارنة بـ 1500). |

> ⚠️ **يؤدي تحديث برنامج تشغيل بطاقة الشبكة (NIC) إلى إعادة تعيين هذه الخصائص المتقدمة إلى القيم الافتراضية.**بعد تحديث أو استبدال برنامج تشغيل المهايئ،**أعد تطبيق** `ReceiveBufferLen=256` و `PendingReceives=64`، وإلا ستتوقف لوحة المصفوفة عن العمل مرة أخرى على الرغم من أنه «لم يتغير شيء في الأجهزة». هذا هو السبب الأول لرفض جهاز كان يعمل سابقًا الاتصال فجأة.

قم بالتطبيق من PowerShell **بصلاحيات عالية** (استبدل اسم المحول الخاص بك، على سبيل المثال `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **يغطي `lattice network --fix` محولات USB 10GbE.** يقوم الآن باكتشاف نوع المحول وضبط الكلمة الرئيسية الصحيحة لحلقة الاستقبال: `*ReceiveBuffers`→2048 لبطاقات الشبكة PCIe (Intel I219، إلخ)، أو `ReceiveBufferLen`→256 + `PendingReceives`→64 لوحدة التحكم Realtek **USB** 10GbE (التي لا تكشف عن `*ReceiveBuffers`). يتم تقييد القيم المستهدفة بالحد الأقصى المبلغ عنه لكل برنامج تشغيل (`NumericParameterMaxValue`)، بحيث لا يتم أبدًا كتابة قيمة خارج النطاق. قم بتشغيله من محطة طرفية **ذات صلاحيات عالية**؛ مثل أي تعديل يعتمد على السجل، لا يسري التغيير إلا بعد إعادة تشغيل المهايئ أو إعادة تشغيل النظام. تظل الأوامر اليدوية `Set-NetAdapterAdvancedProperty` المذكورة أعلاه بديلاً جيدًا — فهي تُطبق فوريًّا (إعادة ربط المحول) دون الحاجة إلى إعادة التشغيل.

### أساسيات الشبكة (جميع وصلات LATTICE)

- **تحديد العناوين:** محلية للوصلة `169.254.0.0/16` (GigE Vision LLA). يأخذ المضيف عنوانًا ثابتًا `169.254.x.x/16`؛ بينما تقوم الكاميرات وDAQ-E بتعيين عناوينها تلقائيًا ضمن نفس النطاق. لا حاجة إلى DHCP أو بوابة.
- **حجم الحزمة:**يُفضل استخدام الحجم الضخم (9000)، ولكن دع عملية الاستكشاف التلقائي تحدد الحجم المناسب — حيث يعيد القياس عند كل اتصال ويتجاوز بالفعل الحد الأقصى لـ ICMP الخاص بالكاميرا البالغ 1500 بايت عبر اختبار GVSP، لذا يستقر على الحزمة الجامبو أينما كان السلك قادرًا على نقلها فعليًّا. لا تحدد القيمة الثابتة باستخدام `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` إلا إذا كنت تعرف أفضل من الاختبار، و افضل الإعداد لكل أمر على الإعداد الدائم: حيث إن التثبيت يتخطى الاختبار، لذا إذا كان المسار لا يستطيع فعليًا نقل 9000**فإن كل** عملية التقاط ستنتهي بفشل مع `SC_ERR_TIMEOUT -1011` (انظر [متغيرات البيئة](#environment-variables)).
- **يتناسب حجم حلقة الاستقبال (RX ring) مع `ReceiveBufferLen`:**عند القيمة الافتراضية لـ `32`، تبلغ سعة الحلقة القابلة للاستخدام حوالي 0.26 ميغابايت (صغيرة جدًّا لأي تسلسل متعدد الكاميرات)؛ وعند الحد الأقصى `256` تكون كبيرة (~13.5 ميغابايت مقاسة على مضيف 10GbE في المختبر)، مما يوفر سعة احتياطية حقيقية. ثم يُتقرر ما إذا كان التكوين سيتصل أم لا من خلال فحص القبول المراعي للاستنزاف**و** فحص الإفراط في الاشتراك التراكمي أدناه — وليس مقارنة أولية بين السرعة المتفجرة وسعة الحلقة.

### معدل الإطارات في الثانية (fps) للمصفوفة ونموذج التسلسل

كيفية قراءة لوحة إعدادات المصفوفة (و`lattice analyze-array` / `analyze_array_network` في جهاز SDK):

- **يتم جمع معدل الاندفاع لكل كاميرا على حدة بتنسيق البكسل الفعلي لكل كاميرا.**الكاميرات الأحادية**M3M**تبث**Mono12 (2 بت/بكسل)**؛**M3C**تنقل الكاميرات ذات نظام باير 8 أو 12 بت (تُصدر TRI032S بصمت BayerRG12 حتى عند طلب BayerRG8). لذا فإن الإطار بدقة كاملة من 4 كاميرات يبلغ حجمه**حوالي 12.6 ميغابايت إذا كانت جميعها 8 بت، ولكنه يصل إلى حوالي 25 ميغابايت مع ثلاث كاميرات أحادية 12 بت****. يقوم نظام العرض بتحديد تنسيق كل كاميرا بناءً على طرازها (ذاكرة التخزين المؤقتة للهوية)، لذا فإن الدفعة تتطابق مع ما ينقله السلك فعليًا — وليس افتراضًا موحدًا بـ BayerRG8.
- **يبلغ الحد الأقصى لمعدل نقل محول إيثرنت USB 200 ميغابايت/ثانية بغض النظر عن المواصفات المذكورة على اللوحة.** جدول الكفاءة الذي يحول معدل الوصلة إلى رقم مستدام مشتق من PCIe؛ حيث تعلن بطاقة الشبكة USB عن معدل ارتباط *إيثرنت* الخاص بها، لكنها مقيدة بنظام ناقل USB وبرنامج التشغيل الخاص به. وقد سجل دونجل USB 10 جيجابت في الثانية معدلًا «مستدامًا» يبلغ حوالي 1063 ميغابايت/ثانية — وهو رقم لم يتم التحقق منه قط — وأدى التباطؤ الناتج إلى تلف 6–18 % من الإطارات مع استمرار الإبلاغ عن معدل إطارات في الثانية (fps) مستهدف سليم. أصبحت بطاقات الشبكة (NIC) المتصلة عبر USB محدودة الآن بـ **200 ميجابايت/ثانية** كحد (الحد هو الناقل نفسه، لذا لا يتناسب مع السعة المعلنة؛ فمحول USB 1 جيجابت في الثانية يبلغ ~80 ميجابايت في الثانية ولا يتأثر بذلك). ويشير `wire_ceiling_source` في سجل القدرات إلى ذلك صراحةً، بينما يشير `nic_is_usb` يشير إلى ذلك. يمكن تجاوز هذا الحد في كلتا الحالتين باستخدام `--wire-ceiling-mbps`.
- **القبول يراعي معدل التفريغ، وليس المقارنة بين الدفعة الكاملة والحلقة.** يجب أن تتناسب الدفعة المتزامنة فقط مع *التراكم المؤقت* = `max(0, Σ per-cam arrival − host drain) × emit_window`، وليس الدفعة بأكملها. في بنية مضيف سريع / كاميرا بطيئة (**PCIe**مضيف 10G + 4 كاميرات 1 GbE: معدل الوصول ≈ 320 ميغابايت/ثانية، معدل التفريغ ≈ 1063 ميغابايت/ثانية)، فإن المضيف يفرغ البيانات أسرع من ملء الكاميرات، والتراكم ≈ 0، لذا فإن الإرسال المتزامن بدقة كاملة**يسمح**على الرغم من أن الدفعة البالغة 25 ميغابايت تتجاوز سعة الحلقة البالغة 13.5 ميغابايت. إذا تم توصيل الكاميرات الأربع نفسها بمحول**USB**10 جيجابت في الثانية، فإن معدل التفريغ يكون 200 ميغابايت/ثانية، وليس 1063 — حيث يتفوق معدل الوصول على معدل التفريغ، وتظهر الخسارة على شكل إطارات تالفة بدلاً من معدل إطارات أقل. على مضيف 1 جيجابت في الثانية، يؤدي الحد الأدنى لـ DLThr للكاميرات البالغ 31.25 ميجابايت/ثانية إلى تفوق معدل الوصول على معدل التفريغ → فيقوم**بحظر** بشكل صحيح (بالنسبة *لهذه* الفئة من الحجب، قم بتقليل منطقة الاهتمام (ROI) أو استخدم تجميع البكسلات (binning) ≥ 2). يُعد «القبول» أحد **بوابتي** الاتصال — والأخرى هي فحص الإفراط في الاشتراك التجميعي أدناه.
- **يُعد معدل الإطارات في الثانية (fps) المتوقع سقفًا متحفظًا للاسترجاع التسلسلي.**تقوم حلقة الاستلام الخاصة بالمضيف حاليًا بسحب مخزن كل كاميرا**تسلسليًا**(نافذة إرسال واحدة تقريبًا لكل كاميرا)، لذا فإن الدورة محددة بـ `max(readout+emit, N × emit)` مع تثبيت الإرسال لكل كاميرا على**رابط الوصول**الخاص بالكاميرا (1 جيجابت إيثرنت ≈ 80 ميجابايت/s)، وليس ارتباط المضيف الصاعد. بالنسبة لمصفوفة مكونة من 4 كاميرات بدقة كاملة و12 بت، يبلغ معدل**~2.8 إطارًا في الثانية**، وهو ما يتطابق مع المعدل المقاس الذي يتراوح بين ~2.7 و3.0 إطارًا في الثانية، وهو**مستقل عن التعريض**، لذا في المشاهد المعتمة قد تنخفض القيمة الفعلية قليلاً عن الحد الأقصى مع إطالة مدة التعريض. الاسترجاع التسلسلي هو المحدد الحقيقي لعدد الإطارات في الثانية؛ ومن شأن جعل العملية متوازية أن يرفع الحد الأقصى ليقترب من معدل الإرسال الفردي.
- **الإفراط التراكمي في الاشتراك هو عائق قوي للاتصال.**يبلغ الحد الأدنى لتخصيص النطاق الترددي لكل كاميرا**8 ميغابايت/**s**(`ARRAY_PER_CAM_FLOOR_BPS`)، لذا بمجرد بلوغ الحد الأدنى، يمكن أن يتجاوز الطلب الإجمالي (`per_cam × N`)**الحد الأقصى الآمن من التصادم**(`sustained × sim_emit_factor`). الحدود القصوى العملية للدقة الكاملة على شبكة 1 جيجابت إيثرنت:**6 كاميرات بـ 1500 MTU، و9 كاميرات باستخدام إطارات جامبو**. هذا الحد الأقصى هو خاصية تقتصر على السلك والحد الأدنى فقط — فهو**مستقل عن حجم الإطار**، لذا**فالتجميع (binning) ومعدل العائد على الاستثمار (ROI) الأصغر لا يساعدان** (فهما يقللان البايتات لكل *إطار*، وليس البايتات التي تتحكم فيها GevSCPD لكل *ثانية*)؛ الحلول الوحيدة هي عدد أقل من الكاميرات، أو إرسال الإطارات الجامبو من طرفإلى النهاية، أو بطاقة شبكة أسرع. وتتمثل الأعراض في فقدان حزم GVSP، وليس انخفاضًا تدريجيًا في معدل الإطارات في الثانية (fps)، لذا فإن `analyze-array` يُفرغ أرقام معدل الإطارات في الثانية القابلة للتحقيق ويطبع `**OVER-SUBSCRIBED**`، و`array-connect` مع دقة ثابتة **يرفض الاتصال** (وإلا فإن عملية «الانخفاض التدريجي» تقوم بتجميع الإطارات في مجموعات أصغر، وهو ما لا يزيل هذه الفئة من الحجب أيضًا). يقوم الرمز `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` بتخفيف مستوى الرفض إلى تحذير صاخب لأعمال الاختبار — انظر [متغيرات البيئة](#environment-variables).

### سلامة المصفوفة — أي نظام فرعي يفقد الإطارات

يحمل `GET /api/camera/array/<array_id>/capability` الخاص بمصفوفة متصلة كتلة حية
`health`، يُعاد تقييمه في نافذة متجددة **مدة 10 ثوانٍ**. وهو يقسم فقدان الإطارات
إلى سببين يتطلبان إصلاحات متعاكسة، بدلاً من الإبلاغ عن معدل «غير مكتمل»
واحد لا يحدد أيًّا منهما:

| الحقل | المعنى | النظام الفرعي |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (لكل رقم تسلسلي) | **وصل الإطار وكان تالفًا من الناحية الهيكلية**— فقدان حزم GVSP. |**الشبكة**: سعة السلك، وتيرة الإرسال، حلقة استقبال بطاقة الشبكة (NIC)، وحدة النقل القصوى (MTU) |
| `never_arrived_rate_pct` (لكل مسلسل) | **لم يصل الإطار على الإطلاق**— لم يتم تشغيل الكاميرا، أو لم يخرج منها أي شيء. |**المشغل / المزامنة**: كابل M8، `--line`، `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | أسوأ معدل لكل كاميرا. | — |
| `per_cam_rate_pct` | المعدل الإجمالي غير المكتمل لكل كاميرا (كلا السببين معًا). | — |
| `stable_for_seconds` | المدة التي بقيت فيها كل كاميرا دون 0.01٪. | — |

عند تجاوز نسبة 5٪، يسجل الخادم الخلفي سطرًا `[array-health <id>] WARN` يحدد الانقسام — عند
الخرق الأول، وعند تغيير نطاق الخطورة، ومرة كل دقيقة طالما استمر الخرق، ومرة واحدة عند
انتهائه. يُطبع النصف التالف `[gvsp-corrupt <SN>]` عند أول ظهور لكل كاميرا و
السبب، ثم تقرير إجمالي كل 60 ثانية. لا يزال كل تقييم يُسجل في ملف سجل الخلفية؛
وتتحرك العدادات مع كل مخزن مؤقت بغض النظر عما يتم طباعته.

يسجل السجل نفسه الرقم الذي يتوقف عنده التخصيص بأكمله:

| الحقل | المعنى |
| --- | --- |
| `wire_ceiling_mbps` | ميزانية النطاق الترددي المستدامة السارية للمضيف، ميغابايت في الثانية. |
| `wire_ceiling_source` | مصدر هذا الرقم، بالكلمات — على سبيل المثال `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` أو `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | يتم تعيينه بواسطة `true` عندما يتم تعيينه بواسطة `--wire-ceiling-mbps` (أو حقل **Wire Budget** في واجهة المستخدم الرسومية) يضبطه. |
| `nic_is_usb` | `true` لمحول إيثرنت USB — انظر الحد الأقصى البالغ 200 ميجابايت/ثانية أعلاه. |

**قراءته:** قيمة `gvsp_corrupt_rate_pct` غير صفرية مع `never_arrived_rate_pct` عند 0
تعني أن التشغيل ومزامنة الكابل مثاليان وأن 100% من الخسارة تقع على مسار الشبكة
— قم بخفض `--wire-ceiling-mbps` وأعد التوصيل. أما النمط العكسي فيشير إلى
كابل المزامنة أو خط التشغيل بدلاً من ذلك.

> **`--target-fps` ليس العامل المسبب للإطارات التالفة.** يتم كتابة معدل GevSCPD
> مرة واحدة عند الاتصال، لذا فإن خفض معدل التشغيل يغير دورة العمل وليس
> معدل اندفاع الإرسال المتزامن. لم يسفر خفض الطلب المقاس بمقدار 5× عن أي تحسن؛
> أدى خفض السقف الأقصى للسرعة من 240 إلى 200 ميغابايت/ثانية إلى خفض نسبة التلف في نفس الجهاز من 10.4 %
> إلى 0.00 %.

> **التقلص التلقائي في منتصف التدفق غير متاح في البرنامج الثابت TRI032S.** لا يمكن للمصفوفة قيد التشغيل
> إصلاح هذا الأمر بنفسها؛ قم بفصل الاتصال وإعادة توصيله حتى يتمكن محدد وقت الاتصال من
> إعادة التخطيط وفقًا للحد الأقصى الجديد.

### الأعراض → الحل

| الأعراض (إعدادات المصفوفة / الاتصال / `analyze_array_network`) | السبب | الحل |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`، `Reduce ROI to enable` | إعادة تعيين `ReceiveBufferLen` إلى 32 (عادةً بعد تحديث برنامج التشغيل) | اضبط `ReceiveBufferLen`→256، و`PendingReceives`→64؛ أعد فتح اللوحة (أعد تشغيل الخادم الخلفي إذا كان قد خزن حجم الحلقة القديم في ذاكرة التخزين المؤقت) |
| إعادة التشغيل/التوقف عند الإغلاق؛ لاحقًا تظهر أخطاء WMI في `Invalid class`، ولا يتم تمكين بطاقة الشبكة، فقدان محركات أقراص USB | برنامج تشغيل Realtek USB 10GbE القديم لعام 2016 → شاشة الموت الزرقاء (BSOD) `0x9F` → إيقاف التشغيل قسريًا | قم بتحديث برنامج تشغيل المحول إلى الإصدار 10.67 أو أعلى (2026)، ثم إعادة تطبيق إعدادات حلقة الاستقبال المذكورة أعلاه |
| ينجح الاتصال لكنه يعرض دقة أقل من الدقة الأصلية | يقوم Smart-prep تلقائيًاتقليص الإطار تلقائيًا ليتناسب مع السلك | قم بترقية الوصلة / اقبل التقليص / `--force-tier slip-emit-and-capture` |
| المصفوفة تُبلغ عن معدل إطارات في الثانية مستهدف سليم لكنها تقدم جزءًا بسيطًا منه؛ `health.gvsp_corrupt_rate_pct` غير صفر، `never_arrived_rate_pct` 0 | الميزانية المفترضة للسلك لدى المضيف تبالغ في تقدير ما يدعمه فعليًّا (شائع في محول إيثرنت USB، أو مسار PCIe ضيق، أو بنية مشتركة) | أعد الاتصال بقيمة `--wire-ceiling-mbps` أقل وأعد فحص كتلة الحالة. **ليس** `--target-fps` — يتم تثبيت وتيرة GevSCPD عند الاتصال |
| الكاميرات مفقودة من المجموعات المنشورة؛ `health.never_arrived_rate_pct` غير صفر، `gvsp_corrupt_rate_pct` 0 | مسار التشغيل / المزامنة — الكاميرات لا تعمل، وليست مشكلة في الشبكة | تحقق من كابل المزامنة M8 و`--line`؛ تأكد من أن كل عضو في حالة تأهب (`TriggerMode=On`) |
| تجاوزت القيم `**OVER-SUBSCRIBED**` / `Wire budget` الحد المسموح به في `analyze-array`، أو رفض الاتصال بدقة مثبتة (`array over-subscribes the wire`) | يتجاوز إجمالي الطلب لكل كاميرا (8 ميغابايت/ثانية كحد أدنى × عدد الكاميرات N) الحد الأقصى الآمن من التصادم — 6 كاميرات بدقة كاملة على شبكة 1 جيجابت إيثرنت @1500 MTU، 9 مع إطارات جامبو | عدد أقل من الكاميرات، أو إطارات جامبو من طرف إلى طرف، أو بطاقة شبكة أسرع. **لن يساعد ROI/binning** (الحد الأقصى لا يعتمد على حجم الإطار). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` يتجاوز الحد الأقصى في الاختبار (يقبل فقدان الحزم) |

---

## `chloros-cli daq`

أوامر مستشعر Spectral. فئتان:
- **`pool-*`**— عملاء «HTTP» خفيفون يتحكمون في المستشعر من خلال المجمع الدائم للخلفية.**هذا هو المسار المدعوم، وهو المسار الوحيد الموجود في إصدار CLI.** تمتلك الخلفية عملية النقل، لذا فإن واجهة المستخدم الرسومية (GUI) ونصوص CLI وSDK تتشارك جميعها مقبضًا نشطًا واحدًا بدلاً من التنافس على المنفذ التسلسلي.
- **كل شيء آخر**(`test`، `record`، `live`، `stream`، `connect`، `info`، `net`، `ota`، `sample-rate`، `calibrate`، `serve`، `ws`، `udp`، `mqtt`، `reflectance`، `login`، `logout`، `status`) — الوصول المباشر إلى الأجهزة، موثق أدناه من أجل الاكتمال. تتطلب هذه الرموز حزمة `daq` Python، وهي**غير مضمنة في أي منتج مرفق**: حيث يستبعدها ملف CLI المُجمَّع (تقوم `scripts/Build-CLI.ps1` بتعيين `--nofollow-import-to=daq`، كما أن وسائل النقل `pyserial` / `bleak` / `zeroconf` معها)، كما أن حزمة PyPI SDK لا تحتوي عليها أيضًا. وهي تعمل فقط من خلال سحب نسخة المصدر، لذا تعامل معها على أنها مسار تطوير داخلي لـ MAPIR بدلاً من اعتبارها شيئًا يمكن الوصول إليه.
- **`discover` / `list`** يجمع بين الاثنين: فهي أوامر مباشرة للأجهزة من نسخة المصدر، ولكن في الإصدار النهائي يتم الرجوع إلى `pool-discover` وتقوم الخلفية بإجراء المسح. لذا فإن المسح يعمل في كل مكان — وهو أمر مهم لأنه الطريقة الوحيدة لمعرفة عنوان MAC BLE لجهاز DAQ-M.

> **`chloros-cli daq --help`** (و `-h` / `help`) يسرد الأوامر الفرعية لـ `pool-*` — يتم توجيه المساعدة عمدًا إلى عميل المجمع بحيث تعكس الأوامر التي يتم تشغيلها فعليًّا. إذا قمت باستدعاء أمر فرعي مباشر للأجهزة على إصدار جاهز، فسيتم إنهاء العملية بخطأ صريح يحدد الحزمة المفقودة ويوجهك مرة أخرى إلى `pool-*`؛ ولا يحدث أي فشل بصمت. (يُستثنى من ذلك `discover` / `list` — حيث يتم إعادة توجيههما إلى `pool-discover` ويعملان بشكل طبيعي.)
>
> **كل ما يحتاجه العميل يمكن الوصول إليه عبر `pool-*`** — الاتصال، والبث، وتسجيل الملفات `.daq` المعايرة، وتبديل ملفات تعريف السعة. يمكن أيضًا تشغيل وحدة DAQ من Python باستخدام `chloros_sdk.connect_daq_sensor()`، الذي يستخدم نفس المسار المجمع.

### سير عمل «الاتصال الأول» لمستشعر DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### مرجع `pool-*`

| الأمر الفرعي | الغرض |
| --- | --- |
| `daq pool-connect` (smart-detect) | فتح مستشعر في تجمع الخلفية. |
| `daq pool-connect --port PORT` | DAQ-U على منفذ تسلسلي محدد. |
| `daq pool-connect --ble` | DAQ-M عبر BLE، مع مسح تلقائي لعنوان MAC. |
| `daq pool-connect --mac MAC` | DAQ-M عبر BLE على عنوان MAC معروف (يتضمن `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E عبر إيثرنت على مضيف معروف. |
| `daq pool-connect --eth` | DAQ-E عبر إيثرنت، اكتشاف المضيف تلقائيًا (mDNS + اللجوء إلى ARP؛ يعمل من ذاكرة تخزين مؤقتة فارغة لـ ARP على Windows و Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | ضبط نافذة التكامل / حالة AE. |
| `daq pool-connect --no-stream` | الاتصال دون بدء البث بعد (يُستأنف باستخدام `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | ملف تعريف تصحيح السعة. الإعداد الافتراضي في الخلفية هو `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | مسح كل وسيلة نقل بحثًا عن أجهزة استشعار يمكنك الاتصال بها، دون إجراء الاتصال. **هذه هي الطريقة التي يمكنك من خلالها العثور على عنوان MAC الخاص بـ DAQ-M عبر تقنية BLE.** يتم التوجيه تلقائيًا إلى `daq discover` / `daq list` في الإصدارات الجاهزة. لا يتم سرد المستشعرات المفتوحة بالفعل في المجموعة — حيث يتوقف جهاز DAQ-M المتصل عن الإعلان — لذا استخدم `pool-list` لتلك المستشعرات. |
| `daq pool-list` | اعرض كل مستشعر في مجموعة الخلفية. |
| `daq pool-disconnect --sensor-id ID [--all]` | تحرير. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | أحدث N إطارات طيفية. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | استئناف / إيقاف مؤقت للبث. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | بدء / إيقاف تسجيل .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | تبديل ملف تعريف تصحيح الحد الأقصى أثناء التشغيل. |

### الأوامر الفرعية المباشرة للأجهزة (متوفرة في نسخة المصدر فقط — غير موجودة في الإصدارات النهائية)

> مدرجة من أجل الاكتمال. تتطلب هذه الأوامر حزمة `daq` Python بالإضافة إلى `pyserial` / `bleak` / `zeroconf`، ولا يتم تضمين أي منها في الإصدارات المُجمَّعة CLI أو على PyPI SDK — فهي تعمل فقط من نسخة المصدر MAPIR. **إذا كنت تستخدم إصدارًا مُطروحًا Chloros، فاستخدم الأوامر `pool-*` المذكورة أعلاه بدلاً من ذلك**؛ فهي تغطي الاتصال، والبث، والتسجيل، واختيار الحد الأقصى.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

فتح مشروع Chloros محفوظ والاتصال به وتشغيله (مجلد يحتوي على `cameras.json` + `sensors.json` + `project.json`). يتم توجيه كل شيء عبر الخلفية بحيث تنتج واجهة المستخدم الرسومية (GUI) وCLI حالة متطابقة للأجهزة.

### مرجع الأوامر الفرعية

| الأمر الفرعي | الغرض |
| --- | --- |
| `project open PATH` | طباعة قائمة الأجهزة الخاصة بالمشروع (الكاميرات، المصفوفات، المستشعرات). |
| `project devices PATH [--reconnect]` | عرض قائمة أو إعادة تشغيل عملية الاكتشاف. |
| `project connect PATH [--cameras-only] [--sensors-only]` | توصيل كل كاميرا / مصفوفة / مستشعر محفوظ. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | التقاط صورة واحدة من كاميرا أو مصفوفة محددة. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | سلسلة من N إطارات من كاميرا أو مصفوفة محددة (`-n/--count` القيمة الافتراضية 5؛ `-i/--interval` ثوانٍ بين الإطارات، القيمة الافتراضية 0). تعمل التسلسلات المتتالية للمصفوفات على إزالة التكرار من المجموعات المتزامنة المتكررة (مراقب التقادم) بحيث لا يمكن لمصفوفة ذات دورات جزئية إعادة N نسخة من إطار واحد؛ وتطبع النتائج لكل تكرار. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | البث إلى القرص عبر مهمة خلفية. `--poll-interval` = عدد الثواني بين عمليات الاستقصاء `/stats` (الافتراضي 2.0). |
| `project sensor read PATH NAME [--json]` | أحدث إطار طيفي. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | تسجيل ملف .daq. |
| `project run PATH RECIPE.yaml` | تنفيذ وصفة التقاط YAML/JSON. `--dry-run` يتحقق من صحة الوصفة دون تشغيلها. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | حساب محاذاة المصفوفة — انظر [جدول العلامات أدناه](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | طباعة ملف تعريف المحاذاة الحالي. |
| `project align clear PATH NAME` | حذف ملف التعريف المخزّن مؤقتًا. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | تعديل تحويل أحد الأجهزة التابعة. |
| `project align export PATH NAME --to FILE` | حفظ ملف التعريف في JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | تحميل ملف تعريف محفوظ. |

#### خيارات `project align calibrate`

| العلامة | القيمة الافتراضية | الوصف |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | طريقة المحاذاة. **تختلف هذه التهجئات عن `lattice align-calibrate`**، التي تستخدم الصيغ المختصرة `orb` / `akaze` / `phase`؛ ولا يمكن استبدال الأمرين ببعضهما البعض في هذه العلامة. |
| `--model {translation, rigid, affine, homography}` | `affine` | تحويل النموذج ليتناسب. |
| `--frames N` | `1` | مزامنة لقطات الإطارات مع المتوسط. |
| `--reference SN` | الكاميرا الرئيسية | الرقم التسلسلي للكاميرا المرجعية؛ يتم تحويل كل عضو آخر ليتوافق معها. |
| `--max-features N` | `5000` | الحد الأقصى لعدد ميزات ORB. |
| `--ratio-threshold F` | `0.75` | اختبار . |
| `--ransac-threshold-px F` | `3.0` | عتبة النقاط الداخلية في RANSAC. |
| `--min-matches N` | `15` | **بوابة الجودة** — رفض الحل إذا كان عدد التطابقات الداخلية أقل من هذا العدد. |
| `--max-reproj-err-px F` | `4.0` | **بوابة الجودة** — رفض الحل إذا تجاوز خطأ إعادة الإسقاط RMS هذا الحد. |
| `--checkerboard RxC` | — | هندسة اللوحة لـ `--method checkerboard`، على سبيل المثال `9x6`. |
| `--name PROFILE` | فارغ | اسم الملف الشخصي المضمن في ملف JSON المحفوظ. **ليس اسم المصفوفة** — وهو الموضع `NAME`. |

بوابتا الجودة هما السبب في أن عملية المعايرة قد تنجح في الحل ومع ذلك
ترفض الحفظ: أي ملف تعريف يفشل في أي منهما سيؤدي إلى تسجيل خاطئ صامت في كل
التسجيل الخاطئ لكل عملية التقاط لاحقة دون إظهار أي خطأ، لذا يتم رفضه بدلاً من الاحتفاظ به.

### أمثلة

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### لغة DSL للوصفات

يقبل `project run RECIPE.yaml` ملف YAML أو ملف لغة التوصيفJSON(YAML) يصف تسلسلًا من الإجراءات:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

الإجراءات المدعومة: `apply`، `wait`، `capture`، `stream`، `burst`، `sensor`. الإجراء `burst` يتطلب `name` (مطلوب)، و`count` (الافتراضي 5)، و`interval` (بالثواني، الافتراضي 0)، و`output`، و`format`، و`settings` (نفس شكلإعدادات الكاميرا مثل `apply`)؛ تستخدم سلسلة اللقطات المتتالية نفس آلية المراقبة الخاصة بالمجموعة التي تمت مزامنتها حديثًا مثل `project burst`.

تشغيله:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## متغيرات البيئة

| المتغير | التأثير |
| --- | --- |
| `CHLOROS_BACKEND_URL` | تجاوز الخلفية URL (الافتراضي `http://127.0.0.1:5000`) — **يتم احترامه فقط من قبل مجموعات الأوامر `lattice` و `project` و `daq pool-*`.** الأوامر الأساسية (`process`، و`login`، و`logout`، و`status`، و`export-status`، `time-sync`، `selftest`) تربط `http://127.0.0.1:<port>` وتتجاهل هذا المتغير (يتجنب العنصر الثابت IPv4 Windows `localhost`→`::1` عقوبة تبلغ حوالي 2 ثانية لكل طلب)، لذا فهي تستهدف دائمًا الجهاز المحلي. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` يخفض مستوى رفض الاتصال الناتج عن الاشتراك الزائد في المصفوفة (الطلب الإجمالي لكل كاميرا &gt; الحد الأقصى الآمن من التصادم مع `pin_resolution`) إلى «تحذير صاخب ومتابعة»، مع قبول فقدان حزم GVSP. للاستخدام في الاختبارات فقط — انظر [نموذج الإطارات في الثانية (fps) والاندفاع (burst) للمصفوفة](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | يتم تعيينه بواسطة «CLI» نفسه؛ يُعلم الخلفية بتمكين المعالجة المتوازية. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` يتخطى اختبار التراجع لـ GVSP (نتائج ICMP فقط). **هذا يؤدي إلى إيقاف تشغيل حزم الجامبو، ولا يقتصر الأمر على تقليل السجلات فحسب** — فالكاميرا لا ترد على اختبارات DF إلا حتى 1500 على كل مسار، لذا فإن هذا الاختبار هو الشيء الوحيد القادر على اكتشاف حزم الجامبو. يوفر حوالي 1 ثانية لكل كاميرا لكل اتصال؛ ويكلف حوالي 1.45× السعة القصوى للسلك إذا كانت الشبكة *قادرة* على نقل حزم جامبو. يُصدر ملف «SDK» تحذيرًا عند ضبط هذا الإعداد. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | تثبيت حجم حزمة GVSP على N بايت؛ وتخطي الاختبار بالكامل. يُفضل استخدام الإعداد لكل أمر (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) بدلاً من الضبط الدائم: فالحجم المثبت يتوقف عن التكيف مع الشبكة أمامه، وتثبيت 9000 على مسار لا يستطيع نقل حزم جامبو يجعل **كل** عملية التقاط تنتهي بفشل مع `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | تجاوز دليل استخراج ملف واحد في Nuitka. يستخدم CLI تلقائيًا `/mnt/ssd/tmp` إن وجد. |

---

## رموز الخروج

| الرمز | المعنى |
| --- | --- |
| `0` | نجاح. |
| `1` | فشل عام (معظم أخطاء الأوامر الفرعية). |
| `2` | خطأ في الوسيطة. |
| `130` | تم المقاطعة بواسطة Ctrl+C. |

---

## إرشادات لاستكشاف الأخطاء وإصلاحها

- **&quot;يلزم تسجيل الدخول&quot;** → قم بتشغيل `chloros-cli login EMAIL PASSWORD` مرة واحدة على هذا الجهاز.
- **&quot;لا يمكن الوصول إلى الخادم الخلفي&quot;** → قم بتشغيل تطبيق سطح المكتب Chloros، أو قم بتشغيل الملف الثنائي للخلفية مباشرةً (`chloros-backend`)، أو تحقق من `CHLOROS_BACKEND_URL` إذا كان الأمر عن بُعد.
- **فشل أوامر `lattice` مع ظهور الرسالة &quot;لم يتم العثور على برامج تشغيل كاميرا LATTICE &quot;** → لم يتم تثبيت بيئة تشغيل Arena SDK؛ حيث يأتي برنامج CLI مزودًا بـ `win32api` مضمّنًا في Windows، لكن بيئة تشغيل C هي جزء من مُثبّت واجهة المستخدم الرسومية.
- **يظهر في &quot;اتصال المصفوفة&quot; / &quot;إعدادات المصفوفة&quot; &quot;سوف تنخفض الإطارات&quot; أو &quot;Reduce ROI to enable&quot;** → حلقة الاستقبال في بطاقة الشبكة للمضيف صغيرة جدًا (عادةً ما يتم إعادة ضبطها إلى 32 بعد تحديث برنامج تشغيل بطاقة الشبكة). انظر [إعداد بطاقة الشبكة للمضيف وضبطها](#host-nic-setup--tuning-lattice-arrays) — اضبط `ReceiveBufferLen=256` و `PendingReceives=64`.
- **يتوقف الجهاز عند إعادة التشغيل/الإغلاق، ثم يظهر خطأ WMI `Invalid class` / لا يتم تمكين بطاقة الشبكة / محركات أقراص USB مفقودة** → برنامج تشغيل محول USB 10GbE قديم يتسبب في `DRIVER_POWER_STATE_FAILURE` (شاشة الموت الزرقاء BSOD `0x9F`). قم بتحديث برنامج تشغيل المحول — انظر [إعداد بطاقة الشبكة المضيفية وضبطها](#host-nic-setup--tuning-lattice-arrays).
- **تحذير بشأن ذاكرة التبادل في Jetson** → أضف ملفمدعوم بالملفات؛ حيث يُظهر الأمر «CLI» الأوامر الدقيقة `fallocate` / `swapon`.
- **الأوامر المباشرة لـ DAQ مفقودة** → المتوقع: يُستبعد حزمة `daq` عمدًا من الإصدار المرفق `chloros-cli`، لذا لا يتوفر سوى `pool-*` (كما أن موقع PyPI SDK لا يتضمنها أيضًا). استخدم `pool-*`، الذي يتحكم في نفس المستشعر عبر الخلفية، أو `chloros_sdk.connect_daq_sensor()` من Python.

---

## انظر أيضًا

- [مرجع Python SDK](sdk-reference.md) — المكافئ البرمجي لكل أمر من أوامر CLI.
- [دليل مستشعرات DAQ](../daq/README.md) — التوصيلات والمعايرة الخاصة بكل مستشعر.
- الوثائق عبر الإنترنت: `https://mapir.gitbook.io/chloros/cli`
