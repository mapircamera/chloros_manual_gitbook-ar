# تثبيت Linux

يتم توزيع Chloros لنظام Linux على شكل حزم `.deb` التي تقوم بتثبيت واجهة سطر الأوامر (CLI) والخادم الخلفي ومجموعة أدوات تطوير البرامج (SDK) لـ Python (بالنسبة للنظام `python3`، بالإضافة إلى حزمة «wheel» متوافقة مع الإصدار لـ virtualenvs).

تحتوي أسماء ملفات الحزم على الإصدار والبنية: `chloros_1.3.0_amd64.deb` لنظام x86_64، و `chloros_1.3.0_arm64_jp6.deb` لـ arm64 (NVIDIA Jetson، JetPack 6). استبدل الملف الذي قمت بتنزيله فعليًّا في الأوامر أدناه.

{% hint style="info" %}
**جديد في الإصدار 1.3.0: يتم توزيع البرامج غير مضغوطة.** يتم تثبيت الخلفية وواجهة سطر الأوامر (CLI) كمجلدين، `/usr/lib/chloros/backend/` و `/usr/lib/chloros/cli/`، بدلاً من ملفات فردية ذاتية الاستخراج. لم تعد هذه المكونات تُفك ضغطها تلقائيًا في مجلد مؤقت عند كل تشغيل، لذا فإنها تبدأ بشكل أسرع بكثير (على x86_64، كانت الخلفية تستغرق حوالي 100 ثانية في فك الضغط عند كل بدء تشغيل) ولم تعد تحتاج إلى عدة غيغابايت من المساحة الخالية في `/tmp`. كما أن حجم `.deb` لنظام amd64 أصبح حوالي نصف حجمه السابق.
{% endhint %}

***

## Linux_amd64 (x86_64)

### متطلبات النظام

| المتطلب | الحد الأدنى | الموصى به |
| --- | --- | --- |
| **التوزيعة** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **المعالج** | x86_64 (Intel/AMD) | Intel Core i7 أو أفضل |
| **الذاكرة (RAM)** | 8 جيجابايت | 16 جيجابايت أو أكثر |
| **بطاقة الرسومات** | لا شيء (معالجة بواسطة المعالج) | بطاقة رسومات NVIDIA GPU بسعة ذاكرة VRAM تبلغ 4GB+ (12GB+ تفتح ميزة `GPU_PARALLEL`، و7GB+ تبقي ميزة Texture Aware خارج مسار الصورة الواحدة) |
| **سعة التخزين** | 2GB مساحة خالية | SSD بسعة 10GB+ مساحة خالية |
| **Python** | Python 3.7+ (لـ SDK) | Python 3.10+ |

> **لا يتم دعم Ubuntu 20.04 و Debian 11.** قائمة التبعيات الخاصة بـ `.deb`
> مستمدة مما ترتبط به الخلفية Chloros فعليًّا، وهذا يشمل
> إصدارًا من `libc6` أحدث من glibc 2.31 المرفق مع إصداري focal و bullseye، لذا يرفض `apt`
> التثبيت تمامًا بدلاً من السماح بفشله لاحقًا أثناء وقت التشغيل.

### التثبيت

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**استخدم `apt install ./…`، وليس `dpkg -i`.** يقوم `apt` بحل تبعيات الحزمة كما يقوم بتثبيت ما يوصي به: متطلبات Python SDK، `python3-numpy` و `python3-requests`. أما `dpkg -i` فلا يقوم بتثبيت أي منهما. تعد واجهة سطر الأوامر (CLI) والواجهة الخلفية (backend) مستقلتين وتعملان بدونهما، لكن `import chloros_sdk` يحتاج إليهما؛ ويذكر إخراج التثبيت أي مكونات مفقودة. احتفظ بـ `./`، حتى يقوم `apt` بتثبيت الملف بدلاً من البحث عن حزمة بهذا الاسم. تعلن الحزمة أيضًا عن `libgl1`، الذي يقوم OpenCV المضمّن بتحميله وقد تفتقر إليه صورة الخادم البسيطة.
{% endhint %}

تحقق من التثبيت:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### متطلبات النظام

| المتطلب | الحد الأدنى | الموصى به |
| --- | --- | --- |
| **المنصة** | NVIDIA Jetson مع JetPack 6 | Jetson Orin NX 16GB أو AGX Orin |
| **JetPack** | JetPack 6.x | أحدث إصدار من JetPack 6 |
| **الذاكرة (RAM)** | 8GB (مشتركة بين GPU و CPU) | 16GB+ مشتركة (12GB+ هي الحد الأدنى للمعالجة المتوازية) |
| **سعة التخزين** | 2GB مساحة خالية | محرك أقراص SSD NVMe مع 10GB+ مساحة خالية |
| **Python** | Python 3.7+ (لـ SDK) | Python 3.10+ |

### التثبيت

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

نفس تخطيط amd64 `.deb` (يتضمن Python SDK ومخازن الاستقبال وخدمات الخلفية)، لأجهزة Jetson Orin / Orin NX / Orin Nano على **JetPack 6** (L4T R36). PyTorch الخاص به هو إصدار CUDA الخاص بـ JetPack 6 لـ Orin، لذا فإن معالجة GPU، بما في ذلك أداة إزالة التباين التي تراعي النسيج (Texture Aware debayer)، تعمل على وحدة معالجة الرسومات (GPU) الخاصة بـ Jetson. لا تحتوي الحزمة على CUDA نفسها: فهي تستخدم مكتبات CUDA وcuDNN التي يقوم JetPack 6 بتثبيتها (`sudo apt install nvidia-jetpack` على صورة تفتقر إليها). في حالة عدم توفر وحدة معالجة رسومات (GPU) تدعم CUDA، تتم المعالجة على وحدة المعالجة المركزية (CPU)، ويستخدم `--debayer texture-aware` أداة إزالة التباين القياسية (تسجل الخلفية سطرًا واحدًا في السجل يوضح السبب). للاطلاع على سلوك Jetson فيما يتعلق بالذاكرة والحرارة والنشر الميداني، راجع [دليل NVIDIA Jetson](nvidia-jetson-guide.md).

***

## تثبيت Python SDK (جميع أنظمة Linux)**يقوم `.deb` بتثبيت SDK نيابة عنك.** يقوم كلا الحزمتين (amd64 وJetson) بتثبيته للنظام `python3`، لذا يعمل `import chloros_sdk` مباشرةً بعد التثبيت، دون الحاجة إلى pip أو اتصال بالشبكة. أما حزمته (`chloros_sdk`، و`lattice_sdk`، و`lattice_cam`، `arena_api` ومحملات المشاريع) تُضم إلى `/usr/lib/python3/dist-packages`، التي تمتلكها الحزمة: حيث يتم استبدالها عند الترقية، وإزالتها عند إزالة الحزمة. يتطابق الإصدار دائمًا مع واجهة سطر الأوامر (CLI) والواجهة الخلفية (backend) التي قمت بتثبيتها.

متطلبات حزمة SDK نفسها، `python3-numpy` و `python3-requests`، **موصى بها، وليست إلزامية** من قبل الحزمة: يقوم `sudo apt install ./chloros_<version>_<arch>.deb` بتثبيتها، بينما لا يقوم `dpkg -i` بذلك. يُبلغ إخراج التثبيت عن إصدار SDK الذي يستورده `python3`، وأي متطلبات مفقودة، وأي نسخة من `chloros_sdk` موجودة في أي مكان آخر على مسار `python3` والتي تحجب نسخة الحزمة (مثل تلك التي تم تثبيتها مسبقًا باستخدام `pip install --user`).**في بيئة virtualenv**، التي لا ترى حزم النظام، قم بتثبيت ملف wheel المرفق فيها (بشكل غير متصل بالإنترنت تمامًا):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**من PyPI** — فقط لأجهزة لا تحتوي على حزمة Chloros مثبتة، والتي يعمل SDK الخاص بها على تشغيل خلفية على جهاز آخر (انظر [وضع الخلفية البعيدة](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

إضافات اختيارية، للتثبيت عبر pip (بيئة virtualenv أو PyPI):

| الإضافة | الأمر | تضيف |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` للبث المباشر للتقدم |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` لنقل BLE (DAQ-M) |

تحقق من SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
يتواصل SDK الخاص بـ Python مع الخلفية التي يقوم `.deb` بتثبيتها عبر HTTP API محلي (`http://127.0.0.1:5000`). يقوم كل من `ChlorosLocal` ومساعدات الاتصال بتشغيله تلقائيًا عند الحاجة؛ أما `connect_all()` الخاص بالمشروع المحفوظ فلا يقوم بذلك أبدًا، لذا، على جهاز بدون شاشة، قم بتشغيل الخلفية كـ [خدمة](#always-on-ptp-for-headless-hosts). استخدم دائمًا عنوان IPv4 الحرفي بدلاً من `localhost` — حيث يمكن أن يُحل `localhost` إلى `::1` ويستغرق حوالي ثانيتين لكل طلب.
{% endhint %}

***

## الإعداد الأولي

### 1. تسجيل الدخول

يتطلب الوصول إلى واجهة سطر الأوامر (CLI) ومجموعة أدوات التطوير (SDK) اشتراكًا مدفوعًا في فئة Chloros+ (**Copper** أو أعلى)، ويتم فرض ذلك من جانب الخادم: يحصل المتصل الذي تم تسجيل خروجه على `401 AUTH_REQUIRED`، بينما يحصل المتصل في الفئة المجانية (Iron) على `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

يتم تخزين بيانات الاعتماد مؤقتًا في `~/.chloros/user_session.json`.

{% hint style="info" %}
**الترقيات تحافظ على بيانات تسجيل الدخول الخاصة بك؛ أما الإزالة فتؤدي إلى مسحها.** يؤدي تثبيت إصدار أحدث من `.deb` فوق تثبيت موجود (بما في ذلك عبر `chloros-cli update`) إلى الاحتفاظ بـ `~/.chloros/user_session.json` والترخيص المخزّن مؤقتًا، وبالتالي تستمر مهام CLI/SDK التي تعمل بدون واجهة مستخدم في العمل. تؤدي إزالة الحزمة (`sudo apt remove chloros`) إلى مسح بيانات تسجيل الدخول المخزنة وذاكرة التخزين المؤقتة للترخيص لكل مستخدم على الجهاز، لذا يتطلب التثبيت الجديد تثبيت `chloros-cli login` مرة أخرى.
{% endhint %}

### 2. التحقق من حالة الترخيص

```bash
chloros-cli status
```

يعمل `chloros-cli status` على أي فئة (بما في ذلك الفئة المجانية)، لذا يمكنك دائمًا معرفة سبب توفر الوصول أو عدم توفره.

### 3. تشغيل تشخيصات النظام

```bash
chloros-cli selftest
```

يتم تشغيل سبع عمليات فحص بالترتيب، ويُخرج الأمر قيمة غير صفرية في حالة فشل أي منها:

| # | الفحص | ما يثبت |
| --- | --- | --- |
| 1 | **الإصدار** | تُبلغ واجهة سطر الأوامر (CLI) عن إصدارها (`v1.3.0`). |
| 2 | **المنفذ متاح** | المنفذ 5000 متاح، *أو* تم الرد عليه بالفعل من قبل خادم خلفي لـ Chloros يعمل بشكل سليم (ويُعتبر ذلك نجاحًا). |
| 3 | **بدء تشغيل الخادم الخلفي** | يتم تشغيل الملف الثنائي للخادم الخلفي. |
| 4 | **اختبار API (`/api/test`)** | تستجيب الخلفية بـ `status: ok`. |
| 5 | **معلومات النظام** | يُعرض `GPU: <name>, CUDA: <bool>, PyTorch: <version>` من `/api/system-info`. |
| 6 | **نماذج مزيل الضوضاء** | يعثر على نماذج `*.pth.enc` (على نظام Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + مزيل الضوضاء**| ميزة Texture Aware قابلة للاستخدام فعليًّا — تتطلب CUDA**و** ملف نموذج واحد على الأقل. |

تنتهي العملية بـ `N/7 checks passed`، مع سرد أي حالات فشل حسب الاسم.

### 4. معالجة أول مجموعة بيانات لك

```bash
chloros-cli process ~/datasets/flight001
```

***

## الملفات والدلائل

### لكل مستخدم يحتفظ

Chloros ببيانات اعتماده وتكوين واجهة سطر الأوامر (CLI) في دليل واحد مشترك بين الأنظمة، **`~/.chloros/`** (في نظام Windows، `%USERPROFILE%\.chloros\`). وهناك ذاكرتان مؤقتتان خاصتان بنظام Linux تتبعان معايير XDG بدلاً من ذلك — حيث يتم الالتزام بـ `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` عند تعيينهما.

| المسار | الغرض |
| --- | --- |
| `~/.chloros/user_session.json` | ذاكرة التخزين المؤقتة لجلسة تسجيل الدخول التي يتم كتابتها بواسطة `chloros-cli login` (يتم الاحتفاظ بها عبر ترقيات الحزم؛ ويتم مسحها عند إزالة الحزمة) |
| `~/.chloros/working_directory.txt` | تجاوز مجلد المشروع الافتراضي (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | تفضيلات لغة واجهة سطر الأوامر (CLI) (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | إعداد اللغة المشترك مع واجهة المستخدم الرسومية لـ Windows — ويكون لـ `language` هنا الأولوية على `cli_language.json` |
| `~/.chloros/update_cache.json` | ذاكرة تخزين مؤقتة لمدة ساعة واحدة لفحص تحديثات بدء تشغيل Linux/Jetson |
| `~/.chloros/backend.log` | سجل الخلفية عند تشغيل الخلفية بواسطة واجهة سطر الأوامر (CLI) |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | حزم معايرة LATTICE المخزنة مؤقتًا لكل كاميرا، مرتبة حسب الرقم التسلسلي وتجزئة الحزمة |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | تجاوزات المستخدم الاختيارية لملفات تعريف تصحيح السقف في DAQ |
| `~/.config/chloros/system_config.json` | ملف تعريف الأجهزة المخزن مؤقتًا من «التكيف الحسابي الديناميكي» — احذفه لإجبار النظام على إجراء كشف جديد للأجهزة |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | سجلات خادم الخلفية، ملف واحد لكل عملية تشغيل |
| `~/Documents/MAPIR/Chloros Projects/` | مجلد المشروع الافتراضي في حالة عدم تعيين أي تجاوز |

### على مستوى النظام

| المسار | الغرض |
| --- | --- |
| `/usr/bin/chloros-cli` | برنامج نصي مغلف — يعيّن `LD_LIBRARY_PATH` للمكتبات الأصلية المجمعة (و`TMPDIR` إلى `/mnt/ssd/tmp` عند وجود هذا المجلد)، ثم يقوم بتنفيذ الملف الثنائي الفعلي |
| `/usr/bin/chloros-backend` | برنامج نصي مغلف — نفس الشيء بالنسبة للخلفية، حيث يقوم بتمييزها على أنها إصدار إنتاجي مُثبَّت |
| `/usr/lib/chloros/cli/`، `/usr/lib/chloros/backend/` | واجهة سطر الأوامر (CLI) والخلفيّة المُجمَّعتان، المثبتتان بعد فك الضغط (جديد في الإصدار 1.3.0)؛ البرامج هي `cli/chloros-cli` و `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`، `/usr/lib/chloros/chloros-backend` | نصوص برمجية للمشغل مكونة من سطرين تعمل على تشغيل البرامج الفعلية المذكورة أعلاه — يتم الاحتفاظ بها في هذه المسارات لأن حزمة SDK وCLI والنصوص البرمجية القديمة تبحث عنها هنا |
| `/usr/lib/chloros/arena_runtime/` | بيئة تشغيل SDK الخاصة بـ Arena المطلوبة لكاميرات LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | نماذج إزالة الضوضاء المشفرة المستخدمة بواسطة أداة إزالة التباين التي تراعي النسيج (Texture Aware debayer) |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (و `lattice_sdk`، و `lattice_cam`، و `arena_api`، وهي أدوات تحميل المشاريع) | حزمة Python SDK، المثبتة لنظام `python3` والمملوكة للحزمة |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | حزمة Python SDK wheel المطابقة تمامًا لهذا الإصدار، لاستخدامها في بيئة virtualenv |
| `/usr/lib/chloros/exiftool` | مشغل ExifTool المضمّن (يتم ربطه برابط رمزي بـ `/usr/local/bin/exiftool` فقط في حالة عدم وجود exiftool للنظام). اعتبارًا من الإصدار 1.3.0، يأتي البرنامج النصي ExifTool مزودًا بمكتبة Perl خاصة به في `/usr/lib/chloros/exiftool-dist/`، لذا فهو يعمل دون الحاجة إلى حزمة النظام `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | تكوين قناة التحديث الذي يقرأه `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | يضبط `net.ipv4.ip_unprivileged_port_start = 319` بحيث يمكن للخلفية ربط منافذ PTP دون الحاجة إلى صلاحيات الجذر |
| `/etc/sysctl.d/60-chloros-gige.conf` | يزيد سعة مخازن الاستقبال لمآخذ التوصيل في النواة (`net.core.rmem_max`، `net.core.rmem_default`) إلى 32 ميغابايت لتدفقات كاميرات LATTICE، وتطبيقها عند التثبيت (يتم تجاهل أي قيمة أعلى تم تعيينها مسبقًا). في الإعداد الافتراضي للنواة، يتم إسقاط حزم الكاميرا في النواة وتفقد المصفوفة الإطارات ومعدل الإطارات؛ ويقوم `chloros-cli lattice network` بالإبلاغ عن المخازن المؤقتة عندما تكون منخفضة. انظر [أجهزة Linux المضيفة — مخازن الاستقبال للمآخذ](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | يوجه المحمل الديناميكي إلى `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | يمنح المستخدم المسجل الدخول حق الوصول إلى جسر DAQ-U التسلسلي عبر USB (CP2102N، `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | خدمة خلفية تعمل دائمًا قابلة للاشتراك تعمل الخلفية باسم المستخدم الذي تحدده (مثبتة، **غير ممكّنة**) — الخدمة التي يجب تمكينها، انظر [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | نفس الخدمة التي تشغل الخلفية بصفتها **root**(مثبتة،**غير ممكّنة**)، يتم الاحتفاظ بها حتى يظل الجهاز الذي قام بتمكينها قيد التشغيل |
| `/var/log/chloros/` | السجل التفصيلي لخدمة الجذر، الذي يمكن قراءته من قبل مجموعة `adm` (يتم حذفه عند إزالة الحزمة) |
| `/usr/share/applications/chloros-cli.desktop` | عنصر قائمة التطبيقات «Chloros CLI» الذي يفتح محطة طرفية |

**جدار الحماية (جديد في الإصدار 1.3.0).** إذا كان `ufw` أو `firewalld` نشطًا وقت التثبيت، فإن الحزمة تفتح منافذ UDP `5002` (بيانات DAQ-E)، و`5353` (اكتشاف mDNS)، و`319` و`320` (PTP)، وتزيل تلك القواعد عند إزالة الحزمة. إذا كان جدار الحماية مثبتًا ولكنه غير نشط، فإنه لا يضيف أي شيء ويقوم بطباعة المنافذ التي يجب السماح بها في حالة تمكينه لاحقًا. ولا يفتح أي منفذ TCP: يفشل تحديث البرنامج الثابت لـ DAQ-E، الذي يتصل فيه المستشعر مرة أخرى بهذا الكمبيوتر عبر TCP لجمع الصورة، مع ظهور الخطأ `… device never connected back on TCP <port> …` عندما يحجب جدار الحماية النشط هذا الاتصال.

## موقع الملف القابل للتنفيذ للخلفية

يقوم كل من واجهة سطر الأوامر (CLI) ومجموعة أدوات تطوير البرامج (SDK) بالكشف التلقائي عن الخلفية:

| المكون | المسار |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| الخلفية | `/usr/lib/chloros/chloros-backend` (مشغل لـ `/usr/lib/chloros/backend/chloros-backend`) |

يمكن تجاوز مسار الخلفية باستخدام علامة CLI `--backend-exe` أو معلمة منشئ SDK `backend_exe`، والمنفذ باستخدام `--port` (الافتراضي `5000`).

{% hint style="info" %}
يشير `CHLOROS_BACKEND_URL` إلى مجموعات الأوامر **`lattice`**،**`project`**، و**`daq pool-*`** في الخلفية البعيدة. أما الأوامر الأساسية (`process`، `login`، `logout`، `status`، `export-status`، `time-sync`، `selftest`) تتجاهله عمدًا وتستهدف دائمًا `http://127.0.0.1:<port>`.
{% endhint %}

***

## كاميرات LATTICE ومستشعرات الضوء DAQ على نظام Linux

تعمل جميع مجموعات أوامر live-hardware على نظام Linux (amd64 و Jetson):

* **`chloros-cli lattice`** — اكتشاف كاميرات LATTICE والمصفوفات المتزامنة والاتصال بها وتهيئتها والتقاط الصور منها. يضم `.deb` بيئة تشغيل Arena SDK التي تتطلبها هذه الكاميرات ويسجلها لدى المحمل الديناميكي.
* **`chloros-cli daq pool-*`** — توصيل مستشعرات الضوء DAQ-U/M/E عبر تجمع الخلفية، وبث الأطياف المعايرة، وتسجيل ملفات `.daq` وتبديل ملفات تعريف الغطاء، وبالنسبة لـ DAQ-E، تغيير إعدادات الشبكة الخاصة به وإعادة تشغيله وتحديث البرنامج الثابت الخاص به. يقوم `chloros-cli daq discover` بالبحث عن المستشعرات التي يمكنك توصيلها، ويقوم `chloros-cli daq --help` بإدراج قائمة بالأوامر الفرعية لـ `pool-*`.
* **`chloros-cli project`** — تشغيل مشروع محفوظ (كاميراته وأجهزة الاستشعار وإعدادات المعالجة) بدون واجهة مستخدم.
* **`chloros-cli time-sync`** — فحص «الجراندماستر» PTP الذي تعمل عليه الخلفية Chloros لكاميرات LATTICE وأجهزة استشعار DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` مطلوب من قبل `pool-latest`، و`pool-stream`، و`pool-record`، و`pool-set-cap`؛ ويُظهر `pool-list` المعرّفات الموجودة حاليًا في المجموعة.

{% hint style="info" %}
**`--eth-host` يتخطى عملية الاكتشاف.** يكتشف `pool-connect --eth` جهاز DAQ-E عبر mDNS، مع احتياطي ARP يعمل من ذاكرة التخزين المؤقتة الباردة لـ ARP؛ ويؤدي تمرير عنوان IP أو اسم المضيف الخاص بالمستشعر مع `--eth-host` إلى الاتصال به مباشرةً.
{% endhint %}

**أذونات المسلسل DAQ-U** تتم معالجتها بواسطة قاعدة udev المثبتة (`uaccess` + المجموعة `dialout`). إذا ظل جهاز استشعار موصولًا بالفعل غير قابل للوصول، فقم بإعادة تحميل القواعد أو إعادة توصيله:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

راجع [مرجع واجهة CLI](../CLI.md) للاطلاع على مجموعة الأوامر الكاملة.

### PTP دائم التشغيل لأجهزة المضيف غير المزودة بشاشة

تقوم الحزمة بتثبيت خدمتين من systemd للخلفية، وكلاهما غير ممكّن. على جهاز Jetson بدون شاشة أو خادم يجب أن يحافظ على تشغيل مزامنة الوقت PTP بشكل مستمر لمستشعرات DAQ-E وكاميرات LATTICE، أو حيث تقوم نصوص SDK بتوصيل المشاريع المحفوظة (لا يقوم `connect_all()` بتشغيل الخلفية أبدًا)، قم بتمكين الخدمة **بصفتك المستخدم**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

تقوم `chloros-backend@<user>.service` بتشغيل الخلفية باسم ذلك المستخدم: عمليات الالتقاط التي تكتبها تخصه، وسجلها التفصيلي موجود في `~/.cache/chloros/logs` الخاص به، وتستخدم `chloros-cli login` (`~/.chloros`)، لذا قم بتسجيل الدخول مرة واحدة باسم هذا المستخدم أولاً. يؤدي تثبيت الحزمة إلى عرض هذا الأمر مع ملء اسم المستخدم الخاص بك.

بدون خدمة، لا تعمل الخلفية (ومعها «الماستر الرئيسي» لـ PTP) إلا عندما يتم تشغيلها — بواسطة أمر «`chloros-cli`» أو برنامج نصي لـ SDK.

تربط الوحدة الخلفية بإعدادات بيئة «`127.0.0.1:5000`» (إعدادات بيئة `CHLOROS_HOST` / `CHLOROS_PORT` داخل الوحدة؛ يمكن تجاوزها باستخدام `sudo systemctl edit chloros-backend@.service`) وتعيد تشغيلها في حالة الفشل بعد 5 ثوانٍ.

**خدمة الجذر.**يقوم `chloros-backend.service` بتشغيل الخلفية بصفة**الجذر**. ولا تزال هذه الخدمة مثبتة بحيث يستمر الجهاز الذي قام بتمكينها في العمل، ولكن كل ما تكتبه الخلفية التي تعمل بصفة الجذر ينتمي إلى الجذر، لذا لا يمكن للمستخدم حذف لقطاته الخاصة؛ ويوجد سجلها في `/var/log/chloros`، ويمكن قراءته بواسطة مجموعة `adm`. للتبديل إلى حساب المستخدم الخاص بك:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**الخلفية الخاصة بالخدمة مشتركة، ولا يتم إيقافها أبدًا بواسطة العميل.** تستخدمها نصوص `chloros-cli` وSDK كما تجدها: يعمل `process` عليها، ويقوم بتحميل مشروعه الخاص بدلاً من آخر مشروع. حيث إن واجهة سطر الأوامر (CLI) كانت ستعيد تشغيل الخلفية (`--restart`، أو `process` التي لم يعثر استيرادها على شيء)، فإنه يحدد اسم الوحدة، ويطبع `sudo systemctl restart <unit>` ويخرج من `1`، لأن الخلفية التي تتوقف بهذه الطريقة ستظل معطلة. أعد تشغيلها بنفسك باستخدام `sudo systemctl restart chloros-backend@$USER.service`. لا تتوقف الخلفية الخاصة بالخدمة أبدًا من تلقاء نفسها عند الخمول، ويرفض `chloros-cli backend stop` ذلك، مع تسمية الأمر `systemctl stop`.**كيف يحصل PTP على منافذه.** يستخدم PTP منافذ UDP 319/320، وكلاهما أقل من الحد الأدنى المعتاد للمنافذ المميزة البالغ 1024. يقوم `postinst` الخاص بالحزمة بكتابة `/etc/sysctl.d/60-chloros-ptp.conf` باستخدام `net.ipv4.ip_unprivileged_port_start = 319`، مما يسمح للخلفية بربطهما أثناء التشغيل بصفتك المستخدم. اعتبارًا من الإصدار 1.3.0، لا تقوم الحزمة بتعيين أي قدرات ملفات (`setcap`) على الخلفية، ولم يعد `libcap2-bin` تابعًا: في الخلفية غير المضغوطة، قد تمنع قدرة الملف تشغيلها، لذا لا تقم بإضافة واحدة بنفسك.

***

## أمثلة على البرمجة النصية في Bash

{% hint style="info" %}
**رموز الخروج الملائمة للبرامج النصية.**`chloros-cli process` يخرج من `0` عند النجاح و**بقيمة غير صفرية عند الفشل — بما في ذلك التشغيل الذي طلب منتجات صور ولكنه لم يكتب أي منها** (يُظهر `Processing finished but wrote no image products.` ويذكر اسم مجلد المشروع والأسباب المعتادة). تُبلغ عمليات التشغيل الناجحة عن عدد منتجات الصور التي تمت كتابتها (`Image products written: N`، بالإضافة إلى `(plus M clamp masks)` عند كتابة الأقنعة). رموز الخروج: `0` للنجاح، و`1` للفشل، و`2` لخطأ في الوسيطة، و`130` للتوقف المفاجئ.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

القيم الصالحة لـ `--format` هي أربع قيم بالضبط، وتحتوي على مسافات — لذا يجب دائمًا وضعها بين علامتي اقتباس:

| قيمة `--format` | مجلد الإخراج |
| --- | --- |
| `TIFF (16-bit)` *(افتراضي)* | `tiff16` |
| قيمة `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` يقبل `standard` (الافتراضي) أو `texture-aware` (Chloros+).

### المعالجة التلقائية باستخدام Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### مثال على PythonSDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## استكشاف الأخطاء وإصلاحها

### عدم العثور على واجهة سطر الأوامر (CLI) بعد التثبيت

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### عدم بدء تشغيل الخلفية بعد إضافة إمكانيات الملفات

لا تقم بتشغيل `setcap` على الخلفية. اعتبارًا من الإصدار 1.3.0، لا تحدد الحزمة أي إمكانيات ملفات، وتأتي منافذ PTP 319/320 من إعداد `sysctl` الموصوف في [كيف يحصل PTP على منافذه](#always-on-ptp-for-headless-hosts). تضع إمكانية الملف الخلفية غير المضغوطة في وضع التنفيذ الآمن الذي لا يمكنها فيه العثور على مكتباتها الخاصة، وبالتالي لا يتم تشغيلها. إذا قمت بإضافة واحدة، فقم بإزالتها:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP لا يعمل / لا يمكن ربط المنفذ 319

تأكد من خفض الحد الأدنى للمنفذ غير المتميز، وأعد تطبيقه على عملية التشغيل الحالية إذا لم يكن قد تم ذلك:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

ثم تحقق من «الجراندماستر»:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice` فشل الأوامر مع ظهور رسالة «لم يتم العثور على برامج تشغيل كاميرا LATTICE»

تعذر على الخلفية تحميل بيئة تشغيل كاميرا LATTICE المثبتة معها. يتم تشغيل كل أمر `lattice` على الخلفية؛ ولا تحتوي واجهة CLI نفسها على أي برامج تشغيل للكاميرا. أعد تثبيت Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): يقوم `.deb` بتثبيت بيئة التشغيل بجانب الخلفية، في `/usr/lib/chloros/arena_runtime`، وتسجيلها لدى المحمل الديناميكي. للتحقق من هذا التسجيل:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
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

توجد سجلات الخلفية الخاصة بفشل التشغيل في `~/.cache/chloros/logs/`. إذا كان المنفذ 5000 تابعًا لخدمة خلفية (`systemctl status chloros-backend@$USER.service`)، فقم بإعادة تشغيل الخدمة بدلاً من إنهاء العملية الخاصة بها.

### لم يتم الكشف عن CUDA

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

يُبلغ ملف `chloros-cli selftest` عن نفس المشكلة في سطر واحد: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### المكتبات المشتركة المفقودة

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### بطء بدء التشغيل على أنظمة بطاقات SD

اعتبارًا من الإصدار 1.3.0، يتم تثبيت الخلفية وواجهة سطر الأوامر (CLI) دون فك ضغط، لذا لم تعدا تقومان بفك ضغط نفسيهما إلى دليل مؤقت عند كل تشغيل. تُنقل الملفات المؤقتة إلى `/mnt/ssd/tmp` تلقائيًا عند وجود هذا المجلد؛ وإلا يمكنك توجيهها إلى نظام ملفات سريع:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## تحديث Chloros على Linux

الأمر `update` مخصص لنظامي Linux وJetson فقط. وهو يتحقق من الإصدار المنشور في قناة التحديث المُعدة في `/etc/chloros/update.conf` ويعرض تنزيل وتثبيت الإصدار المطابق `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

في نظام Linux/Jetson، تقوم واجهة CLI أيضًا بإجراء فحص تحديث غير معطل عند كل عملية تشغيل (يتم تخزين النتيجة مؤقتًا لمدة ساعة واحدة في `~/.chloros/update_cache.json`) ويعرض `Update available: vX.Y.Z` عند وجود إصدار أحدث. تظل إعداداتك ومشاريعك وبيانات تسجيل الدخول محفوظة بعد التحديث.

تؤدي ترقية الحزمة إلى إيقاف كل خدمة خلفية أثناء التثبيت، لذا يفقد البرنامج النصي قيد التشغيل الكاميرات وأجهزة الاستشعار الخاصة به، ثم يعيد تشغيل كل خدمة تم تمكينها (أما الخدمة التي تم تشغيلها يدويًّا ولكن لم يتم تمكينها أبدًا، فتظل متوقفة، كما هو الحال بعد إعادة التشغيل).

## إلغاء التثبيت

```bash
sudo apt remove chloros
```

يؤدي الإزالة إلى إيقاف وتعطيل كل خدمة خلفية، واستعادة الحد الأدنى الافتراضي للمنافذ غير المتميزة (1024)، وإزالة ملف إعدادات مخزن الاستقبال المؤقت (تظل القيم قيد التشغيل سارية حتى إعادة التشغيل)، والرابط الرمزي لـ exiftool المضمّن، وتكوين محمل Arena، وقواعد جدار الحماية التي أضافتها الحزمة، وPython SDK الذي قامت بتثبيته، كما تقوم بمسح بيانات الاعتماد المخزنة مؤقتًا. يتم ترك مشاريعك وملفات بيانات `~/.chloros/` كما هي. كما يقوم `sudo apt purge chloros` بحذف `/var/log/chloros`.

***

## الخطوات التالية

* [دليل NVIDIA Jetson](nvidia-jetson-guide.md) — التحسين والنشر الخاصان بـ Jetson
* [CLI: سطر الأوامر](../CLI.md) — دليل CLI
* [API: Python SDK](../api-python-sdk.md) — دليل SDK
* [مرجع CLI](../reference/cli-reference.md) و[مرجع SDK](../reference/sdk-reference.md) — قوائم شاملة للأوامر/API للإصدار 1.3.0
* [التكيف الديناميكي للحوسبة](../processing-architecture/dynamic-compute-adaptation.md) — كيف يتكيف نظام Chloros مع أجهزتك
