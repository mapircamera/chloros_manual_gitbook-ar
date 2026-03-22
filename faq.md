---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# الأسئلة الشائعة

<details>

<summary>هل يمكنني معالجة صور من كاميرات ليست من ماركة MAPIR باستخدام Chloros؟</summary>

لا، لا يدعم Chloros سوى معالجة صور كاميرات MAPIR. يرجى الاطلاع على قائمة [طرازات الكاميرات المدعومة](supported-cameras.md) لمزيد من المعلومات. نحن نقدم خدمة معالجة الصور من الكاميرات الأخرى على MAPIR Cloud، انظر القائمة الكاملة [هنا](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>هل يمكنني معايرة صوري من حيث الانعكاس بدون هدف معايرة؟</summary>

لا. بدون صورة لهدف المعايرة تم التقاطها في نفس الوقت الذي تم فيه التقاط الصور غير المستهدفة، لن تتمكن من ربط قيم بكسل الصورة بنسبة انعكاس معروفة. إذا لم تقم أيضًا بتضمين السجل من مستشعر الضوء MAPIR، فلن يتم قياس طيف الضوء المحيط، ولن تكون نتائج الانعكاس دقيقة.

</details>

<details>

<summary>هل يمكنني تعديل صوري قبل معالجتها في Chloros؟</summary>

لا. يفترض Chloros أن البيانات المدخلة لم يتم تعديلها. لا تقم بتغيير أسماء الملفات.

</details>

<details>

<summary>هل يمكنني ضبط كاميراتي MAPIR و Survey3 على التعرض التلقائي ومعالجة الصور في Chloros؟</summary>

لا. يجب أن تكون مجموعات بيانات الصور في Survey3 ذات تعريض ثابت/مغلق، لذا لا يمكن استخدام سرعة الغالق التلقائية أو ISO التلقائي. يجب أن تكون جميع الصور من نفس طراز الكاميرا ذات سرعة غالق و ISO (تعريض) متطابقة.

</details>

<details>

<summary>هل يمكن لـ Chloros معالجة أو تحليل الصور الأورثوموسايك؟</summary>

لا. لا يتم دعم سوى صور الكاميرا الفردية في MAPIR، وليس الصور المجمعة مثل خريطة الأورثوموزايك.

</details>

<details>

<summary>كيف يمكنني تسريع خطوة الكشف عن الهدف في Chloros؟</summary>

في جدول مستعرض الملفات، سيؤدي التحديد المسبق للصور المستهدفة في العمود الأيمن إلى إخبار Chloros بالبحث فقط في تلك الصور عن أهداف المعايرة، مما يسرع المعالجة بشكل كبير.

</details>

<details>

<summary>إذا كنت سأقوم بتحميل صوري إلى <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud،</a> فهل يجب عليّ معالجتها في Chloros قبل التحميل؟</summary>

إذا كنت تخطط للتحميل إلى منصة المعالجة عبر الإنترنت [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription)، فلا تقم بتحرير الصور قبل التحميل. ستقوم Cloud بإجراء جميع عمليات المعالجة نفسها وأكثر من ذلك.

</details>

<details>

<summary>هل ستدعم MAPIR ميزة X في أي وقت؟ أتمنى حقًا أن تقدم MAPIR ميزة X.</summary>

نحن مهتمون دائمًا بتلقي التعليقات حول منتجاتنا. إذا وجدت مشكلة في منتجاتنا، أو كان لديك اقتراح حول كيفية تحسين منتجاتنا، يرجى [الاتصال بنا](https://www.mapir.camera/community/contact) لمشاركة أفكارك. يتم توجيه معظم أنشطة البحث والتطوير لدينا من خلال الاستماع إلى أهم احتياجات عملائنا.

</details>

<details>

<summary>هل Chloros متاح لـ Linux؟</summary>

نعم! يدعم Chloros 1.1.0 Linux amd64 (x86\_64) و arm64 (NVIDIA Jetson JetPack 6) عبر حزم `.deb`. يتم دعم CLI و Python و SDK بشكل كامل على Linux. لا توجد واجهة مستخدم رسومية لـ Linux — تتم جميع التفاعلات عبر [CLI](CLI.md) أو [Python SDK](api-python-sdk.md). انظر [نظرة عامة على Linux](linux/linux-overview.md) للحصول على التفاصيل.

</details>

<details>

<summary>هل يمكنني تشغيل Chloros على NVIDIA Jetson؟</summary>

نعم! يدعم Chloros 1.1.0 منصات NVIDIA Jetson بما في ذلك Jetson Nano و Orin Nano و Orin NX و AGX Orin التي تعمل بنظام JetPack 6. يقوم Chloros تلقائيًا باكتشاف طراز Jetson الخاص بك وتحسين استراتيجية المعالجة الخاصة به. راجع [دليل NVIDIA Jetson](linux/nvidia-jetson-guide.md) للحصول على إرشادات الإعداد والنشر.

</details>

<details>

<summary>هل يقوم Chloros بالتحسين تلقائيًا لأجهزتي؟</summary>

نعم! يتضمن Chloros 1.1.0 [التكيف الحسابي الديناميكي](processing-architecture/dynamic-compute-adaptation.md) الذي يكتشف تلقائيًا وحدة المعالجة المركزية (CPU) ووحدة معالجة الرسومات (GPU) وذاكرة الوصول العشوائي (RAM) وأجهزة استشعار الحرارة (على Jetson). ثم يختار استراتيجية المعالجة المثلى — من `GPU_PARALLEL` على الأنظمة ذات الذاكرة العالية إلى `GPU_SINGLE` على الأجهزة المقيدة إلى `CPU_PARALLEL` على الأنظمة التي لا تحتوي على وحدة معالجة رسومات NVIDIA. لا حاجة إلى تكوين يدوي.

</details>

<details>

<summary>ما هو مسار المعالجة ذو 4 خيوط؟</summary>

يستخدم Chloros 1.1.0 بنية مسار معالجة ذات 4 خيوط لمستخدمي Chloros+: يقوم الخيط 1 (الكشف) بتحميل الصور واكتشاف أهداف المعايرة، ويقوم الخيط 2 (المعايرة) بحساب معايرة الانعكاس، ويقوم الخيط 3 (المعالجة) بإجراء إزالة التباين وحساب الفهرس المعجل بواسطة وحدة معالجة الرسومات (GPU)، ويقوم الخيط 4 (التصدير) بكتابة ملفات الإخراج. يمكن أن تكون هناك صور متعددة في خيوط مختلفة في وقت واحد لتحقيق أقصى إنتاجية. انظر [مسار المعالجة](processing-architecture/processing-pipeline.md) للحصول على التفاصيل.

</details>

<details>

<summary>كيف أقوم بتشغيل التشخيصات على تثبيت Chloros الخاص بي؟</summary>

استخدم الأمر `selftest` لتشغيل 7 تشخيصات للنظام بما في ذلك فحص الإصدار، وتوافر المنفذ، وبدء تشغيل الخلفية، واتصال API، ومعلومات النظام، ونماذج إزالة الضوضاء، وتوافر CUDA:

```bash
chloros-cli selftest
```

هذا مفيد بشكل خاص على أنظمة Linux/Jetson للتحقق من إعدادات GPU و CUDA.

</details>
