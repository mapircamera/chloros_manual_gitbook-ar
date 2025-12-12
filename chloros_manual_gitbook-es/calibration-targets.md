---
description: لوحات تم قياسها معمليًا تُستخدم لمعايرة البيانات الملتقطة في مرحلة ما بعد المعالجة
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# أهداف المعايرة

يقدم MAPIR أهداف معايرة مختلفة لتغطية مجموعة من التطبيقات. يحتوي جهاز T4-R50 المدمج الموضح أدناه على 4 ألواح تم قياس انعكاس الضوء فيها من 250 إلى 2500 نانومتر.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

تحتوي الأهداف المرجعية المنتشرة T4 على منحنيات الانعكاس التالية، [تنزيل البيانات هنا](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 الانعكاس:: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 الانعكاس :: 400-1000nm</p></figcaption></figure>

بالنظر إلى الرسم البياني للانعكاس، يمكنك أن ترى أن القيم هي الطول الموجي (المحور السيني) مقابل نسبة الانعكاس (المحور الصادي). عندما نلتقط صورة لهدف المعايرة، فإننا نقوم بعد ذلك بإنشاء علاقة بين قيمة البكسل ونسبة الانعكاس، ضمن الطيف الذي يتأثر به كل نطاق من نطاقات مستشعر الكاميرا.

وهذا يعني أنه مع كل صورة تلتقطها بكاميراتنا، يمكنك استخدام صورة لأهداف الانعكاس لدينا، مثل [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) أو [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) لمعايرة الصور من أجل الانعكاس. بمجرد معايرة كل بكسل في الصورة يساوي نسبة الانعكاس.

إذا قمت بإخراج الصور التي تمت معايرتها في Chloros بتنسيق JPG أو TIFF النموذجي، فسيتم حساب نسبة الانعكاس عن طريق قسمة قيمة البكسل على عمق البت لتنسيق الصورة. لذلك بالنسبة لـ JPG نقسم على 255، وبالنسبة لـ TIFF نقسم على 65,535. يمكنك أيضًا اختيار إخراج تنسيق PERCENT في Chloros، وبعد ذلك سيتراوح كل بكسل من قيمة النسبة المئوية من 0.0 إلى 1.0 (انعكاس من 0% إلى 100%). فقط ضع في اعتبارك أن بعض تطبيقات الصور لا يمكنها قبول الصور المئوية (الفاصلة العائمة)، كما أنها كبيرة الحجم من ناحية التخزين.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
