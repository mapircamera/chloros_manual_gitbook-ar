# CLI: سطر الأوامر

> **مرجع كامل:**[مرجع CLI](reference/cli-reference.md) يوثق**كل علامة من علامات كل أمر فرعي** وهو مُحسّن للمساعدات الذكية — الصق رابط URL الخاص به في مساعدك واطلب أمرًا فعالًا: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **نصيحة لأدوات الذكاء الاصطناعي:** تتوفر أي صفحة من هذا الدليل بتنسيق Markdown الخام عن طريق إضافة `.md` إلى URL الخاص بها (على سبيل المثال: `https://mapir.gitbook.io/chloros/reference/cli-reference.md`)، ويقوم `https://mapir.gitbook.io/chloros/llms.txt` بفهرسة الدليل بأكمله لاستخدامه في نماذج اللغة الكبيرة (LLM).

> **هل تريد أن يقوم مساعدك بتشغيل Chloros بنفسه؟** `chloros-mcp`، وهو خادم MCP مُثبَّت عليه Python SDK، يعمل مع Claude Desktop وClaude Code وCursor وعملاء MCP الآخرين: انظر [استخدام Chloros مع مساعدات الذكاء الاصطناعي](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## ما هي واجهة سطر الأوامر (CLI
)

`chloros-cli` هي واجهة سطر الأوامر لمحرك المعالجة نفسه الذي يستخدمه تطبيق Chloros
. وهو عميل HTTP
خفيف يعمل عبر الخلفية الخاصة بـ Chloros
(خادم محلي على `127.0.0.1:5000`) — حيث تبدأ معظم الأوامر تشغيل الخلفية تلقائيًا، لذا فإن استدعاءً واحدًا لـ `chloros-cli process …` هو كل ما يحتاجه البرنامج النصي. ثم تظل الخلفية قيد التشغيل استعدادًا للأمر التالي: يتوقف عند `chloros-cli logout` (ما لم يكن التطبيق مفتوحًا أو تكون عملية تشغيل أو إنشاء قيد التنفيذ)، أو عند `chloros-cli backend stop`، أو تلقائيًا بعد 30 دقيقة في حالة عدم وجود عميل أو عملية تشغيل أو جهاز متصل. يظهر ذلك في `chloros-cli backend status`.

يعمل على **نظامي Windows
10/11 (x64)**و**نظام Linux
(x86_64، وNVIDIA Jetson arm64 على JetPack 6)**، في أي محطة طرفية، دون الحاجة إلى واجهة مستخدم رسومية. تحقق من التثبيت باستخدام:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

نظرة عامة على مجموعات الأوامر:

* **المعالجة والحساب** — `process`، `login`، `logout`، `status`، `export-status`، `language` (38 لغة — انظر [اللغات المدعومة](supported-languages.md))، `set-project-folder` / `get-project-folder` / `reset-project-folder`، `selftest`، `update` (نظام Linux
/Jetson فقط)
* **الأجهزة الحية** — `lattice` (التحكم في كاميرا LATTICE، أكثر من 45 أمرًا فرعيًا)، `daq pool-*` (مستشعرات الضوء DAQ)، `time-sync` (PTP)
* **الأتمتة** — `project` (تشغيل مشروع Chloros
محفوظ بدون واجهة مستخدم، بما في ذلك وصفات التقاط YAML)

خيارات عامة جديرة بالمعرفة: `--port N` (منفذ الخلفية، الافتراضي `5000`)، `-v/--verbose`، `--restart` (إعادة تشغيل الخلفية قسريًّا)، `--backend-exe PATH`. راجع [مرجع واجهة سطر الأوامر (CLI
)](reference/cli-reference.md) للاطلاع على القائمة الكاملة.

***

## التثبيت

يأتي **CLI** **مضمنًا في برنامج تثبيت Chloros** على كل نظام أساسي — ولا يوجد تنزيل منفصل لـ CLI
. احصل على برنامج التثبيت من صفحة [التنزيل](download.md).

### Windows


يقوم برنامج التثبيت بوضع واجهة CLI
في:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

ويضيف هذا المجلد إلى نظامك `PATH` مرة واحدة (تقوم عملية الترقية أيضًا بإزالة الإدخالات المكررة التي خلفتها برامج التثبيت السابقة) — **افتح محطة طرفية جديدة**بعد التثبيت حتى يتم التعرف على `PATH` المحدث. يقوم المثبت أيضًا بوضع نصوص تشغيل المشغل (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) في جذر التثبيت بالإضافة إلى**اختصار Chloros
CLI
** في قائمة «ابدأ»، يفتح كل منها محطة طرفية مع `chloros-cli` جاهزًا للاستخدام.

### Linux


قم بتثبيت `.deb` الخاص بهيكلية نظامك باستخدام `apt` (وليس `dpkg -i`، الذي يتخطى المتطلبات الموصى بها لـ Python
SDK
):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

يؤدي هذا إلى تثبيت `chloros-cli` إلى `/usr/bin/chloros-cli` (الموجود بالفعل في `PATH`)، وبرامج الخلفية وCLI
التي تم فك ضغطها في `/usr/lib/chloros/backend/` و`/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` هو مشغل يقوم بتشغيل الخلفية)، وبيئة تشغيل Arena SDK
التي تحتاجها الخلفية لكاميرات LATTICE، وPython
SDK
للنظام `python3`. انظر [التثبيت على نظام Linux
](linux/linux-installation.md) لمزيد من التفاصيل.

### التحقق

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## تسجيل الدخول والترخيص

يتطلب الوصول إلى واجهة سطر الأوامر (CLI
) (وPython
SDK
) **خطة Chloros
+ مدفوعة**— تتوفر هذه الميزة في أي فئة مدفوعة؛ بينما لا تتوفر في الفئة المجانية. يتم فرض الحد الأدنى**من جانب الخادم** بواسطة الخلفية، وليس بواسطة ملف CLI
الثنائي: يتم رفض أي استدعاء دون تسجيل دخول برمز الخطأ `401 AUTH_REQUIRED`، ويتم رفض المكالمة التي تتم بعد تسجيل الدخول في الفئة المجانية برمز الخطأ `403 PLAN_UPGRADE_REQUIRED`، سواء كانت قادمة من `chloros-cli` أو SDK
أو عميل HTTP
مخصص. قم بالترقية على [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

قم بتسجيل الدخول **مرة واحدة لكل جهاز**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**كلمات المرور التي تحتوي على أحرف خاصة**(يتم تشويه `$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` بواسطة شل (يكتشف واجهة سطر الأوامر CLI
ذلك عند ظهور خطأ 401 ويقوم بإعادة المحاولة تلقائيًا، لكن استخدام علامات الاقتباس المفردة يتجنب المشكلة تمامًا).
{% endhint %}

يتم تخزين الجلسة مؤقتًا في `~/.chloros/user_session.json` وتستمر في العمل دون اتصال بالإنترنت طوال فترة السماح الخاصة بالخطة (30 يومًا للخطط الشهرية، وحتى انتهاء الصلاحية للخطط السنوية). يعمل `chloros-cli status` حتى بدون خطة مدفوعة، لذا يظل سبب الرفض مرئيًا دائمًا.

{% hint style="danger" %}
**هل تريد جدولة مهام بدون واجهة مستخدم؟ قم بتسجيل الدخول أولاً.**لا تفشل الأوامر التي تُنشئ عمليات في الخلفية (`process`، `status`، `export-status`، …) يتم تشغيله**بدون جلسة مخزنة مؤقتًا**لا يفشل سريعًا — بل ينتقل إلى موجه تفاعلي `Email:` / `Password:` على stdin. وبالتالي، فإن مهمة cron غير المراقبة أو خطوة CI**ستتوقف في انتظار الإدخال**. قم بتشغيل `chloros-cli login EMAIL 'PASSWORD'` مرة واحدة على الجهاز قبل جدولة أي شيء.
{% endhint %}

***

## أول عملية معالجة لك

وجه `process` إلى مجلد يحتوي على ملفات التسجيل — حيث يقوم تلقائيًا باكتشاف Survey3
(`.raw` + `.jpg`)، LATTICE (`.tif`/`.tiff`)، أو `.dng`، أو مزيجًا منها:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

تتدفق التقدمات مباشرةً لكل خيط من خطوط الأنابيب (الكشف، التحليل، المعالجة، التصدير)، وينتهي التشغيل الناجح بالإبلاغ عن عدد منتجات الصور التي تم كتابتها: `Image products written: N`، يليه `(plus M clamp masks)` عندما يقوم التشغيل أيضًا بكتابة [أقنعة التثبيت](output-image-formats.md#clamp-masks) (لا تُحسب هذه كمنتجات).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


إذا كانت الخلفية (Chloros
) قيد التشغيل بالفعل (على سبيل المثال، تطبيق سطح المكتب)، فإن `process` تستخدمها وتفتح مشروعها فيها. وترفض العملية أثناء فتح التطبيق على مشروع فتحه التطبيق: قم بالتشغيل من التطبيق، أو أغلق ذلك المشروع. إذا كان التشغيل السابق لا يزال قيد الانتهاء، فإنه ينتظر ما يصل إلى 15 ثانية قبل الرفض. `--restart` يفرض استخدام خلفية جديدة. في نظام Linux
، لا تتم إعادة تشغيل الخلفية التي تعمل كخدمة systemd أبدًا: يستخدمها `process` كما هي، وفي الحالات التي كان سيُعيد تشغيلها فيها (`--restart`، أو عملية استيراد لم تعثر على شيء)، فإنه يعرض الرسالة `sudo systemctl restart <unit>` ويخرج من `1`.

### مكان حفظ المخرجات

يكتب `process` في **مجلد المشروع**، وليس في مجلد الإدخال الخاص بك:

* في حالة عدم وجود `-o`: يتم إنشاء المشروع ضمن مجلد المشاريع الافتراضي الخاص بك (المشترك مع واجهة المستخدم الرسومية؛ قم بإدارته باستخدام `get-project-folder` / `set-project-folder`؛ عند عدم تعيين أي مجلد، يتم إنشاؤه في مجلد المستخدم الخاص بك)، ويتم تسميته بـ `-n/--project-name` أو طابع زمني (`YYYYMMDD_HHMMSS`) عند عدم ذكر الاسم.
* مع `-o PATH`: هذا المجلد **هو** مجلد المشروع. إذا كان يحتوي بالفعل على ملف `project.json`، فسيتم إنشاء ملف شقيق بامتداد `_1`/`_2`… بدلاً من الكتابة فوقه. يتم أخذ المسار النسبي `PATH` بالنسبة للمجلد الذي يتم تشغيل `chloros-cli` منه.

{% hint style="info" %}
**المسارات النسبية (تم تغييرها في الإصدار 1.3.0).** كل مسار تفتحه الخلفية — `-o`/`--output` عند أي أمر (بما في ذلك `output` الافتراضي لأوامر الالتقاط)، `--input`، و`--burst-dir`، و`--profile`، و`--save`، و`--mask` — يتم تحديده بناءً على المجلد الذي يتم تشغيل `chloros-cli` منه، لذا فإن `-o captures/` يكتب إلى `<current folder>/captures/`. في الإصدار 1.2.0، كانت الخلفية تحلها في مجلد التثبيت الخاص بها، مما أدى إلى فشل العملية مع ظهور رسالة «الوصول مرفوض». أما بالنسبة للخلفية البعيدة (`CHLOROS_BACKEND_URL`)، فيتم إرسال المسارات كما تمت كتابتها ويتم حلها على ذلك الجهاز.
{% endhint %}

{% hint style="warning" %}
**نظام Windows
: اجعل مجلد المشروع قصيرًا ما لم تكن المسارات الطويلة قيد التشغيل.** مع إيقاف تشغيل المسارات الطويلة في Windows
(الإعداد الافتراضي لنظام Windows
)، لا يجوز أن يتجاوز طول أي مسار ملف 260 حرفًا، وتقع المنتجات حتى حوالي 130 حرفًا أسفل مجلد المشروع. يقوم `process` بالتحقق من ذلك قبل البدء: يتم رفض مجلد المشروع الذي يكون عميقًا جدًّا مع ظهور الرسالة **&quot;مجلد المشروع عميق جدًّا بالنسبة لهذا الكمبيوتر&quot;**، وأطول مجلد مشروع مناسب، وكيفية تمكين المسارات الطويلة — ولا تتم معالجة أي شيء. استخدم `-o` (أو `set-project-folder`) أقصر، أو قم بتفعيل المسارات الطويلة في Windows
(خيار **تمكين المسارات الطويلة في Windows** في المثبت، وهو محدد افتراضيًا) وأعد تشغيل Chloros
.
{% endhint %}

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

مجلد الكاميرا هو `LATT-<sensor>-<lens>-F<filter>` لـ LATTICE (مطابقًا لـ EXIF الخاص بالالتقاط `Model`) و `<model>_<filter>` (على سبيل المثال: `Survey3N_RGN`) بالنسبة لـ Survey3
. أما مجلد التنسيق فيتبع `--format`: `tiff16`، `tiff8`، `png8`، و`jpg8`، أو `tiff32` بالنسبة لـ `TIFF (32-bit, Percent)`.

بالنسبة للكاميرا M3C متعددة الأطياف، قد يظهر أيضًا مجلد `clamp_masks/` في مجلد التنسيق، بجانب مجلدات المنتجات (على سبيل المثال `tiff16/clamp_masks/Reflectance_Calibrated_Images/` و `tiff32/clamp_masks/Radiance_Images/`). ويحتوي هذا المجلد على أقنعة لكل بكسل لمنتجات الإشعاع والانعكاس التي قام فيها فصل النطاق بتثبيت القيمة السالبة عند الصفر — انظر [أقنعة التثبيت](output-image-formats.md#clamp-masks). ولا يحتوي أي مجلد منتج على قناع أبدًا.

{% hint style="info" %}
**يحتفظ كل منتج مُصدَّر باسم الملف المصدر.**لا يزال تصدير الإشعاع من `capture_..._raw.tif` يُسمى `capture_..._raw.tif` — إنه موجود فقط في `tiff32/Radiance_Images/`.**المجلد هو ما يحدد المنتج، وليس اسم الملف**، لذا استخدم تعبيرًا عامًا للبحث عن المجلد، وليس عن اللاحقة `*radiance*`.
{% endhint %}

### الخيارات التي ستستخدمها فعليًّا

| العلامة | القيمة الافتراضية | الوظيفة |
| --- | --- | --- |
| `-o, --output PATH` | مجلد المشروع الافتراضي | موقع مجلد المشروع (انظر أعلاه). |
| `-n, --project-name NAME` | الطابع الزمني | اسم المشروع. |
| `--format FMT` | `TIFF (16-bit)` | أحد الخيارات التالية: `TIFF (16-bit)`، `TIFF (8-bit)`، أو `TIFF (32-bit, Percent)`، أو `PNG (8-bit)`، أو `JPG (8-bit)`. لا يقوم Chloros
بزيادة الدقة: في جميع الصور ذات 8 بت، يتم كتابة التنسيق الأعمق على أنه `TIFF (8-bit)`، ويتم ذكر ذلك مرة واحدة. |
| `--indices NAME [NAME ...]` | لا شيء | مؤشرات الغطاء النباتي المراد تصديرها (انظر [مؤشرات الغطاء النباتي](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = إزالة التباين العصبي، أبطأ، أعلى جودة (Chloros
+، يتطلب وحدة معالجة رسومات CUDA: في حالة عدم توفرها، يستخدم التشغيل `standard`؛ يُرفض استخدامه مع لقطات LATTICE، التي تستخدم دائمًا `standard`). |
| `--vignette / --no-vignette` | قيد التشغيل | تصحيح التظليل. |
| `--reflectance / --no-reflectance` | قيد التشغيل | معايرة الانعكاس؛ بالنسبة لـ LATTICE، يُعد هذا أيضًا مفتاح تشغيل/إيقاف منتج الانعكاس. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | فرض نقطة دخول مسار المعالجة لملفات TIFF الخاصة بـ LATTICE. |

بالنسبة لكل شيء آخر — ضبط الكشف عن الهدف، PPK، دبابيس التعريض، علامات محاذاة المصفوفة — راجع [قسم `process` من مرجع CLI
](reference/cli-reference.md).

***

## اختيار ما سيتم تصديره (منتجات LATTICE)

تتفرع معالجة LATTICE إلى **كل منتج قابل للتطبيق في مسار واحد**. أربعة مفاتيح تبديل لكل منتج تكون جميعها**مفعّلة افتراضيًا**؛ استخدم النموذج `--no-` لإلغاء تفعيل أحدها:

| مفتاح التبديل | المنتج |
| --- | --- |
| `--debayered` | إزالة التمويه الخطي → `Debayered_Images/`: إزالة الإزاحة المظلمة لحزمة المعايرة، ثم إزالة التمويه مع مراعاة الحواف، بدون تصحيح الخطية أو تسوية المجال أو فصل الألوان — وهو نفس الملف الذي يحفظه الالتقاط عند مستوى `debayered`. يتم الاحتفاظ به في الاتجاه الأصلي للمستشعر (المسجل في XMP). |
| `--preview` | معاينة العرض (توازن اللون الأبيض + جاما؛ تمديد الألوان الزائفة للصور متعددة الأطياف) → `Preview_Images/` |
| `--radiance` | الإشعاع بنوع float32، و/م²/sr/nm → `Radiance_Images/` (دائمًا `tiff32/`) |
| `--reflectance` | uint16 الانعكاسية، جاهز لـ Pix4D → `Reflectance_Calibrated_Images/` |

الكاميرات الرئيسية RGB
لا تصدر سوى الإشعاع بعد إزالة التباين اللوني + المعاينة — فالإشعاع/الانعكاس لكل نطاق ليس ذا معنى بالنسبة لمستشعر واسع النطاق، لذا فإن خيارات التبديل هذه لا تؤدي إلى أي تأثير بالنسبة لها. Survey3
`.raw` يتجاهل خيارات التبديل ويتبع المسار القياسي للانعكاس/الهدف.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (الافتراضي `auto`) يختار مرجع الانعكاسية: `auto` يجعل [هدف المعايرة](calibration-targets.md) الموجود داخل الإطار والذي اجتاز اختبار ضمان الجودة المرجع المطلق، ويعود إلى تقسيم الضوء الهابط لمستشعر الضوء في نظام جمع البيانات (DAQ) (ρ = π·L/E) في حالة عدم وجود هدف (الهدف الذي يجتاز فحص ضمان الجودة مع وجود علامة يتنافس مع جهاز DAQ في النتيجة المقاسة)؛ `target` صارم (لا يوجد استبدال بجهاز DAQ)؛ `daq` يعتمد على DAQ بشكل حاسم. يمكن توفير مسح الهدف المقاس لكل وحدة باستخدام `--target-reflectance-dir`.

{% hint style="info" %}
**قراءة وحدات البكسل الانعكاسية:**قيمة DN التي تعني ρ = 1.0 هي**لكل مصدر** — تضع ملفات LATTICE العلامة `Chloros:PixelScale=32768` في XMP؛ تستخدم ملفات Survey3
الرقم 65535 ولا تحمل علامة `Chloros:PixelScale` (لكنها تحمل علامات معالجة `Chloros:*` أخرى). اقرأ العلامة وقسم عليها بدلاً من افتراض قيمة ثابتة. توجد التفاصيل والحالة الاستثنائية الوحيدة المتعمدة التي لا تحتوي على مقياس في [مرجع CLI
](reference/cli-reference.md).
{% endhint %}

**تبدأ المعالجة دائمًا من `raw`.** لا يتم إعادة تغذية المنتجات المشتقة (صادرات إزالة التباين/الإشعاع/الانعكاس) عبر مسار المعالجة أبدًا — حيث إن إعادة استيرادها ومعالجتها سيؤدي إلى تطبيق حسابات المعايرة مرتين، لذا يتخطاها برنامج Chloros
ويشير إلى ذلك. `--input-level` هو مخرج الطوارئ المقصود عندما تحتاج فعليًّا إلى فرض نقطة دخول.

***

## عند فشل التشغيل

اعتبارًا من الإصدار 1.2.0، يفشل `process` بشكل واضح بدلاً من «النجاح» دون عرض أي نتائج:

* التشغيل الذي **طلب منتجات ولكنه لم يكتب أيًا منها**— فقط `project.json` و `calibration_data.json` — يُخرج `Processing finished but wrote no image products.` و**يخرج بقيمة غير صفرية**، لذلك يمكن للبرامج النصية اكتشافه. الأسباب المعتادة: لم يتم التعرف على مجلد الإدخال على أنه عملية التقاط (تحقق من التخطيط و`--input-level`)، أو أن كل منتج مطلوب كان غير قابل للتطبيق على تلك الكاميرات (على سبيل المثال، طلب الإشعاع/الانعكاس من كاميرات تعمل بنظام RGB
فقط).
* **التشغيل المتعمد للبيانات الوصفية فقط** (إيقاف تشغيل جميع المنتجات، بدون `--indices`) لا يزال يُعتبر ناجحًا — حيث يُعد إخراج صورة فارغة هو النتيجة الصحيحة في هذه الحالة.
* أعد التشغيل باستخدام `--verbose` وتحقق من سجل الخلفية بحثًا عن الأسطر `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`، التي تشرح حالات التخطي لكل كاميرا.
* في نظام Windows
مع تعطيل المسارات الطويلة، يتم رفض مجلد المشروع الذي يتجاوز حد المسار البالغ 260 حرفًا **قبل** بدء المعالجة، مع استخدام أطول مجلد مشروع يتناسب مع الحد (انظر [مكان تخزين المخرجات](#where-the-outputs-land)).
* تقوم أوامر الالتقاط (`lattice capture` و `lattice array-capture` ومجموعات الالتقاط الأخرى) بتسمية كل ملف لم تتمكن من كتابته ثم تخرج برمز `1`؛ وتتوقف عمليات الالتقاط المتكررة عندما يمتلئ محرك الأقراص.

رموز الخروج: `0` نجاح · `1` فشل عام · `2` خطأ في الوسيطة · `130` تم مقاطعته بواسطة Ctrl+C.

***

## مؤشرات الغطاء النباتي

قم بتشغيل `--indices` مع اسم واحد أو أكثر من الأسماء المحددة مسبقًا؛ حيث يتم حفظ كل مؤشر في مجلده الخاص `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

الأسماء المحددة مسبقًا البالغ عددها 22 التي تقبلها `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

يتم إضافة اسم لم يرد في قائمة المشروع بعد إلى المشروع. أما الاسم الذي ليس من الأسماء المحددة مسبقًا، أو الذي لا تستطيع نطاقات الكاميرات تكوينه، فيتم تخطيه مع ظهور تحذير يحدد اسمه. إذا تعذر على واجهة سطر الأوامر (CLI
) قراءة قائمة الفهرس الحالية للمشروع، فإنها تترك القائمة دون تغيير وتصدر تحذيرًا بدلاً من استبدالها.

{% hint style="warning" %}
**توجد ثلاث قوائم فهرس — لا تخلط بينها.**تحتوي القائمة المنسدلة «إعدادات المشروع» في واجهة المستخدم الرسومية (GUI) على 27 صيغة (تضيف `FCI1`، `FCI2`، `GARI`، `GEMI`، و`LCI` — وهذه الخمس صيغ مخصصة لواجهة المستخدم الرسومية فقط و**ليست** صالحة لـ `--indices`). يستخدم الأمر `lattice index --preset` (المباشر/غير المتصل) قائمة منفصلة خاصة به تضم 22 إعدادًا مسبقًا. تم توثيق الصيغ وحسابات النطاقات في [صيغ المؤشرات متعددة الأطياف](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## مستشعرات الضوء DAQ: لمحة سريعة

تقوم عائلة `daq pool-*` بتشغيل مستشعرات الطيف DAQ من MAPIR
(DAQ-U عبر USB، وDAQ-M عبر BLE، وDAQ-E عبر Ethernet) من خلال المجمع الدائم للخلفية — حيث تشترك واجهة المستخدم الرسومية (GUI) وواجهة سطر الأوامر (CLI
) ومجموعة أدوات التطوير (SDK
) جميعًا في مقبض واحد مباشر. **`pool-*` هو مسار DAQ المدعوم في واجهة CLI
المرفقة**؛ أما الأوامر الفرعية الأخرى لـ `daq` التي قد ترى إشارة إليها فهي واجهة داخلية خاصة بـ MAPIR
مخصصة للمصدر فقط، وتُغلق مع ظهور خطأ صريح يوجهك إلى `pool-*`.

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

`pool-record` بدون `--duration` يعمل حتى `pool-record --stop`؛ دليل الإخراج الافتراضي هو `~/Documents/DAQ Live View/` **على جهاز الخلفية**. يتم اختيار ملف تعريف تصحيح الحد الأقصى عند الاتصال (`--cap-id`، الإعداد الافتراضي للخلفية هو `sunshine_cosine`) ويمكن تبديله أثناء التشغيل باستخدام `pool-set-cap` — يتم تناول ملفات تعريف الحد الأقصى ونطاق المعايرة الخاص بالمستشعر في فصول DAQ من هذا الدليل.

{% hint style="warning" %}
**DAQ-E على مضيف متعدد بطاقات الشبكة (NIC):** قد يفشل الاكتشاف التلقائي الأول لـ `pool-connect --eth` بعد التشغيل حتى مع وجود مستشعر سليم. `--eth-host <ip-or-hostname>` هو الشكل الموثوق به — استخدمه كلما فشل الاكتشاف.
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

إلى جانب ذلك: يقدم `chloros-cli time-sync` تقريرًا عن وحدة التحكم الرئيسية (Grandmaster) لـ PTP التي يشغلها المضيف Chloros
(تتبعها كاميرات LATTICE ومستشعرات DAQ-E كوحدات تابعة للحصول على طوابع زمنية عبر الأجهزة)، ويفتح `chloros-cli project` مشروع Chloros
محفوظًا ويقوم بتشغيل كاميراته ومصفوفاته وأجهزة استشعاره دون واجهة مستخدم — بما في ذلك وصفات التقاط YAML المبرمجة.

هذه المجموعات الثلاث (`lattice`، `project`، `daq pool-*`) هي أيضًا العائلات الوحيدة التي تدعم `CHLOROS_BACKEND_URL` لتشغيل خلفية **بعيدية**؛ حيث تستهدف الأوامر الأساسية دائمًا الجهاز المحلي.

توجد إرشادات تفصيلية كاملة في فصول LATTICE من هذا الدليل؛ وتوجد كل علامة في [مرجع واجهة CLI
](reference/cli-reference.md).

***

## استكشاف الأخطاء وإصلاحها: أهم 5 مشكلات

| الأعراض | الحل |
| --- | --- |
| `Login required`، أو توقف مهمة مجدولة عند موجه `Email:` | قم بتشغيل `chloros-cli login EMAIL 'PASSWORD'` مرة واحدة على هذا الجهاز — فالأوامر التي لا تحتوي على جلسة مخزنة مؤقتًا تعمل بشكل تفاعلي بدلاً من الفشل الفوري. |
| `backend unreachable` | قم بتشغيل تطبيق سطح المكتب Chloros
، أو قم بتشغيل الخلفية بنفسك: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` على نظام Windows
(المجلد `backend` الموجود في مجلد التثبيت الخاص بك؛ لا يوجد في `PATH`)، `chloros-backend` على نظام Linux
. يوضح `chloros-cli backend status` ما إذا كان هناك واحد قيد التشغيل؛ `lattice`/`project`/`daq pool-*` لا تبدأ أي منها أبدًا. إذا قمت بتوجيهها إلى خلفية بعيدة، فتحقق من `CHLOROS_BACKEND_URL`. |
| تم حظر اتصال المصفوفة: `FRAMES WILL DROP` / `Reduce ROI to enable` | تمت إعادة تعيين حلقة الاستقبال في بطاقة الشبكة (NIC) للمضيف إلى الإعدادات الافتراضية — وهو السبب الأول لرفض جهاز كان يعمل سابقًا الاتصال، وعادةً ما يحدث ذلك بعد تحديث برنامج تشغيل بطاقة الشبكة. قم بتشغيل `chloros-cli lattice network --fix` من محطة طرفية **ذات صلاحيات عالية** (أو قم بتعيين `ReceiveBufferLen=256`، `PendingReceives=64`)؛ راجع *إعداد بطاقة شبكة المضيف وضبطها* في المرجع. |
| إنهاء الأمر الفرعي `daq`: «يتطلب حزمة DAQ الكاملة…» | استخدم أوامر `daq pool-*`، التي تتحكم في نفس المستشعر عبر الخلفية: الاتصال، والبث، والتسجيل، واختيار الحد الأقصى، وبالنسبة لـ DAQ-E إعدادات الشبكة، وإعادة التشغيل، وتحديثات البرامج الثابتة (أو `chloros_sdk.connect_daq_sensor()` من Python
). |
| يعرض Jetson تحذيرًا بشأن ذاكرة التبادل قبل المجلدات الكبيرة | أضف ذاكرة تبادل مدعومة بالملفات — تعرض واجهة CLI
الأوامر `fallocate`/`swapon` المحددة التي يجب تشغيلها. |

***

## الحصول على المساعدة

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **كل علامة، كل أمر فرعي:** [مرجع واجهة CLI
](reference/cli-reference.md)
* **المكافئ في Python
:** [Python
SDK
](api-python-sdk.md) و[مرجع SDK
](reference/sdk-reference.md)
* **الدعم:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
