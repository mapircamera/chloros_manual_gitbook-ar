# البدء السريع لـ CLI (pool-*)

تقوم محركات `chloros-cli` المرفقة بتشغيل مستشعرات DAQ من خلال مجموعة أوامر **`daq pool-*`** — وهي عملاء HTTP خفيفون يعملون على تشغيل المستشعر من خلال مجموعة المستشعرات الدائمة الخاصة بالخلفية (Chloros). تتحكم الخلفية في عملية النقل، لذا تتشارك واجهة المستخدم الرسومية (GUI) وواجهة سطر الأوامر (CLI) ونصوص SDK جميعها في مقبض واحد نشط بدلاً من التنافس على المنفذ. يمكن الوصول إلى كل ما يحتاجه العميل من خلال `pool-*`: الاتصال، والبث، وتسجيل ملفات `.daq` المعايرة، وتبديل ملفات تعريف السعة.

`pool-*` هي واجهة DAQ المدعومة في الإصدارات التي تم طرحها. `chloros-cli daq --help` تسرد الأوامر الفرعية لـ `pool-*`، و`daq discover` / `daq list` يقومان بتنفيذ نفس عملية المسح التي يقوم بها `pool-discover`. عندما لا يتوفر أمر فرعي لـ DAQ للأجهزة المباشرة، فإنه يخرج مع خطأ صريح يحدد الحزمة المفقودة ويوجهك مرة أخرى إلى `pool-*` — لا شيء يفشل بصمت. (لا يوفر `pip install chloros-sdk` أوامر الأجهزة المباشرة أيضًا.)

***

## المتطلبات الأساسية

* **يجب أن تكون الخلفية (Chloros) قيد التشغيل** — أوامر `pool-*` هي عملاء HTTP، وليست برامج تشغيل للأجهزة. في نظام Windows، قم بتشغيل تطبيق سطح المكتب Chloros (حيث يقوم بتشغيل الخلفية). على أنظمة Linux/Jetson التي لا تحتوي على شاشة، قم بتمكين الخدمة باسم المستخدم الخاص بك: `sudo systemctl enable --now chloros-backend@$USER.service` (انظر [تثبيت Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **تسجيل الدخول إلى Chloros+ (المستوى المدفوع)**: قم بتشغيل `chloros-cli login` أولاً. يتم التطبيق من جانب الخادم — بدون تسجيل دخول، تفشل الأوامر مع `401 AUTH_REQUIRED`؛ أما في المستوى المجاني (Iron)، فتفشل مع `403 PLAN_UPGRADE_REQUIRED`.
* تستهدف الأوامر `http://127.0.0.1:5000` افتراضيًا؛ أما عائلة `daq pool-*` فتلتزم بمتغير البيئة `CHLOROS_BACKEND_URL` إذا كان الخادم الخلفي الخاص بك يعمل في مكان آخر.

***

## جلسة مدتها خمس دقائق

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — فتح مستشعر في المجموعة

| المتغير | المعنى |
| --- | --- |
| `daq pool-connect` | الكشف الذكي: البحث عن أي جهاز DAQ على هذا الجهاز. |
| `daq pool-connect --port PORT` | DAQ-U على منفذ تسلسلي محدد (على سبيل المثال: `COM3`، `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | جهاز DAQ-M عبر BLE، تم مسح عنوان MAC تلقائيًا. |
| `daq pool-connect --mac MAC` | DAQ-M على عنوان MAC معروف لشبكة BLE (يشير إلى `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E على اسم مضيف أو عنوان IP معروف — **المسار الموثوق**. |
| `daq pool-connect --eth` | DAQ-E مع الاكتشاف التلقائي (mDNS، مع اللجوء إلى ARP). انظر التحذير أدناه. |

علامات الضبط، جميعها اختيارية:

| العلامة | المعنى |
| --- | --- |
| `--integration-time MS` / `-t MS` | وقت التكامل اليدوي بالمللي ثانية. |
| `--frame-avg N` / `-f N` | متوسط عدد الإطارات لكل طيف مُبلغ عنه. |
| `--no-ae` | تعطيل التعريض التلقائي (يكون التعريض التلقائي (AE) قيد التشغيل افتراضيًا). |
| `--no-stream` | الاتصال دون بدء البث (يمكن استئنافه لاحقًا باستخدام `pool-stream --start`). |
| `--cap-id CAP` | ملف تعريف تصحيح الحد الأقصى؛ الإعداد الافتراضي للخلفية هو `sunshine_cosine`. انظر [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

يؤدي توصيل جهاز DAQ-E موجود بالفعل في المجموعة تحت عنوان آخر خاص به — سواء كان اسم المضيف `daq-e-<id>.local` أو عنوان IP الخاص به — إلى إرجاع الإدخال الموجود في المجموعة بدلاً من فتح الوحدة مرة ثانية.

{% hint style="warning" %}
**تحذير بشأن الاكتشاف التلقائي لـ `--eth`.** على مضيف متعدد المنافذ (يحتوي على أكثر من واجهة شبكة نشطة)، قد يظهر *أول* `pool-connect --eth` بعد التشغيل فارغًا حتى مع سلامة المستشعر — فقد يفوت عملية البحث عن واجهة المستشعر أثناء وجود ذاكرة التخزين المؤقتة لـ ARP في حالة غير نشطة. إذا لم يعثر `--eth` على أي شيء، فحاول مرة أخرى، أو تخطى عملية الاكتشاف تمامًا باستخدام `--eth-host <ip-or-hostname>`، وهو المسار الموثوق به على الأجهزة ذات العناوين المتعددة. اسم مضيف DAQ-E هو `daq-e-<id>.local` (على سبيل المثال: `daq-e-def330.local`)؛ كما أن عنوان IP العادي الخاص به يعمل أيضًا.
{% endhint %}

## `pool-discover` — المسح دون الاتصال

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

يبحث عن أجهزة الاستشعار التي يمكنك الاتصال بها ويعرض عنوان الاتصال بها، دون الاتصال فعليًّا. `--only` يأخذ مجموعة فرعية مفصولة بفواصل من `usb,ble,eth`؛ `--timeout SEC` يحدد وقت المسح لكل وسيلة نقل (الافتراضي 5 ثوانٍ، محدد بين 1–20 ثانية). لا يتم إدراج أجهزة الاستشعار المفتوحة بالفعل في المجموعة (يتوقف جهاز DAQ-M المتصل عن الإعلان) — استخدم `pool-list` لتلك الأجهزة. في الإصدار المُوزَّع، يقوم كل من `chloros-cli daq discover` و `daq list` بتشغيل هذا المسح نفسه.

## `pool-list` — عرض الأجهزة المتصلة

يعرض كل مستشعر في مجموعة الخلفية، بما في ذلك `sensor_id` الذي تحتاجه جميع الأوامر الأخرى:

| الطراز | شكل `sensor_id` | مثال |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 بايت موصولة بشرطة | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

يتم تمييز وحدة DAQ-E التي تم إيقاف تشغيل PTP بها (يتم حفظ الإعداد على الوحدة) في `pool-list` بـ `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` والأمر الذي يعيد تشغيله — انظر [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — قراءة إطارات الطيف

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

تُرجع الإطار الأحدث، أو أحدث إطارات `--recent N`؛ وتُصدر `--json` مخرجات قابلة للقراءة آليًا لاستخدامها في البرمجة النصية. الإطارات عبارة عن إشعاع طيفي (W/m²/nm) مُعاير إشعاعيًا على شبكة مكونة من 135 نقطة، في النطاق 340–1010 نانومتر، مع تطبيق ملف تعريف غطاء المستشعر بالفعل. للحصول على قيم إشعاعية كمية، قم بحساب المتوسط لـ 15 ثانية على الأقل من الإطارات — وهذه خاصية للأداة وليست عيبًا.

## `pool-stream` — إيقاف البث مؤقتًا أو استئنافه

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — تسجيل ملف `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| العلامة | القيمة الافتراضية | المعنى |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | مدة التسجيل بالثواني؛ `0` يعني الاستمرار في التشغيل حتى إصدار الأمر `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | دليل الإخراج. مع وجود الخلفية على هذا الكمبيوتر، يتم تحديد المسار النسبي بناءً على المجلد الذي قمت بتشغيل الأمر فيه؛ أما مع وجود خلفية بعيدة، فيتم تحديد المسار **على الجهاز الذي تعمل عليه الخلفية**. |
| `--device-name NAME` | — | التسمية المخزنة مع التسجيل. |
| `--stop` | — | إيقاف تسجيل قيد التنفيذ. |
| `--confirm-after SEC` | `20` | بعد البدء، انتظر هذه المدة حتى يتم كتابة الصف الأول، وقم بإصدار تحذير بصوت عالٍ (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`) إذا لم يصل أي شيء. `0` يعطل عملية الفحص. |
| `--heartbeat SEC` | `30` | باستخدام `--duration`، اطبع عدد الصفوف كل SEC ثانية وحذر إذا توقف العدد عن الزيادة. `0` يعطل هذه الميزة. |

{% hint style="info" %}
يتم التسجيل في الخلفية، لذا ينتهي المطاف بملف `.daq` في نظام ملفات **الجهاز الخلفي** — بشكل افتراضي في `~/Documents/DAQ Live View/` هناك، وليس بالضرورة في المكان الذي قمت فيه بتشغيل واجهة CLI. تتضمن أسماء الملفات معرّف المستشعر والطابع الزمني.
{% endhint %}

## `pool-set-cap` — تحديد الغطاء المركب

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

يحدد معرّف الغطاء ملف التصحيح المقاس في المصنع والمطبق على كل طيف، و**يجب أن يتطابق مع الغطاء المركب فعليًا على المستشعر** — لا يمكن للمستشعر ولا للبرنامج اكتشاف الغطاء من تلقاء نفسه، ويتم توثيق هذا الاختيار في كل ملف `.daq`. القيمة الافتراضية في كل مكان هي `sunshine_cosine` (يُشحن كل جهاز DAQ مزودًا بغطاء مصحح جيب التمام من Sunshine، بتوهين يبلغ حوالي 12× حسب التصميم — ويؤدي تغيير الغطاء دون الإعلان عنه إلى تصحيح الأطياف بشكل خاطئ بنفس العامل تقريبًا).

| `--cap-id` | متوفر في |
| --- | --- |
| `sunshine_cosine` (الافتراضي) | DAQ-U، DAQ-M، DAQ-E |
| `fov_15`، `fov_45`، `fov_90` | DAQ-U، DAQ-E |
| `fov_30`، `fov_60` | DAQ-U فقط |
| `none` | DAQ-E فقط — انظر الملاحظة |

يتم رفض معرّف الغطاء الذي يقع خارج مجموعة المستشعر عند الاتصال مع ظهور خطأ واضح. `none` (DAQ-E) يعني أن الغطاء قد أُزيل فعليًّا — ولا يزال يطبق ملف تعريف هندسيًّا من المصنع لموزع الضوء الزجاجي المُغلق في DAQ-E، لذا فهو ليس عديم التأثير، ويُعتبر جهاز DAQ-E العاري تكوينًا مختبريًّا، وليس تكوينًا ميدانيًّا مدعومًا. (يُعتبر جهاز DAQ-U العاري عاريًا تمامًا ولا يحتاج إلى ملف تعريف تصحيح على الإطلاق؛ أما جهاز DAQ-M فيُستخدم مع غطاء Sunshine الخاص به.)

## `pool-disconnect` — تحرير المستشعرات

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` يحرر المستشعر بغض النظر عمن فتحه، لذا فإنه يحرر أيضًا المستشعر الذي تركه برنامج نصي تعطل في المجمع (بشرط الانتهاء أولًا من التسجيل عليه). أما المستشعر الذي ليس موجودًا في المجمع فلا يوجد ما يتم تحريره: فيقوم الأمر بطباعة `<sensor-id>: already released (not in the pool)` ثم يخرج من `0`. أما عملية التحرير التي رفضتها الخلفية أو لم تؤكدها، فتخرج من `1`.

## `pool-enable-ptp` — إعادة تشغيل PTP في وحدة DAQ-E

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

يحتفظ جهاز DAQ-E بإعدادات تشغيل/إيقاف PTP في ذاكرته الخاصة، لذا تظل الوحدة التي تم إيقاف تشغيل PTP بها في حالة إيقاف — ولا تتم مزامنة طوابعها الزمنية مع PTP — حتى يتم إعادة تشغيلها. يقوم `pool-list` بتمييز مثل هذه الوحدة وطباعة هذا الأمر لها. لا يقوم برنامج Chloros أبدًا بإعادة تشغيل الوحدة من تلقاء نفسه، لأن إيقاف التشغيل قد يكون متعمدًا. جديد في الإصدار 1.3.0.

## إعدادات جهاز DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` يعرض البرنامج الثابت للمستشعر (وما إذا كان إصدار Chloros هذا يحتوي على نسخة أحدث منه)، وعنوانه وكيفية حصوله عليه، وإعدادات الشبكة المحفوظة في حال اختلافها عن الإعدادات الحالية، والبث عند التشغيل، و PTP. `pool-set-network` يعيد تشغيل المستشعر لتطبيق التغيير ويعيد توصيله بالإعدادات التي كان عليها؛ بينما `--no-reboot` يحفظها فقط. يجب أن يكون العنوان الثابت موجودًا على شبكة هذا الكمبيوتر، وإلا فإن Chloros يرفضه؛ `--force` يتجاوز ذلك، فقط بالنسبة للكمبيوتر الذي سيتم إعادة توجيهه أو توجيهه إليه. يتم رفض عمليات إعادة التشغيل أثناء قيام المستشعر بالتسجيل. `pool-update-firmware` هو ما يعادل خيار **التحديث إلى \<version\>** في واجهة المستخدم الرسومية (GUI) بالنسبة لسطر الأوامر، وهو الطريقة لتحديث DAQ-E على نظام Linux، الذي لا يحتوي على واجهة مستخدم رسومية. جديد في الإصدار 1.3.0.

***

## ملخص الأوامر

| الأمر | الغرض |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | فتح مستشعر في مجموعة الخلفية. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | البحث عن أجهزة الاستشعار دون الاتصال (يبحث عن عنوان MAC الخاص بتقنية BLE لجهاز DAQ-M). |
| `daq pool-list` | عرض كل مستشعر في المجموعة مع `sensor_id` الخاص به. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | أحدث N إطارات طيفية تمت معايرتها. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | استئناف / إيقاف مؤقت للبث. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | بدء / إيقاف تسجيل `.daq` (من جانب الخلفية). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | تبديل ملف تعريف تصحيح الحد الأقصى أثناء التشغيل. |
| `daq pool-disconnect --sensor-id ID [--all]` | تحرير مستشعر واحد، أو جميعها. |
| `daq pool-enable-ptp --sensor-id ID` | إعادة تشغيل PTP لجهاز DAQ-E الذي يكون PTP فيه معطلاً. |
| `daq pool-info --sensor-id ID [--json]` | البرنامج الثابت لجهاز DAQ-E، والعنوان، وإعدادات الشبكة المحفوظة، والبث عند التشغيل، وPTP. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP أو عنوان ثابت؛ إعادة تشغيل المستشعر وإعادة توصيله. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | البث عند التشغيل. |
| `daq pool-reboot --sensor-id ID` | إعادة تشغيل جهاز DAQ-E وإعادة توصيله. |
| `daq pool-update-firmware --sensor-id ID` | تثبيت البرنامج الثابت لجهاز DAQ-E الخاص بهذا الإصدار على جهاز استشعار أقدم. |

***

## استكشاف أخطاء الاتصال الأول بجهاز DAQ-E

1. لا يحتوي جهاز DAQ-E على مصباح LED للحالة — تأكد من توفر الطاقة عبر مؤشر PoE/الوصلة الموجود على المحول أو منفذ المحقن، وانتظر بضع ثوانٍ بعد التشغيل حتى يتم التمهيد والانضمام إلى الشبكة.
2. يجب أن يكون الجهاز الخلفي في **نفس مجال البث** الذي يوجد فيه المستشعر — حيث لا يعبر mDNS أجهزة التوجيه.
3. يجب أن يسمح جدار الحماية بمرور بروتوكول mDNS (UDP 5353) وبيانات DAQ-E (UDP 5002) وبروتوكول PTP (UDP 319/320). يضيف برنامج التثبيت الخاص بـ Windows قاعدة لهذا الغرض، أما برنامج Linux `.deb` فيفتحها في جدار حماية `ufw` / `firewalld` كان نشطًا وقت التثبيت؛ يجب السماح بها في جدار الحماية التابع لجهة خارجية، أو في جدار الحماية الذي تم تمكينه لاحقًا.
4. ألا يزال لا يوجد رد من `--eth`؟ استخدم `--eth-host` مع اسم مضيف الجهاز (`daq-e-<id>.local`) أو عنوان IP الخاص به، مما يتخطى عملية الاكتشاف.

***
{% hint style="info" %}
**نصيحة لمساعدات الذكاء الاصطناعي.** يتم تقديم كل صفحة من هذا الدليل بتنسيق Markdown الخام — أضف `.md` إلى عنوان URL المختصر للصفحة بالخط الصغير (هذه الصفحة: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`)؛ الفهرس القابل للقراءة آليًا هو `https://mapir.gitbook.io/chloros/llms.txt`. للحصول على الوثائق الكاملة على مستوى العلامات الخاصة بـ `chloros-cli daq` وكل مجموعة أوامر أخرى، احصل على [مرجع واجهة سطر الأوامر (CLI)](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`)؛ مسار Python هو `chloros_sdk.connect_daq_sensor()` في [مرجع SDK](../reference/sdk-reference.md).
{% endhint %}
