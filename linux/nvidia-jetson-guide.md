# دليل NVIDIA Jetson

XPROTX000047 يتيح XPROTX على NVIDIA Jetson معالجة الصور متعددة الأطياف في الحافة — في الميدان، وعلى الطائرات بدون طيار، وفي المنشآت البعيدة. XPROTX000048 يقوم XPROTX بالكشف تلقائيًا عن طراز Jetson الخاص بك ويُحسّن استراتيجية المعالجة بما يتناسب مع أجهزتك.

***

## طرازات Jetson المدعومة

| الطراز                | ذاكرة الوصول العشوائي (RAM)            | استراتيجية المعالجة                                   | الاستخدام الموصى به                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 جيجابايت مشتركة | `GPU_PARALLEL` (4 عمال)                            | أقصى أداء، مجموعات بيانات كبيرة                      |
| **Jetson Orin NX**   | 8-16 جيجابايت مشتركة  | `GPU_PARALLEL` (3 عمال، 16 جيجابايت) / `GPU_SINGLE` (8 جيجابايت) | التوصية الأساسية للنشر الجوي والميداني |
| **Jetson Orin Nano** | 8 جيجابايت مشتركة     | `GPU_SINGLE` (عامل واحد)                               | حوسبة حافة المستوى المبتدئ                                 |
| **Jetson Nano**      | 4-8 جيجابايت مشتركة   | `GPU_SINGLE` (عامل واحد)                               | مستوى مبتدئ، محدود الذاكرة                          |

{% hint style="info" %}
**قد لا يتم دعم طرازات Jetson القديمة** (TX2، TX1، Xavier NX). سيختلف الأداء بناءً على ذاكرة GPU المتاحة وقدرات CUDA.
{% endhint %}

***

## المتطلبات

* **JetPack 6.x** (يوصى بأحدث إصدار)
* **NVIDIA CUDA** (مضمنة مع JetPack)
* **ترخيص Chloros+** (مطلوب للوصول إلى CLI/SDK)

## التثبيت

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

للحصول على تفاصيل عامة حول تثبيت Linux، راجع [تثبيت Linux](linux-installation.md).

***

## التكيف الحسابي الديناميكي على Jetson

يكتشف Chloros طراز Jetson الخاص بك تلقائيًا ويختار استراتيجية المعالجة المثلى. **لا يلزم إجراء أي ضبط يدوي.**

### كيف يعمل

عند بدء التشغيل، يقوم Chloros بتحديد ملامح نظامك:

1. **يكتشف طراز Jetson** عبر `/proc/device-tree/model`
2. **يقرأ وحدة معالجة الرسومات (GPU) والذاكرة المشتركة المتاحة**

3.**يختار استراتيجية معالجة** (`GPU_PARALLEL` أو `GPU_SINGLE` أو `CPU_PARALLEL`)
4. **يضبط عدد العمال ونوع خط الأنابيب وتخصيص الذاكرة** تلقائيًا

### السلوك لكل نموذج

| نموذج Jetson                | الاستراتيجية       | العمال | خط الأنابيب                       | التزامن |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (كفاءة في استخدام الذاكرة) | تسلسلي  |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | متسلسل  |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | متسلسل  |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (مسار GPU كامل)    | متزامن  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | متزامن  |

{% hint style="success" %}
**Jetson Orin NX 16GB** هو الخيار الأمثل للنشر على الحافة — فهو يتبنى استراتيجية `GPU_PARALLEL` مع 3 عمال متزامنين، مما يوفر معالجة GPU متوازية حقيقية في هيكل مدمج.
{% endhint %}

الفرق الرئيسي بين المنصات هو **الذاكرة**. يجب على Jetson Nano المزود بذاكرة مشتركة سعة 8 جيجابايت معالجة الصور واحدة تلو الأخرى باستخدام نهج متجاور فعال من حيث استهلاك الذاكرة، بينما يمكن لـ Orin NX المزود بذاكرة سعة 16 جيجابايت تشغيل 3 صور عبر وحدة معالجة الرسومات (GPU) في وقت واحد باستخدام خط أنابيب مدمج ذي إنتاجية أعلى.

للاطلاع على مرجع التكيف الحسابي الكامل، انظر [التكيف الحسابي الديناميكي](../processing-architecture/dynamic-compute-adaptation.md).

***

## إدارة الحرارة

تتمتع أجهزة Jetson بقدرة حرارية محدودة، خاصة في عمليات النشر المغلقة أو المحمولة جواً. يتضمن Chloros مراقبة حرارية تلقائية وخفض السرعة:

| درجة الحرارة         | الإجراء                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 درجة مئوية**          | التشغيل العادي — سرعة معالجة كاملة          |
| **70 درجة مئوية** (تحذير)  | تقليل حجم الدفعة تلقائيًا                   |
| **80 درجة مئوية** (حرجة) | تخفيض قوي — تقليل التزامن         |
| **90 درجة مئوية** (إيقاف التشغيل) | إيقاف معالجة وحدة معالجة الرسومات (GPU) تمامًا — يلزم التبريد |

{% hint style="warning" %}
**تأكد من توفر تهوية كافية ومشتت حراري** من أجل معالجة مستمرة، خاصة في العلب الميدانية المغلقة أو الأنظمة المحمولة جواً. سيؤدي التقييد الحراري إلى تقليل إنتاجية المعالجة لحماية الأجهزة.
{% endhint %}

***

## إدارة الذاكرة

تستخدم أجهزة Jetson **ذاكرة موحدة** — حيث تشترك وحدة معالجة الرسومات (GPU) ووحدة المعالجة المركزية (CPU) في نفس ذاكرة الوصول العشوائي (RAM) الفعلية. وهذا يعني أن ذاكرة VRAM المعلنة (على سبيل المثال، 15.3 جيجابايت في Orin NX 16 جيجابايت) ليست ذاكرة مخصصة لوحدة معالجة الرسومات (GPU)؛ بل يتم مشاركتها مع نظام التشغيل والعمليات الأخرى.

### توصيات بشأن مساحة التبديل

بالنسبة لمجموعات البيانات الكبيرة أو معالجة إزالة التباين (debayer) التي تراعي النسيج، قد يوصي Chloros بإنشاء مساحة تبديل:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**تقديرات الذاكرة لكل صورة:**

* إزالة التباين القياسية: ~10 ميجابايت لكل صورة
* إزالة التباين التي تراعي النسيج: ~15 ميجابايت لكل صورة

يحسب Chloros تلقائيًا الذاكرة المطلوبة بناءً على حجم مجموعة البيانات الخاصة بك ويحذرك إذا كان التبديل موصى به.

### خطة بديلة في حالة نفاد الذاكرة (OOM)

إذا تم الكشف عن حالة نفاد الذاكرة أثناء المعالجة:

1. يقوم Chloros تلقائيًا بتقليل عدد عمال وحدة معالجة الرسومات (GPU)
2. يتحول من مسار `fused_gpu` إلى مسار `tiled_gpu` (أكثر كفاءة في استخدام الذاكرة)
3. يستمر في المعالجة بإنتاجية مخفضة بدلاً من التعطل

***

## النشر الميداني

### اعتبارات الطاقة

| طراز Jetson     | استهلاك الطاقة النموذجي | ملاحظات                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10 وات              | USB-C أو مقبس أسطواني    |
| Jetson Orin Nano | 7-15 وات              | مقبس أسطواني تيار مستمر          |
| Jetson Orin NX   | 10-25 وات             | مقبس أسطواني تيار مستمر          |
| Jetson AGX Orin  | 15-60 واط             | USB-C PD أو مقبس أسطواني |

خطط لميزانية الطاقة الخاصة بك من أجل معالجة مستمرة — يحدث استهلاك الطاقة الأقصى أثناء الخيط 3 (المعالجة) الذي يعتمد بشكل مكثف على وحدة معالجة الرسومات (GPU).

### توصيات التخزين

* **NVMe SSD** موصى به بشدة لنشر arm64
* بطاقات SD بطيئة جدًا للمعالجة — استخدمها كوسائط تمهيد فقط
* خطط لحجم يبلغ 2-3 أضعاف حجم بيانات الصورة الأولية للناتج المعالج

### التشغيل بدون شاشة عبر SSH

Chloros CLI مثالي لنشر Jetson بدون شاشة:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

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

### المعالجة الأساسية لـ Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK على Jetson

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
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## أنظمة Jetson الموصى بها للاستخدام الميداني

بالنسبة لعمليات النشر الميدانية والجوية، ضع في اعتبارك خيارات لوحات الناقل Jetson Orin NX 16GB التالية:

* **جوية/طائرات بدون طيار**: أنظمة ذات تصنيف مقاومة للاهتزاز (MIL-STD)، خفيفة الوزن (أقل من 300 جرام)، تبريد سلبي
* **الميدان القاسي**: علب مقاومة للماء بمعيار IP67/IP69K مع اتصال كاميرا PoE GigE
* **الحد الأدنى/الميزانية**: مجموعات المطورين مع علب إضافية

اتصل بـ [MAPIR الدعم](https://www.mapir.camera/community/contact) للحصول على توصيات محددة بشأن الأجهزة لسيناريو النشر الخاص بك.

***

## الخطوات التالية

* [Linux التثبيت](linux-installation.md) — تفاصيل عامة حول تثبيت Linux
* [التكيف الحسابي الديناميكي](../processing-architecture/dynamic-compute-adaptation.md) — مرجع كامل لاستراتيجية الحوسبة
* [مسار المعالجة](../processing-architecture/processing-pipeline.md) — فهم مسار المعالجة ذي 4 خيوط
* [CLI : سطر الأوامر](../CLI.md) — مرجع كامل لـ CLI
* [API : Python SDK](../api-python-sdk.md) — مرجع SDK الكامل
