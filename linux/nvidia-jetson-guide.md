# دليل NVIDIA Jetson

يتيح برنامج Chloros على NVIDIA Jetson معالجة الصور متعددة الأطياف في الأجهزة الطرفية — في الميدان، وعلى الطائرات بدون طيار، وفي المنشآت البعيدة. Chloros 1.2.0 يكتشف طراز Jetson الخاص بك عند بدء التشغيل ويحسن استراتيجية المعالجة وفقًا للأجهزة التي يعثر عليها. **لا حاجة إلى ضبط يدوي.**

***

## طرازات Jetson المدعومة

| الطراز                | ذاكرة الوصول العشوائي (RAM)            | استراتيجية المعالجة                                     | الاستخدام الموصى به                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 جيجابايت مشتركة | `GPU_PARALLEL` (2 وحدة عمل)                              | أقصى أداء، مجموعات بيانات كبيرة                      |
| **Jetson Orin NX**   | 8-16 جيجابايت مشتركة | `GPU_PARALLEL` (وحدتان عاملتان، 16 جيجابايت) / `GPU_SINGLE` (8 جيجابايت)   | التوصية الأساسية للنشر الجوي والميداني |
| **Jetson Orin Nano** | 8 جيجابايت مشتركة     | `GPU_SINGLE` (وحدة عمل واحدة، تسلسلي)                     | حوسبة الحافة للمبتدئين                                 |

{% hint style="info" %}
تتطلب حزمة Linux arm64 **JetPack 6**، المتوفرة على عائلة Jetson Orin. لا يمكن للطرازات الأقدم (Nano، TX2، Xavier NX) تشغيل JetPack 6 ولا تدعمها الحزمة الحالية.
{% endhint %}

***

## المتطلبات

* **JetPack 6.x** (يوصى باستخدام أحدث إصدار)
* **NVIDIA CUDA** (مضمنة مع JetPack)
* **خطة Chloros+ المدفوعة** — فئة Copper أو أعلى (مطلوبة لجميع عمليات الوصول إلى CLI/SDK؛ يتم فرضها من جانب الخادم)

## التثبيت

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

للحصول على تفاصيل عامة حول تثبيت Linux ومواقع الملفات واستكشاف الأخطاء وإصلاحها، راجع [تثبيت Linux](linux-installation.md).

{% hint style="info" %}
**ضع دليل الاستخراج على وحدة تخزين سريعة.** تقوم الملفات الثنائية المُجمَّعة بفك ضغط نفسها إلى دليل مؤقت عند كل تشغيل — وهو أمر بطيء للغاية عند استخدام بطاقة SD. يستخدم Chloros `/mnt/ssd/tmp` تلقائيًا عند وجوده؛ وإلا فقم بتعيين `TMPDIR` إلى مسار على محرك NVMe الخاص بك (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## التكيف الحسابي الديناميكي على Jetson

### كيف يعمل

عند بدء التشغيل، يقوم Chloros بتحديد خصائص نظامك:

1. **يكتشف طراز Jetson** عبر `/proc/device-tree/model`
2. **يقرأ ذاكرة GPU/CPU المشتركة المتاحة** (يستخدم Jetson ذاكرة موحدة)
3. **يختار استراتيجية معالجة** (`GPU_PARALLEL`، أو `GPU_SINGLE`، أو `CPU_PARALLEL`)
4. **يحدد عدد العمال ونوع مسار المعالجة وتخصيص الذاكرة** تلقائيًا

يتم اتخاذ القرار بناءً على **إجمالي ذاكرة الوصول العشوائي (RAM) المشتركة**، وليس بناءً على اسم النموذج:

* **عندما يقل إجمالي ذاكرة الوصول العشوائي (RAM) عن 12 جيجابايت**(جميع أجهزة Jetson بسعة 8 جيجابايت): `GPU_SINGLE` مع**عامل واحد — معالجة تسلسلية متعمدة**. الذاكرة غير كافية للعاملين المتزامنين، لذا تتم معالجة الصور واحدة تلو الأخرى. في أجهزة Jetson ذات**ذاكرة وصول عشوائي (RAM) تبلغ 8 جيجابايت أو أقل**، يتخطى الخيط 3 مجموعة العمال تمامًا ويقوم بتنفيذ عمله لكل صورة أثناء المعالجة.
* **12 جيجابايت أو أكثر**(Orin NX 16 جيجابايت، AGX Orin): الذاكرة الموحدة مؤهلة لاستخدام `GPU_PARALLEL`، ولكن**يقتصر عدد العمال على 2 في أجهزة Jetson** — حيث تعتمد وحدة معالجة الرسومات (GPU) وذاكرة الوصول العشوائي (RAM) لعمليات العمال وسياقات CUDA لكل عامل على نفس المجموعة المشتركة، لذا فإن زيادة عدد العمال قد تؤدي إلى حدوث أعطال بسبب نفاد الذاكرة.

يمكنك تجاوز الاختيار التلقائي باستخدام متغير البيئة `CHLOROS_STRATEGY` — انظر [التكيف الحسابي الديناميكي](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### السلوك لكل نموذج

| نموذج Jetson                | الاستراتيجية       | عدد العمال | التنفيذ                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | حلقة متسلسلة داخل العملية (`tiled_gpu` في حالة ضغط الذاكرة) |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | حلقة تسلسلية داخل العملية                     |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 2       | عمليات عاملة متزامنة، مسار `fused_gpu`  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2       | عمليات عاملة متزامنة، مسار `fused_gpu`  |

الفرق الرئيسي بين المنصات هو **الذاكرة**. يجب على جهاز Jetson بسعة 8GB معالجة الصور واحدة تلو الأخرى باستخدام نهج متجاور موفر للذاكرة عندما يكون الضغط مرتفعًا، بينما يمكن لجهاز Orin بسعة 16GB+ تشغيل صورتين عبر وحدة معالجة الرسومات (GPU) في وقت واحد باستخدام خط الأنابيب المدمج ذي الإنتاجية الأعلى.

### ميزانية وحدة معالجة الرسومات (GPU) لكل طراز

يحتوي كل طراز من طرازات Jetson أيضًا على ملف تعريف للأجهزة يحدد الحد الأقصى لمقدار المعالجة التي يمكن أن تستهلكها المجمعة المشتركة، ويقوم بتحجيم أحجام الدُفعات:

| الطراز | الحد الأقصى لميزانية وحدة معالجة الرسومات (GPU) | مضاعف حجم الدُفعة | المخصص للنظام/الشاشة |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0.8 | 2.0 جيجابايت |
| **Jetson Orin NX** | 75% | ×1.0 | 3.0 جيجابايت |
| **Jetson AGX Orin** | 80% | ×1.5 | 4.0 GB |

تقوم ذاكرة الوصول العشوائي (RAM) المكتشفة بتعديل ملف التعريف: في حالة جهاز Jetson الذي يُبلغ عن **16GB أو أكثر**، يتم رفع مضاعف الدُفعات الخاص به بمقدار ×1.2. حجم الدفعة الأساسي قبل تطبيق المضاعفات هو 8 صور.

للاطلاع على المرجع الكامل لتكييف الحوسبة، انظر [التكييف الديناميكي للحوسبة](../processing-architecture/dynamic-compute-adaptation.md).

***

## الحد الأقصى لتردد وحدة معالجة الرسومات (GPU) لميزة «Texture Aware» على Nano وOrin Nano

تقوم عملية إزالة التباين «Texture Aware» بتشغيل استدلال الشبكة العصبية على وحدة معالجة الرسومات (GPU)، مما قد يؤدي إلى ظهور **تحذيرات من زيادة التيار**على طرازات Jetson منخفضة الطاقة (فئة 10-15 واط) عند سرعة ساعة وحدة معالجة الرسومات (GPU) القصوى. قبل بدء معالجة «Texture Aware» على**Jetson Nano أو Orin Nano**، يقوم Chloros بالتحقق من الحد الأقصى لتردد وحدة معالجة الرسومات (GPU) وتحديد سقفه عند**510 ميجاهرتز** (510000000) إذا كان أعلى من ذلك حاليًا:

* إذا تمكنت CLI من الكتابة في عقدة sysfs الخاصة بتردد وحدة معالجة الرسومات (GPU)، يتم **تطبيق الحد الأقصى تلقائيًا** ويتم عرض رسالة تأكيد.
* إذا لم يكن الأمر كذلك (يتطلب صلاحيات الجذر)، فإن CLI يعرض الأمر الدقيق `sudo` لتطبيق الحد يدويًّا، وينتظر لحظة حتى تتمكن من قراءته، ثم يواصل العمل — تستمر المعالجة ولكن قد تظهر تحذيرات من زيادة التيار.

لتطبيق الحد الأقصى بنفسك قبل المعالجة:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

تعمل الطرز ذات الطاقة الأعلى (Orin NX 25W، AGX Orin 60W) بأقصى سرعة لوحدة معالجة الرسومات (GPU)؛ ولا يتم تطبيق أي حد أقصى. ولا يقوم أداة إزالة التباين القياسية (Standard debayer) أبدًا بتفعيل الحد الأقصى على أي طراز.

{% hint style="info" %}
**يعمل مسار «Texture Aware» على Jetson دائمًا على صورة واحدة في كل مرة.** سيحتاج كل عامل إلى سياق CUDA خاص به (~1 جيجابايت) بالإضافة إلى نسخته الخاصة من نموذج إزالة الضوضاء، وهو ما لا تستطيع الذاكرة الموحدة تحمله — لذا على جهاز Jetson، يتم تثبيت مسار «Texture Aware» على عامل واحد مع تسلسل الوصول إلى وحدة معالجة الرسومات (GPU). توقع أن يكون أداء «Texture Aware» أبطأ بشكل ملحوظ من «Standard» على أي جهاز Jetson.
{% endhint %}

***

## إدارة الحرارة

تتمتع أجهزة Jetson بقدرة حرارية محدودة، خاصةً في حالات النشر في الأماكن المغلقة أو المحمولة جوًا. يقوم Chloros بمراقبة درجة حرارة SoC وتقليل أحجام الدُفعات تلقائيًا:

| درجة الحرارة         | الإجراء                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | التشغيل العادي — سرعة معالجة كاملة          |
| **70°C** (تحذير)  | يتم تقليص حجم الدُفعات تدريجيًا (من 100% إلى 50% بين 70°C و80°C) |
| **80°C** (حالة حرجة) | تقييد شديد (50% → 0% بين 80°C و 90°C) |
| **90°C** (إيقاف التشغيل) | إيقاف معالجة وحدة معالجة الرسومات (GPU) تمامًا — يلزم التبريد |

{% hint style="warning" %}
**تأكد من توفر تهوية كافية وتبديد حراري** لضمان استمرار المعالجة، خاصةً في العلب الميدانية المغلقة أو الأنظمة المحمولة جوًّا. سيؤدي التخفيض الحراري إلى تقليل معدل إنتاجية المعالجة لحماية الأجهزة.
{% endhint %}

***

## إدارة الذاكرة

تستخدم أجهزة Jetson **ذاكرة موحدة** — حيث تشترك وحدة معالجة الرسومات (GPU) ووحدة المعالجة المركزية (CPU) في نفس ذاكرة الوصول العشوائي (RAM) الفعلية. ذاكرة VRAM المُبلغ عنها (على سبيل المثال ~15.3 جيجابايت في جهاز Orin NX سعة 16 جيجابايت) ليست ذاكرة مخصصة لوحدة معالجة الرسومات (GPU)؛ بل هي نفس ذاكرة الوصول العشوائي (RAM) التي يستخدمها نظام التشغيل وجميع العمليات الأخرى.

### تحذير بشأن المبادلة والتوصيات

قبل المعالجة على Jetson، يقوم CLI بحساب عدد الصور RAW الموجودة في مجلد الإدخال الخاص بك (`.tif`، `.tiff`، `.raw`، `.dng` — لا يتم احتساب معاينات JPG)، ويقدّر الحد الأقصى للذاكرة التي يحتاجها التشغيل، وي**حذّر قبل البدء** إذا كان من المحتمل أن تكون ذاكرة الوصول العشوائي (RAM) + مساحة التبادل غير كافية. يحمل التحذير العنوان `LOW MEMORY WARNING - Jetson Detected`، ويعرض عدد الصور، وذاكرة الوصول العشوائي (RAM)، ومساحة التخزين المؤقت الحالية، والحد الأقصى التقديري، ثم يقدم الأوامر `fallocate` / `chmod` / `mkswap` / `swapon` المصممة خصيصًا لمشروعك (ولا تقل أبدًا عن 8 جيجابايت). يتوقف البرنامج لبضع ثوانٍ حتى لا تضيع الرسالة عند التمرير للخلف، ثم تستمر المعالجة.**تقديرات الذاكرة المستخدمة في التحذير:**

| وضع إزالة التباين | الأساسي | لكل صورة |
| --- | --- | --- |
| قياسي | ~1.5 جيجابايت | ~10 ميجابايت |
| مع مراعاة النسيج | ~2.5 جيجابايت (النموذج + وقت تشغيل Python) | ~15 ميغابايت |

يتم إصدار التحذير عندما يتجاوز الذروة المقدرة ذاكرة الوصول العشوائي (RAM) + مساحة التبديل ناقص هامش أمان قدره 1 جيجابايت، ولا يحسب سوى مساحة التبديل **المدعومة بالملفات** — وسيتم الإبلاغ عن الإعداد الذي يعتمد على zram فقط.

لإضافة مساحة التبادل يدويًّا (مثال: 8 جيجابايت):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

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
```### معالجة OOM (نفاد الذاكرة)

أثناء المعالجة، يراقب Chloros ذاكرة وحدة معالجة الرسومات (GPU) ويقوم بتقليل الأداء بشكل سلس بدلاً من التعطل:

1. عندما يتجاوز استخدام ذاكرة وحدة معالجة الرسومات (GPU) **85%**، يتم تقليل أحجام الدُفعات بشكل استباقي
2. إذا استمر حدوث حالة نفاد الذاكرة، يتم **تقليل حجم الدُفعة إلى النصف**، ثم تقليله إلى النصف مرة أخرى عند كل حالة نفاد ذاكرة متتالية؛ وتؤدي كل دُفعة ناجحة لاحقة إلى التراجع عن هذه العقوبة بخطوة واحدة
3. في ظل الضغط المستمر، يتراجع مسار المعالجة من `fused_gpu` إلى مسار `tiled_gpu` الأكثر كفاءة في استخدام الذاكرة، ثم إلى المعالجة بواسطة وحدة المعالجة المركزية (CPU) كحل أخير

***

## النشر الميداني

### اعتبارات الطاقة

| طراز Jetson     | استهلاك الطاقة النموذجي | ملاحظات                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15 واط              | مقبس أسطواني تيار مستمر          |
| Jetson Orin NX   | 10-25 واط             | مقبس أسطواني تيار مستمر          |
| Jetson AGX Orin  | 15-60 واط             | USB-C PD أو مقبس أسطواني |

خطط لميزانية الطاقة الخاصة بك من أجل معالجة مستمرة — يحدث استهلاك الطاقة الأقصى أثناء الخيط 3 (المعالجة) الذي يعتمد بشكل مكثف على وحدة معالجة الرسومات (GPU).

### توصيات التخزين

* **محرك الأقراص الصلبة SSD NVMe** موصى به بشدة لنشر أنظمة arm64
* بطاقات SD بطيئة جدًا للمعالجة — استخدمها كوسائط تمهيد فقط
* خصص مساحة تبلغ 2-3 أضعاف حجم بيانات الصورة الأولية للمخرجات المعالجة

### التشغيل بدون شاشة عبر SSH

يعد Chloros وCLI مثاليين لعمليات نشر Jetson بدون شاشة:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### الخلفية التي تعمل دائمًا لمزامنة الوقت في LATTICE / DAQ-E

إذا كان جهاز Jetson الخاص بك يتحكم في كاميرات LATTICE أو مستشعرات الضوء DAQ-E دون شاشة، فقم بتمكين خدمة systemd الخلفية حتى يعمل جهاز PTP grandmaster بشكل مستمر (الوحدة مثبتة ولكنها غير ممكّنة افتراضيًا):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

انظر [تثبيت Linux](linux-installation.md#always-on-ptp-for-headless-hosts) للحصول على التفاصيل، بما في ذلك كيفية قيام الحزمة بجعل منافذ PTP 319/320 قابلة للربط دون الحاجة إلى صلاحيات الجذر.

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

تُرجع `chloros-cli process` قيمة غير صفرية عندما لا تكتب عملية تشغيل طلبت منتجات أي صور، لذا فإن حالة الفشل في systemd مفيدة للمراقبة.

قم بإقرانها بمؤقت systemd للمعالجة المجدولة:

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
    --indices NDVI NDRE GNDVI
```

### Python وSDK على Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
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
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## أنظمة Jetson الموصى بها للاستخدام الميداني

بالنسبة لعمليات النشر الميدانية والجوية، ضع في اعتبارك خيارات لوحات الحامل Jetson Orin NX سعة 16 جيجابايت التالية:

* **الطيران/الطائرات بدون طيار**: أنظمة ذات تصنيف مقاومة للاهتزاز (MIL-STD)، وخفيفة الوزن (أقل من 300 جرام)، وتبريد سلبي
* **البيئات الميدانية القاسية**: أغلفة مقاومة للماء بمعيار IP67/IP69K مع إمكانية توصيل كاميرا GigE عبر PoE
* **الحد الأدنى/الميزانية المحدودة**: مجموعات المطورين المزودة بعلب إضافية

اتصل بـ [دعم MAPIR](https://www.mapir.camera/community/contact) للحصول على توصيات محددة بشأن الأجهزة التي تناسب سيناريو النشر الخاص بك.

***

## الخطوات التالية

* [Linux التثبيت](linux-installation.md) — تفاصيل عامة حول تثبيت Linux
* [التكيف الديناميكي للحوسبة](../processing-architecture/dynamic-compute-adaptation.md) — مرجع كامل لاستراتيجيات الحوسبة
* [مسار المعالجة](../processing-architecture/processing-pipeline.md) — فهم مسار المعالجة ذي الخيوط الأربعة
* [CLI : سطر الأوامر](../CLI.md) — دليل CLI
* [API : Python SDK](../api-python-sdk.md) — دليل SDK
* [مرجع CLI](../reference/cli-reference.md) و[مرجع SDK](../reference/sdk-reference.md) — قوائم شاملة بالأوامر/API للإصدار 1.2.0
