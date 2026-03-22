---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# أهداف المعايرة

تقدم MAPIR مجموعة متنوعة من أهداف المعايرة لتغطية مجموعة واسعة من التطبيقات. يحتوي الطراز T4-R50 المدمج الموضح أدناه على 4 لوحات تم قياس انعكاس الضوء لها في النطاق من 250 إلى 2,500 نانومتر.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>تتميز أهداف المرجع المنتشر T4 بمنحنيات الانعكاس التالية، [تنزيل البيانات هنا](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR انعكاس T4 :: 250-2500 نانومتر</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR انعكاس T4 :: 400-1000 نانومتر</p></figcaption></figure>تتميز أهداف المرجع المنتشرة T4P بمنحنيات الانعكاس التالية، [تنزيل البيانات هنا](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR انعكاس T4P :: 250-2500 نانومتر</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR انعكاس T4P :: 400-1000 نانومتر</p></figcaption></figure>بالنظر إلى الرسم البياني للانعكاس، يمكنك ملاحظة أن القيم تمثل الطول الموجي (المحور x) مقابل نسبة الانعكاس (المحور y). عندما نلتقط صورة لهدف المعايرة، فإننا ننشئ علاقة بين قيمة البكسل ونسبة الانعكاس، ضمن الطيف الذي يكون كل نطاق من نطاقات مستشعر الكاميرا حساسًا له.

وهذا يعني أنه مع كل صورة تلتقطها بكاميراتنا، يمكنك استخدام صورة لأهداف الانعكاس الخاصة بنا، مثل [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) أو [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) لمعايرة الصور من حيث الانعكاس. بمجرد المعايرة، يصبح كل بكسل في الصورة مساوياً لنسبة الانعكاس المئوية.

إذا قمت بإخراج الصور المعايرة في Chloros بتنسيق JPG النموذجي أو TIFF، فسيتم حساب نسبة الانعكاس بقسمة قيمة البكسل على عمق البت لتنسيق الصورة. لذلك، بالنسبة لتنسيق JPG، قم بالقسمة على 255، وبالنسبة لتنسيق TIFF، قم بالقسمة على 65,535. يمكنك أيضًا اختيار تنسيق الإخراج PERCENT في Chloros، وعندها سيتراوح كل بكسل بين قيمة مئوية من 0.0 إلى 1.0 (انعكاس من 0% إلى 100%). فقط ضع في اعتبارك أن بعض تطبيقات الصور لا تقبل الصور المئوية (النقطة العائمة)، كما أنها كبيرة الحجم من حيث التخزين.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
