# CLI : سطر الأوامر

> **مرجع كامل:**[مرجع CLI](reference/cli-reference.md) يوثق**كل علامة من علامات كل أمر فرعي** وهو مُحسّن للمساعدات الذكية — الصق الرابط URL في مساعدك واطلب أمرًا فعالًا: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **نصيحة لأدوات الذكاء الاصطناعي:** تتوفر أي صفحة من هذا الدليل بتنسيق Markdown الخام عن طريق إضافة `.md` إلى عنوانها URL (على سبيل المثال: `https://mapir.gitbook.io/chloros/reference/cli-reference.md`)، ويقوم `https://mapir.gitbook.io/chloros/llms.txt` بفهرسة الدليل بأكمله لاستخدامه في نماذج اللغة الكبيرة (LLM).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## ما هوCLI


`chloros-cli` هو واجهة سطر الأوامر لمحرك المعالجة نفسه الذي يستخدمه تطبيق سطح المكتبChloros
. وهو عميل خفيف (HTTP
) يعمل عبر الخلفية (Chloros
) (خادم محلي على `127.0.0.1:5000`) — حيث تبدأ معظم الأوامر تشغيل الخلفية تلقائيًا، لذا فإن استدعاءً واحدًا لـ `chloros-cli process …` هو كل ما يحتاجه البرنامج النصي.

يعمل على **Windows
10/11 (x64)**و**Linux
(x86_64، وNVIDIA Jetson arm64 على JetPack 6)**، في أي محطة طرفية، دون الحاجة إلى واجهة مستخدم رسومية. تحقق من التثبيت باستخدام:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

نظرة عامة على مجموعات الأوامر:

* **المعالجة والحساب** — `process`، `login`، `logout`، `status`، `export-status`، `language` (38 لغة — انظر [اللغات المدعومة](supported-languages.md))، `set-project-folder` / `get-project-folder` / `reset-project-folder`، `selftest`، `update` (Linux
/Jetson فقط)
* **الأجهزة الحية** — `lattice` (التحكم في كاميرا LATTICE، أكثر من 45 أمرًا فرعيًا)، `daq pool-*` (مستشعرات الضوء DAQ)، `time-sync` (PTP)
* **الأتمتة** — `project` (تشغيل مشروعChloros
محفوظ بدون واجهة مستخدم، بما في ذلك وصفات الالتقاط بتنسيق YAML)

خيارات عامة جديرة بالمعرفة: `--port N` (منفذ الخلفية، الافتراضي `5000`)، `-v/--verbose`، `--restart` (إعادة تشغيل الخلفية قسريًّا)، `--backend-exe PATH`. راجع [مرجعCLI
](reference/cli-reference.md) للاطلاع على القائمة الكاملة.

***

## التثبيت

يأتي برنامج «CLI
» **مضمنًا في أداة تثبيت «Chloros
»** على جميع الأنظمة الأساسية — ولا يوجد تنزيل منفصل لـ «CLI
». احصل على أداة التثبيت من صفحة [التنزيل](download.md).

###Windows


تقوم أداة التثبيت بوضع ملف «CLI
» في:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

ويضيف هذا المجلد إلى نظامك `PATH` — **افتح محطة طرفية جديدة**بعد التثبيت حتى يتم التقاط `PATH` المحدث. كما يقوم المثبت بوضع نصوص تشغيل (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) في جذر التثبيت بالإضافة إلى**اختصار في قائمة البدءChlorosCLI
** اختصار في قائمة «ابدأ»، يفتح كل منها محطة طرفية مع `chloros-cli` جاهزًا للاستخدام.

###Linux


قم بتثبيت `.deb` المناسب لهيكل نظامك:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

يؤدي هذا إلى تثبيت `chloros-cli` إلى `/usr/bin/chloros-cli` (الموجود بالفعل في الإصدار `PATH`) والواجهة الخلفية إلى الإصدار `/usr/lib/chloros/chloros-backend`، إلى جانب بيئة تشغيل ArenaSDK
اللازمة لكاميرات LATTICE. انظر [تثبيتLinux
](linux/linux-installation.md) للحصول على التفاصيل.

### التحقق

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## تسجيل الدخول والترخيص

CLI
(وPython
SDK
) يتطلب الوصول **خطة مدفوعةChloros
+**— تتوفر هذه الميزة في أي فئة مدفوعة؛ أما الفئة المجانية فلا تتوفر فيها. يتم فرض الحد الأدنى**من جانب الخادم** بواسطة الخلفية، وليس بواسطة ملفCLI
الثنائي: يتم رفض المكالمة عند تسجيل الخروج برمز `401 AUTH_REQUIRED`، أما الاستدعاءات التي تتم أثناء تسجيل الدخول في الخطة المجانية، فيتم رفضها برمز الخطأ `403 PLAN_UPGRADE_REQUIRED`، سواء كانت قادمة من `chloros-cli` أو منSDK
أو من عميلHTTP
مخصص. قم بالترقية على [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

قم بتسجيل الدخول **مرة واحدة لكل جهاز**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**كلمات المرور التي تحتوي على أحرف خاصة**(يتم تشويه كلمة المرور `$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` بواسطة شل (يكتشفCLI
ذلك عند ظهور الخطأ 401 ويقوم بإعادة المحاولة تلقائيًا، لكن استخدام الأقواس المفردة يتجنب المشكلة تمامًا).
{% endhint %}

يتم تخزين الجلسة مؤقتًا في `~/.chloros/user_session.json` وتستمر في العمل دون اتصال بالإنترنت طوال فترة السماح الخاصة بالخطة (30 يومًا للخطط الشهرية، وحتى انتهاء الصلاحية للخطط السنوية). يعمل `chloros-cli status` حتى بدون خطة مدفوعة، لذا يظل سبب الرفض مرئيًا دائمًا.

{% hint style="danger" %}
**هل تريد جدولة مهام بدون واجهة مستخدم؟ قم بتسجيل الدخول أولاً.**لا تفشل الأوامر التي تُنشئ عمليات في الخلفية (`process`، `status`، `export-status`، …) يتم تشغيله**بدون جلسة مخزنة مؤقتًا**لا يفشل سريعًا — بل ينتقل إلى موجه تفاعلي `Email:` / `Password:` على stdin. وبالتالي، فإن مهمة cron غير المراقبة أو خطوة CI**ستتوقف في انتظار الإدخال**. قم بتشغيل `chloros-cli login EMAIL 'PASSWORD'` مرة واحدة على الجهاز قبل جدولة أي شيء.
{% endhint %}

***

## أول عملية معالجة لك

وجه `process` إلى مجلد يحتوي على اللقطات — فهو يكتشف تلقائيًاSurvey3
(`.raw` + `.jpg`)، LATTICE (`.tif`/`.tiff`)، أو `.dng`، أو مزيجًا منها:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

يتم بث التقدم المباشر لكل خيط من خطوط الأنابيب (الكشف، التحليل، المعالجة، التصدير)، وينتهي التشغيل الناجح بالإبلاغ عن عدد منتجات الصور التي تمت كتابتها (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### مكان حفظ المخرجات

يكتب `process` في **مجلد المشروع**، وليس في مجلد الإدخال الخاص بك:

* في حالة عدم وجود `-o`: يتم إنشاء المشروع ضمن مجلد المشروع الافتراضي الخاص بك (المشترك مع واجهة المستخدم الرسومية؛ قم بإدارته باستخدام `get-project-folder` / `set-project-folder`، والبديل `~/Chloros Projects`)، ويتم تسميته بـ `-n/--project-name` أو طابع زمني (`YYYYMMDD_HHMMSS`) عند حذفه.
* مع `-o PATH`: هذا المجلد **هو** مجلد المشروع. إذا كان يحتوي بالفعل على `project.json`، يتم إنشاء مجلد شقيق بامتداد `_1`/`_2`… بدلاً من الكتابة فوقه.

داخل المشروع، يتم تجميع المنتجات **حسب الكاميرا، ثم حسب تنسيق الملف**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

مجلد الكاميرا هو `LATT-<sensor>-<lens>-F<filter>` لـ LATTICE (مطابقًا لـ EXIF لالتقاط الصورة `Model`) و `<model>_<filter>` (على سبيل المثال: `Survey3N_RGN`) لـSurvey3
. ويأتي مجلد التنسيقات بعد `--format`: `tiff16`، و`tiff8`، `png8`، `jpg8`، أو `tiff32` لـ `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**يحتفظ كل منتج مُصدَّر باسم الملف الأصلي (SOURCE).**لا يزال تصدير radiance لـ `capture_..._raw.tif` يُسمى `capture_..._raw.tif` — إنه موجود فقط في `tiff32/Radiance_Images/`.**المجلد هو ما يحدد المنتج، وليس اسم الملف**، لذا استخدم تعبيرًا عامًا (glob) للبحث عن الدليل، وليس عن اللاحقة `*radiance*`.
{% endhint %}

### الخيارات التي ستستخدمها فعليًّا

| العلامة | القيمة الافتراضية | الوظيفة |
| --- | --- | --- |
| `-o, --output PATH` | مجلد المشروع الافتراضي | موقع مجلد المشروع (انظر أعلاه). |
| `-n, --project-name NAME` | الطابع الزمني | اسم المشروع. |
| `--format FMT` | `TIFF (16-bit)` | أحد الخيارات التالية: `TIFF (16-bit)`، `TIFF (32-bit, Percent)`، أو `PNG (8-bit)`، أو `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | لا شيء | مؤشرات الغطاء النباتي المراد تصديرها (انظر [مؤشرات الغطاء النباتي](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = إزالة التباين العصبي، أبطأ، أعلى جودة (Chloros
+، وحدة معالجة الرسومات NVIDIA). |
| `--vignette / --no-vignette` | قيد التشغيل | تصحيح التدرج اللوني. |
| `--reflectance / --no-reflectance` | قيد التشغيل | معايرة الانعكاس؛ بالنسبة لـ LATTICE، يُعد هذا أيضًا مفتاح تبديل منتج الانعكاس. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | فرض نقطة دخول مسار المعالجة لملفات TIFF الخاصة بـ LATTICE. |

بالنسبة لكل شيء آخر — ضبط الكشف عن الهدف، PPK، دبابيس التعريض، علامات محاذاة المصفوفات — راجع [القسم `process` من مرجعCLI
](reference/cli-reference.md).

***

## اختيار العناصر المراد تصديرها (منتجات LATTICE)

تتفرع معالجة LATTICE إلى **كل منتج قابل للتطبيق في مسار واحد**. أربعة مفاتيح تبديل لكل منتج تكون جميعها**مفعّلة افتراضيًا**؛ استخدم النموذج `--no-` لإلغاء تفعيل أحدها:

| زر التبديل | المنتج |
| --- | --- |
| `--debayered` | إزالة التمويه الخطي → `Debayered_Images/` |
| `--preview` | معاينة العرض (توازن اللون الأبيض + جاما؛ تمديد الألوان الزائفة للصور متعددة الأطياف) → `Preview_Images/` |
| `--radiance` | الإشعاع بتصنيف float32، واط/م²/سر/نانومتر → `Radiance_Images/` (دائمًا `tiff32/`) |
| `--reflectance` | uint16 الانعكاسية، جاهز لـ Pix4D → `Reflectance_Calibrated_Images/` |

RGB
لا تصدر الكاميرات الرئيسية سوى الإشعاع بعد إزالة التباين + المعاينة — ولا معنى للإشعاع/الانعكاس لكل نطاق في مستشعر النطاق العريض، لذا فإن خيارات التبديل هذه لا تؤدي إلى أي تأثير بالنسبة لها.Survey3
`.raw` يتجاهل خيارات التبديل ويتبع المسار القياسي للانعكاس/الهدف.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (الافتراضي `auto`) يختار مرجع الانعكاس: `auto` يجعل [هدف المعايرة](calibration-targets.md) داخل الإطار الذي اجتاز اختبار ضمان الجودة المرجع المطلق ويعود إلى تقسيم الضوء الساقط لمستشعر DAQ (ρ = π·L/E) في حالة عدم وجود هدف؛ `target` صارم (لا يوجد استبدال DAQ)؛ `daq` يعتمد على DAQ بشكل مطلق. يمكن توفير مسح الهدف المقاس لكل وحدة باستخدام `--target-reflectance-dir`.

{% hint style="info" %}
**قراءة وحدات البكسل للانعكاس:**DN الذي يعني ρ = 1.0 هو**لكل مصدر** — ملفات LATTICE تضع علامة `Chloros:PixelScale=32768` في XMP؛ بينما تستخدم ملفاتSurvey3
الرقم 65535 (ولا تحتوي على علامات `Chloros:*`). اقرأ العلامة وقسم عليها بدلاً من افتراض قيمة ثابتة. توجد التفاصيل والحالة الاستثنائية الوحيدة المتعمدة التي لا تحتوي على مقياس في [مرجعCLI
](reference/cli-reference.md).
{% endhint %}

**تبدأ المعالجة دائمًا من `raw`.** لا يتم إعادة تغذية المنتجات المشتقة (صادرات إزالة التباين/الإشعاع/الانعكاس) عبر خط الأنابيب أبدًا — حيث إن إعادة استيرادها ومعالجتها سيؤدي إلى تطبيق حسابات المعايرة مرتين، لذا يتخطاهاChloros
ويشير إلى ذلك. يُعد `--input-level` مخرجًا احتياطيًا متعمدًا عندما تحتاج فعليًّا إلى فرض نقطة دخول.

***

## عند فشل التشغيل

اعتبارًا من الإصدار 1.2.0، يفشل `process` بشكل واضح بدلًا من «النجاح» دون عرض أي نتائج:

* التشغيل الذي **طلب منتجات ولكنه لم يكتب أي منها**— فقط `project.json` و `calibration_data.json` — يُظهر `Processing finished but wrote no image products.` و**يخرج بقيمة غير صفرية**، بحيث يمكن للبرامج النصية اكتشاف ذلك. الأسباب المعتادة: لم يتم التعرف على مجلد الإدخال على أنه عملية التقاط (تحقق من التخطيط و`--input-level`)، أو أن كل منتج مطلوب كان غير قابل للتطبيق على تلك الكاميرات (على سبيل المثال، طلب الإشعاع/الانعكاس من كاميراتRGB
فقط).
* **التشغيل المتعمد للبيانات الوصفية فقط** (إيقاف تشغيل جميع المنتجات، بدون `--indices`) لا يزال يعتبر ناجحًا — حيث يُعد إخراج صورة فارغة هو النتيجة الصحيحة في هذه الحالة.
* أعد التشغيل باستخدام `--verbose` وتحقق من سجل الخلفية بحثًا عن الأسطر `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`، التي تشرح حالات التخطي لكل كاميرا على حدة.

رموز الخروج: `0` نجاح · `1` فشل عام · `2` خطأ في الوسيطة · `130` تم مقاطعتها بواسطة Ctrl+C.

***

## مؤشرات الغطاء النباتي

قم بتشغيل `--indices` مع اسم واحد أو أكثر من الأسماء المعدة مسبقًا؛ حيث يتم حفظ كل مؤشر في مجلده الخاص `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

الأسماء المحددة مسبقًا البالغ عددها 22 التي تقبلها `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**توجد ثلاث قوائم فهارس — لا تخلط بينها.**تحتوي القائمة المنسدلة «إعدادات المشروع» في واجهة المستخدم الرسومية (GUI) على 27 صيغة (تضيف `FCI1`، `FCI2`، `GARI`، `GEMI`، `LCI` — هذه الصيغ الخمس مخصصة لواجهة المستخدم الرسومية فقط و**ليست** صالحة لـ `--indices`). يستخدم الأمر `lattice index --preset` (المباشر/غير المتصل) قائمة منفصلة خاصة به تحتوي على 22 إعدادًا مسبقًا. تم توثيق الصيغ وحسابات النطاقات في [صيغ المؤشرات متعددة الأطياف](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## مستشعرات الضوء DAQ: لمحة سريعة

تقوم عائلة `daq pool-*` بتشغيل مستشعرات الطيف DAQ منMAPIR
(DAQ-U عبر USB، DAQ-M عبر BLE، DAQ-E عبر Ethernet) من خلال المجموعة الدائمة للخلفية — حيث تشترك واجهة المستخدم الرسومية (GUI) وCLI
وSDK
جميعًا في مقبض واحد مباشر. **`pool-*` هو مسار DAQ المدعوم فيCLI
المُرفق**؛ أما الأوامر الفرعية الأخرى لـ `daq` التي قد ترى الإشارة إليها فهيMAPIR
— وهي واجهة داخلية مخصصة للمصدر فقط — وتنتهي بخطأ صريح يوجهك إلى `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` بدون `--duration` يعمل حتى `pool-record --stop`؛ دليل الإخراج الافتراضي هو `~/Documents/DAQ Live View/` **على جهاز الخادم الخلفي**. يتم اختيار ملف تعريف تصحيح السعة عند الاتصال (`--cap-id`، الإعداد الافتراضي للخلفية هو `sunshine_cosine`) ويمكن تبديله في الوقت الفعلي بـ `pool-set-cap` — يتم تناول ملفات تعريف الحد الأقصى والنطاق المعاير للمستشعر في فصول DAQ من هذا الدليل.

{% hint style="warning" %}
**DAQ-E على مضيف متعدد بطاقات الشبكة (NIC):** قد يفشل الاكتشاف التلقائي الأول لـ `pool-connect --eth` بعد التشغيل حتى مع وجود مستشعر سليم. `--eth-host <ip-or-hostname>` هو الشكل الموثوق — استخدمه كلما فشل الاكتشاف.
{% endhint %}

***

## كاميرات LATTICE و PTP وأتمتة المشاريع

تغطي عائلة `lattice` (أكثر من 45 أمرًا فرعيًا) عمل كاميرات LATTICE من البداية إلى النهاية: الاكتشاف، والتقاط الصور الفردية، والمصفوفات المتزامنة الدائمة مع تدفق الاتصال «smart-prep» في واجهة المستخدم الرسومية، والمعاينة المباشرة عبر المتصفح، والمحاذاة، وحسابات الفهرس، وتشخيص بطاقة الشبكة (NIC) الخاصة بالمضيف. نموذج:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

إلى جانب ذلك: يقدم `chloros-cli time-sync` تقارير عن وحدة التحكم الرئيسية (Grandmaster) لـ PTP التي يعمل عليها المضيفChloros
(تعمل كاميرات LATTICE ومستشعرات DAQ-E كوحدات تابعة لها من أجل الطوابع الزمنية عبر الأجهزة)، ويفتح `chloros-cli project` مشروعًا محفوظًا منChloros
ويقوم بتشغيل الكاميرات والمصفوفات وأجهزة الاستشعار دون واجهة مستخدم — بما في ذلك وصفات التقاط YAML المبرمجة.

هذه المجموعات الثلاث (`lattice`، `project`، `daq pool-*`) هي أيضًا العائلات الوحيدة التي تدعم `CHLOROS_BACKEND_URL` لتشغيل خلفية **بعيدة**؛ حيث تستهدف الأوامر الأساسية دائمًا الجهاز المحلي.

توجد إرشادات تفصيلية كاملة في فصول LATTICE من هذا الدليل؛ وتوجد كل العلامات في [مرجعCLI
](reference/cli-reference.md).

***

## استكشاف الأخطاء وإصلاحها: أهم 5

| الأعراض | الحل |
| --- | --- |
| `Login required`، أو توقف مهمة مجدولة عند موجه `Email:` | قم بتشغيل `chloros-cli login EMAIL 'PASSWORD'` مرة واحدة على هذا الجهاز — فالأوامر التي لا تحتوي على جلسة مخزنة مؤقتًا تعمل بشكل تفاعلي بدلاً من الفشل السريع. |
| `backend unreachable` | قم بتشغيل تطبيق سطح المكتبChloros
، أو قم بتشغيل الملف الثنائي للخلفية مباشرةً (`chloros-backend`). إذا قمت بتوجيه `lattice`/`project`/`daq pool-*` إلى خلفية بعيدة، فتحقق من `CHLOROS_BACKEND_URL`. |
| تم حظر اتصال المصفوفة: `FRAMES WILL DROP` / `Reduce ROI to enable` | إعادة تعيين حلقة استقبال بطاقة الشبكة (NIC) للمضيف إلى الإعدادات الافتراضية — السبب الأول لرفض جهاز كان يعمل سابقًا الاتصال، وعادةً ما يحدث ذلك بعد تحديث برنامج تشغيل بطاقة الشبكة. قم بتشغيل `chloros-cli lattice network --fix` من محطة طرفية **ذات صلاحيات مرتفعة** (أو قم بتعيين `ReceiveBufferLen=256`، `PendingReceives=64`)؛ راجع *إعداد وضبط بطاقة الشبكة للمضيف* في المرجع. |
| يخرج الأمر الفرعي `daq` برسالة: «يتطلب حزمة daq الكاملة…» | أمر متوقع في الإصدارات المُشحَّنة — حيث إن حزمةCLI
المُجمَّعة لا تشمل سوى عائلة أوامر `daq pool-*`، التي تغطي الاتصال، والتدفق، والتسجيل، واختيار الحد الأقصى. استخدم `pool-*` (أو `chloros_sdk.connect_daq_sensor()` منPython
). |
| يعرض Jetson تحذيرًا بشأن ذاكرة التبادل قبل المجلدات الكبيرة | أضف ذاكرة تبادل مدعومة بملف — يُصدر ملفCLI
الأوامر الدقيقة `fallocate`/`swapon` المطلوب تشغيلها. |

***

## الحصول على المساعدة

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **كل علامة، كل أمر فرعي:** [CLI
مرجع](reference/cli-reference.md)
* **المكافئ فيPython
:** [Python
SDK
](api-python-sdk.md) و[SDK
مرجع](reference/sdk-reference.md)
* **الدعم:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
