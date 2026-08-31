# استخدام Chloros مع المساعدات الذكية

تم إعداد هذا الدليل لفئتين من المستخدمين: البشر، والمساعدات الذكية التي يعتمد عليها البشر بشكل متزايد في عملهم. تنشر كل صفحة القيم الدقيقة والإعدادات الافتراضية والأوامر القابلة للنسخ واللصق، بحيث يمكن للمساعد (مثل Claude أو ChatGPT أو Copilot أو وكيل البرمجة، ...) كتابة أتمتة Chloros فعالة من المحاولة الأولى.

إصدار Chloros: **

1.2.0**. منصات CLI/SDK: Windows 10/11 x64 وLinux (x86\_64 / Jetson aarch64).

## ما الذي يجب أن تسلمه لمساعدك

| المورد | URL | الغرض منه |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | فهرس قابل للقراءة آليًا لكل صفحة في هذا الدليل. |
| **CLI مرجع** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | واجهة أوامر `chloros-cli` الكاملة: كل أمر، وعلامة، وإعداد افتراضي، ورمز خروج، وقاعدة مجلد الإخراج. مكتوب لاستخدام نماذج اللغة الكبيرة (LLM). |
| **مرجع SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | `chloros_sdk` Python API الكامل: الفئات، والتوقيعات، والاستثناءات، والأمثلة العملية. مكتوب لاستخدام طلاب الماجستير في القانون (LLM). |
| **أي صفحة بتنسيق Markdown الخام** | أضف `.md` إلى الصفحة URL | على سبيل المثال `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` يعرض الصفحة بتنسيق Markdown الخام — وهو مثالي للصقها في نافذة السياق أو استردادها من وكيل. |

الروابط داخل الدليل: [CLI مرجع](reference/cli-reference.md) · [SDK مرجع](reference/sdk-reference.md).

{% hint style="info" %}
صفحتا المرجع هاتان مستقلتان بذاتهما: فلا يحتاج المساعد الذي قرأ إحداهما إلى بقية الدليل لكتابة نصوص برمجية صحيحة.
{% endhint %}

## وصفات سريعة

انسخ `<placeholders>`، وأكمل الفراغات، ثم الصق النص في مساعدك.

### 1. معالجة مجلد رحلة جوية في NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. المراقبة الدفعية لدليل «captures»

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. توصيل مصفوفة LATTICE والتقاط البيانات

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. تسجيل أطياف مستشعر الضوء DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
تتم عمليات البرمجة النصية لـ DAQ من سطر الأوامر دائمًا عبر مجموعة `daq pool-*` (`pool-connect`، `pool-list`، `pool-latest`، `pool-stream`، `pool-record`، `pool-set-cap`، `pool-disconnect`). أما الأوامر الفرعية الأخرى لـ `daq` التي قد يبتكرها مساعدك، فهي غير متوفرة في الإصدارات الموزعة وتؤدي إلى الخروج مع ظهور خطأ.
{% endhint %}

## لماذا تعمل البرامج النصية المكتوبة بواسطة الذكاء الاصطناعي بشكل جيد مع Chloros

كل من هذه الأمور يمثل سلوكًا حقيقيًّا ومُثبتًا لـ Chloros 1.2.0 — فهي تقضي على أنماط الفشل التقليدية للأتمتة المكتوبة آليًّا:

* **لا حاجة إلى إجراءات إعداد معقدة.**تقوم أدوات المساعدة للاتصال الذكي في SDK (`connect_camera`، `connect_array`، `connect_daq_sensor`) ونقاط دخول المعالجة (`ChlorosLocal`، `process_folder`)**تقوم بتشغيل الخلفية المحلية تلقائيًا**. لا يحتاج البرنامج النصي الذي تم إنشاؤه إلى فتح واجهة المستخدم الرسومية (GUI) أو خادم تم تشغيله يدويًّا — بل يحتاج فقط إلى تثبيت حزمة desktop/CLI.
* **خط الأنابيب بأكمله عبارة عن استدعاء واحد.** يقوم `chloros_sdk.process_folder("path", indices=["NDVI"])` بتشغيل الاستيراد → المعايرة → الانعكاس → تصدير المؤشر من البداية إلى النهاية. كلما قلت مساحة السطح، قلّت الأماكن التي يمكن أن يحدث فيها خطأ في البرنامج النصي الذي تم إنشاؤه.
* **تقوم عمليات التشغيل التي لا تنتج أي مخرجات بالتشخيص الذاتي.** بعد `process()`، يتم إرفاق ملخص العملية بالنتيجة، كما يتم إعادة إرسال كل تلميح متعلق بالمعالجة (مثل *سبب* عدم إنتاج العملية لأي ناتج) يتم إعادة إصداره أيضًا كـ Python `UserWarning` — لذا حتى البرنامج النصي الذي لا يفحص قاموس النتائج أبدًا يعرض التشخيص.
* **يفشل CLI بشكل واضح.**تُطبع عملية `chloros-cli process` التي طلبت نواتج لكنها لم تكتب أيًا منها `Processing finished but wrote no image products.` و**تخرج برمز خروج غير صفري**، لذا تكتشفها البرامج النصية للصدفة وCI من خلال فحص بسيط لرمز الخروج. أما عمليات التشغيل الناجحة فتُبلغ عن `Image products written: N`.

هناك تباين واحد يجب أن يعرفه المساعد: إن `process()` الخاص بـ SDK **لا** يظهر عمدًا عند تشغيل بدون منتجات — بل يُبلغ عن ذلك من خلال الملخص/التلميحات بدلاً من ذلك. إذا كان يجب أن يتوقف مسار Python عند تشغيل فارغ، فتحقق من الملخص (كما تفعل الوصفة 2).

## تحذيرات

* **يلزم تسجيل الدخول إلى Chloros+.**يتطلب كل من CLI وSDK**مستوى مدفوع** Chloros+، ويتم فرض ذلك من جانب الخادم: تفشل الطلبات باستخدام `401 AUTH_REQUIRED` في حالة عدم تسجيل الدخول، وكذلك مع `403 PLAN_UPGRADE_REQUIRED` في الفئة المجانية. قم بتشغيل `chloros-cli login` مرة واحدة لكل جهاز قبل تشغيل البرامج النصية التي تم إنشاؤها. انظر [Chloros+ تسجيل الدخول](chloros+-login.md).
* **أوامر الالتقاط تتحكم في الأجهزة الفعلية.** تقوم أوامر `lattice` / `daq` / `project` وكائنات الجلسة SDK بالاتصال بالكاميرات وأجهزة الاستشعار المادية وبث الصور منها وتشغيلها. راجع البرنامج النصي الذي تم إنشاؤه قبل تشغيله لأول مرة، وقم بتشغيله في وجود الجهاز.
* **افحص المخرجات بشكل عشوائي.** تحقق من مجلدات النتائج وبعض قيم البكسل قبل نشر النتائج. وعلى وجه الخصوص، يتم تحجيم ملفات TIFF الخاصة بالانعكاسية حسب المصدر — اقرأ علامة XMP الخاصة بـ `Chloros:PixelScale` (LATTICE: 32768 = انعكاس 1.0؛ Survey3: 65535) بدلاً من افتراض وجود مقسوم عليه. توثق كلتا الصفحتين المرجعيتين ذلك تحت عنوان «قراءة بكسلات الانعكاس».
* **مشاكل صغيرة تعطل الكود المُنشأ:**`pool-record` يكتب إلى نظام ملفات**المضيف الخلفي** (الافتراضي `~/Documents/DAQ Live View/`)؛ على الأجهزة التي تحتوي على عدة واجهات شبكة، يُفضل استخدام `daq pool-connect --eth-host <ip-or-hostname>` بدلاً من الاكتشاف التلقائي؛ واستخدم `http://127.0.0.1:5000` (وليس `localhost` أبدًا) في أي مكان يظهر فيه URL كخلفية.
