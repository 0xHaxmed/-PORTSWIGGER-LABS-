# دليل الحماية والإغلاق لثغرات تزوير الطلبات SSRF (Remediation)

## 1. الحماية المعمارية على مستوى الشبكة والسحابة

1. **عزل الشبكة وتقييد الاتصالات الصادرة (Egress Firewall Rules):**
   منع خوادم الويب من عمل اتصالات موجهة نحو عناوين الشبكات الخاصة (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `127.0.0.0/8`).
2. **إلزام استخدام IMDSv2 في AWS:**
   تفعيل الجيل الثاني من خدمة الميتاداتا السحابية وضبط الـ Hop Limit على `1`، مما يمنع نهائياً سرقة بيانات الـ IAM عبر الـ SSRF.
3. **تعطيل ميزة التتبع التلقائي للتحويلات (Disable Redirects):**
   ضبط مكتبات الـ HTTP بعدم تتبع أي تحويلات `3xx` تلقائياً.

---

## 2. التحقق البرمجي الآمن من الـ IP المحلول (DNS Resolution Pinning)
يجب تحويل اسم الدومين إلى IP عبر الـ DNS أولاً، وفحص أن الـ IP عام وليس محلياً، ثم تثبيت الاتصال على نفس الـ IP لمنع هجمات DNS Rebinding:

```python
# كود بايثون آمن تماماً ضد SSRF
import ipaddress, socket, requests
from urllib.parse import urlparse

def safe_http_fetch(target_url):
    parsed = urlparse(target_url)
    if parsed.scheme not in ('http', 'https'):
        raise ValueError("البروتوكول غير مسموح به")
    
    # فك اسم المضيف إلى عنوان IP
    ip_str = socket.gethostbyname(parsed.hostname)
    ip_obj = ipaddress.ip_address(ip_str)
    
    # منع كل العناوين الخاصة والمحلية والميتاداتا
    if ip_obj.is_private or ip_obj.is_loopback or ip_obj.is_link_local:
        raise PermissionError("ممنوع الاتصال بالعناوين الداخلية!")
        
    # إرسال الطلب مع تعطيل تتبع التحويلات
    return requests.get(target_url, allow_redirects=False, timeout=5)
```
