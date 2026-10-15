# اجتياز المسارات وتضمين الملفات (Path Traversal & File Inclusion) — دليل شامل

[![التصنيف](https://img.shields.io/badge/CWE-CWE--22-red)]()
[![الفئة](https://img.shields.io/badge/Category-Path%20Traversal-purple)]()

---

## 1. ما هي ثغرة Path Traversal؟
ثغرة **اجتياز المسارات (Path Traversal / Directory Traversal)** تمكن المهاجم من قراءة ملفات عشوائية مخزنة على نظام تشغيل السيرفر خارج مجلد الويب المسموح به.

تشمل هذه الملفات:
- الكود المصدري للموقع (Source Code).
- ملفات إعدادات الاتصال بقواعد البيانات ومفاتيح الـ API.
- ملفات النظام الحساسة (مثل `/etc/passwd` في Linux أو `C:\Windows\win.ini` في Windows).

---

## 2. الفرق الجوهري بين Path Traversal و LFI و RFI

| الثغرة | آلية الحدوث | أقصى أثر يمكن الوصول إليه |
|---|---|---|
| **Path Traversal (قراءة الملفات)** | التلاعب بالمسار لقراءة ملفات ثابتة عبر دوال مثل `file_get_contents()` أو قراءة الصور. | قراءة الملفات الحساسة وسرقة أسرار النظام (Arbitrary File Read). |
| **Local File Inclusion (LFI)** | تمرير المسار إلى دالة تنفيذ برمجية مثل `include()` أو `require()` في PHP. | تنفيذ أوامر على السيرفر (RCE) عبر تقنيات مثل Log Poisoning أو Session Poisoning. |
| **Remote File Inclusion (RFI)** | إجبار السيرفر على سحب وتنفيذ ملف من سيرفر خارجي يملكه المهاجم. | تنفيذ أوامر فوري على السيرفر (Instant RCE). |

---

## 3. أشهر أساليب تخطي الفلاتر (Bypass Techniques)
1. **المسار المطلق (Absolute Path):** إذا كان الفلتر يحذف `../`، فإن إرسال `/etc/passwd` مباشرة يتخطاه في العديد من الدوال.
2. **التنظيف غير التكراري (Non-recursive Stripping):** استخدام `....//....//etc/passwd`؛ عند حذف `../` لمرة واحدة يتبقى `../../`.
3. **التشفير المزدوج (Double URL Encoding):** تشفير السلاش والنقط مرتين `%252e%252e%252f` لخداع البروكسي بينما يفكها السيرفر مرتين.
4. **تجاوز شرط بداية المسار (Prefix Validation):** تلبية شرط السيرفر ثم الرجوع للخلف: `/var/www/images/../../../etc/passwd`.
5. **حقن البايت الفارغ (Null Byte Injection):** استخدام `%00` لإنهاء السلسلة النصية في دوال لغة C وخداع فحص الامتداد (`/etc/passwd%00.png`).
