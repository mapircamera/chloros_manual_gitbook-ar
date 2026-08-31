# تثبيت Linux

يتم توزيع Chloros لـ Linux كحزم `.deb` التي تقوم بتثبيت CLI والخادم الخلفي. يُعد Python SDK حزمة pip منفصلة (مُضمَّنة أيضًا داخل `.deb` كحزمة wheel متوافقة مع الإصدار).

تحتوي أسماء ملفات الحزم على الإصدار والبنية: `chloros_1.2.0_amd64.deb` لـ x86_64، و `chloros_1.2.0_arm64_jp6.deb` لإصدارات JetPack 6 Jetson. استبدل الملف الذي قمت بتنزيله فعليًا في الأوامر أدناه.

***

## Linux amd64 (x86_64)

### متطلبات النظام

| المتطلب | الحد الأدنى | الموصى به |
| --- | --- | --- |
| **توزيع** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **المعالج** | x86_64 (Intel/AMD) | Intel Core i7 أو أفضل |
| **الذاكرة (RAM)** | 8 جيجابايت | 16 جيجابايت أو أكثر |
| **بطاقة الرسومات** | لا شيء (معالجة عبر وحدة المعالجة المركزية) | بطاقة رسومات NVIDIA GPU بسعة ذاكرة VRAM تبلغ 4 جيجابايت أو أكثر (تؤدي سعة 12 جيجابايت أو أكثر إلى فتح ميزة `GPU_PARALLEL`، بينما تؤدي سعة 7 جيجابايت أو أكثر إلى إبقاء ميزة Texture Aware خارج مسار الصورة الفردية) |
| **سعة التخزين** | 2 جيجابايت مساحة خالية | SSD بسعة 10 جيجابايت أو أكثر من المساحة الخالية |
| **Python** | Python 3.7 أو أعلى (لـ SDK) | Python 3.10+ |

> **لا يتم دعم Ubuntu 20.04 و Debian 11.** قائمة التبعيات الخاصة بـ `.deb`
> مستمدة مما ترتبط به الخلفية Chloros فعليًا، وهذا يشمل
> `libc6 (>= 2.34)`. يتم تزويد كل من Focal و bullseye بـ glibc 2.31، لذا يرفض `apt`
> التثبيت تمامًا بدلاً من السماح بفشله لاحقًا أثناء وقت التشغيل.

### التثبيت

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
لا يحل `dpkg -i` التبعيات. إذا أبلغ عن حزم مفقودة، فإن `sudo apt-get install -f` (أو `sudo apt --fix-broken install`) يكمل التثبيت — وهذا هو المسار الطبيعي، وليس خطأً.
{% endhint %}

تحقق من التثبيت:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### متطلبات النظام

| المتطلب | الحد الأدنى | الموصى به |
| --- | --- | --- |
| **المنصة** | NVIDIA Jetson مع JetPack 6 | Jetson Orin NX 16GB أو AGX Orin |
| **JetPack** | JetPack 6.x | أحدث إصدار من JetPack 6 |
| **الذاكرة (RAM)** | 8GB (مشتركة بين GPU و CPU) | 16GB+ مشتركة (12GB+ هي الحد الأدنى لتشغيل وحدات العمل المتوازية على GPU) |
| **سعة التخزين** | 2GB مساحة خالية | محرك أقراص SSD NVMe مع 10GB+ مساحة خالية |
| **Python** | Python 3.7+ (لـ SDK) | Python 3.10+ |

### التثبيت

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

نفس تصميم amd64 `.deb`، مع إصدار CUDA مُعدّل خصيصًا لـ Jetson Orin / Orin NX / Orin Nano. لمعرفة سلوك Jetson فيما يتعلق بالذاكرة والحرارة والنشر الميداني، راجع [دليل NVIDIA Jetson](nvidia-jetson-guide.md).

***

## تثبيت Python وSDK (جميع إصدارات Linux)

يُعد SDK عميلًا خالصًا لـ Python HTTP للخلفية، لذا تعمل الحزمة نفسها على amd64 و arm64. مصدران:**من PyPI** — الإصدار المستقر المنشور:

```bash
pip install chloros-sdk
```

**من ملف wheel المرفق** — مضمون توافقه مع CLI/backend الذي قمت بتثبيته للتو (استخدم هذا عندما يكون إصدار `.deb` الخاص بك أحدث من PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**توزيعات PEP 668** (Ubuntu 23.10+، Debian 12+) ترفض عمليات التثبيت عبر pip على مستوى النظام. استخدم `pip install --user …`، أو بيئة افتراضية، أو `sudo pip install --break-system-packages …`. لا يقوم مُثبِّت الحزمة أبدًا بتثبيت SDK تلقائيًّا في نظامك Python — فهذا الخيار متروك لك.
{% endhint %}

إضافات اختيارية:

| الإضافة | الأمر | تضيف |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` للبث المباشر للتقدم |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` لنقل BLE (DAQ-M) |

تحقق من SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
يقوم `.deb` بتثبيت Chloros و CLI والخلفية. يتواصل Python وSDK مع تلك الخلفية عبر شبكة محلية HTTP وAPI (`http://127.0.0.1:5000`) ويقوم بتشغيلها تلقائيًا عند الحاجة. استخدم دائمًا عنوان IPv4 الحرفي بدلاً من — حيث يمكن أن يُحل عنوان إلى ويستغرق الأمر حوالي ثانيتين لكل طلب.
{% endhint %}

***

## الإعداد الأولي

### 1. تسجيل الدخول

يتطلب الوصول إلى CLI و SDK اشتراكًا مدفوعًا من فئة Chloros+ (**Copper** أو أعلى)، ويتم تطبيق ذلك من جانب الخادم: يحصل المتصل الذي لم يسجل الدخول على `401 AUTH_REQUIRED`، بينما يحصل المتصل الذي يستخدم الفئة المجانية (Iron) على `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

يتم تخزين بيانات الاعتماد مؤقتًا في `~/.chloros/user_session.json`.

{% hint style="warning" %}
**يجب عليك تسجيل الدخول مرة أخرى بعد كل عملية تثبيت أو ترقية.** يقوم البرنامج النصي `prerm` الخاص بالحزمة بمسح `~/.chloros/user_session.json` والترخيص المخزّن مؤقتًا لكل مستخدم على الجهاز عن قصد، بحيث يقوم الإصدار الجديد دائمًا بإعادة التحقق من صحة الترخيص بدلاً من الاعتماد على ذاكرة تخزين مؤقتة قديمة.
{% endhint %}

### 2. تحقق من حالة ترخيصك

```bash
chloros-cli status
```

يعمل `chloros-cli status` على أي فئة (بما في ذلك الإصدار المجاني)، لذا يمكنك دائمًا معرفة سبب توفر الوصول أو عدم توفره.

### 3. تشغيل تشخيصات النظام

```bash
chloros-cli selftest
```

يتم تشغيل سبع عمليات فحص بالترتيب، ويخرج الأمر بقيمة غير صفرية في حالة فشل أي منها:

| # | الفحص | ما يثبت |
| --- | --- | --- |
| 1 | **الإصدار** | يُبلغ CLI عن إصداره (`v1.2.0`). |
| 2 | **المنفذ متاح** | المنفذ 5000 متاح، *أو* تم الرد عليه بالفعل من قبل خادم خلفي Chloros سليم (ويُعتبر ذلك نجاحًا). |
| 3 | **بدء تشغيل الخادم الخلفي** | يتم تشغيل الملف الثنائي للخادم الخلفي. |
| 4 | **اختبار API (`/api/test`)** | تستجيب الخلفية بـ `status: ok`. |
| 5 | **معلومات النظام** | يُعرض `GPU: <name>, CUDA: <bool>, PyTorch: <version>` من `/api/system-info`. |
| 6 | **نماذج مزيل الضوضاء** | يعثر على نماذج `*.pth.enc` (على Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + مزيل الضوضاء**| ميزة Texture Aware قابلة للاستخدام فعليًّا — تتطلب CUDA**و** ملف نموذج واحد على الأقل. |

ينتهي التشغيل بـ `N/7 checks passed`، مع سرد أي حالات فشل حسب الاسم.

### 4. معالجة أول مجموعة بيانات لك

```bash
chloros-cli process ~/datasets/flight001
```

***

## الملفات والمجلدات

### لكل مستخدم

يحتفظ Chloros ببيانات اعتماده وتكوين CLI في دليل واحد متعدد المنصات، **`~/.chloros/`** (على Windows و `%USERPROFILE%\.chloros\`). في المقابل، تتبع ذاكرتا التخزين المؤقتتان الخاصتان بـ Linux قواعد XDG — وهما تلتزمان بـ `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` عند تعيينهما.

| المسار | الغرض |
| --- | --- |
| `~/.chloros/user_session.json` | ذاكرة التخزين المؤقت لجلسة تسجيل الدخول التي يتم كتابتها بواسطة `chloros-cli login` (يتم مسحها عند كل تثبيت/ترقية لحزمة) |
| `~/.chloros/working_directory.txt` | تجاوز مجلد المشروع الافتراضي (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | تفضيلات اللغة لـ CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | إعداد اللغة المشترك مع واجهة المستخدم الرسومية Windows — ويكون لـ `language` هنا الأولوية على `cli_language.json` |
| `~/.chloros/update_cache.json` | ذاكرة تخزين مؤقتة لمدة ساعة واحدة لفحص تحديثات بدء التشغيل لـ Linux/Jetson |
| `~/.chloros/backend.log` | سجل الخلفية عند تشغيل الخلفية بواسطة CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | حزم معايرة LATTICE المخزنة مؤقتًا لكل كاميرا، مرتبة حسب الرقم التسلسلي وتجزئة الحزمة |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | تجاوزات المستخدم الاختيارية لملفات تعريف تصحيح السقف في DAQ |
| `~/.config/chloros/system_config.json` | ملف تعريف الأجهزة المخزن مؤقتًا من «التكيف الحسابي الديناميكي» — احذفه لإجبار النظام على إجراء اكتشاف جديد للأجهزة |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | سجلات خادم الخلفية، ملف واحد لكل عملية تشغيل |
| `~/Chloros Projects/` | مجلد المشروع الافتراضي في حالة عدم تعيين أي تجاوز |

### على مستوى النظام

| المسار | الغرض |
| --- | --- |
| `/usr/bin/chloros-cli` | برنامج نصي مغلف — يضبط `LD_LIBRARY_PATH` للمكتبات الأصلية المجمعة، ثم ينفذ الملف الثنائي الفعلي |
| `/usr/bin/chloros-backend` | برنامج نصي مغلف — نفس الشيء، بالإضافة إلى `CHLOROS_PRODUCTION=1` حتى لا تتمكن بوابة المصادقة في الخلفية من تعطيل نفسها بصمت |
| `/usr/lib/chloros/chloros-cli`، `/usr/lib/chloros/chloros-backend` | الملفات الثنائية المجمعة |
| `/usr/lib/chloros/arena_runtime/` | بيئة تشغيل Arena SDK المطلوبة لكاميرات LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | نماذج إزالة الضوضاء المشفرة المستخدمة بواسطة أداة إزالة التباين Texture Aware |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python و SDK، مجموعة ملفات تتطابق تمامًا مع هذا الإصدار |
| `/usr/lib/chloros/exiftool` | أداة exiftool المدمجة (مرتبطة رمزيًا بـ `/usr/local/bin/exiftool` فقط في حالة عدم وجود أداة exiftool في النظام) |
| `/etc/chloros/update.conf` | تكوين قناة التحديث الذي يقرأه `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | يضبط `net.ipv4.ip_unprivileged_port_start = 319` بحيث يمكن للخلفية ربط منافذ PTP دون حقوق الجذر |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | توجيه المحمل الديناميكي إلى `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | يمنح المستخدم المسجل الدخول حق الوصول إلى جسر DAQ-U التسلسلي عبر USB (CP2102N، `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | خدمة الخلفية التي تعمل دائمًا (مثبتة، **غير ممكّنة**) |
| `/usr/share/applications/chloros-cli.desktop` | عنصر قائمة التطبيقات &quot;Chloros CLI&quot; الذي يفتح محطة طرفية |

## موقع الملف القابل للتنفيذ للخلفية

يقوم كل من CLI و SDK بالكشف التلقائي عن الخلفية:

| المكون | المسار |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| الخلفية | `/usr/lib/chloros/chloros-backend` |

يمكن تجاوز مسار الخلفية باستخدام العلامة `--backend-exe` CLI أو معلمة منشئ `backend_exe` SDK، وتجاوز المنفذ باستخدام `--port` (الافتراضي `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` يشير إلى **`lattice`**،**`project`**، و**`daq pool-*`** في الخلفية البعيدة. أما الأوامر الأساسية (`process`، `login`، `logout`، `status`، `export-status`، `time-sync`، `selftest`) تتجاهلها عمدًا وتستهدف دائمًا `http://127.0.0.1:<port>`.
{% endhint %}

***

## كاميرات LATTICE ومستشعرات الضوء DAQ على Linux

تعمل جميع مجموعات أوامر live-hardware على Linux (amd64 و Jetson):

* **`chloros-cli lattice`** — اكتشاف كاميرات LATTICE والمصفوفات المتزامنة والاتصال بها وتهيئتها والتقاط الصور منها. يضم `.deb` بيئة التشغيل Arena SDK التي تتطلبها هذه المجموعات ويسجلها لدى المحمل الديناميكي.
* **`chloros-cli daq pool-*`** — توصيل مستشعرات الضوء DAQ-U/M/E عبر مجموعة الخلفية، وبث الأطياف المعايرة، وتسجيل ملفات `.daq`. يتم توزيع الإصدار المُجمَّع CLI مع عائلة `pool-*` فقط: `pool-connect`، `pool-disconnect`، `pool-list`، و`pool-latest`، و`pool-stream`، و`pool-record`، و`pool-set-cap`.
* **`chloros-cli project`** — تشغيل مشروع محفوظ (بما في ذلك الكاميرات وأجهزة الاستشعار وإعدادات المعالجة) بدون واجهة مستخدم.
* **`chloros-cli time-sync`** — فحص وحدة التحكم الرئيسية PTP التي تعمل عليها الخلفية Chloros لكاميرات LATTICE ومستشعرات DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` مطلوب من قِبل `pool-latest` و `pool-stream` و `pool-record`، و`pool-set-cap`؛ ويُظهر `pool-list` المعرّفات الموجودة حاليًا في المجموعة.

{% hint style="info" %}
**يفضل استخدام `--eth-host` لأول اتصال DAQ-E على جهاز متعدد العناوين.** تقوم ميزة الاكتشاف التلقائي بالبحث في mDNS وقد تفوت واجهة المستشعر بسبب ذاكرة التخزين المؤقتة ARP غير النشطة، لذا قد يفشل أول اتصال بـ `pool-connect --eth` بعد التشغيل حتى لو كان المستشعر يعمل بشكل سليم تمامًا. يؤدي تمرير عنوان IP أو اسم المضيف الخاص بالمستشعر إلى تخطي عملية الاكتشاف تمامًا.
{% endhint %}

**أذونات المسلسل DAQ-U** تُدار بواسطة قاعدة udev المثبتة (`uaccess` + المجموعة `dialout`). إذا ظل جهاز استشعار موصولًا بالفعل غير قابل للوصول، فقم بإعادة تحميل القواعد أو إعادة توصيله:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

انظر [مرجع CLI](../CLI.md) للاطلاع على مجموعة الأوامر الكاملة.

### PTP دائم التشغيل لأجهزة المضيف غير المزودة بشاشة

عند التثبيت الأول، يتم إنشاء وحدة systemd `chloros-backend.service` ولكنها **غير ممكّنة**. على جهاز Jetson بدون شاشة أو خادم يجب أن يحافظ على تشغيل مزامنة الوقت PTP بشكل مستمر لأجهزة استشعار DAQ-E وكاميرات LATTICE، قم بتمكينه:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

بدونها، يعمل PTP فقط أثناء تشغيل الخلفية Chloros — أي أثناء جلسة CLI/SDK النشطة.

تقوم الوحدة بربط الخلفية بـ `127.0.0.1:5000` (إعدادات البيئة `CHLOROS_HOST` / `CHLOROS_PORT` داخل الوحدة؛ التجاوز باستخدام `sudo systemctl edit chloros-backend.service`) وتعيد تشغيلها في حالة الفشل بعد 5 ثوانٍ.

**كيف يحصل PTP على منافذه.** يستخدم PTP UDP 319/320، وكلاهما أقل من الحد الأدنى المعتاد للمنافذ المتميزة 1024. يقوم `postinst` الخاص بالحزمة بكتابة `/etc/sysctl.d/60-chloros-ptp.conf` باستخدام `net.ipv4.ip_unprivileged_port_start = 319`، مما يسمح للخلفية بربطهما أثناء التشغيل بصفتك مستخدمًا. كما يتم تطبيق `setcap cap_net_bind_service,cap_net_raw=+ep` على الملف الثنائي للخلفية كإجراء احترازي إضافي — ولهذا السبب يُعد `libcap2-bin` أحد التبعيات المعلنة للحزمة.***

## أمثلة على البرمجة النصية في Bash

{% hint style="info" %}
**رموز خروج ملائمة للبرامج النصية.**يقوم `chloros-cli process` بإنهاء `0` عند النجاح و**بقيمة غير صفرية عند الفشل — بما في ذلك التشغيل الذي طلب منتجات صور ولكنه لم يكتب أيًا منها** (يُظهر `Processing finished but wrote no image products.` ويذكر اسم مجلد المشروع والأسباب المعتادة). تُبلغ عمليات التشغيل الناجحة عن عدد منتجات الصور التي تمت كتابتها (`Image products written: N`). رموز الخروج: `0` نجاح، `1` فشل، `2` خطأ في الوسيطة، `130` توقف.
{% endhint %}

### معالجة مجموعات بيانات متعددة

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### المعالجة باستخدام إعدادات مخصصة

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

القيم الصالحة لـ `--format` هي أربع قيم بالضبط، وتحتوي على مسافات — لذا يجب وضعها دائمًا بين علامتي اقتباس:

| قيمة `--format` | مجلد الإخراج |
| --- | --- |
| `TIFF (16-bit)` *(افتراضي)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` يقبل `standard` (الافتراضي) أو `texture-aware` (Chloros+).

### المعالجة التلقائية باستخدام Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### مثال على Python وSDK

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

### لم يتم العثور على CLI بعد التثبيت

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### رفض الإذن

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### «فشل setcap» أثناء التثبيت

يقوم `.deb` بتطبيق `cap_net_bind_service` على `/usr/lib/chloros/chloros-backend` حتى يتمكن من ربط منافذ PTP 319/320 دون الحاجة إلى صلاحيات الجذر. إذا كان `libcap2-bin` مفقودًا عند التثبيت، يتم تخطي الاستدعاء. قم بتثبيته ثم أعد تثبيت الحزمة:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP لا يبدأ / لا يمكن ربط المنفذ 319

تأكد من خفض الحد الأدنى للمنافذ غير المتميزة، وأعد تطبيقه على عملية التشغيل الحالية إذا لم يكن كذلك:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

ثم تحقق من جهاز grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### «لم يتم العثور على برامج تشغيل كاميرا LATTICE»

لا يتم تحليل وقت تشغيل Arena SDK. تأكد من وجود تكوين المحمل الذي تكتبه الحزمة ومن تحديثه:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### فشل تشغيل الخلفية

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

توجد سجلات الخلفية الخاصة بفشل التشغيل في `~/.cache/chloros/logs/`.

### لم يتم الكشف عن CUDA

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

يُبلغ ملف `chloros-cli selftest` عن الأمر نفسه في سطر واحد: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### المكتبات المشتركة مفقودة

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### بطء بدء التشغيل على أنظمة بطاقات SD

تقوم الملفات الثنائية المُجمَّعة باستخراج نفسها إلى دليل مؤقت عند كل تشغيل. إذا كان `/mnt/ssd/tmp` موجودًا، فإن Chloros يستخدمه تلقائيًا؛ وإلا فاضبط `TMPDIR` على نظام ملفات سريع:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## تحديث Chloros على Linux

الأمر `update` مخصص لـ Linux/Jetson فقط. وهو يتحقق من الإصدار المنشور في قناة التحديث المُعدة في `/etc/chloros/update.conf` ويعرض تنزيل وتثبيت الإصدار المطابق `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

على Linux/Jetson، يقوم الأمر CLI أيضًا بإجراء فحص تحديث غير معوق عند كل عملية تشغيل (يتم تخزين النتيجة مؤقتًا لمدة ساعة واحدة في `~/.chloros/update_cache.json`) ويعرض `Update available: vX.Y.Z` عند وجود إصدار أحدث. يتم الحفاظ على إعداداتك ومشاريعك بعد التحديث؛ وستحتاج إلى تسجيل الدخول مرة أخرى بعد ذلك.

## إلغاء التثبيت

```bash
sudo apt remove chloros
```

يؤدي إلغاء التثبيت إلى إيقاف تشغيل `chloros-backend.service`، واستعادة الحد الأدنى الافتراضي للمنافذ غير المتميزة (1024)، ويزيل الرابط الرمزي لـ exiftool المرفق وتكوين أداة التحميل Arena، كما يمسح بيانات الاعتماد المخزنة مؤقتًا. تظل مشاريعك وملفات بيانات `~/.chloros/` كما هي.

***

## الخطوات التالية

* [دليل NVIDIA Jetson](nvidia-jetson-guide.md) — التحسين والنشر الخاصان بـ Jetson
* [CLI : سطر الأوامر](../CLI.md) — دليل CLI
* [API : Python SDK](../api-python-sdk.md) — دليل SDK
* [مرجع CLI](../reference/cli-reference.md) و[مرجع SDK](../reference/sdk-reference.md) — قوائم شاملة للأوامر/API للإصدار 1.2.0
* [التكيف الديناميكي للحوسبة](../processing-architecture/dynamic-compute-adaptation.md) — كيفية تكيف Chloros مع أجهزتك
