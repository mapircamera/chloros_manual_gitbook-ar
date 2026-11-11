# دليل NVIDIA Jetson يتيح برنامج

Chloros على NVIDIA Jetson معالجة الصور متعددة الأطياف على الحافة — في الميدان، وعلى الطائرات بدون طيار، وفي المنشآت البعيدة. يقوم Chloros 1.3.0 باكتشاف طراز جهاز Jetson الخاص بك عند بدء التشغيل ويُحسّن استراتيجية المعالجة وفقًا للأجهزة التي يكتشفها. **لا يلزم إجراء أي ضبط يدوي.**
{% hint style="info" %}
**تعمل حزمة Jetson 1.3.0 على وحدة معالجة الرسومات (GPU) الخاصة بـ Jetson.** تم تصميم `chloros_1.3.0_arm64_jp6.deb` لـ JetPack 6 ويستخدم إصدار PyTorch المدمج مع CUDA الخاص بـ JetPack 6، لذا فإن المعالجة — بما في ذلك أداة إزالة التباين التي تراعي النسيج (Texture Aware debayer) — تعمل على وحدة معالجة الرسومات (GPU) الخاصة بـ Orin. لا تحتوي الحزمة على CUDA نفسها: فهي تستخدم مكتبات CUDA وcuDNN التي يقوم JetPack 6 بتثبيتها (`sudo apt install nvidia-jetpack` على صورة تفتقر إلى هاتين المكتبتين). وفي حالة عدم توفر وحدة معالجة رسومات (GPU) تدعم CUDA، تتم المعالجة على وحدة المعالجة المركزية (CPU) ويستخدم Texture Aware أداة إزالة التباين القياسية (Standard debayer).
{% endhint %}

***

## طرازات Jetson المدعومة

| الطراز                | ذاكرة الوصول العشوائي (RAM)            | استراتيجية المعالجة                                     | الاستخدام الموصى به                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 جيجابايت مشتركة | `GPU_PARALLEL` (2 وحدة عمل)                              | أقصى أداء، مجموعات بيانات كبيرة                      |
| **Jetson Orin NX**   | 8-16 جيجابايت مشتركة  | `GPU_PARALLEL` (وحدتا عمل، 16 جيجابايت) / `GPU_SINGLE` (8 جيجابايت)   | التوصية الأساسية للنشر الميداني              |
| **Jetson Orin Nano** | 8 جيجابايت مشتركة     | `GPU_SINGLE` (وحدة عمل واحدة، تسلسلي)                     | الحوسبة الطرفية للمبتدئين                                 |

{% hint style="info" %}
تتطلب حزمة Linux arm64 **JetPack 6**، المتوفرة على عائلة Jetson Orin. لا يمكن للطرازات الأقدم (Nano، TX2، Xavier NX) تشغيل JetPack 6 ولا تدعمها الحزمة الحالية.
{% endhint %}

***

## المتطلبات

* **JetPack 6.x** (يوصى باستخدام أحدث إصدار)
* **NVIDIA CUDA و cuDNN** من JetPack 6 (`sudo apt install nvidia-jetpack` إذا كانت الصورة الخاصة بك تفتقر إليهما)
* **خطة Chloros+ المدفوعة** — فئة Copper أو أعلى (مطلوبة لجميع عمليات الوصول إلى CLI/SDK؛ يتم فرضها من جانب الخادم)

## التثبيت

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

يقوم `.deb` بتثبيت SDK لـ Python الخاص بالنظام `python3`، المطابق لهذا الإصدار. يحتاج virtualenv فقط إلى تثبيت ملف wheel المرفق بداخله: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (يعمل دون اتصال بالإنترنت تمامًا). للحصول على تفاصيل التثبيت العامة لنظام Linux، ومواقع الملفات، واستكشاف الأخطاء وإصلاحها، راجع [التثبيت على Linux](linux-installation.md).

{% hint style="info" %}
**لم يعد هناك حاجة إلى فك الضغط عند التشغيل.** اعتبارًا من الإصدار 1.3.0، يتم تثبيت الخلفية وواجهة سطر الأوامر (CLI) دون فك الضغط ضمن `/usr/lib/chloros/`، لذا لم تعد تُفك ضغطها تلقائيًا إلى دليل مؤقت في كل مرة يتم فيها تشغيلها. لا تزال الملفات المؤقتة تُنقل تلقائيًا إلى `/mnt/ssd/tmp` عند وجود هذا المجلد؛ وإلا يمكنك تعيين `TMPDIR` إلى مسار على محرك NVMe الخاص بك (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## التكيف الحسابي الديناميكي على Jetson

### كيف يعمل

عند بدء التشغيل، يقوم Chloros بتحديد ملامح نظامك:

1. **يكتشف طراز Jetson** عبر `/proc/device-tree/model`
2. **يقرأ الذاكرة المشتركة المتاحة لوحدة معالجة الرسومات (GPU) ووحدة المعالجة المركزية (CPU)** (يستخدم Jetson ذاكرة موحدة)
3. **يسأل PyTorch عما إذا كان بإمكانه استخدام CUDA** — مع تثبيت CUDA وcuDNN من JetPack 6، يمكنه ذلك
4. **يختار استراتيجية المعالجة** (`GPU_PARALLEL`، أو `GPU_SINGLE`، أو `CPU_PARALLEL`) وعدد العمال تلقائيًا

يتم اتخاذ القرار بناءً على **إجمالي ذاكرة الوصول العشوائي (RAM) المشتركة**، وليس بناءً على اسم النموذج:

* **عندما يقل إجمالي ذاكرة الوصول العشوائي (RAM) عن 12 جيجابايت**(جميع أجهزة Jetson بسعة 8 جيجابايت):**عامل واحد — معالجة تسلسلية متعمدة**. الذاكرة غير كافية لتشغيل عمال متزامنين، لذا تتم معالجة الصور واحدة تلو الأخرى. في أجهزة Jetson ذات**ذاكرة وصول عشوائي (RAM) تبلغ 8 جيجابايت أو أقل**، يتخطى الخيط 3 مجموعة العمال تمامًا ويقوم بتنفيذ العمل الخاص بكل صورة أثناء المعالجة. ويُطلق سجل الخلفية على هذه الاستراتيجية اسم `GPU_SINGLE`.
* **12 جيجابايت أو أكثر**(Orin NX 16 جيجابايت، AGX Orin): تتوافق الذاكرة الموحدة مع `GPU_PARALLEL`، لكن**عدد العمال محدد بـ 2 على أجهزة Jetson** — حيث تعتمد وحدة معالجة الرسومات (GPU) وذاكرة الوصول العشوائي (RAM) لعمليات العمال وسياقات CUDA لكل عامل على نفس المجموعة المشتركة، لذا فإن زيادة عدد العمال تزيد من خطر حدوث أعطال بسبب نفاد الذاكرة.
* **عدم توفر CUDA قابلة للاستخدام** (مكتبات CUDA الخاصة بـ JetPack مفقودة): `CPU_PARALLEL` عند 12 جيجابايت أو أكثر، صورة واحدة في كل مرة أدناه.

يمكنك تجاوز الاختيار التلقائي باستخدام متغير البيئة `CHLOROS_STRATEGY` — انظر [التكيف الحسابي الديناميكي](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### السلوك حسب النموذج

| نموذج Jetson                | الاستراتيجية       | العمال | التنفيذ                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | حلقة تسلسلية داخل العملية (`tiled_gpu` في حالة ضغط الذاكرة) |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | حلقة تسلسلية داخل العملية                     |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 2       | عمليات عاملة متزامنة، مسار `fused_gpu`  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2       | عمليات عاملة متزامنة، مسار `fused_gpu`  |

يتمثل الاختلاف الرئيسي بين المنصات في **الذاكرة**. يجب على جهاز Jetson بسعة 8GB معالجة الصور واحدة تلو الأخرى باستخدام نهج التبليط الموفر للذاكرة عندما يكون الضغط مرتفعًا، بينما يمكن لجهاز Orin بسعة 16GB+ تشغيل صورتين عبر وحدة معالجة الرسومات (GPU) في وقت واحد باستخدام خط الأنابيب المدمج ذي الإنتاجية الأعلى.

### ميزانية وحدة معالجة الرسومات (GPU) لكل طراز

يحتوي كل طراز من طرازات Jetson أيضًا على ملف تعريف للأجهزة يحدد الحد الأقصى لمقدار المعالجة التي يمكن أن تستهلكها المجمعة المشتركة، ويقوم بتحجيم أحجام الدُفعات:

| الطراز | الحد الأقصى لميزانية وحدة معالجة الرسومات (GPU) | مضاعف حجم الدُفعة | المخصص للنظام/الشاشة |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0.8 | 2.0 جيجابايت |
| **Jetson Orin NX** | 75% | ×1.0 | 3.0 جيجابايت |
| **Jetson AGX Orin** | 80% | ×1.5 | 4.0 GB |

تقوم ذاكرة الوصول العشوائي (RAM) المكتشفة بضبط ملف التعريف: ففي حالة جهاز Jetson الذي يُبلغ عن **16GB أو أكثر**، يتم رفع مضاعف الدفعة إلى ×1.2. يبلغ حجم الدفعة الأساسي قبل تطبيق المضاعفات 8 صور.

للاطلاع على المرجع الكامل لتكييف الحوسبة، انظر [التكييف الديناميكي للحوسبة](../processing-architecture/dynamic-compute-adaptation.md).

***

## الحد الأقصى لتردد وحدة معالجة الرسومات (GPU) لـ Texture Aware على Nano و Orin Nano

يقوم برنامج إزالة التباين في Texture Aware بتشغيل استدلال الشبكة العصبية على وحدة معالجة الرسومات (GPU)، مما قد يؤدي إلى **تحذيرات من زيادة التيار**في طرازات Jetson منخفضة الطاقة (فئة 10-15 واط) عند سرعة ساعة وحدة معالجة الرسومات (GPU) القصوى. عند تمرير `--debayer texture-aware` على**Jetson Nano أو Orin Nano**، تتحقق واجهة CLI من الحد الأقصى لتردد وحدة معالجة الرسومات (GPU) وتحدده بـ **510 ميجاهرتز** (510000000) إذا كان التردد الحالي أعلى من ذلك:

* إذا كان بإمكان واجهة الأوامر (CLI) الكتابة في عقدة sysfs الخاصة بتردد وحدة معالجة الرسومات (GPU)، يتم **تطبيق الحد الأقصى تلقائيًا** ويتم عرض رسالة تأكيد.
* إذا لم تتمكن (تتطلب صلاحيات الجذر)، فإن واجهة CLI تُعرض الأمر `sudo` بالضبط لتطبيق الحد يدويًّا، وتنتظر لحظة حتى تتمكن من قراءته، ثم تواصل العمل — تستمر المعالجة ولكن قد تظهر تحذيرات من زيادة التيار.

لتطبيق الحد الأقصى بنفسك قبل المعالجة:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

تعمل الطرز ذات الطاقة الأعلى (Orin NX 25W، AGX Orin 60W) بأقصى سرعة لـ GPU؛ ولا يتم تطبيق أي حد أقصى. لا يقوم أبدًا أداة إزالة التباين القياسية (Standard debayer) بتفعيل الحد الأقصى على أي طراز.

{% hint style="info" %}
**يعمل «Texture Aware» على Jetson دائمًا على صورة واحدة في كل مرة.** سيحتاج كل عامل إلى سياق CUDA خاص به (~1 جيجابايت) بالإضافة إلى نسخته الخاصة من نموذج إزالة الضوضاء، وهو ما لا تستطيع الذاكرة الموحدة تحمله — لذا على Jetson، يتم تثبيت مسار Texture Aware على عامل واحد مع تسلسل الوصول إلى وحدة معالجة الرسومات (GPU). توقع أن يكون Texture Aware أبطأ بشكل ملحوظ من Standard على أي جهاز Jetson. يُطبق «Texture Aware» على صور Survey3: تحصل لقطات LATTICE دائمًا على عملية إزالة التباين القياسية (Standard debayer). في حالة عدم توفر وحدة معالجة رسومات (GPU) تدعم CUDA قابلة للاستخدام، يستخدم التشغيل الوضع القياسي (Standard)، وتسجل الخلفية سطرًا واحدًا في السجل بـ `[DEBAYER] … needs a CUDA GPU …` يوضح السبب (كما هو موضح بـ `chloros-cli --verbose`).
{% endhint %}

***

## إدارة الحرارة

تتمتع أجهزة Jetson بقدرة حرارية محدودة، خاصةً في حالات النشر في الأماكن المغلقة أو المحمولة جوًّا. يقوم برنامج Chloros بمراقبة درجة حرارة نظام SoC وتقليل أحجام الدُفعات تلقائيًّا:

| درجة الحرارة         | الإجراء                                            |
| ------------------- | ------------------------------------------------- |
| **< 70°C**          | التشغيل العادي — سرعة معالجة كاملة          |
| **70°C** (تحذير)  | يتم تقليص حجم الدُفعات تدريجيًا (100% → 50% بين 70°C و80°C) |
| **80°C** (حالة حرجة) | تقييد شديد (50% → 0% بين 80°C و90°C) |
| **90°C** (إيقاف التشغيل) | إيقاف معالجة وحدة معالجة الرسومات (GPU) تمامًا — يلزم التبريد |

{% hint style="warning" %}
**تأكد من توفر تهوية كافية وتبديد حراري** من أجل معالجة مستمرة، خاصةً في العلب الميدانية المغلقة أو الأنظمة المحمولة جوًا. سيؤدي التخفيض الحراري إلى تقليل معدل إنتاجية المعالجة لحماية الأجهزة.
{% endhint %}

***

## إدارة الذاكرة

تستخدم أجهزة Jetson **ذاكرة موحدة** — حيث تشترك وحدة معالجة الرسومات (GPU) ووحدة المعالجة المركزية (CPU) في نفس ذاكرة الوصول العشوائي (RAM) الفعلية. ذاكرة VRAM المُبلغ عنها (على سبيل المثال ~15.3GB على جهاز Orin NX 16GB) ليست ذاكرة مخصصة لوحدة معالجة الرسومات (GPU)؛ بل هي نفس ذاكرة الوصول العشوائي (RAM) التي يستخدمها نظام التشغيل وجميع العمليات الأخرى.

### تحذير بشأن المبادلة والتوصيات

قبل المعالجة على Jetson، تقوم واجهة سطر الأوامر (CLI) بحساب عدد الصور RAW الموجودة في مجلد الإدخال الخاص بك (`.tif`، `.tiff`، `.raw`، `.dng` — لا يتم احتساب معاينات JPG)، ويقدّر الحد الأقصى للذاكرة التي يحتاجها التشغيل، وي**حذر قبل البدء** إذا كان من المحتمل أن تكون ذاكرة الوصول العشوائي (RAM) + مساحة التبادل غير كافية. يحمل التحذير العنوان `LOW MEMORY WARNING - Jetson Detected`، ويعرض عدد الصور، ذاكرة الوصول العشوائي (RAM)، ومساحة التخزين المؤقت الحالية، والذروة المقدرة، ثم يعطي الأوامر الدقيقة `fallocate` / `chmod` / `mkswap` / `swapon` المحددة الحجم لمشروعك مشروعك (لا يقل حجمها أبدًا عن 8 جيجابايت). يتوقف البرنامج لبضع ثوانٍ حتى لا تضيع الرسالة عند التمرير للخلف، ثم تستمر المعالجة.**تقديرات الذاكرة المستخدمة في التحذير:**

| وضع إزالة التباين | الأساسي | لكل صورة |
| --- | --- | --- |
| قياسي | ~1.5 جيجابايت | ~10 ميجابايت |
| مراعٍ للنسيج | ~2.5 جيجابايت (النموذج + وقت تشغيل Python) | ~15 ميجابايت |

يتم تشغيل التحذير عندما يتجاوز الذروة المقدرة سعة ذاكرة الوصول العشوائي (RAM) + مساحة التبادل (swap) مطروحًا منها هامش أمان قدره 1 جيجابايت، ولا يحسب سوى مساحة التبادل **المدعومة بالملفات** — وسيظل الإعداد الذي يعتمد على zram فقط محددًا.

لإضافة مساحة التبادل يدويًّا (مثال: 8 جيجابايت):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### معالجة OOM (نفاد الذاكرة)

أثناء المعالجة، يراقب Chloros ذاكرة وحدة معالجة الرسومات (GPU) ويقوم بتقليل الأداء بشكل سلس بدلاً من التعطل:

1. عندما يتجاوز استخدام ذاكرة وحدة معالجة الرسومات (GPU) **85%**، يتم تقليل أحجام الدُفعات بشكل استباقي
2. إذا استمر حدوث حالة نفاد الذاكرة، يتم **تقليل حجم الدُفعة إلى النصف**، ثم تقليله إلى النصف مرة أخرى عند كل حالة OOM متتالية؛ وتؤدي كل دُفعة ناجحة لاحقة إلى التراجع عن هذه العقوبة خطوة واحدة
3. في ظل الضغط المستمر، يتحول مسار المعالجة من `fused_gpu` إلى مسار `tiled_gpu` الأكثر كفاءة في استخدام الذاكرة، ثم إلى المعالجة عبر وحدة المعالجة المركزية (CPU) كحل أخير

نظرًا لأن وحدة معالجة الرسومات (GPU) تشترك في ذاكرة الوصول العشوائي (RAM) الخاصة بجهاز Jetson، فإن ذاكرة الوصول العشوائي (RAM) بالإضافة إلى مساحة التخزين المؤقتة (swap) لا تزال تحدد حدود التشغيل — ويجب الانتباه إلى تحذير مساحة التخزين المؤقتة المذكور أعلاه.

***

## النشر الميداني

### اعتبارات الطاقة

| طراز Jetson     | استهلاك الطاقة النموذجي | ملاحظات                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15 واط              | مقبس تيار مستمر أسطواني          |
| Jetson Orin NX   | 10-25 واط             | مقبس تيار مستمر أسطواني          |
| Jetson AGX Orin  | 15-60 واط             | USB-C PD أو مقبس تيار مستمر أسطواني |

خطط لميزانية الطاقة الخاصة بك من أجل معالجة مستمرة — يحدث استهلاك الطاقة الأقصى أثناء الخيط 3 (المعالجة) الذي يعتمد بشكل مكثف على وحدة معالجة الرسومات (GPU).

### توصيات التخزين

* **محرك أقراص SSD NVMe** موصى به بشدة لنشر أنظمة arm64
* بطاقات SD بطيئة جدًا للمعالجة — استخدمها كوسائط تمهيد فقط
* خطط لحجم يبلغ 2-3 أضعاف حجم بيانات الصور الأولية للناتج المعالج

### التشغيل بدون شاشة عبر SSH
تعد واجهة

Chloros الخاصة بـ CLI مثالية لعمليات نشر Jetson بدون شاشة:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### الخلفية التي تعمل دائمًا لمزامنة الوقت في LATTICE / DAQ-E

إذا كان جهاز Jetson الخاص بك يتحكم في كاميرات LATTICE أو مستشعرات الضوء DAQ-E بدون شاشة، فقم بتمكين خدمة systemd الخلفية **بصفتك المستخدم** حتى يعمل جهاز PTP grandmaster بشكل مستمر وتجد نصوص SDK الخلفية قيد التشغيل (الوحدة مثبتة ولكنها غير ممكّنة افتراضيًا):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

وعندئذٍ تعمل الخلفية بصفتك أنت: فاللقطات التي تسجلها تخصك، وتستخدم `chloros-cli login` الخاص بك. انظر [تثبيت Linux](linux-installation.md#always-on-ptp-for-headless-hosts) للحصول على التفاصيل، بما في ذلك حساب الجذر `chloros-backend.service` وكيفية قيام الحزمة بجعل منافذ PTP 319/320 قابلة للربط دون الحاجة إلى صلاحيات الجذر.

### المعالجة الآلية باستخدام systemd

قم بإنشاء خدمة systemd للمعالجة الآلية:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

تُخرج `chloros-cli process` قيمة غير صفرية عندما لا تكتب عملية التشغيل التي طلبت المنتجات أي صور، لذا فإن حالة الفشل في systemd مفيدة للمراقبة.

اقترن بمؤقت systemd للمعالجة المجدولة:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## أمثلة على سير العمل

### المعالجة الأساسية على Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI
```

### PythonSDK
على Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### المعالجة المجمعة لرحلات طيران متعددة

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## أنظمة Jetson الموصى بها للاستخدام الميداني

بالنسبة لعمليات النشر الميدانية والجوية، ضع في اعتبارك خيارات لوحات الحامل Jetson Orin NX سعة 16 جيجابايت التالية:

* **الاستخدام الجوي/الطائرات بدون طيار**: أنظمة ذات تصنيف مقاومة للاهتزاز (MIL-STD)، وخفيفة الوزن (أقل من 300 جرام)، وتبريد سلبي
* **البيئات الميدانية القاسية**: علب مقاومة للماء بمعيار IP67/IP69K مزودة بقدرة توصيل الكاميرات عبر PoE GigE
* **الخيارات البسيطة/المقتصدة**: مجموعات المطورين المزودة بعلب إضافية

اتصل بـ [دعم MAPIR](https://www.mapir.camera/community/contact) للحصول على توصيات محددة بشأن الأجهزة المناسبة لسيناريو النشر الخاص بك.

***

## الخطوات التالية

* [تثبيت Linux](linux-installation.md) — تفاصيل عامة حول تثبيت Linux

* [التكيف الحسابي الديناميكي](../processing-architecture/dynamic-compute-adaptation.md) — مرجع كامل لاستراتيجيات الحوسبة
* [مسار المعالجة](../processing-architecture/processing-pipeline.md) — فهم مسار المعالجة ذي الخيوط الأربعة
* [CLI: سطر الأوامر](../CLI.md) — دليل CLI

* [API: Python SDK](../api-python-sdk.md) — دليل SDK

* [مرجع CLI](../reference/cli-reference.md) و[مرجع SDK](../reference/sdk-reference.md) — قوائم شاملة للأوامر/API للإصدار 1.3.0
