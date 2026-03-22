---
metaLinks: {}
---

# البدء

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros هو تطبيق برمجي من [MAPIR](https://www.mapir.camera) لمعالجة الصور وبيانات المستشعرات الأخرى.

***{% hint style="success" %}**ما الجديد في Chloros 1.1.0**: دعم Linux الأصلي (amd64 و arm64)، الحوسبة الطرفية NVIDIA Jetson، التكيف الحسابي الديناميكي، خط أنابيب معالجة رباعي الخيوط، أوامر وخيارات CLI جديدة. انظر [التنزيل](download.md) للاطلاع على سجل التغييرات الكامل.
{% endhint %}

يتوفر Chloros في 3 أوضاع للتطبيق:

## Chloros: تطبيق واجهة المستخدم الرسومية لسطح المكتب

نافذة منفصلة مستقلة مع جميع الميزات. _لنظام Windows فقط._

## [Chloros CLI: واجهة سطر الأوامر](CLI.md)

معالجة دفعية عبر سطر الأوامر. مثالي للأتمتة وكتابة البرامج النصية والتشغيل بدون واجهة مستخدم. متوفر على **Windows و Linux amd64 و Linux arm64 (NVIDIA Jetson)**. _تتطلب واجهة سطر الأوامر ترخيص Chloros+ للوصول إليها._

## [Chloros API: Python SDK](api-python-sdk.md)

واجهة Python البرمجية للأتمتة وسير العمل المخصص. مثالية لمسارات البحث، والتكامل مع تطبيقات Python الحالية، وإنشاء أدوات مخصصة. متوفرة على **جميع المنصات** عبر `pip install chloros-sdk`. _تتطلب واجهة برمجة التطبيقات (API) ترخيص Chloros+ للوصول إليها._***

## المنصات المدعومة

| المنصة | واجهة المستخدم الرسومية (GUI) | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | نعم | نعم | نعم |
| **Linux amd64 (x86_64)** | لا | نعم | نعم |
| **Linux arm64 (NVIDIA Jetson)** | لا | نعم | نعم |

للحصول على إرشادات تثبيت Linux، راجع قسم [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

على الرغم من أن Chloros مجاني للاستخدام في معظم المهام، فقد تجد أنك بحاجة إلى المزيد. وهنا يمكن أن تفيدك الترخيص المدفوع لـ Chloros+. مع ترخيص Chloros+، يمكنك فتح ميزات جديدة مثل:

* **المعالجة متعددة الخيوط**: تسريع معالجة الصور بشكل كبير للمشاريع الأكبر حجمًا من خلال معالجة الصور في وقت واحد عبر خط الأنابيب.
* **تسريع GPU (CUDA)**: استفد من خيارات ذاكرة GPU الأعلى المتوفرة حاليًا لزيادة سرعة خط أنابيب معالجة الصور. نوصي بذاكرة VRAM سعة 4 جيجابايت أو أكثر للحصول على أفضل النتائج.
* **Chloros+**[**CLI**](CLI.md)**الوصول**: قم بتشغيل Chloros+ من سطر الأوامر لأتمتة البرنامج ودمجه في برامجك الخاصة.
* **Chloros+**[**API**](api-python-sdk.md)**الوصول:** قم بتشغيل Chloros+ من Python للتحكم البرمجي، مما يتيح التكامل السلس مع مسارات البحث الخاصة بك، وسير عمل تحليل البيانات، والتطبيقات المخصصة.
* **استخدام أجهزة متعددة**: تسمح كل ترخيص Chloros+ بتسجيل جهازين أو أكثر. استخدم حساب MAPIR Cloud الخاص بك لإدارة الأجهزة المسجلة. أضف دعمًا لمزيد من الأجهزة عن طريق ترقية ترخيص Chloros+ الخاص بك.
* **طريقة متقدمة لإزالة التباين مع مراعاة النسيج:** أداة إزالة التباين عالية الجودة تراعي الحواف مقترنة بنموذج إزالة الضوضاء بالذكاء الاصطناعي/التعلم الآلي الذي يزيل تقريبًا كل ضوضاء إزالة التباين. 
* **صيغ مؤشرات متعددة الأطياف مخصصة:** أدخل مؤشرات متعددة الأطياف مخصصة في حاسبات النقطية Chloros، سواء للمعالجة أو لصندوق الرمل الخاص بعرض الصور.
* **Linux والحوسبة الطرفية:** قم بتشغيل Chloros على منصات Linux x86\_64 وARM64 بما في ذلك NVIDIA Jetson للمعالجة الميدانية والطرفية. انظر [نظرة عامة على Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">أسعار Chloros+ والتسجيل</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
