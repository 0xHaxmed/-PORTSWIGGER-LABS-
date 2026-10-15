# Lab 05 — Information Disclosure in Version Control History (تسريب كلمة المرور في تاريخ Git)

[![التصنيف](https://img.shields.io/badge/الفئة-Information%20Disclosure-blue)]()
[![النوع](https://img.shields.io/badge/النوع-Information%20Disclosure%20(Exposed%20.git%20Repository)-orange)]()
[![مستوى الخطورة](https://img.shields.io/badge/الخطورة-عالي%20(High)-critical)]()
[![المصدر](https://img.shields.io/badge/المصدر-PortSwigger%20Academy-blue)](https://portswigger.net/web-security/information-disclosure)

> هذا الشرح جزء من سلسلة [معامل تسريب المعلومات (Information Disclosure)](./README.md) — أكاديمية PortSwigger.

---

## 1. بطاقة تعريف اللاب (Overview)

| الحقل | التفاصيل |
|---|---|
| **اسم اللاب** | Lab 05 — Information Disclosure in Version Control History (تسريب كلمة المرور في تاريخ Git) |
| **تصنيف الثغرة** | Information Disclosure (Exposed .git Repository) |
| **معيار الضعف (CWE)** | CWE-527: Exposure of Version-Control Repository to an Unauthorized Control Sphere |
| **مستوى الخطورة** | عالي (High) |
| **الهدف من اللاب** | تحميل مجلد .git المكشوف، والبحث في سجل التعديلات السابقة لاستخراج باسورد الأدمن المحذوف، ثم حذف carlos. |

---

## 2. فكرة اللاب وسيناريو المشكلة (Concept & Vulnerability Mechanism)

أثناء تطوير الموقع، قام المطور بكتابة كلمة مرور الأدمن داخل كود ملف `admin.conf` وعمل له حفظ وإيداع (`git commit`). في وقت لاحق، تذكر المطور أن كتابة الباسورد في الكود خطأ، فقام بمسح السطر من الملف وعمل `commit` جديد باسم `Remove hardcoded admin password`.

**الخلل الفكري هنا:**
1. نظام Git يحتفظ بالتاريخ الأبدي لكل بايت تم حفظه في أي وقت، ومجرد مسح الكود في commit لاحق لا يمسحه من سجلات الـ Git!
2. عند نشر الموقع على السيرفر، تم نسخ مجلد المشروع بالكامل ومعه المجلد المخفي `/.git`.
3. استطاع المهاجم تحميل مجلد `/.git` واستعراض الـ Commits السابقة ورؤية التعديل (git diff) وقراءة كلمة المرور التي تم حذفها!

---

## 3. التحليل الجذري للسبب (Root Cause Analysis)

### 🔴 وجهة نظر المهاجم (Attacker Perspective):
المهاجم يتحقق من وجود مسار `/.git/`، ويستخدم أدوات سحب المستودعات لإعادة بناء شجرة المشروع واستعراض الفروقات (Diffs) لاستخراج الأسرار المحذوفة.

### 🟢 وجهة نظر المدافع والمطور (Defender Perspective):
نشر مجلد `.git` على خادم الويب العام، والاعتقاد الخاطئ بأن عمل commit لحذف سر يلغي وجوده من تاريخ المستودع.

---

## 4. خطوات الاستغلال والحل بالتفصيل (Proof of Concept - PoC)

1. تأكد من أن مجلد Git متاح بزيارة: `/.git/HEAD` في المتصفح (ستظهر لك عبارة `ref: refs/heads/master`).
2. استخدم أداة مثل `wget` أو `git-dumper` لتحميل مجلد `.git` بالكامل إلى جهازك:
```bash
git-dumper https://<lab-id>.web-security-academy.net/.git/ ./repo
```
3. ادخل إلى مجلد المستودع في جهازك: `cd ./repo`.
4. اكتب أمر استعراض السجل: `git log`.
5. ستلاحظ وجود commit حديث بعنوان مشبوه: `Remove admin password from config`.
6. اكتب أمر فحص التعديلات في ذلك الـ Commit:
`git show <commit-hash>` (أو `git diff HEAD~1`).
7. ستشاهد السطر المحذوف باللون الأحمر وفيه كلمة المرور الحقيقية للأدمن:
`- admin.password = a8b7c6d5e4f3...`
8. توجه لصفحة تسجيل الدخول `/login` وسجل بحساب `administrator` وباسورده المسروق، ثم احذف `carlos`.

---

## 5. أدلة الترافيك (Evidence: Request / Response)

### نموذج طلب الاستغلال (Exploit Request):
```http
GET /.git/COMMIT_EDITMSG HTTP/1.1
Host: <lab-id>.web-security-academy.net
```

### نموذج استجابة السيرفر (Server Response):
```http
HTTP/1.1 200 OK

Remove hardcoded admin password from config
```

---

## 6. الأثر الواقعي للثغرة (Impact)

تسريب السورس كود الكامل للمشروع وتاريخه التطويري، واستخراج كلمات المرور والمفاتيح السرية الحساسة.

---

## 7. الحماية والإغلاق البرمجي (Remediation)

1. حظر الوصول لمجلد `/.git` في إعدادات خادم الويب نهائياً.
2. استخدام أدوات مثل `git-filter-repo` أو `BFG Repo-Cleaner` لتطهير تاريخ الـ Git تماماً في حال تسرب سر في أي وقت.

---

## 8. المراجع الرسمية (References)

- [PortSwigger Web Security Academy — Information Disclosure](https://portswigger.net/web-security/information-disclosure)
- [OWASP Top 10 — A05:2021 Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)
- [CWE Database Details](https://cwe.mitre.org/)

---
*تم إعداد هذا التوثيق للأغراض التعليمية واختبار الاختراق الأخلاقي المصرح به.*
