# دليل الحماية والإغلاق البرمجي لثغرات التحكم بالوصول (Remediation)

## 1. المبادئ الهندسية الأساسية

1. **الحرمان افتراضياً (Deny by Default):**
   أي رابط، أو وظيفة، أو مورد في التطبيق يجب أن يكون محظوراً بالكامل، ما لم يُصرّح له صراحة بالفتح لعموم المستخدمين.
2. **الفحص في كل طلب (Validate on Every Request):**
   لا تفترض أبداً أن المستخدم الذي اجتاز الخطوة الأولى لا يزال مأذوناً له في الخطوة الثانية دون التحقق من جلسته مجدداً.
3. **فصل الصلاحيات تماماً عن واجهة المستخدم (Decouple UI from Auth):**
   إخفاء الروابط أو الأزرار في الواجهة هو قرار خاص بتجربة المستخدم (UX)، ولا يمثل أي حماية أمنية على الإطلاق.

---

## 2. حلول معمارية برمجية عملية

### أولاً: تطبيق طبقة فحص صلاحيات مركزية (Centralized Middleware / RBAC)
بدلاً من كتابة شروط يدوية متفرقة داخل كل دالة، استخدم Middleware موحد يربط الصلاحيات بالأدوار:

```python
# كود محمي بايثون / فلاسك
def require_permission(permission_name):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not current_user.is_authenticated:
                abort(401) # غير مسجل الدخول
            if not current_user.has_permission(permission_name):
                abort(403) # مسجل الدخول لكن الصلاحية مرفوضة
            return f(*args, **kwargs)
        return decorated_function
    return decorator

@app.route('/admin/users/delete', methods=['POST'])
@require_permission('DELETE_USERS')
def handle_delete_user():
    # كود الحذف الآمن
    pass
```

### ثانياً: منع ثغرات IDOR بالتحقق من الملكية المشتركة
عند استعلام قاعدة البيانات، لا تبحث بالمعرّف المرسل من المستخدم وحده، بل اربطه بمعرّف المستخدم المأخوذ من الجلسة الآمنة (Session):

```php
// كود محمي PHP PDO
$accountId = $_GET['account_id'];
$currentUserId = $_SESSION['logged_in_user_id'];

// استعلام آمن يتحقق من أن السجل مملوك للمستخدم صاحب الجلسة
$stmt = $pdo->prepare("SELECT * FROM accounts WHERE id = ? AND owner_id = ?");
$stmt->execute([$accountId, $currentUserId]);
$account = $stmt->fetch();

if (!$account) {
    http_response_code(403);
    die("غير مصرح لك بمشاهدة هذا الحساب");
}
```

### ثالثاً: استخدام DTOs لمنع ثغرات Mass Assignment
لا تعتمد على إدخال الـ JSON مباشرة إلى الـ Model. حدد كائناً خاصاً بالمدخلات المسموحة فقط:

```csharp
// في C# ASP.NET Core
public class UpdateProfileRequest
{
    [Required]
    public string FullName { get; set; }
    [EmailAddress]
    public string Email { get; set; }
    // لا نضع حقل RoleId أو IsAdmin هنا أبداً!
}
```

### رابعاً: ضبط خوادم التوجيه والبروكسي (Reverse Proxy Hardening)
* تأكد من أن خوادم Nginx / Apache تقوم بتعطيل أو تصفية أي ترويسات تخص إعادة التوجيه الداخلي (`X-Original-URL`, `X-Rewrite-URL`, `X-Forwarded-Prefix`) قبل تمرير الطلب لتطبيق الباك إند.
