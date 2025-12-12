# الكلوروس + تسجيل الدخول

## دخول الكلوروس والكلوروس (المتصفح).

تسمح لك قائمة الشريط الجانبي <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> للمستخدم بتسجيل الدخول إلى حساب Chloros+ الخاص بك وفتح ميزات إضافية.

عند تسجيل الدخول ستظهر تفاصيل حسابك:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## تسجيل الدخول CLI

قم بتسجيل الدخول باستخدام بيانات اعتماد Chloros+ الخاصة بك لتمكين معالجة CLI.

**بناء الجملة:**

```bash
chloros-cli login <email> <password>
```

**مثال:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% تلميح النمط = "تحذير" %}
**الأحرف الخاصة**: استخدم علامات الاقتباس المفردة حول كلمات المرور التي تحتوي على أحرف مثل `$`، أو `!`، أو مسافات.
{% نهاية %}

**الإخراج:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### انتهاء صلاحية الخطة

توضح انتهاء صلاحية الخطة في واجهة المستخدم الرسومية متى سيصبح ترخيصك غير صالح. بالنسبة للاشتراكات الشهرية المتكررة، تنتهي صلاحية الاشتراك في نهاية الشهر. بالنسبة للاشتراكات السنوية، يكون ذلك بعد عام من بدء الاشتراك. يتطلب فحص الترخيص اتصالاً شهريًا بالإنترنت للتحقق، مع فترة سماح مدتها 30 يومًا.

### حد الجهاز

تقدم كل خطة Chloros+ عددًا مختلفًا من الأجهزة المسجلة. سيتم احتساب كل جهاز تقوم بتسجيل الدخول إليه باستخدام حساب Chloros+ ضمن عدد أجهزتك المسجلة. يمكنك إعادة تسمية وإزالة جهاز على صفحة حساب MAPIR Cloud الخاصة بك.

<table><thead><tr><th width="168.5999755859375" align="right">كلوروس+ الخطة</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">الأجهزة مدعوم</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
