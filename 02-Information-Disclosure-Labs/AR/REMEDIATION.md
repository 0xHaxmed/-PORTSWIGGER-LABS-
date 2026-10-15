# دليل الحماية والإغلاق لتسريب المعلومات (Remediation)

## 1. المبادئ الهندسية الوقائية

1. **إغلاق وضع التطوير فوراً في الإنتاج:** تأكد بنسبة 100% أن `DEBUG=False` في جميع خوادم الـ Production.
2. **صفحات أخطاء موحدة ومحايدة (Generic Error Pages):** يجب ألا يرى المستخدم أي تفاصيل فنية عن الخطأ، بل صفحة لطيفة مثل: "حدث خطأ غير متوقع، يرجى المحاولة لاحقاً".
3. **تجريد ملفات النشر من المخلفات:** استخدام ملفات `.dockerignore` و `.gitignore` لمنع تضمين `.git` والنسخ الاحتياطية في صور الحاويات.
4. **تعطيل طرق الـ HTTP التشخيصية:** حظر طلبات `TRACE` و `TRACK` على مستوى خادم الويب.

---

## 2. إعدادات السيرفر العملية المحصنة

### أولاً: خادم Apache
```apache
# تعطيل طريقة TRACE بالكامل
TraceEnable Off

# منع استعراض المجلدات في حال غياب ملف index
Options -Indexes

# حظر الوصول لمجلدات Git وملفات الباك آب
<FilesMatch "(\.(bak|old|orig|save|swp)|~)$">
    Require all denied
</FilesMatch>

<DirectoryMatch "/\.(git|svn)">
    Require all denied
</DirectoryMatch>
```

### ثانياً: خادم Nginx
```nginx
# حظر الوصول لملفات Git والملفات الحساسة
location ~ /\.(git|env|svn) {
    deny all;
    return 404;
}

location ~* \.(bak|old|orig|save|swp)$ {
    deny all;
    return 404;
}
```

### ثالثاً: معالجة الأخطاء برمجياً (مثال Java / Spring)
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleAllExceptions(Exception ex) {
        // نسجل تفاصيل الخطأ في اللوج الداخلي الآمن فقط
        logger.error("Internal Error: ", ex);
        // نعيد للمستخدم رسالة عامة برمز 500 دون أي تفاصيل فنية
        return new ResponseEntity<>("حدث خطأ أثناء معالجة الطلب.", HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```
