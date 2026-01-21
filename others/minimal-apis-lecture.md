# 📚 **Minimal APIs ببساطة** — محاضرة سريعة

> **شرح عملي بـ 10 صفحات فقط!**

---

## 🎯 **مقدمة سريعة**

### يعني إيه API؟

```
انت (Client) → طلب → الويتر (API) → أمر → المطبخ (Server) → رد → أنت
```

**API** = طريقة آمنة بتسمح للـ frontend يطلب بيانات من الـ backend.

### يعني إيه Minimal؟

بدل ما تكتب Controllers وRequests وResources (معقد):
```php
// ❌ معقد: 5 ملفات
Route::apiResource('posts', PostController::class);
```

تكتب مباشرة في الـ route (بسيط):
```php
// ✅ بسيط: ملف واحد!
Route::get('/posts', fn() => Post::all());
Route::post('/posts', fn(Request $r) => Post::create($r->validated()));
```

---

## 🔧 **الـ Concepts الأساسية**

### HTTP Methods:

| Method | معنى | مثال |
|:------:|:----:|:----:|
| **GET** | اجلب بيانات | `GET /posts` |
| **POST** | انشئ جديد | `POST /posts + data` |
| **PATCH** | عدّل جزء | `PATCH /posts/1` |
| **DELETE** | احذف | `DELETE /posts/1` |

### Status Codes:

| الكود | المعنى |
|:----:|:------:|
| **200** | نجح + بيانات |
| **201** | اتنشأت حاجة جديدة |
| **204** | نجح بدون data |
| **400** | الطلب غلط |
| **401** | ما متسجل |
| **403** | ما عندك صلاحية |
| **404** | مش موجود |
| **422** | الـ validation فشل |
| **500** | حاجة غلط في الكود |

---

## 💻 **مثال عملي: Todo API**

### الـ Routes (`routes/api.php`):

```php
<?php

use App\Models\Todo;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

// 🔍 اجلب كل الـ Todos
Route::get('/todos', fn() => Todo::all());

// 🔍 اجلب Todo واحد
Route::get('/todos/{id}', function ($id) {
    $todo = Todo::find($id) ?? abort(404);
    return $todo;
});

// ✨ انشئ Todo جديد
Route::post('/todos', function (Request $request) {
    $request->validate([
        'title' => 'required|string|max:255',
        'is_completed' => 'nullable|boolean'
    ]);
    
    return response()->json(
        Todo::create($request->validated()),
        201
    );
});

// ✏️ عدّل Todo
Route::patch('/todos/{id}', function (Request $request, $id) {
    $todo = Todo::find($id) ?? abort(404);
    $todo->update($request->validate([
        'title' => 'nullable|string|max:255',
        'is_completed' => 'nullable|boolean'
    ]));
    return $todo;
});

// 🗑️ احذف Todo
Route::delete('/todos/{id}', function ($id) {
    Todo::find($id)?->delete() ?? abort(404);
    return response()->json(null, 204);
});
```

### الـ Model:

```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Todo extends Model
{
    protected $fillable = ['title', 'is_completed'];
    protected $casts = ['is_completed' => 'boolean'];
}
```

---

## 🛠️ **خطوات سريعة**

```bash
# 1. انشئ مشروع
composer create-project laravel/laravel api-demo && cd api-demo

# 2. انشئ Model و Migration
php artisan make:model Todo -m

# 3. في Migration، أضف الـ fields:
# $table->id();
# $table->string('title');
# $table->boolean('is_completed')->default(false);
# $table->timestamps();

# 4. شغّل Migration
php artisan migrate

# 5. اكتب الـ Routes (اللي فوق)

# 6. شغّل السيرفر
php artisan serve

# 7. اختبر:
curl http://localhost:8000/api/todos
curl -X POST http://localhost:8000/api/todos \
  -H "Content-Type: application/json" \
  -d '{"title":"Learn APIs"}'
```

---

## ❌ **معالجة الأخطاء**

### 404 — Resource مش موجود:

```php
Route::get('/todos/{id}', function ($id) {
    return Todo::findOrFail($id); // ترجع 404 تلقائياً
});
```

### 422 — Validation فشل:

```php
// Laravel بتعمل الـ validation تلقائياً وترجع 422
$request->validate([
    'title' => 'required|string|max:255',
]);
```

### Error Response:

```json
{
  "message": "The given data was invalid.",
  "errors": {
    "title": ["The title field is required."]
  }
}
```

---

## 🚀 **Advanced بس سريع**

### 1️⃣ **Authentication (Sanctum)**:

```php
// محمي بـ token:
Route::middleware('auth:sanctum')->post('/todos', function (Request $r) {
    return auth()->user()->todos()->create($r->validated());
});

// Login:
Route::post('/login', function (Request $r) {
    $user = User::where('email', $r->email)->first();
    
    if (!Hash::check($r->password, $user->password)) 
        return response()->json(['error' => 'Invalid'], 401);
    
    return ['token' => $user->createToken('api')->plainTextToken];
});
```

### 2️⃣ **Pagination**:

```php
Route::get('/todos', fn(Request $r) => 
    Todo::paginate($r->get('per_page', 15))
);

// GET /todos?per_page=10
```

### 3️⃣ **Filtering & Sorting**:

```php
Route::get('/todos', function (Request $r) {
    $query = Todo::query();
    
    if ($r->has('completed'))
        $query->where('is_completed', $r->boolean('completed'));
    
    if ($r->has('sort'))
        $query->orderBy($r->sort, $r->get('order', 'desc'));
    
    return $query->paginate();
});

// GET /todos?completed=1&sort=created_at&order=asc
```

### 4️⃣ **API Versioning**:

```php
Route::prefix('v1')->group(fn() => 
    Route::apiResource('todos', TodoControllerV1::class)
);

Route::prefix('v2')->group(fn() => 
    Route::apiResource('todos', TodoControllerV2::class)
);
```

---

## ✨ **Best Practices**

```php
// ✅ 1. استخدم Resources للـ clean responses:
Route::get('/todos', fn() => TodoResource::collection(Todo::all()));

// ✅ 2. Validate دايماً:
$request->validate(['title' => 'required|string|max:255']);

// ✅ 3. استخدم الـ HTTP Methods صح:
// GET = اجلب | POST = انشئ | PATCH = عدّل | DELETE = احذف

// ✅ 4. الرد بـ الـ status code الصحيح:
response()->json($data, 201);  // 201 للـ POST
response()->json(null, 204);   // 204 للـ DELETE

// ✅ 5. منع SQL Injection (Laravel بتعملها تلقائياً):
Todo::create($request->validated()); // ✓ آمن
Todo::create($request->all());       // ✗ خطر!
```

---

## 📝 **Cheat Sheet**

### الـ Routes الـ 5 الأساسية:

```php
Route::get('/posts', ...);                    // Read all
Route::get('/posts/{id}', ...);               // Read one
Route::post('/posts', ...);                   // Create
Route::patch('/posts/{id}', ...);             // Update
Route::delete('/posts/{id}', ...);            // Delete

// أو بـ سطر واحد:
Route::apiResource('posts', PostController::class);
```

### Middleware:

```php
// Authentication:
Route::middleware('auth:sanctum')->group(fn() => ...);

// Rate limiting (60 طلب في الدقيقة):
Route::middleware('throttle:60,1')->group(fn() => ...);
```

### Quick Commands:

```bash
php artisan serve                    # تشغيل السيرفر
php artisan make:model Todo -m       # Model + Migration
php artisan migrate                  # تشغيل Migrations
php artisan route:list               # عرض الـ Routes
```

---

## 🎓 **Mini-Challenge**

ابني API لـ **Products**:

```
POST   /api/products       → انشئ منتج
GET    /api/products       → اجلب كل المنتجات (مع pagination)
GET    /api/products/{id}  → اجلب منتج واحد
PATCH  /api/products/{id}  → عدّل منتج
DELETE /api/products/{id}  → احذف منتج

Validation: name (required), price (numeric), category (in:a,b,c)
```

---

## ❓ **FAQ سريع**

**Q: Minimal أم MVC؟**  
A: Minimal للـ بسيط والسريع، MVC للـ معقد.

**Q: ليه Resources؟**  
A: عشان تتحكم في الحقول اللي بتظهر وتنسق الـ response.

**Q: إزاي أحمي الـ API؟**  
A: Validate دايماً، استخدم Auth، Rate Limiting.

**Q: احتاج Controllers؟**  
A: للـ logic البسيطة: لا، للـ معقدة: أيوا.

---

## 🎉 **الخلاصة**

✅ **ابدأ بسيط** — Minimal routes  
✅ **Validate** — دايماً تحقق من البيانات  
✅ **Status codes** — استخدم الـ صحيحة  
✅ **Test** — curl أو Postman  
✅ **Expand** — أضف features لما احتجت  

---

**Happy API Building! 🚀**
