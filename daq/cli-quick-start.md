# CLI البدء السريع (pool-*)

تقوم محركات `chloros-cli` المرفقة بتشغيل مستشعرات DAQ من خلال **`daq pool-*`** — وهي عبارة عن عملاء HTTP خفيفة تعمل على تشغيل المستشعر من خلال مجموعة المستشعرات الدائمة الخاصة بالخلفية Chloros. تمتلك الخلفية عملية النقل، لذا فإن واجهة المستخدم الرسومية (GUI) وبرامج CLI وSDK النصية تتشارك جميعها معالجًا واحدًا نشطًا بدلاً من التنافس على المنفذ. يمكن الوصول إلى كل ما يحتاجه العميل من خلال `pool-*`: الاتصال، والبث، وتسجيل ملفات `.daq` المعايرة، وتبديل ملفات تعريف السعة.

كما أن `pool-*` هو **الوحيد** من بين واجهات DAQ في الإصدارات المنشورة. يعرض `chloros-cli daq --help` قائمة بالأوامر الفرعية لـ `pool-*`، ويؤدي استدعاء أمر فرعي لـ DAQ مباشر للأجهزة في إصدار تم توزيعه إلى الخروج مع خطأ صريح يحدد الحزمة المفقودة ويوجهك مرة أخرى إلى `pool-*` — ولا يحدث أي فشل بصمت. (لا تعمل الأوامر المباشرة للأجهزة إلا من نسخة مصدر MAPIR؛ كما أن `pip install chloros-sdk` لا توفرها أيضًا.)

***

## المتطلبات الأساسية

* **يجب أن يكون الخادم الخلفي Chloros قيد التشغيل** — فأوامر `pool-*` هي عملاء لـ HTTP، وليست برامج تشغيل للأجهزة. على Windows، قم بتشغيل تطبيق سطح المكتب Chloros (حيث يقوم بتشغيل الخلفية). على جهاز Linux/Jetson بدون شاشة، قم بتمكين الخدمة: `sudo systemctl enable --now chloros-backend.service`.
* **تسجيل الدخول إلى Chloros+ (المستوى المدفوع)**: قم بتشغيل `chloros-cli login` أولاً. يتم التطبيق من جانب الخادم — بدون تسجيل دخول، تفشل الأوامر مع `401 AUTH_REQUIRED`؛ وفي المستوى المجاني (Iron) تفشل مع `403 PLAN_UPGRADE_REQUIRED`.
* تستهدف الأوامر `http://127.0.0.1:5000` افتراضيًا؛ أما عائلة `daq pool-*` فتراعي متغير البيئة `CHLOROS_BACKEND_URL` إذا كان الخادم الخلفي الخاص بك يعمل في مكان آخر.

***

## جلسة مدتها خمس دقائق

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

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
| `daq pool-connect --port PORT` | جهاز DAQ-U على منفذ تسلسلي محدد (على سبيل المثال: `COM3`، `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | وحدة DAQ-M عبر BLE، مع مسح تلقائي لعنوان MAC. |
| `daq pool-connect --mac MAC` | DAQ-M على عنوان MAC معروف لشبكة BLE (يشير إلى `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E على اسم مضيف أو عنوان IP معروف — **المسار الموثوق**. |
| `daq pool-connect --eth` | DAQ-E مع الاكتشاف التلقائي (mDNS، مع اللجوء إلى ARP). انظر التحذير أدناه. |

علامات الضبط، جميعها اختيارية:

| العلامة | المعنى |
| --- | --- |
| `--integration-time MS` / `-t MS` | وقت التكامل اليدوي بالمللي ثانية. |
| `--frame-avg N` / `-f N` | متوسط عدد الإطارات لكل طيف مُبلغ عنه. |
| `--no-ae` | تعطيل التعرض التلقائي (التعرض التلقائي (AE) قيد التشغيل افتراضيًا). |
| `--no-stream` | الاتصال دون بدء البث (يمكن استئنافه لاحقًا باستخدام `pool-stream --start`). |
| `--cap-id CAP` | ملف تعريف تصحيح الحد الأقصى؛ الإعداد الافتراضي للخلفية هو `sunshine_cosine`. انظر [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**تحذير بشأن الاكتشاف التلقائي لـ `--eth`.** على مضيف متعدد الوصلات (يحتوي على أكثر من واجهة شبكة نشطة واحدة)، قد يظهر *أول* `pool-connect --eth` بعد التشغيل فارغًا حتى مع سلامة المستشعر — فقد يفوت عملية البحث عن الاكتشاف واجهة المستشعر أثناء وجود ذاكرة التخزين المؤقتة لـ ARP في حالة غير نشطة. إذا لم يعثر `--eth` على أي شيء، فكرر المحاولة، أو تخطى عملية الاكتشاف تمامًا باستخدام `--eth-host <ip-or-hostname>`، وهو المسار الموثوق به على الأجهزة ذات العناوين المتعددة. اسم مضيف DAQ-E هو `daq-e-<id>.local` (على سبيل المثال: `daq-e-def330.local`)؛ كما أن عنوان IP العادي الخاص به يعمل أيضًا.
{% endhint %}

## `pool-list` — عرض الأجهزة المتصلة

يعرض كل مستشعر موجود في مجموعة الخلفية، بما في ذلك `sensor_id` الذي تحتاجه جميع الأوامر الأخرى:

| الطراز | شكل `sensor_id` | مثال |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 بايت موصولة بشرطة | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — قراءة إطارات الطيف

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

تُرجع الإطار الأحدث، أو أحدث `--recent N` إطارات؛ `--json` تُصدر مخرجات قابلة للقراءة آليًا من أجل البرمجة النصية. الإطارات عبارة عن إشعاع طيفي (W/m²/nm) مُعاير إشعاعيًا على شبكة مكونة من 135 نقطة، في النطاق 340–1010 نانومتر، مع تطبيق ملف تعريف غطاء المستشعر بالفعل. للحصول على قيم الإشعاع الكمية، احسب المتوسط لـ 15 ثانية على الأقل من الإطارات — وهذه خاصية للجهاز وليست عيبًا.

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
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | دليل الإخراج، يتم تحديده **على الجهاز الذي يعمل عليه الخادم الخلفي**. |
| `--device-name NAME` | — | التسمية المخزنة مع التسجيل. |
| `--stop` | — | إيقاف تسجيل قيد التنفيذ. |

{% hint style="info" %}
يتم التسجيل في الخلفية، لذا فإن ملف `.daq` يُخزّن في نظام ملفات **الجهاز الذي يعمل عليه الخادم الخلفي** — بشكل افتراضي في `~/Documents/DAQ Live View/` هناك، وليس بالضرورة في المكان الذي قمت فيه بتشغيل CLI. تتضمن أسماء الملفات معرف المستشعر والطابع الزمني.
{% endhint %}

## `pool-set-cap` — تحديد الغطاء المركب

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

يحدد معرّف الغطاء ملف التصحيح المقاس في المصنع والمطبق على كل طيف، ويجب **أن يتطابق مع الغطاء المركب فعليًا على المستشعر** — لا يمكن للمستشعر ولا للبرنامج اكتشاف الغطاء من تلقاء نفسه، ويتم توثيق هذا الاختيار في كل ملف `.daq`. القيمة الافتراضية في كل مكان هي `sunshine_cosine` (يُشحن كل جهاز DAQ مزودًا بغطاء مصحح جيب التمام من Sunshine، مع توهين يبلغ حوالي 12× حسب التصميم — ويؤدي تغيير الغطاء دون الإعلان عنه إلى تصحيح الأطياف بشكل خاطئ بنفس العامل تقريبًا).

| `--cap-id` | متوفر في |
| --- | --- |
| `sunshine_cosine` (الافتراضي) | DAQ-U، DAQ-M، DAQ-E |
| `fov_15`، `fov_45`، `fov_90` | DAQ-U، DAQ-E |
| `fov_30`، `fov_60` | DAQ-U فقط |
| `none` | DAQ-E فقط — انظر الملاحظة |

يتم رفض معرّف الغطاء الذي يقع خارج مجموعة المستشعر عند الاتصال مع ظهور خطأ واضح. `none` (DAQ-E) يعني أن الغطاء قد أُزيل فعليًّا — ولا يزال يُطبق ملف تعريف هندسي مصنعيًّا لموزع الضوء الزجاجي المُغلق في DAQ-E، لذا فهو ليس عملية غير فعالة (no-op)، ويُعتبر جهاز DAQ-E العاري تكوينًا مختبريًّا، وليس تكوينًا ميدانيًّا مدعومًا. (يُعتبر جهاز DAQ-U «عاريًا» بالفعل ولا يحتاج إلى ملف تعريف تصحيح على الإطلاق؛ أما جهاز DAQ-M فيُستخدم مع غطاء Sunshine الخاص به.)

## `pool-disconnect` — تحرير المستشعرات

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## ملخص الأوامر

| الأمر | الغرض |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | فتح مستشعر في مجموعة الخلفية. |
| `daq pool-list` | عرض كل مستشعر في المجموعة مع `sensor_id` الخاص به. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | أحدث N إطارات طيفية معايرة. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | استئناف / إيقاف البث مؤقتًا. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | بدء / إيقاف تسجيل `.daq` (من جانب الخلفية). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | تبديل ملف تعريف تصحيح الحد الأقصى أثناء التشغيل. |
| `daq pool-disconnect --sensor-id ID [--all]` | تحرير مستشعر واحد أو جميعها. |

***

## استكشاف أخطاء الاتصال الأول بـ DAQ-E

1. لا يحتوي جهاز DAQ-E على مصباح LED للحالة — تأكد من توفر الطاقة عبر مؤشر PoE/الربط الموجود على المحول أو منفذ المحقن، وانتظر بضع ثوانٍ بعد التشغيل حتى يتم التمهيد والانضمام إلى الشبكة.
2. يجب أن يكون الجهاز الخلفي في **نفس مجال البث** الذي يوجد فيه المستشعر — حيث لا يعبر مDNS أجهزة التوجيه.
3. في Windows، اقبل مطالبة جدار الحماية Defender عند التشغيل الأول (mDNS UDP 5353، بيانات DAQ-E UDP 5002، PTP UDP 319/320).
4. ألا يزال جهاز `--eth` لا يستجيب؟ استخدم جهاز `--eth-host` مع اسم مضيف الوحدة (`daq-e-<id>.local`) أو عنوان IP — فهذا هو المسار الموثوق به، خاصةً على المضيفات متعددة العناوين.

***{% hint style="info" %}**نصيحة لمساعدات الذكاء الاصطناعي.** يتم تقديم كل صفحة من هذا الدليل بتنسيق Markdown الخام — أضف `.md` إلى رابط الصفحة (slug) المكتوب بأحرف صغيرة URL (هذه الصفحة: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`)؛ فسيكون الفهرس القابل للقراءة آليًّا هو `https://mapir.gitbook.io/chloros/llms.txt`. للحصول على الوثائق الكاملة على مستوى العلامات الخاصة بـ `chloros-cli daq` وكل مجموعة أوامر أخرى، احصل على [مرجع CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`)؛ المسار الخاص بـ Python هو `chloros_sdk.connect_daq_sensor()` في [مرجع SDK](../reference/sdk-reference.md).
{% endhint %}
