# تثبيت Linux

يتم توزيع Chloros لـ Linux على شكل حزم `.deb` التي تقوم بتثبيت CLI والواجهة الخلفية. يتم تثبيت Python SDK بشكل منفصل عبر pip.

***

## Linux amd64 (x86_64)

### متطلبات النظام

| المتطلب | الحد الأدنى | الموصى به |
| --- | --- | --- |
| **التوزيع** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **المعالج** | x86_64 (Intel/AMD) | Intel Core i7 أو أفضل |
| **الذاكرة (RAM)** | 8 جيجابايت | 16 جيجابايت أو أكثر |
| **بطاقة الرسومات** | لا شيء (معالجة وحدة المعالجة المركزية) | وحدة معالجة رسومات NVIDIA مع ذاكرة VRAM سعة 4 جيجابايت أو أكثر |
| **التخزين** | 2 جيجابايت من المساحة الخالية | SSD مع 10 جيجابايت أو أكثر من المساحة الخالية |
| **Python** | Python 3.7+ (لـ SDK) | Python 3.10+ |

### التثبيت

قم بتنزيل حزمة `.deb` وتثبيتها:

```bash
sudo dpkg -i chloros-amd64.deb
```

تحقق من التثبيت:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### متطلبات النظام

| المتطلب | الحد الأدنى | الموصى به |
| --- | --- | --- |
| **المنصة** | NVIDIA Jetson مع JetPack 6 | Jetson Orin NX 16GB أو AGX Orin |
| **JetPack** | JetPack 6.x | أحدث إصدار من JetPack 6 |
| **الذاكرة (RAM)** | 8 جيجابايت (مشتركة بين GPU/CPU) | 16 جيجابايت+ مشتركة |
| **التخزين** | 2 جيجابايت مساحة خالية | SSD NVMe مع 10 جيجابايت+ مساحة خالية |
| **Python** | Python 3.7+ (لـ SDK) | Python 3.10+ |

### التثبيت

قم بتنزيل حزمة JetPack 6 `.deb` وتثبيتها:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

تحقق من التثبيت:

```bash
chloros-cli --version
```

للحصول على إعدادات Jetson التفصيلية بما في ذلك إدارة الحرارة والنشر الميداني، راجع [دليل NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Python SDK التثبيت (كل Linux)

يتم تثبيت Python SDK بشكل منفصل عبر pip ويعمل على كل من amd64 و arm64:

```bash
pip install chloros-sdk
```

لإدراج دعم البث التدريجي الاختياري:

```bash
pip install chloros-sdk[progress]
```

تحقق من SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
تقوم حزمة `.deb` بتثبيت Chloros و CLI والخلفيّة. تعد حزمة Python وSDK حزمة pip منفصلة تتواصل مع الخلفية عبر HTTP وAPI محلي.
{% endhint %}

***

## دلائل التكوين

Chloros على Linux يتبع [مواصفات الدليل الأساسي لـ XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| الغرض | مسار Linux | المكافئ Windows |
| --- | --- | --- |
| **التكوين** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **البيانات / المشاريع** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **ذاكرة التخزين المؤقت / بيانات الاعتماد** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## مواقع الملفات التنفيذية للخلفية

تقوم حزمة `.deb` بتثبيت الخلفية في موقع قياسي. تقوم الحزمتان CLI و SDK بالكشف التلقائي عن مسار الخلفية:

| طريقة التثبيت | مسار الخلفية |
| --- | --- |
| حزمة `.deb` | `/usr/lib/chloros/chloros-backend` |
| يدوي / مخصص | `/opt/mapir/chloros/backend/chloros-backend` |

يمكنك تجاوز مسار الخلفية باستخدام علامة `--backend-exe` CLI أو معلمة منشئ `backend_exe` SDK.

***

## الإعداد الأولي

### 1. تنشيط الترخيص

يلزم ترخيص Chloros+ للوصول إلى CLI و SDK:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. تحقق من حالة ترخيصك

```bash
chloros-cli status
```

### 3. قم بمعالجة أول مجموعة بيانات لك

```bash
chloros-cli process ~/datasets/flight001
```

### 4. قم بتشغيل تشخيص النظام

تحقق من أن نظامك قد تم تكوينه بشكل صحيح:

```bash
chloros-cli selftest
```

يؤدي هذا إلى تشغيل 7 فحوصات تشخيصية تشمل الإصدار وبدء تشغيل الخلفية وAPI الاتصال وتوافر CUDA/GPU.

***

## أمثلة على البرمجة النصية في Bash

### معالجة مجموعات بيانات متعددة

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### المعالجة باستخدام إعدادات مخصصة

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### المعالجة التلقائية باستخدام Cron

أضف إلى crontab (`crontab -e`) لمعالجة مجموعات البيانات الجديدة تلقائيًا:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK مثال

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## استكشاف الأخطاء وإصلاحها

### CLI غير موجود بعد التثبيت

إذا لم يتم العثور على `chloros-cli` بعد تثبيت حزمة `.deb`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### رفض الإذن

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### فشل بدء تشغيل الخلفية

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### لم يتم الكشف عن CUDA

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### المكتبات المشتركة مفقودة

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## تحديث Chloros على Linux

استخدم أمر التحديث المدمج للتحقق من وجود تحديثات وتثبيتها:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## الخطوات التالية

* [دليل NVIDIA Jetson](nvidia-jetson-guide.md) — التحسين والنشر الخاصان بـ Jetson
* [CLI : سطر الأوامر](../CLI.md) — مرجع كامل لأوامر CLI
* [API : Python SDK](../api-python-sdk.md) — مرجع SDK الكامل
* [التكيف الحسابي الديناميكي](../processing-architecture/dynamic-compute-adaptation.md) — كيف يتكيف Chloros مع أجهزتك
