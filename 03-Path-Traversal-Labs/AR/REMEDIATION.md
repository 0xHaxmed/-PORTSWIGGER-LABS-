# دليل الحماية والإغلاق لاجتياز المسارات (Remediation)

## 1. الحل الجذري الأقوى: التحقق من المسار القياسي (Canonical Path Verification)
الحل الصحيح والوحيد الذي يضمن منع الثغرة بنسبة 100% هو تحويل المسار إلى شكله المطلق النهائي على القرص، ثم التأكد من أنه يبدأ بمجلد الصور المصرح به:

### كود الحماية بلغة Java:
```java
File file = new File(BASE_DIRECTORY, userInputFilename);
String canonicalPath = file.getCanonicalPath();

// التحقق من أن المسار النهائي يقع بالكامل داخل المجلد المسموح به
if (!canonicalPath.startsWith(new File(BASE_DIRECTORY).getCanonicalPath() + File.separator)) {
    throw new SecurityException("محاولة اختراق مسارات غير مصرح بها!");
}

return Files.readAllBytes(file.toPath());
```

### كود الحماية بلغة Python:
```python
import os

base_dir = os.path.abspath("/var/www/uploads")
requested_path = os.path.abspath(os.path.join(base_dir, user_filename))

if not requested_path.startswith(base_dir + os.path.sep):
    abort(403, "غير مصرح بالوصول لهذا المسار")

with open(requested_path, "rb") as f:
    return f.read()
```

## 2. إجراءات حماية إضافية
1. **استخدام معرّفات وسيطة (Indirect Mapping):** إعطاء كل ملف رقم ID في الداتابيز، وعدم السماح للمستخدم بتمرير أسماء ملفات أبداً.
2. **تقييد صلاحيات الخادم (Least Privilege):** تشغيل تطبيق الويب بمستخدم محدود الصلاحيات (`www-data`) لا يملك صلاحية القراءة على مجلدات النظام الحساسة.
