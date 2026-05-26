# 👑 RoyalCity RP — دليل الإعداد

## ✅ الملفات المضمّنة
| الملف | الوصف |
|---|---|
| `index.html` | الموقع الرئيسي |
| `admin.html` | لوحة التحكم |
| `.env` | إعدادات بيئة — **لا ترفعه!** |
| `.gitignore` | لحماية الملفات الحساسة |

---

## 🔧 إعداد المتغيرات

### 1. Discord Client ID
افتح `index.html` وابحث عن:
```js
const CLIENT_ID = ''; // ← ضع هنا CLIENT_ID
```
اسحب الـ ID من [Discord Developer Portal](https://discord.com/developers/applications) وضعه هنا.

### 2. Supabase
بيانات Supabase موجودة في الملفين وهي:
- `SUPABASE_URL` — رابط مشروعك
- `SUPABASE_ANON_KEY` — المفتاح العام

**مهم:** تأكد من تفعيل RLS على كل الجداول في Supabase!

### 3. Webhooks الديسكورد
⚠️ **لا ترسل Webhook من الفرونت!**

الطريقة الصحيحة:
1. احفظ رابط الـ Webhook في جدول `forms` في Supabase
2. أنشئ **Supabase Database Webhook** أو **Edge Function** ترسله للديسكورد عند إضافة تقديم جديد

---

## 🗄️ جداول Supabase المطلوبة

```sql
-- طاقم الإدارة
create table staff (
  id uuid default gen_random_uuid() primary key,
  user_id text unique not null,  -- Discord ID
  role integer not null default 2, -- 2=Mod, 3=Admin, 4=Owner
  created_at timestamptz default now()
);

-- الأخبار
create table news (
  id uuid default gen_random_uuid() primary key,
  title text not null,
  content text not null,
  created_at timestamptz default now()
);

-- القوانين
create table rules (
  id uuid default gen_random_uuid() primary key,
  title text not null,
  content text not null,
  "order" integer default 0,
  created_at timestamptz default now()
);

-- نماذج التقديم
create table forms (
  id uuid default gen_random_uuid() primary key,
  title text not null,
  desc text,
  webhook text,  -- محمي بـ RLS — يراه الأدمن فقط
  questions jsonb not null default '[]',
  is_open boolean default true,
  created_at timestamptz default now()
);

-- البلاك ليست
create table blacklist (
  id uuid default gen_random_uuid() primary key,
  user_id text unique not null,
  reason text not null,
  created_at timestamptz default now()
);

-- الطلبات المقدّمة
create table applications (
  id uuid default gen_random_uuid() primary key,
  form_id uuid references forms(id),
  user_id text not null,
  username text not null,
  avatar text,
  answers jsonb not null default '[]',
  created_at timestamptz default now(),
  unique(form_id, user_id)
);
```

### 🔒 سياسات RLS المهمة

```sql
-- المستخدمون يقرؤون النماذج بدون webhook
create policy "public_read_forms" on forms
  for select using (true);

-- المستخدمون يُرسلون تقديماتهم فقط
create policy "insert_own_application" on applications
  for insert with check (true);

-- منع قراءة webhook من الفرونت
-- (أزِل عمود webhook من الـ SELECT في الفرونت — مفعّل بالفعل في الكود)
```

---

## 🔐 ملاحظات الأمان

- ✅ **RLS مفعّل** على كل الجداول
- ✅ **Webhook لا يُرسل** من الفرونت
- ✅ **CSP محكمة** تمنع XSS وتحميل موارد خارجية
- ✅ **Rate Limiting** على تسجيل الدخول والتقديمات
- ✅ **تنظيف المدخلات** قبل عرضها أو حفظها
- ✅ **لا innerHTML** من بيانات المستخدمين
- ✅ **التوكن يُمسح** من الـ URL مباشرة بعد الحفظ

---

**© 2025 RoyalCity RP — جميع الحقوق محفوظة لـ RoyalCity**
