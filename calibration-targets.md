---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# أهداف المعايرة

تقدم MAPIR أهداف معايرة متنوعة لتغطية مجموعة واسعة من التطبيقات. يحتوي الطراز T4-R50 المدمج الموضح أدناه على 4 لوحات تم قياس انعكاس الضوء فيها في النطاق من 250 إلى 2,500 نانومتر.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>تتميز أهداف المرجع المنتشرة T4 بمنحنيات الانعكاس التالية، [تنزيل البيانات هنا](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR انعكاس T4 :: 250-2500 نانومتر</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR انعكاسية T4 :: 400-1000 نانومتر</p></figcaption></figure>تتميز الأهداف المرجعية المنتشرة T4P بمنحنيات الانعكاس التالية، [تنزيل البيانات من هنا](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR انعكاسية T4P :: 250-2500 نانومتر</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR انعكاسية T4P :: 400-1000 نانومتر</p></figcaption></figure>بالنظر إلى الرسم البياني للانعكاسية، يمكنك ملاحظة أن القيم تمثل الطول الموجي (المحور السيني) مقابل النسبة المئوية للانعكاسية (المحور الصادي). وعندما نلتقط صورة لهدف المعايرة، فإننا ننشئ علاقة بين قيمة البكسل والنسبة المئوية للانعكاسية، ضمن الطيف الذي يتجاوب معه كل نطاق من نطاقات مستشعر الكاميرا.

وهذا يعني أنه مع كل صورة تلتقطها بكاميراتنا، يمكنك استخدام صورة لأهداف الانعكاسية الخاصة بنا، مثل [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) أو [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) لمعايرة الصور من حيث الانعكاسية. وبمجرد المعايرة، يصبح كل بكسل في الصورة مساويًا لنسبة الانعكاسية المئوية.

بالنسبة لمخرجات **Survey3** ، إذا قمت بإخراج الصور المعايرة في Chloros بتنسيق JPG النموذجي أو TIFF، فإن نسبة الانعكاس تُحسب بقسمة قيمة البكسل على عمق البت لتنسيق الصورة. لذلك، بالنسبة لتنسيق JPG، تقسم على 255، وبالنسبة لـ TIFF، تقسم على 65,535. يمكنك أيضًا اختيار إخراج تنسيق PERCENT في Chloros، وعندها سيتراوح كل بكسل بين قيمة مئوية من 0.0 إلى 1.0 (انعكاس من 0% إلى 100%). فقط ضع في اعتبارك أن بعض تطبيقات الصور لا تقبل الصور ذات النسب المئوية (النقاط العائمة)، كما أنها كبيرة الحجم من حيث سعة التخزين.

{% hint style="info" %}
**يستخدم انعكاس LATTICE مقياس بكسل مختلفًا.** يتم تخزين انعكاس LATTICE بحيث يمثل DN 32768 = 100% انعكاس (وليس 65535)، ويحتوي كل ملف على علامة XMP `Chloros:PixelScale` تحدد مقياسه. اقرأ العلامة وقسم عليها بدلاً من افتراض قيمة ثابتة — انظر [تنسيقات الصور الناتجة](output-image-formats.md).
{% endhint %}

## أهداف المعايرة مع كاميرات LATTICE

مع كاميرات LATTICE، يُعد هدف المعايرة **اختياريًا** بالنسبة للانعكاسية: يمكن لـ Chloros بدلاً من ذلك ربط الانعكاسية بالإشعاع الهابط المقاس بواسطة مستشعر ضوء DAQ (ρ = π·L/E). يتم اختيار المرجع من خلال إعداد مصدر الانعكاسية (إعدادات المشروع في واجهة المستخدم الرسومية؛ `--reflectance-source` في CLI؛ `reflectance_source` في SDK):

| القيمة | السلوك |
| --- | --- |
| `auto` *(افتراضي)* | الهدف الموجود داخل الإطار الذي اجتاز اختبار ضمان الجودة (QA) هو **المرجع المطلق**؛ وعندما لا يتوفر هدف أو يفشل اختبار ضمان الجودة، يلجأ Chloros إلى تقسيم DAQ للضوء المتدفق لأسفل. |
| `target` | الهدف فقط بشكل صارم — لا يوجد استبدال من DAQ. |
| `daq` | DAQ هو المرجع — القياس المنحدر هو المرجع دائمًا. |

سلوك إضافي للأهداف في LATTICE:

* **هندسة الأهداف** — يتم دعم الألواح المُعلمة بـ ArUco، والألواح ذات منطقة الاهتمام (ROI) الثابتة، والأهداف الشريطية؛ وتأتي الهندسة من تكوين الأهداف الخاص بالمشروع.
* **بيانات الهدف المقاسة لكل وحدة** — يشير `--target-reflectance-dir DIR` إلى دليل يحتوي على مسح انعكاس الهدف المقاس لكل وحدة (`<serial>.csv`، يتم البحث عنه بواسطة الرقم التسلسلي/QR لوحدة الهدف). في حالة الفشل، يلجأ Chloros إلى الأطياف الاسمية T3/T4P.
* **التثبيت الزمني** — يقوم الهدف المكتشف بمعايرة الإطارات المحيطة به ويتم الاحتفاظ به بين عمليات رصد الهدف.

توجد دلالات العلامات الكاملة والأمثلة في [مرجع CLI](reference/cli-reference.md) (انظر «مفاتيح التبديل للتصدير لكل منتج»).

### F988

«يتم معايرة انعكاسية F988 باستخدام لوحة انعكاسية داخل المشهد: يقع النطاق خارج النطاق المعاير لمستشعر الضوء DAQ، لذا يقوم Chloros بتطبيق أحدث لقطة للوحة ويحتفظ بها بين عمليات رصد اللوحة.»

إذا تم تشغيل F988 بمعايرة تعتمد على DAQ فقط، فإن Chloros يرفض الانعكاسية المستندة إلى DAQ لهذا النطاق ويوضح السبب (سبب التخطي `dls-uncalibrated-band-988`)؛ ويُعد سير عمل اللوحة هو المسار المدعوم.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
