# استخدام Chloros مع المساعدين الذكائيين

تمت كتابة هذا الدليل لفئتين من المستخدمين: البشر، ومساعدات الذكاء الاصطناعي التي يعتمد عليها البشر بشكل متزايد في عملهم. تنشر كل صفحة القيم الدقيقة والإعدادات الافتراضية والأوامر القابلة للنسخ واللصق، بحيث يمكن للمساعد (Claude، ChatGPT، Copilot، وكيل البرمجة، ...) كتابة أتمتة Chloros فعالة من المحاولة الأولى.

إصدار Chloros: **

1.3.0**. منصات CLI/SDK: Windows 10/11 x64 وLinux (x86\_64 / Jetson aarch64).

## ما الذي يجب تزويد مساعدك به

| المورد | URL | الغرض منه |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | فهرس قابل للقراءة آليًّا لكل صفحة في هذا الدليل. |
| **مرجع CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | واجهة الأوامر الكاملة لـ `chloros-cli`: كل أمر، وعلامة، وإعداد افتراضي، ورمز خروج، وقاعدة مجلد الإخراج. مكتوب لاستخدام LLM. |
| **مرجع SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | واجهة برمجة التطبيقات (API) الكاملة لـ `chloros_sdk` بلغة Python: الفئات، والتوقيعات، والاستثناءات، والأمثلة العملية. مكتوب لاستخدام نماذج اللغة الكبيرة (LLM). |
| **أي صفحة بتنسيق Markdown الخام** | أضف `.md` إلى URL الصفحة | على سبيل المثال، يعرض `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` الصفحة بتنسيق Markdown الخام — وهو مثالي للصقها في نافذة السياق أو استردادها من وكيل. |

روابط داخل الدليل: [مرجع CLI](reference/cli-reference.md) · [مرجع SDK](reference/sdk-reference.md).

{% hint style="info" %}
صفحتا المرجع هاتان مكتفيتان بذاتهما: فالمساعد الذي قرأ إحداهما لا يحتاج إلى بقية الدليل لكتابة نصوص برمجية صحيحة.
{% endhint %}

## اجعل مساعدك يشغل Chloros مباشرةً: `chloros-mcp`

كل ما ورد أعلاه يتطلب من مساعدك *كتابة* نص برمجي أو أمر. مع `chloros-mcp`، يقوم المساعد الذي يتحدث [بروتوكول سياق النموذج](https://modelcontextprotocol.io) (MCP)، مثل Claude Desktop أو Claude Code أو Cursor، بتشغيل Chloros نفسه. تطلب بلغة بسيطة فيقوم باستدعاء أدوات الخادم:

* **معالجة الصور:** إنشاء مشروع، واستيراد مجلد من اللقطات، واختيار الإعدادات، والمعالجة، والإبلاغ عما تم تصديره من عملية التشغيل.
* **تشغيل الأجهزة:** توصيل كاميرات LATTICE، والمصفوفات، ومستشعرات الضوء DAQ، والمعاينة، وضبط التعرض واللون، والتقاط الصور، والتسجيل، وتحديث البرامج الثابتة.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

تغطي أدوات الخادم وظائف [Python SDK](api-python-sdk.md). يتم تثبيتها مع حزمة SDK، لذا لا يوجد عادةً أي شيء آخر لتثبيته: في نظام Windows، يقوم مُثبِّت Chloros بوضع `chloros-mcp.exe` في مجلد `Scripts` الخاص بـ Python، أما في نظام Linux، فيضع `.deb` ملف `chloros-mcp` في `/usr/bin`.

**كود Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**أضف هذا إلى `claude_desktop_config.json` (يمكن فتحه من خلال**الإعدادات &gt; المطور &gt; تحرير التكوين**)، ثم أعد تشغيل Claude Desktop.**Cursor** يأخذ نفس الإدخال في `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

إذا أبلغ العميل أن الخادم فشل في التشغيل، فمن المحتمل أن `chloros-mcp` غير موجود في `PATH` الذي يراه: أدخل المسار الكامل بدلاً من ذلك (`where chloros-mcp` في نظام Windows، و`which chloros-mcp` في نظام Linux).

يجب أن يكون برنامج Chloros مثبتًا على نفس جهاز الكمبيوتر وأن يكون لديك حساب مسجل الدخول عليه، سواء في التطبيق أو عبر `chloros-cli login`، على أي فئة مدفوعة من Chloros+ (Copper أو أعلى). لا يمكن للخادم تسجيل الدخول نيابة عنك.

{% hint style="info" %}
إعداد العميل لكل تطبيق، والخيارات، وكل أداة، وكيفية عمل كل عملية: [خادم MCP (مساعدو الذكاء الاصطناعي)](reference/sdk-reference.md#mcp-server-ai-assistants) في مرجع SDK.
{% endhint %}

## وصفات الأوامر

انسخ، واملأ `<placeholders>`، ثم الصق في مساعدك.

### 1. معالجة مجلد رحلة جوية إلى NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. المراقبة الدفعية لمجلد «captures»

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. توصيل مصفوفة LATTICE والتقاط الصور

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

كل من هذه الأمثلة يمثل سلوكًا حقيقيًّا ومُثبتًا لـ Chloros 1.3.0 — فهي تزيل أنماط الفشل الكلاسيكية للأتمتة المكتوبة آليًّا:

* **لا حاجة إلى إجراءات إعداد معقدة.**تعمل أدوات المساعدة للاتصال الذكي في حزمة تطوير البرامج (SDK) (`connect_camera`، `connect_array`، `connect_daq_sensor`) ونقاط دخول المعالجة (`ChlorosLocal`، `process_folder`)**تقوم بتشغيل الخلفية المحلية تلقائيًا**. لا يحتاج البرنامج النصي الذي تم إنشاؤه إلى فتح واجهة المستخدم الرسومية (GUI) أو خادم يتم تشغيله يدويًّا — بل يحتاج فقط إلى تثبيت حزمة سطح المكتب/واجهة سطر الأوامر (CLI).
* **خط الأنابيب بأكمله عبارة عن استدعاء واحد.** يقوم `chloros_sdk.process_folder("path", indices=["NDVI"])` بتنفيذ الاستيراد → المعايرة → الانعكاس → تصدير المؤشر من البداية إلى النهاية. مساحة سطح أقل، وأماكن أقل لحدوث أخطاء في البرنامج النصي الذي تم إنشاؤه.
* **تقوم عمليات التشغيل التي لا تنتج أي مخرجات بالتشخيص الذاتي.** بعد `process()`، يتم إرفاق ملخص العملية بالنتيجة، كما يتم إعادة إرسال كل تلميح متعلق بالمعالجة (مثل *سبب* عدم إنتاج العملية لأي مخرجات) يُعاد إصداره أيضًا كPython `UserWarning` — لذا فإن حتى البرنامج النصي الذي لا يفحص قاموس النتائج أبدًا يعرض التشخيص.
* **تُظهر واجهة سطر الأوامر (CLI) فشلًا واضحًا.**عملية `chloros-cli process` التي طلبت مخرجات لكنها لم تكتب أيًا منها تُظهر `Processing finished but wrote no image products.` و**تخرج بقيمة غير صفرية**، لذا تكتشفها البرامج النصية للصدفة (shell scripts) وبيئة التكامل المستمر (CI) من خلال فحص بسيط لرمز الخروج. تُبلغ عمليات التشغيل الناجحة عن `Image products written: N`، بالإضافة إلى `(plus M clamp masks)` عند كتابة الأقنعة. وبالمثل، تذكر أوامر التسجيل كل ملف لم تتمكن من كتابته وتخرج برمز `1`.
* **المسارات النسبية تعني ما يتوقعه البرنامج النصي.** مع الخلفية المحلية، يتم تحليل المسار النسبي للإخراج (`-o out/` في واجهة سطر الأوامر CLI، و`output_dir="out/"` في حزمة تطوير البرامج SDK) استنادًا إلى المجلد الذي يتم تشغيل الأمر أو البرنامج النصي منه.

هناك تباين واحد يجب أن يعرفه المساعد: إن `process()` في SDK **لا** يُثير استثناءً عمدًا عند التشغيل الذي لا ينتج عنه أي مخرجات — بل يُبلغ عن ذلك من خلال الملخص/التلميحات بدلاً من ذلك. إذا كان يجب أن يتوقف مسار Python عند التشغيل الفارغ، فتحقق من الملخص (كما يفعل الوصفة 2).

## تحذيرات

* **يلزم تسجيل الدخول إلى Chloros+.**تتطلب واجهة سطر الأوامر (CLI) ومجموعة أدوات التطوير (SDK) مستوى**مدفوع** من Chloros+، ويتم فرض ذلك من جانب الخادم: تفشل الطلبات مع الخطأ `401 AUTH_REQUIRED` في حالة عدم تسجيل الدخول، ومع الخطأ `403 PLAN_UPGRADE_REQUIRED` في المستوى المجاني. قم بتشغيل `chloros-cli login` مرة واحدة لكل جهاز قبل تشغيل البرامج النصية التي تم إنشاؤها. انظر [تسجيل الدخول إلى Chloros+](chloros+-login.md).
* **أوامر الالتقاط تتحكم في الأجهزة الفعلية.** `lattice` / `daq` / `project`، وكائنات جلسة عمل SDK، وأدوات `chloros-mcp`، تعمل على الاتصال بالكاميرات وأجهزة الاستشعار المادية وبث البيانات منها وتشغيلها. راجع البرنامج النصي الذي تم إنشاؤه قبل تشغيله لأول مرة، وقم بتشغيله مع وجود الأجهزة في متناول اليد.
* **افحص المخرجات بشكل عشوائي.** تحقق من مجلدات المنتج وبعض قيم البكسل قبل نشر النتائج. وعلى وجه الخصوص، يتم تحجيم ملفات TIFF الخاصة بالانعكاس حسب المصدر — اقرأ علامة XMP الخاصة بـ `Chloros:PixelScale` (تحتوي ملفات LATTICE عليها: 32768 = انعكاس 1.0؛ ملفات Survey3 لا تحتوي عليها، وتستخدم 65535 = 1.0) بدلاً من افتراض وجود مقسوم عليه. توثق كلتا الصفحتين المرجعيتين هذا الأمر تحت عنوان «قراءة بكسلات الانعكاس».
* **مشاكل صغيرة تعرقل الكود المُولَّد:**يكتب `pool-record` إلى نظام ملفات**المضيف الخلفي** (الافتراضي هو `~/Documents/DAQ Live View/`)؛ على الأجهزة التي تحتوي على عدة واجهات شبكة، يُفضل استخدام `daq pool-connect --eth-host <ip-or-hostname>` بدلاً من الاكتشاف التلقائي؛ استخدم `http://127.0.0.1:5000` (وليس `localhost` أبدًا) في أي مكان يظهر فيه URL للخلفية؛ يجب أن يتخطى البرنامج النصي الذي يتنقل عبر شجرة مخرجات المشروع مجلدات `clamp_masks/` (فملفاتها `*_clampmask.tif` هي أقنعة وليست صورًا — انظر [تنسيقات الصور الناتجة](output-image-formats.md#clamp-masks))؛ وفي نظام Windows مع تعطيل المسارات الطويلة، حافظ على قصر أسماء مجلدات المشروع — حيث يتم رفض أي مجلد مشروع يتجاوز عمقه حوالي 130 حرفًا قبل بدء المعالجة.
