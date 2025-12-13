---
description: لوحات مقاسة في المختبر تستخدم لمعايرة البيانات الملتقطة في مرحلة ما بعد المعالجة
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---
# أهداف المعايرة

تقدم MAPIR أهداف معايرة متنوعة لتغطية مجموعة من التطبيقات. يحتوي جهاز T4-R50 المدمج الموضح أدناه على 4 لوحات تم قياس انعكاس الضوء منها من 250 إلى 2500 نانومتر.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>تتميز أهداف المرجع المنتشر T4 بمنحنيات الانعكاس التالية، [تنزيل البيانات هنا](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 الانعكاس :: 250-2500 نانومتر</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR انعكاس T4 :: 400-1000 نانومتر</p></figcaption></figure>بالنظر إلى الرسم البياني للانعكاس، يمكنك أن ترى أن القيم هي الطول الموجي (المحور x) مقابل نسبة الانعكاس (المحور y). عندما نلتقط صورة لهدف المعايرة، نقوم بعد ذلك بإنشاء علاقة بين قيمة البكسل ونسبة الانعكاس، ضمن الطيف الذي تستشعره كل نطاقات مستشعر الكاميرا.

هذا يعني أنه مع كل صورة تلتقطها بكاميراتنا، يمكنك استخدام صورة لأهداف الانعكاس الخاصة بنا، مثل [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) أو [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) لمعايرة الصور من حيث الانعكاس. بمجرد المعايرة، يصبح كل بكسل في الصورة مساويًا لنسبة الانعكاس المئوية.

إذا قمت بإخراج الصور المعايرة في Chloros كملف JPG أو TIFF نموذجي، فسيتم حساب نسبة الانعكاس عن طريق قسمة قيمة البكسل على عمق بت تنسيق الصورة. لذلك، بالنسبة لملف JPG، قسّم على 255، وبالنسبة لملف TIFF، قسّم على 65535. يمكنك أيضًا اختيار إخراج تنسيق PERCENT في Chloros، ومن ثم سيتراوح كل بكسل بين قيمة مئوية من 0.0 إلى 1.0 (انعكاسية من 0% إلى 100%). فقط ضع في اعتبارك أن بعض تطبيقات الصور لا تقبل الصور المئوية (النقطة العائمة)، كما أنها كبيرة الحجم من حيث التخزين.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
