# تسجيل الدخول إلى Chloros+

## تسجيل الدخول إلى Chloros و Chloros (المتصفح)

تتيح لك <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> تتيح لك قائمة الشريط الجانبي تسجيل الدخول إلى حساب Chloros+ الخاص بك وإلغاء قفل الميزات الإضافية.

عند تسجيل الدخول، ستظهر تفاصيل حسابك:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## تسجيل الدخول إلى CLI

سجل الدخول باستخدام بيانات اعتماد Chloros+ لتمكين معالجة CLI. في Linux (بدون واجهة مستخدم رسومية)، هذه هي الطريقة الوحيدة لتفعيل ترخيصك.

**الصيغة:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**مستخدمو SDK**: يوفر Python SDK أيضًا طريقة برمجية `logout()` لمسح بيانات الاعتماد المخزنة مؤقتًا. انظر [وثائق Python SDK](api-python-sdk.md#logout) للحصول على التفاصيل.
{% endhint %}

**مثال:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**الأحرف الخاصة**: استخدم علامات الاقتباس المفردة حول كلمات المرور التي تحتوي على أحرف مثل `$` أو `!` أو مسافات.
{% endhint %}

**الإخراج:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### تخزين بيانات الاعتماد

يتم تخزين بيانات الاعتماد المخزنة مؤقتًا في موقع خاص بالمنصة:

| المنصة | مسار ذاكرة التخزين المؤقت لبيانات الاعتماد |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### انتهاء صلاحية الخطة

يُظهر انتهاء صلاحية الخطة في واجهة المستخدم الرسومية (GUI) متى ستصبح ترخيصك غير صالح. بالنسبة للاشتراكات الشهرية المتكررة، يكون انتهاء الصلاحية في نهاية الشهر. بالنسبة للاشتراكات السنوية، يكون انتهاء الصلاحية بعد عام من بدء الاشتراك. يتطلب فحص الترخيص اتصالاً شهرياً بالإنترنت للتحقق، مع فترة سماح مدتها 30 يوماً.

### حد الأجهزة

تقدم كل خطة Chloros+ عددًا مختلفًا من الأجهزة المسجلة. سيتم احتساب كل جهاز تقوم بتسجيل الدخول إليه باستخدام حساب Chloros+ ضمن عدد الأجهزة المسجلة لديك. يمكنك إعادة تسمية جهاز أو إزالته من صفحة حساب MAPIR Cloud الخاص بك.

<table><thead><tr><th width="168.5999755859375" align="right">خطة Chloros+</th><th align="center">النحاس</th><th align="center">BRONZE</th><th align="center">فضي</th><th align="center">ذهبي</th></tr></thead><tbody><tr><td align="right">الأجهزة المدعومة</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
