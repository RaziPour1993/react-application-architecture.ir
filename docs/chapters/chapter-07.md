# فصل ۷: پیاده‌سازی احراز هویت و امن‌سازی اپلیکیشن {#h1_199 .chapterTitle}

authentication و authorization سنگ بناهای اپلیکیشن‌های امن هستند. authentication را مثل چک‌این کردن در هتل در نظر بگیرید؛ هویت کاربر را از طریق تأیید اطلاعات ورود تأیید می‌کند. authorization مثل کارت‌کلیدی است که دریافت می‌کنید؛ تعیین می‌کند کاربر یک‌بار وارد شد چه کارهایی می‌تواند انجام دهد. در اپلیکیشن ما، authentication تأیید می‌کند کسی کاربر ثبت‌نام‌شده است و authorization تضمین می‌کند فقط نویسندهٔ یک ایده می‌تواند آن را ویرایش یا حذف کند.

علاوه بر authentication و authorization، باید اپلیکیشن و کاربرانش را در برابر آسیب‌پذیری‌های امنیتی با جلوگیری از اجرای کد مخرب به‌جای کاربر محافظت کنیم. موضوعات زیر را پوشش می‌دهیم:

- درک authentication و authorization
- پیاده‌سازی authentication با [httpOnly cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- محافظت از بخش‌های اپلیکیشن با authorization policy
- ساخت flowهای ثبت‌نام، ورود و خروج
- محافظت از routeهای احراز هویت‌شده
- جلوگیری از حملات [XSS](https://owasp.org/www-community/attacks/xss/) با sanitization محتوا
- امن‌سازی اپلیکیشن با security header

در پایان این فصل، سیستم authentication امنی خواهیم داشت که اپلیکیشن را محافظت می‌کند و practiceهای امنیتی را پیاده‌سازی می‌کند تا اپلیکیشن و کاربرانمان ایمن باشند.

## پیش‌نیازهای فنی {#h1_200}

قبل از شروع، باید پروژه را راه‌اندازی کنیم. برای توسعهٔ پروژه، ابزارهای زیر باید روی کامپیوتر نصب باشند:

- [Node.js](https://nodejs.org/) نسخه ۲۴ یا بالاتر. [npm](https://www.npmjs.com/) نسخه ۱۱ یا بالاتر همراه Node عرضه می‌شود. می‌توانیم با اجرای `node ‑v` و `npm ‑v` در ترمینال تأیید کنیم. روش‌های مختلفی برای نصب Node.js و npm وجود دارد. مقالهٔ مفیدی با جزئیات بیشتر اینجا هست: [https://www.nodejsdesignpatterns.com/blog/5-ways-to-install-node-js](https://www.nodejsdesignpatterns.com/blog/5-ways-to-install-node-js).
- **[VS Code](https://code.visualstudio.com/)** (اختیاری)، ویرایشگر محبوب برای JavaScript و [TypeScript](https://www.typescriptlang.org/). متن‌باز است، پشتیبانی خوبی از TypeScript دارد و افزونه‌های زیادی ارائه می‌دهد. از [https://code.visualstudio.com](https://code.visualstudio.com) قابل دانلود است.

کد این کتاب در [https://github.com/PacktPublishing/React-Application-Architecture-for-Production-Second-Edition](https://github.com/PacktPublishing/React-Application-Architecture-for-Production-Second-Edition) در GitHub موجود است. آن را clone کنید و به ریشهٔ مخزن وارد شوید:

```bash
git clone https://github.com/PacktPublishing/React-Application-Architecture-for-Production-Second-Edition.git
```

مخزن شامل پوشه‌های فصل‌ها با کد هر فصل، به‌علاوهٔ پوشهٔ مشترک `api` است که سرور API مورد استفاده در همهٔ فصل‌ها را شامل می‌شود.

ما روی **فصل ۷** کار می‌کنیم پس به پوشهٔ `chapter‑07` بروید:

```bash
cd React-Application-Architecture-for-Production-Second-Edition/chapter-07
```

سپس dependencyها را نصب کنید:

```bash
npm install
```

همچنین باید متغیرهای محیطی را فراهم کنیم:

```bash
cp .env.example .env
```

در این مرحله، frontend باید آماده و در حال اجرای [http://localhost:5173](http://localhost:5173) باشد.

باید سرور API را هم اجرا کنیم.

یک پنجرهٔ ترمینال جدید باز کنید و به پوشهٔ `api` بروید:

```bash
cd React-Application-Architecture-for-Production-Second-Edition/api
```

اسکریپت setup را برای **فصل ۷** اجرا کنید تا همه‌چیز تنظیم شود:

```bash
npm run setup 07
```

سپس سرور API را استارت کنید:

```bash
npm run dev
```

سرور API حالا باید در [http://localhost:9999](http://localhost:9999) در حال اجرا باشد.

برای اطلاعات بیشتر دربارهٔ جزئیات setup، فایل `README.md` را بررسی کنید.

## احراز هویت {#h1_201}

**authentication** فرآیند تأیید هویت کاربر است. وقتی کاربر وارد اپلیکیشن ما می‌شود، باید هویتش را تأیید کنیم و سپس به‌عنوان کاربر در حال ناوبری بین صفحات مختلف و ارسال درخواست به API بخاطرش بمانیم. این برای ساخت اپلیکیشن‌هایی که محتوای شخصی‌سازی‌شده ارائه می‌دهند و داده‌های حساس را محافظت می‌کنند، اساسی است.

بدون authentication نمی‌توانیم بین کاربران مختلف تمایز قائل شویم یا دسترسی به امکانات خاصی را محدود کنیم. هر کاربر محتوای یکسانی می‌دید و هرکسی به هر داده‌ای دسترسی داشت. این برای وبسایت‌های عمومی کار می‌کند، اما بیشتر اپلیکیشن‌ها باید بدانند چه کسی از آن‌ها استفاده می‌کند.

authentication را با رویکرد [token-based](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication) پیاده‌سازی می‌کنیم. وقتی کاربران با اطلاعات ورودشان وارد می‌شوند، API ما آن‌ها را تأیید کرده و tokenهای authentication را برمی‌گرداند. این tokenها در httpOnly cookie ذخیره می‌شوند — cookieهایی که JavaScript نمی‌تواند به آن‌ها دسترسی داشته باشد. این مهم است چون از دزدیده شدن tokenها توسط scriptهای مخربی که ممکن است در صفحه‌مان اجرا شوند محافظت می‌کند، چون httpOnly cookie توسط JavaScript سمت client قابل دسترسی نیست. ارزش ذکر دارد که اگرچه httpOnly cookie خطر حملات XSS برای دزدیدن tokenها را کاهش می‌دهند، اما به‌تنهایی در برابر حملات CSRF محافظت نمی‌کنند. برای کمک به کاهش [CSRF](https://owasp.org/www-community/attacks/csrf)، ویژگی [`SameSite`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite) cookie به ما کمک می‌کند. با تنظیم `SameSite` روی `Strict` یا `Lax`، مرورگر cookie را فقط از همان سایت ارسال می‌کند و از ارسال آن با درخواست‌های cross-site جلوگیری می‌شود.

شکل زیر نشان می‌دهد flow authentication چگونه کار می‌کند:

![شکل ۷.۱ — Flow authentication](/images/B31385_7_1.png)

**شکل ۷.۱ — Flow authentication**

در این flow، کاربر اطلاعات ورودش را به API ارسال می‌کند، API آن‌ها را تأیید کرده و tokenهای authentication را برمی‌گرداند. این tokenها به‌صورت httpOnly cookie در مرورگر ذخیره شده و خودکار با درخواست‌های بعدی برای شناسایی کاربر ارسال می‌شوند.

API ما از دو نوع token برای مدیریت authentication استفاده می‌کند:

- **Access token**: tokenهای کوتاه‌مدت (معمولاً ۱۵ دقیقه) که هویت کاربر را برای هر درخواست اثبات می‌کنند. این‌ها با هر فراخوانی API برای شناسایی کاربر ارسال می‌شوند.
- **Refresh token**: tokenهای بلندمدت‌تر (معمولاً ۷ روز) که می‌توانند access token جدید تولید کنند وقتی tokenهای قدیمی منقضی شوند. این به کاربران اجازه می‌دهد بدون نیاز به ورود مجدد هر ۱۵ دقیقه، وارد باقی بمانند.

این سیستم دو tokenی تعادلی بین امنیت و تجربهٔ کاربری برقرار می‌کند. Access tokenها سریع منقضی می‌شوند تا آسیب ناشی از interception محدود شود، در حالی که refresh token به کاربران اجازه می‌دهد برای مدت طولانی‌تری وارد باقی بمانند بدون به خطر انداختن امنیت.

برای پشتیبانی از این flow authentication، باید API client را برای مدیریت tokenها گسترش دهیم، صفحات ثبت‌نام و ورود را پیاده‌سازی کنیم و routeهایی که به authentication نیاز دارند را محافظت کنیم.

### گسترش API client {#h2_202}

وقتی access token ما منقضی می‌شود، API با وضعیت 401 Unauthorized پاسخ می‌دهد. به‌جای خروج فوری کاربر، باید سعی کنیم با استفاده از refresh token access token جدید بگیریم. به این ترتیب، کاربران بدون متوجه شدن منقضی شدن token، وارد باقی می‌مانند.

اول، client را طوری به‌روزرسانی کنیم که همیشه cookieها را خودکار سمت client ارسال کند:

```typescript
// src/lib/api.ts

async function fetchApi<T, TBody = unknown>(
  url: string,
  options: RequestOptions<TBody> = {},
): Promise<T> {
  // ...

  const makeRequest = async (accessToken?: string): Promise<Response> => {
    try {
      return await fetch(fullUrl, {
        // ...
        credentials: typeof window !== 'undefined' ? 'include' : undefined,
      });
    } catch (error) {
      // ...
    }
  };
}
```

با تنظیم گزینهٔ `credentials` روی `include`، cookieها را سمت client خودکار ارسال می‌کنیم. مقدار پیش‌فرض `same-origin` است یعنی cookieها فقط برای درخواست‌های به همان origin ارسال می‌شوند. اما API ما روی origin متفاوتی اجرا می‌شود پس باید cookieها را صریحاً ارسال کنیم. سمت server، باید cookieها را صریحاً در headerها پاس دهیم:

```typescript
const response = await api.get<GetCurrentUserResponse>('/auth/me', {
    headers: {
      Cookie: cookieHeader ?? '',
    },
});
```

همچنین می‌خواهیم از refresh کردن access token پشتیبانی کنیم:

```typescript
// src/lib/api.ts

async function fetchApi<T, TBody = unknown>(
  url: string,
  options: RequestOptions<TBody> = {},
): Promise<T> {
  // ...

  // Handle 401 errors with automatic token refresh
  const isAuthenticationError = 
    !response.ok &&
    response.status === 401 &&
    !url.endsWith('/auth/login') &&
    !url.endsWith('/auth/register')


  if (isAuthenticationError) {
    // Extract cookie header from options for server-side requests
    // On client-side, this will be undefined and credentials: 'include' will be used
    const cookieHeader = headers.Cookie;

    try {
      // Attempt to refresh the token
      const { accessToken } = await refreshToken(cookieHeader);

      // Retry the original request with the new token
      response = await makeRequest(accessToken);
    } catch (refreshError) {
      // If refresh fails, the original 401 error will be handled below
      console.warn('Token refresh failed:', refreshError);
    }
  }

  // ...
}
```

وقتی درخواستی با وضعیت 401 شکست می‌خورد، خودکار سعی می‌کنیم token را refresh کرده و درخواست اصلی را با access token جدید تکرار کنیم.

این تنظیم تضمین می‌کند کاربران بدون وقفه وارد باقی بمانند حتی وقتی access tokenهایشان منقضی می‌شود، و هم در مرورگر و هم روی server درست کار می‌کند.

شکل زیر فرآیند refresh token را نشان می‌دهد:

![شکل ۷.۲ — Flow refresh token](/images/B31385_7_2.png)

**شکل ۷.۲ — Flow refresh token**

وقتی درخواست API با وضعیت 401 شکست می‌خورد، client خودکار endpoint refresh را با استفاده از cookie refresh token فراخوانی می‌کند. اگر refresh token معتبر باشد، API access token جدیدی برمی‌گرداند و client درخواست اصلی را تکرار می‌کند. همهٔ این‌ها در پس‌زمینه و بدون اطلاع کاربر اتفاق می‌افتد.

حالا که API client آمادهٔ مدیریت authentication است، به راهی نیاز داریم تا کاربران از اول به این tokenها دسترسی پیدا کنند. کاربران می‌توانند به‌عنوان کاربر جدید ثبت‌نام کنند یا با اطلاعات موجود وارد شوند.

### صفحه ثبت‌نام {#h2_203}

برای ثبت‌نام کاربر جدید، باید route جدید و یک کامپوننت فرم بسازیم که هنگام ارسال، endpoint ثبت‌نام را فراخوانی می‌کند و کاربر جدید ایجاد کرده و tokenهای authentication را در cookie برمی‌گرداند.

```tsx
// src/app/routes/auth/register.tsx

export default function RegisterPage() {
  const navigate = useNavigate();

  const registerUserMutation = useRegisterUserMutation();

  const onSubmit = (data: RegisterUserData['body']) => {
    registerUserMutation.mutate(data, {
      onSuccess: () => {
        navigate('/dashboard');
      },
    });
  };

  return (
    <div className="container mx-auto px-4 py-16">
      <div className="max-w-md mx-auto">
        <Card>
          <CardHeader>
            <CardTitle className="text-2xl text-center">Register</CardTitle>
          </CardHeader>
          <CardContent>
            <RegisterForm
              onSubmit={onSubmit}
              error={registerUserMutation.error}
              isPending={registerUserMutation.isPending}
            />
            <div className="text-center mt-4">
              <p className="text-sm text-muted-foreground">
                Already have an account?{' '}
                <Link to="/auth/login" className="text-primary hover:underline">
                  Log In
                </Link>
              </p>
            </div>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
```

صفحهٔ ثبت‌نام به این شکل است:

![شکل ۷.۳ — صفحهٔ ثبت‌نام](/images/B31385_7_3.png)

**شکل ۷.۳ — صفحهٔ ثبت‌نام**

این صفحهٔ ثبت‌نام ساده است. فرمی در داخل یک کامپوننت card رندر می‌کند و ارسال را با فراخوانی mutation ثبت‌نام مدیریت می‌کند. وقتی ثبت‌نام موفق باشد، کاربر را به dashboard هدایت می‌کنیم. کامپوننت فرم stateهای خطا و بارگذاری را از mutation دریافت می‌کند تا بتواند پیام‌های خطا را نشان داده و ورودی‌ها را در حین درخواست غیرفعال کند.

منطق واقعی ثبت‌نام در mutation hook قرار دارد.

```typescript
// src/features/auth/api/register.ts

export async function registerUser(
  data: RegisterUserData['body'],
): Promise<RegisterUserResponse> {
  const response = await api.post<RegisterUserResponse>('/auth/register', {
    body: data,
  });

  return zRegisterUserResponse.parse(response);
}

export function getRegisterUserMutationOptions() {
  return mutationOptions({
    mutationFn: registerUser,
  });
}

export function useRegisterUserMutation({
  options,
}: {
  options?: Omit<
    UseMutationOptions<RegisterUserResponse, Error, RegisterUserData['body']>,
    'mutationFn'
  >;
} = {}) {
  return useMutation({
    ...getRegisterUserMutationOptions(),
    ...options,
  });
}
```

تابع ثبت‌نام اطلاعات ورود کاربر را به endpoint `/auth/register` ارسال می‌کند. وقتی ثبت‌نام موفق باشد، API خودکار tokenهای authentication را در httpOnly cookie در پاسخ set می‌کند، پس کاربر فوراً بدون هیچ قدم اضافه‌ای وارد می‌شود.

حالا باید route را برای رندر صفحهٔ ثبت‌نام تنظیم کنیم. این کار در فایل `routes.ts` انجام می‌شود.

```typescript
// src/app/routes.ts

export default [
  route('auth/register', './routes/auth/register.tsx'),
  // ...
];
```

با وجود ثبت‌نام، به راهی برای کاربران موجود هم نیاز داریم تا دوباره وارد اپلیکیشن شوند.

### صفحه ورود {#h2_204}

flow ورود بسیار شبیه ثبت‌نام است. به صفحه‌ای با فرم نیاز داریم که کاربران اطلاعات ورودشان را وارد کنند و وقتی فرم را ارسال کردند، endpoint ورود را فراخوانی کرده و آن‌ها را احراز هویت کنیم.

```tsx
// src/app/routes/auth/login.tsx

export default function LoginPage() {
  const navigate = useNavigate();

  const loginUserMutation = useLoginUserMutation();

  const onSubmit = (data: LoginUserData['body']) => {
    loginUserMutation.mutate(data, {
      onSuccess: () => {
        navigate('/dashboard');
      },
    });
  };

  return (
    <div className="container mx-auto px-4 py-16">
      <div className="max-w-md mx-auto">
        <Card>
          <CardHeader>
            <CardTitle className="text-2xl text-center">Log In</CardTitle>
          </CardHeader>
          <CardContent>
            <LoginForm
              onSubmit={onSubmit}
              error={loginUserMutation.error}
              isPending={loginUserMutation.isPending}
            />
            <div className="text-center mt-4">
              <p className="text-sm text-muted-foreground">
                Don't have an account?{' '}
                <Link
                  to="/auth/register"
                  className="text-primary hover:underline"
                >
                  Register
                </Link>
              </p>
            </div>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
```

صفحهٔ ورود به این شکل است:

![شکل ۷.۴ — صفحهٔ ورود](/images/B31385_7_4.png)

**شکل ۷.۴ — صفحهٔ ورود**

صفحهٔ ورود همان الگوی ثبت‌نام را دنبال می‌کند. ارسال فرم را با یک mutation مدیریت می‌کند و در صورت موفقیت به dashboard هدایت می‌شود.

mutation endpoint ورود را فراخوانی می‌کند تا اطلاعات ورود کاربر را تأیید کند.

```typescript
// src/features/auth/api/login.ts

export async function loginUser(
  data: LoginUserData['body'],
): Promise<LoginUserResponse> {
  const response = await api.post<LoginUserResponse>('/auth/login', {
    body: data,
  });

  return zLoginUserResponse.parse(response);
}

export function getLoginUserMutationOptions() {
  return mutationOptions({
    mutationFn: loginUser,
  });
}

export function useLoginUserMutation({
  options,
}: {
  options?: Omit<
    UseMutationOptions<LoginUserResponse, Error, LoginUserData['body']>,
    'mutationFn'
  >;
} = {}) {
  return useMutation({
    ...getLoginUserMutationOptions(),
    ...options,
  });
}
```

دقیقاً مثل ثبت‌نام، endpoint ورود وقتی اطلاعات ورود معتبر باشند، tokenهای authentication را در httpOnly cookie تنظیم می‌کند. سپس کاربر احراز هویت شده و می‌تواند به بخش‌های محافظت‌شدهٔ اپلیکیشن دسترسی پیدا کند.

همچنین باید route را برای رندر صفحهٔ ورود تنظیم کنیم. این کار در فایل `routes.ts` انجام می‌شود.

```typescript
// src/app/routes.ts

export default [
  route('auth/login', './routes/auth/login.tsx'),
  // ...
];
```

حالا که کاربران می‌توانند ثبت‌نام کنند و وارد شوند، باید راهی برای خروجشان فراهم کنیم.

### خروج کاربر {#h2_205}

**خروج کردن** با ثبت‌نام و ورود فرق دارد چون کاربران فرمی پر نمی‌کنند. در عوض، روی دکمهٔ «خروج» در ناوبری کلیک می‌کنند و ما فوراً endpoint خروج را فراخوانی کرده تا tokenهای authentication آن‌ها پاک شود.

کامپوننت ناوبری ما دکمهٔ خروجی دارد که callback به نام `onLogout` را فراخوانی می‌کند. این callback را در کامپوننت layout که اپلیکیشن را در بر می‌گیرد، پیاده‌سازی می‌کنیم.

```tsx
// src/app/routes/layout.tsx

import { Outlet, useNavigate } from 'react-router';

import { Navigation } from '@/components/navigation';
import { useLogoutUserMutation } from '@/features/auth/api/logout';
import { useUser } from '@/features/auth/hooks/use-user';

export default function Layout() {
  const user = useUser();
  const navigate = useNavigate();
  const logoutUserMutation = useLogoutUserMutation({
    options: {
      onSuccess: () => navigate('/'),
    },
  });
  const handleLogout = async () => {
    logoutUserMutation.mutate();
  };

  return (
    <div>
      <Navigation user={user} onLogout={handleLogout} />
      <main className="min-h-screen bg-background">
        <Outlet />
      </main>
    </div>
  );
}
```

کامپوننت layout از hook `useUser` برای دریافت کاربر فعلی استفاده می‌کند و آن را به ناوبری پاس می‌دهد. وقتی کاربر روی دکمهٔ خروج کلیک می‌کند، تابع `handleLogout` فراخوانی شده و mutation خروج را راه‌اندازی می‌کند. پس از خروج موفق، کاربر را به صفحهٔ اصلی هدایت می‌کنیم.

حالا mutation خروج را پیاده‌سازی کنیم.

```typescript
// src/features/auth/api/logout.ts

export async function logoutUser(): Promise<LogoutUserResponse> {
  const response = await api.post<LogoutUserResponse>('/auth/logout', {
    body: {},
  });

  return zLogoutUserResponse.parse(response);
}

export function getLogoutUserMutationOptions() {
  return mutationOptions({
    mutationFn: logoutUser,
  });
}

export function useLogoutUserMutation({
  options,
}: {
  options?: Omit<
    UseMutationOptions<LogoutUserResponse, Error, void>,
    'mutationFn'
  >;
} = {}) {
  return useMutation({
    ...getLogoutUserMutationOptions(),
    ...options,
  });
}
```

تابع خروج endpoint `/auth/logout` را فراخوانی می‌کند که tokenهای authentication را از cookie پاک می‌کند. پس از این، کاربر دیگر احراز هویت نشده و باید برای دسترسی به صفحات محافظت‌شده دوباره وارد شود.

اما اپلیکیشن ما چگونه می‌داند آیا احراز هویت شده‌ایم یا نه؟ به راهی نیاز داریم تا در سراسر اپلیکیشن به اطلاعات کاربر فعلی دسترسی داشته باشیم.

### دسترسی به کاربر در اپلیکیشن {#h2_206}

معمولاً به اطلاعات کاربر فعلی در بخش‌های زیادی از اپلیکیشن نیاز داریم. ناوبری نام کاربر را نشان می‌دهد، dashboard داده‌های خاص کاربر را نمایش می‌دهد و باید بر اساس مجوزهای کاربر تصمیم بگیریم کدام امکانات را نشان دهیم. به‌جای دریافت کاربر در هر کامپوننت، یک‌بار دریافت کرده و در سراسر اپلیکیشن در دسترس قرار می‌دهیم.

از middleware React Router برای بارگذاری کاربر روی server و در دسترس قرار دادنش برای همهٔ کامپوننت‌ها استفاده می‌کنیم. اول، به تابعی نیاز داریم که اطلاعات کاربر فعلی را از API دریافت کند.

```typescript
// src/features/auth/api/get-me.ts

export async function getMe(
  cookieHeader?: string,
): Promise<GetCurrentUserResponse> {
  const response = await api.get<GetCurrentUserResponse>('/auth/me', {
    headers: {
      Cookie: cookieHeader ?? '',
    },
  });
  return zGetCurrentUserResponse.parse(response);
}
```

این تابع endpoint `/auth/me` را فراخوانی می‌کند تا اطلاعات کاربر فعلی را دریافت کند. پارامتر اختیاری `cookieHeader` را می‌پذیرد چون آن را روی server فراخوانی می‌کنیم که cookieها باید صریحاً پاس داده شوند.

حالا باید این اطلاعات کاربر را دریافت کرده و با استفاده از middleware در سراسر اپلیکیشن در دسترس قرار دهیم. اول، باید middleware را در اپلیکیشن فعال کنیم.

```typescript
// react-router.config.ts

import type { Config } from '@react-router/dev/config';

export default {
  // ...
  future: {
    v8_middleware: true,
  },
} satisfies Config;
```

این توسط [React Router](https://reactrouter.com/) لازم است چون یک feature جدید است که در نسخهٔ اصلی بعدی React Router به‌صورت پیش‌فرض موجود خواهد بود.

حالا user middleware را بسازیم:

```typescript
// src/app/middleware/user.ts

import { createContext, type MiddlewareFunction } from 'react-router';

import { getMe } from '@/features/auth/api/get-me';
import type { CurrentUser } from '@/types/generated/types.gen';

export const userContext = createContext<CurrentUser | null>();

function hasAuthCookies(request: Request): boolean {
  const cookieHeader = request.headers.get('Cookie');
  if (!cookieHeader) return false;

  return (
    cookieHeader.includes('accessToken=') ||
    cookieHeader.includes('refreshToken=')
  );
}

export const userMiddleware: MiddlewareFunction = async (
  { request, context },
  next,
) => {
  try {
    const existingUser = context.get(userContext);

    if (existingUser !== undefined) {
      return next();
    }
  } catch {
  }

  if (!hasAuthCookies(request)) {
    context.set(userContext, null);
    return next();
  }

  try {
    const cookieHeader = request.headers.get('Cookie') || '';
    const user = await getMe(cookieHeader);
    context.set(userContext, user);
  } catch {
    context.set(userContext, null);
  }

  return next();
};
```

این middleware در هر درخواست اجرا شده و اگر cookieهای authentication موجود باشند، کاربر فعلی را دریافت می‌کند. بیایید نحوهٔ کارش را بررسی کنیم:

- **بررسی کاربر موجود**: اگر کاربر قبلاً در context باشد، fetch را رد می‌کنیم تا از درخواست‌های تکراری جلوگیری شود.
- **بررسی cookieهای authentication**: فقط وقتی cookieهای authentication در درخواست موجود باشند سعی می‌کنیم کاربر را دریافت کنیم.
- **دریافت و ذخیرهٔ کاربر**: اگر cookieها وجود داشته باشند، اطلاعات کاربر را دریافت کرده و در context ذخیره می‌کنیم.
- **مدیریت خطای graceful**: اگر دریافت ناموفق باشد (token نامعتبر، خطای شبکه و...)، کاربر را به‌جای crash کردن اپلیکیشن، null قرار می‌دهیم.

این middleware شیء کاربر یا null را در request context React Router (با Context API [React](https://react.dev/) اشتباه گرفته نشود) ذخیره می‌کند، پس کد downstream همیشه می‌داند کاربر احراز هویت شده یا نه.

برای فعال کردن این middleware، باید آن را به آرایهٔ middleware در فایل root اضافه کنیم.

```typescript
// src/app/root.tsx

export const middleware = [userMiddleware];

// ...
```

با اضافه کردن `userMiddleware` به آرایهٔ middleware، قبل از اجرای هر loader در هر درخواست اجرا می‌شود. این تضمین می‌کند کاربر وقتی بهش نیاز داریم در context در دسترس باشد.

حالا می‌توانیم کاربر را از context در root loader دریافت کنیم.

```typescript
// src/app/root.tsx

export async function loader({ context }: Route.LoaderArgs) {
  const user = context.get(userContext);
  return data({ user });
}
```

root loader کاربر را از context دریافت کرده و برمی‌گرداند. این اطلاعات کاربر را از طریق مکانیزم loader data React Router برای همهٔ کامپوننت‌ها در دسترس قرار می‌دهد.

با بارگذاری اطلاعات کاربر در root، می‌توانیم hook سفارشی بسازیم تا از هر کامپوننتی به آن دسترسی داشته باشیم.

```typescript
// src/features/auth/hooks/use-user.ts

import { useRouteLoaderData } from 'react-router';

import type { RootLoaderData } from '@/types/app-context';

export function useUser() {
  const rootData = useRouteLoaderData<RootLoaderData>('root');
  return rootData?.user ?? null;
}
```

hook `useUser` اطلاعات root loader را گرفته و کاربر را از آن استخراج می‌کند. این hook را می‌توان از هر کامپوننتی در اپلیکیشن فراخوانی کرد تا به کاربر فعلی دسترسی پیدا کنیم.

در اینجا نحوهٔ استفادهٔ عملی آن را می‌بینیم.

```tsx
// src/app/routes/layout.tsx
import { useUser } from '@/features/auth/hooks/use-user';

export default function Layout() {
  const user = useUser();
  
  // ...
}
```

هر کامپوننتی حالا می‌تواند `useUser()` را فراخوانی کند تا کاربر فعلی را دریافت کند. اگر کاربر احراز هویت شده باشد، اطلاعاتش را برمی‌گرداند. در غیر این صورت null برمی‌گرداند.

authentication ما کار می‌کند و می‌توانیم در سراسر اپ به کاربر دسترسی داشته باشیم، اما هنوز به محافظت از بعضی routeها نیاز داریم تا فقط کاربران احراز هویت‌شده بتوانند به آن‌ها دسترسی پیدا کنند.

### محافظت از routeها {#h2_207}

بعضی routeها در اپلیکیشن ما فقط باید برای کاربران احراز هویت‌شده قابل دسترسی باشند. مثلاً صفحات dashboard برای کاربرانی که وارد نشده‌اند معنایی ندارند. به راهی نیاز داریم تا این routeها را محافظت کرده و کاربران احراز هویت‌نشده را به صفحهٔ ورود هدایت کنیم.

می‌توانیم middleware بسازیم که بررسی کند آیا کاربر احراز هویت شده یا نه. اگر نشده باشد، قبل از لود شدن route به صفحهٔ ورود هدایتش می‌کند.

```typescript
// src/app/middleware/protected.ts
import { redirect, type MiddlewareFunction } from 'react-router';

import { userContext } from './user';

export const protectedMiddleware: MiddlewareFunction = async (
  { context },
  next,
) => {
  const user = context.get(userContext);

  if (!user) {
    throw redirect('/auth/login');
  }

  return next();
};
```

این middleware ساده اما قدرتمند است. کاربر را از context گرفته و بررسی می‌کند آیا احراز هویت شده یا نه. اگر کاربری نباشد، redirect به صفحهٔ ورود پرتاب می‌کند که جلوی ادامهٔ درخواست را می‌گیرد. اگر کاربری باشد، `next()` را فراخوانی می‌کند تا پردازش درخواست ادامه یابد.

ترتیب اجرای middleware مهم است. user middleware باید قبل از protected middleware اجرا شود چون protected middleware به کاربری نیاز دارد که توسط user middleware بارگذاری شده است.

حالا می‌توانیم این middleware را برای محافظت از routeهای خاص اعمال کنیم. چون routeهای dashboard همه زیر dashboard layout تو در تو هستند، می‌توانیم middleware را آنجا اضافه کنیم تا همهٔ آن‌ها را یکجا محافظت کند.

```tsx
// src/app/routes/dashboard/layout.tsx

export const middleware = [protectedMiddleware];

export default function DashboardLayout() {
  // ...
}
```

با اضافه کردن `protectedMiddleware` به dashboard layout، همهٔ routeهای زیر dashboard بررسی authentication را انجام می‌دهند. اگر کاربری بدون ورود سعی کند به dashboard دسترسی پیدا کند، به صفحهٔ ورود هدایت می‌شود.

با وجود authentication، می‌دانیم کاربرانمان کیستند و می‌توانیم routeهای کامل را محافظت کنیم. با این حال، دانستن اینکه کاربر کیست به ما نمی‌گوید چه کارهایی در اپلیکیشن اجازهٔ انجامشان را دارد. اینجاست که authorization وارد می‌شود.

## مجوزدهی {#h1_208}

Authentication به ما می‌گوید کاربران کیستند، اما authorization تعیین می‌کند چه کارهایی می‌توانند انجام دهند. این دو مفهوم متفاوت هستند که با هم کار می‌کنند. Authentication پاسخ می‌دهد «آیا آن‌ هستی که می‌گویی؟» در حالی که authorization پاسخ می‌دهد «آیا اجازهٔ انجام این عمل را داری؟»

مثلاً، authentication به ما می‌گوید کاربری به نام John وارد شده است. authorization سپس تعیین می‌کند آیا John می‌تواند یک ایدهٔ خاص را ویرایش کند یا نه. شاید John فقط بتواند ایده‌های خودش را ویرایش کند، نه ایده‌هایی که کاربران دیگر ایجاد کرده‌اند. این نوع بررسی مجوز، authorization است.

بدون authorization، کاربران احراز هویت‌شده می‌توانند هر عملی در اپلیکیشن انجام دهند. می‌توانند محتوای کاربران دیگر را حذف کنند، داده‌هایی که متعلق به خودشان نیست را تغییر دهند، یا به امکاناتی دسترسی پیدا کنند که نباید داشته باشند. قوانین authorization مرزهای آنچه هر کاربر می‌تواند انجام دهد را تعریف می‌کنند.

در اپلیکیشن ما، برای چند سناریو به authorization نیاز داریم:

- کاربران فقط می‌توانند ایده‌های خودشان را ویرایش و حذف کنند
- کاربران نمی‌توانند ایده‌های خودشان را نقد کنند
- کاربران فقط یک‌بار می‌توانند هر ایده را نقد کنند
- کاربران فقط می‌توانند پروفایل خودشان را ویرایش کنند

authorization را با رویکرد policy-based پیاده‌سازی می‌کنیم. Policy توابعی هستند که کاربر فعلی و یک منبع را دریافت کرده و برمی‌گردانند آیا کاربر اجازهٔ انجام عملی روی آن منبع را دارد یا نه.

بیایید authorization policy برای منابع مختلف در اپلیکیشنمان بسازیم.

```typescript
// src/features/auth/lib/authorization-policies.ts

import type {
  CurrentUser,
  Idea,
  Review,
  User,
} from '@/types/generated/types.gen';

export const IdeaPolicies = {
  canCreate: (currentUser: CurrentUser | null): boolean => {
    const isAuthenticated = !!currentUser;
    return isAuthenticated;
  },

  canEdit: (currentUser: CurrentUser | null, idea: Idea): boolean => {
    const isIdeaAuthor = currentUser?.id === idea.authorId;
    return isIdeaAuthor;
  },

  canDelete: (currentUser: CurrentUser | null, idea: Idea): boolean => {
    const isIdeaAuthor = currentUser?.id === idea.authorId;
    return isIdeaAuthor;
  },
};

export const ReviewPolicies = {
  canCreate: (
    currentUser: CurrentUser | null,
    idea: Idea,
    existingReviews?: Review[],
  ): boolean => {
    const isAuthenticated = !!currentUser;
    if (!isAuthenticated) return false;

    const isIdeaAuthor = currentUser.id === idea.authorId;
    if (isIdeaAuthor) return false;

    const hasAlreadyReviewed = existingReviews?.some(
      (review) => review.authorId === currentUser.id,
    );
    if (hasAlreadyReviewed) return false;

    return true;
  },

  canEdit: (currentUser: CurrentUser | null, review: Review): boolean => {
    const isReviewAuthor = currentUser?.id === review.authorId;
    return isReviewAuthor;
  },

  canDelete: (currentUser: CurrentUser | null, review: Review): boolean => {
    const isReviewAuthor = currentUser?.id === review.authorId;
    return isReviewAuthor;
  },
};

export const ProfilePolicies = {
  canEdit: (currentUser: CurrentUser | null, user: User): boolean => {
    const isUserAuthor = currentUser?.id === user.id;
    return isUserAuthor;
  },
};
```

این authorization policy قوانین تجاری ما را برای اینکه چه کسی چه کاری می‌تواند انجام دهد، encapsulate می‌کنند. هر policy تابع ساده‌ای است که true یا false برمی‌گرداند بر اساس کاربر فعلی و منبعی که به آن دسترسی پیدا شده است.

policyها الگوی روشنی دارند:

- **policy ایده‌ها**: کاربران باید وارد شده باشند تا بتوانند ایده ایجاد کنند و فقط ایده‌هایی که نویسنده‌شان هستند را می‌توانند ویرایش یا حذف کنند
- **policy نقدها**: کاربران باید وارد شده باشند، نمی‌توانند ایده‌های خودشان را نقد کنند و نمی‌توانند یک ایده را دوبار نقد کنند
- **policy پروفایل**: کاربران فقط می‌توانند پروفایل خودشان را ویرایش کنند

با متمرکز کردن این قوانین در یک جا، به‌روزرسانی و نگهداریشان آسان می‌شود. اگر نیاز باشد تغییر دهیم چه کسی چه کاری می‌تواند انجام دهد، فقط توابع policy را به‌روزرسانی می‌کنیم.

حالا به راهی نیاز داریم تا این policyها را در کامپوننت‌هایمان استفاده کنیم. hookی می‌سازیم که authorization policy را در بر گرفته و استفاده‌شان را در سراسر اپلیکیشن آسان می‌کند.

```typescript
// src/features/auth/hooks/use-authorization.ts

import type { Idea, Review, User } from '@/types/generated/types.gen';

import {
  IdeaPolicies,
  ProfilePolicies,
  ReviewPolicies,
} from '../lib/authorization-policies';

import { useUser } from './use-user';

export function useAuthorization() {
  const currentUser = useUser();

  return {
    // Ideas
    canCreateIdea: () => IdeaPolicies.canCreate(currentUser),
    canEditIdea: (idea: Idea) => IdeaPolicies.canEdit(currentUser, idea),
    canDeleteIdea: (idea: Idea) => IdeaPolicies.canDelete(currentUser, idea),

    // Reviews
    canCreateReview: (idea: Idea, reviews?: Review[]) =>
      ReviewPolicies.canCreate(currentUser, idea, reviews),
    canEditReview: (review: Review) =>
      ReviewPolicies.canEdit(currentUser, review),
    canDeleteReview: (review: Review) =>
      ReviewPolicies.canDelete(currentUser, review),

    // Profile
    canEditProfile: (user: User) => ProfilePolicies.canEdit(currentUser, user),
  };
}
```

hook `useAuthorization` راه راحتی برای بررسی مجوزها در کامپوننت‌هایمان فراهم می‌کند. کاربر فعلی را با hook `useUser` دریافت کرده و آن را به توابع policy مناسب پاس می‌دهد. این API تمیزی برای بررسی‌های authorization در سراسر اپلیکیشن در اختیارمان قرار می‌دهد.

به‌جای import کردن توابع policy و کاربر فعلی در هر کامپوننت، فقط می‌توانیم `useAuthorization()` را فراخوانی کنیم و همهٔ بررسی‌های مجوز لازم را دریافت کنیم.

بیایید ببینیم چگونه این hook را در یک کامپوننت واقعی استفاده می‌کنیم.

```tsx
// src/app/routes/dashboard/ideas/ideas.tsx

import { Plus } from 'lucide-react';
import { Link } from 'react-router';

import { Button } from '@/components/ui/button';
import { useAuthorization } from '@/features/auth/hooks/use-authorization';
import { useCurrentUserIdeasQuery } from '@/features/ideas/api/get-current-user-ideas';
import { IdeasList } from '@/features/ideas/components/ideas-list';

import type { Route } from './+types/ideas';

export default function MyIdeasPage() {
  const ideasQuery = useCurrentUserIdeasQuery();
  const ideas = ideasQuery.data?.data;

  const { canCreateIdea } = useAuthorization();

  return (
    <div className="container mx-auto px-4 py-8">
      <div className="max-w-4xl mx-auto space-y-6">
        <div className="flex items-center justify-between">
          <div>
            <h1 className="text-3xl font-bold mb-2">My Ideas</h1>
            <p className="text-muted-foreground">
              Manage and track your submitted ideas
            </p>
          </div>
          {canCreateIdea() && (
            <Link to="/dashboard/ideas/new">
              <Button>
                <Plus className="mr-2 h-4 w-4" />
                New Idea
              </Button>
            </Link>
          )}
        </div>

        <IdeasList
          ideas={ideas}
          isLoading={ideasQuery.isLoading}
          emptyMessage="You haven't created any ideas yet."
          error={ideasQuery.error}
        />
      </div>
    </div>
  );
}
```

در این مثال، از hook `useAuthorization` استفاده می‌کنیم تا بررسی کنیم آیا کاربر فعلی می‌تواند ایده ایجاد کند. بر اساس این بررسی، دکمهٔ «New Idea» را به‌صورت شرطی رندر می‌کنیم. اگر کاربر مجوز نداشته باشد، دکمه به‌سادگی در UI ظاهر نمی‌شود.

این الگو در سراسر اپلیکیشن کار می‌کند. مجوزها را قبل از نمایش دکمه‌های عمل، قبل از هدایت به صفحات ویرایش، و در فرم‌ها قبل از فعال کردن دکمه‌های ارسال بررسی می‌کنیم. UI با آنچه هر کاربر اجازهٔ انجامش را دارد، تطبیق می‌یابد.

نکتهٔ مهمی که باید ذکر شود این است که بررسی‌های authorization در UI دربارهٔ تجربهٔ کاربری هستند، نه امنیت. آن‌ها جلوی دیدن کاربران از دکمه‌هایی برای اعمالی که نمی‌توانند انجام دهند را می‌گیرند که از سردرگمی و پیام‌های خطا جلوگیری می‌کند. با این حال، باید authorization را سمت server در API هم اعمال کنیم. کاربر مصممی می‌تواند بررسی‌های سمت client را دور بزند پس backend باید مجوزها را قبل از اجازهٔ هر عملی سمت server تأیید کند.

با وجود authentication و authorization، کاربران می‌توانند وارد شوند و اپلیکیشن می‌داند هر کاربر چه کارهایی می‌تواند انجام دهد. اما هنوز باید در برابر تهدیدات امنیتی که هر اپلیکیشن web را تحت تأثیر قرار می‌دهند، صرف‌نظر از authentication، محافظت کنیم.

## امن‌سازی اپلیکیشن {#h1_209}

Authentication به ما می‌گوید کاربران کیستند، اما امنیت دربارهٔ محافظت از آن‌ها و اپلیکیشنمان در برابر حملات مخرب است. حتی با authentication کامل، اپلیکیشن ما در برابر حملاتی آسیب‌پذیر است که داده‌ها را می‌دزدند، session کاربران را hijack می‌کنند یا کد مخرب اجرا می‌کنند.

اپلیکیشن‌های web با تهدیدات امنیتی زیادی روبه‌رو هستند، اما دو مورد از رایج‌ترین‌ها حملات XSS و security headerهایناقص هستند. حملات XSS scriptهای مخرب را در اپلیکیشن ما تزریق می‌کنند در حالی که security headerهایناقص اپلیکیشن را در برابر حملات مختلف مبتنی بر مرورگر آسیب‌پذیر می‌کند. بیایید ببینیم چگونه در برابر این تهدیدات محافظت کنیم.

### Sanitization محتوا {#h2_210}

**Cross-Site Scripting (XSS)** یکی از رایج‌ترین آسیب‌پذیری‌های امنیتی وب است. وقتی اتفاق می‌افتد که مهاجم کد JavaScript مخربی را در اپلیکیشن ما تزریق می‌کند و سپس در مرورگرهای کاربران دیگر اجرا می‌شود. مثلاً، اگر کاربری بتواند محتوایی شامل تگ‌های `<script>` ارسال کند و ما آن محتوا را بدون sanitization نمایش دهیم، script در مرورگر هر کاربری که آن را مشاهده می‌کند اجرا خواهد شد.

این به‌ویژه خطرناک است چون script مخرب با همان مجوزهای اپلیکیشن ما اجرا می‌شود. می‌تواند cookie بخواند، به local storage دسترسی پیدا کند، درخواست‌های API به‌جای کاربر ارسال کند یا کاربران را به سایت‌های مخرب هدایت کند. حتی با وجود اینکه از httpOnly cookie برای authentication استفاده می‌کنیم (که JavaScript نمی‌تواند بهشان دسترسی پیدا کند)، حملات XSS هنوز می‌توانند به‌عنوان کاربر واردشده عمل کنند.

در اپلیکیشن ما، کاربران می‌توانند ایده‌هایی با توضیحات به فرمت [Markdown](https://www.markdownguide.org/) ایجاد کنند. Markdown به HTML تبدیل می‌شود برای نمایش، یعنی HTMLای را رندر می‌کنیم که از ورودی کاربر آمده است. اگر این HTML را sanitize نکنیم، کاربران می‌توانند scriptهای مخرب در محتوای markdown خود تزریق کنند.

از [DOMPurify](https://github.com/cure53/DOMPurify) برای sanitization HTML قبل از رندر استفاده می‌کنیم. DOMPurify هر محتوای بالقوه خطرناک مثل تگ‌های `<script>`، event handlerهای inline و attributeهای خطرناک را حذف می‌کند در حالی که elementهای امن HTML مثل heading، لیست و لینک را حفظ می‌کند.

```tsx
// src/components/markdown-renderer.tsx

import DOMPurify from 'isomorphic-dompurify';
import { useMemo } from 'react';
import { remark } from 'remark';
import remarkGfm from 'remark-gfm';
import remarkHtml from 'remark-html';

export type MarkdownRendererProps = {
  content: string;
  className?: string;
};

export function MarkdownRenderer({
  content,
  className = '',
}: MarkdownRendererProps) {
  const htmlContent = useMemo(() => {
    try {
      // Process markdown to HTML
      const result = remark()
        .use(remarkGfm) // GitHub Flavored Markdown
        .use(remarkHtml, { sanitize: false })
        .processSync(content);

      const sanitizedHtml = DOMPurify.sanitize(result.toString(), {
        ALLOWED_TAGS: [
          'p',
          'br',
          'strong',
          'em',
          'u',
          's',
          'code',
          'pre',
          'h1',
          'h2',
          'h3',
          'h4',
          'h5',
          'h6',
          'ul',
          'ol',
          'li',
          'blockquote',
          'a',
          'table',
          'thead',
          'tbody',
          'tr',
          'th',
          'td',
          'hr',
        ],
        ALLOWED_ATTR: ['href', 'title', 'class'],
        ALLOW_DATA_ATTR: false,
        ALLOWED_URI_REGEXP:
          /^(?:(?:(?:f|ht)tps?|mailto):|[^a-z]|[a-z+.-]+(?:[^a-z+.-:]|$))/i,
      });

      return sanitizedHtml;
    } catch (error) {
      console.error('Error processing markdown:', error);
      return `<p>Error rendering markdown content</p>`;
    }
  }, [content]);

  return (
    <div
      className={`prose prose-sm max-w-none dark:prose-invert ${className}`}
      dangerouslySetInnerHTML={{ __html: htmlContent }}
    />
  );
}
```

بیایید نحوهٔ محافظت این کامپوننت در برابر XSS را بررسی کنیم:

- **پردازش دو مرحله‌ای**: اول، markdown را با استفاده از [remark](https://github.com/remarkjs/remark) به HTML تبدیل می‌کنیم. سپس، HTML را با `DOMPurify` sanitization می‌کنیم. sanitization داخلی remark را عمداً غیرفعال می‌کنیم چون `DOMPurify` بهتر عمل می‌کند.
- **لیست مجاز سخت‌گیرانه**: فقط تگ‌های HTML خاصی که برای متن امن و ضروری هستند مجازند. تگ‌هایی مثل `<script>`، `<iframe>` و `<object>` خودکار مسدود می‌شوند.
- **attributeهای محدود**: فقط attributeهای امن مثل `href`، `title` و `class` مجازند. Event handlerهایی مثل `onclick` مسدود می‌شوند.
- **پروتکل‌های امن**: برای لینکها، فقط پروتکل‌های `http:`، `https:` و `mailto:` مجازند. این از لینکهای `javascript:` که می‌توانند کد اجرا کنند جلوگیری می‌کند.

نتیجه این است که کاربران می‌توانند محتوای غنی markdown با فرمت‌بندی، لینک و لیست بنویسند، اما نمی‌توانند scriptهای مخرب یا elementهای خطرناک HTML تزریق کنند.

Sanitization محتوا در برابر حملات XSS محافظت می‌کند، اما لایهٔ امنیتی دیگری هم نیاز داریم: security header.

### Security headerها {#h2_211}

**Security header** HTTP headerهای خاصی هستند که به مرورگر می‌گویند چگونه رفتار کند هنگام مدیریت اپلیکیشن ما. مثل قوانینی هستند که مرورگر برای محافظت از کاربران در برابر حملات مختلف دنبال می‌کند. بدون این headerها، مرورگرها رفتار پیش‌فرضی دارند که همیشه امن نیستند.

مرورگرهای مدرن از security headerهای زیادی پشتیبانی می‌کنند که می‌توانند از حملاتی مثل clickjacking، MIME-type sniffing و cross-site scripting جلوگیری کنند. با تنظیم این headerها روی همهٔ پاسخ‌هایمان، چندین لایهٔ دفاعی ایجاد می‌کنیم که با هم برای محافظت از اپلیکیشن و کاربرانمان کار می‌کنند.

بیایید تابعی پیاده‌سازی کنیم که همهٔ security headerهای لازم را روی پاسخ‌هایمان اعمال کند.

```typescript
// src/lib/security-headers.ts

export function applySecurityHeaders(
  responseHeaders: Headers,
  nonce: string,
): void {
  const isProd = (process.env.NODE_ENV as string) === 'production';
  const apiUrl = process.env.VITE_API_URL as string | undefined;
  const securityHeaders = getSecurityHeaders(isProd, apiUrl, nonce);

  Object.entries(securityHeaders).forEach(([key, value]) => {
    responseHeaders.set(key, value);
  });
}

function getSecurityHeaders(
  isProd: boolean,
  apiUrl?: string,
  nonce?: string,
): Record<string, string> {
  return {
    'X-Frame-Options': 'DENY',
    'X-Content-Type-Options': 'nosniff',
    'Referrer-Policy': 'strict-origin-when-cross-origin',

    ...(isProd
      ? { 'Strict-Transport-Security': 'max-age=63072000; includeSubDomains' }
      : {}),

    'Content-Security-Policy': [
      "default-src 'self'",
      `script-src 'self' 'nonce-${nonce}'
      `style-src 'self' 'unsafe-inline' https://fonts.googleapis.com`,
      "font-src 'self' https://fonts.gstatic.com",
      "img-src 'self' data: https:",
      `connect-src 'self' ${apiUrl}`,
    ].join('; '),
  };
}
```

در اینجا عملکرد هر security header را می‌بینیم:

- `X‑Frame‑Options: DENY`: جلوی جاسازی سایت ما در iframe را می‌گیرد. این از حملات clickjacking جلوگیری می‌کند که در آن مهاجمان iframeهای نامرئی روی دکمه‌ها قرار می‌دهند تا کاربران را فریب دهند روی آن‌ها کلیک کنند.
- `[X‑Content‑Type‑Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options): nosniff`: جلوی حدس زدن نوع محتوای فایل‌ها توسط مرورگر را می‌گیرد. بدون این، مرورگر ممکن است فایلی را به‌عنوان JavaScript اجرا کند حتی اگر گفته باشیم تصویر است، که می‌تواند به حملات XSS منجر شود.
- `Referrer‑Policy: strict‑origin‑when‑cross‑origin`: کنترل می‌کند چه اطلاعاتی در header `Referer` ارسال شود. این تعادلی بین حریم خصوصی (نشت نکردن URL کامل به سایت‌های دیگر) و عملکرد (اجازهٔ analytics same-origin) برقرار می‌کند.
- [`Strict‑Transport‑Security`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security): مرورگرها را مجبور می‌کند فقط از اتصال HTTPS به سایت ما استفاده کنند. این از دزدیدن traffic توسط downgrade کردن کاربران به HTTP جلوگیری می‌کند. فقط در production فعال می‌کنیم چون در development از HTTP استفاده می‌کنیم.
- [`Content‑Security‑Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP): این قدرتمندترین security header است. کنترل می‌کند مرورگر از کجا می‌تواند منابع را load کند که در ادامه با جزئیات توضیح می‌دهیم.

header **Content-Security-Policy** (یا **CSP**) به مرورگر می‌گوید کدام منابع برای load کردن منابع امن هستند. بدون CSP، مرورگر script، استایل، تصویر و سایر منابع را از هر جایی load می‌کند. CSP لیست مجازی از منابع معتبر ایجاد می‌کند.

CSP ما چند directive دارد:

- `default‑src 'self'`: به‌صورت پیش‌فرض، فقط منابع از origin خودمان load شوند.
- `script‑src 'self' 'nonce‑...'`: فقط scriptهایی از origin ما یا scriptهایی با nonce مطابق load شوند.
- `style‑src 'self' 'unsafe‑inline' https://fonts.googleapis.com`: استایل‌ها از origin ما load شوند، inline style (لازم برای بعضی کتابخانه‌ها) مجاز باشد و Google Fonts مجاز باشد.
- `font‑src 'self' https://fonts.gstatic.com`: فونت‌ها از origin ما و CDN Google Fonts load شوند.
- `img‑src 'self' data: https:`: تصاویر از origin ما، data URI (برای تصاویر inline) و هر source HTTPS load شوند.
- `connect‑src 'self' [API_URL]`: فقط درخواست‌های API به origin ما و سرور API ما ارسال شوند.

nonce (شمارهٔ یکبار مصرف) در directive `script-src` مهم است. به‌جای اجازهٔ همهٔ scriptهای inline با `unsafe-inline`، یک مقدار تصادفی یکتا برای هر درخواست تولید کرده و فقط scriptهایی با آن nonce خاص مجاز هستیم. این از اجرای scriptهای تزریقی جلوگیری می‌کند حتی اگر مهاجمی موفق شود آن‌ها را در HTML ما قرار دهد. برای اطلاعات بیشتر دربارهٔ nonce در CSP می‌توانید [https://content-security-policy.com/nonce](https://content-security-policy.com/nonce) را بررسی کنید.

برای پیاده‌سازی nonce، باید nonce تصادفی جدیدی برای هر درخواست تولید کرده و هم در CSP header و هم در تگ‌های script استفاده کنیم. بیایید middleware بسازیم که nonce تولید کند.

```typescript
// src/app/middleware/nonce.ts

import { randomBytes } from 'node:crypto';

import { createContext, type MiddlewareFunction } from 'react-router';

export function generateNonce(): string {
  return randomBytes(16).toString('base64');
}

export const nonceContext = createContext<string>();

export const nonceMiddleware: MiddlewareFunction = async (
  { context },
  next,
) => {
  const nonce = generateNonce();
  context.set(nonceContext, nonce);

  return next();
};
```

این middleware nonce تصادفی برای هر درخواست تولید کرده و آن را در context ذخیره می‌کند. nonce هم در CSP header و هم در تگ‌های script استفاده می‌شود تا مرورگر بداند scriptهای ما معتبر هستند.

حالا باید security headerها را روی هر پاسخی که اپلیکیشن ما ارسال می‌کند اعمال کنیم.

```tsx
// src/app/entry.server.tsx

import { applySecurityHeaders } from '@/lib/security-headers';

import { nonceContext } from './middleware/nonce';

export const streamTimeout = 5_000;

export default function handleRequest(
  request: Request,
  responseStatusCode: number,
  responseHeaders: Headers,
  routerContext: EntryContext,
  loadContext: RouterContextProvider,
) {
  return new Promise((resolve, reject) => {
    // ...

    const nonce = loadContext.get(nonceContext);

    applySecurityHeaders(responseHeaders, nonce);

    // ...

    const { pipe, abort } = renderToPipeableStream(
      <ServerRouter context={routerContext} url={request.url} nonce={nonce} />,
      {
        nonce,
        // ...
      },
    );
  });
}
```

در فایل `entry.server.tsx`، رندر سمت server اپلیکیشن React را مدیریت می‌کنیم. اینجاست که به response headerها قبل از ارسال به مرورگر دسترسی داریم و مکان مناسبی برای اعمال security headerهایمان است.

تغییرات کلیدی عبارتند از:

- **دریافت nonce**: nonce را از context که توسط middleware nonce تنظیم شده دریافت می‌کنیم
- **اعمال security headerها**: `applySecurityHeaders` را فراخوانی می‌کنیم تا همهٔ security headerها را به پاسخ اضافه کند
- **پاس دادن nonce به React**: nonce را به کامپوننت `ServerRouter` پاس می‌دهیم تا بتواند در تگ‌های script در سراسر اپلیکیشن استفاده شود

nonce به تابع رندر React پاس داده می‌شود که خودکار آن را به تگ‌های script اضافه می‌کند که React تولید می‌کند. این تضمین می‌کند JavaScript اپلیکیشن ما بتواند اجرا شود در حالی که هر script تزریقی مسدود می‌شود.

در نهایت، باید nonce را در root layout در دسترس قرار دهیم تا بتوانیم آن را به Scripts و `ScrollRestoration` که React Router فراهم می‌کند اضافه کنیم.

```tsx
// src/app/root.tsx

import { QueryClientProvider } from '@tanstack/react-query';
import { NuqsAdapter } from 'nuqs/adapters/react-router/v7';
import { useState } from 'react';
import {
  data,
  isRouteErrorResponse,
  Links,
  Meta,
  Outlet,
  Scripts,
  ScrollRestoration,
  useLoaderData,
} from 'react-router';

import { Notifications } from '@/components/notifications';
import { createQueryClient } from '@/lib/react-query';

import type { Route } from './+types/root';
import './app.css';
import { nonceContext, nonceMiddleware } from './middleware/nonce';
import { userContext, userMiddleware } from './middleware/user';

export const middleware = [nonceMiddleware, userMiddleware];

export async function loader({ context }: Route.LoaderArgs) {
  const user = context.get(userContext);
  const nonce = context.get(nonceContext);
  return data({ user, nonce });
}

// ...

export function Layout({ children }: { children: React.ReactNode }) {
  const { nonce } = useLoaderData<typeof loader>();

  return (
    <html lang="en">
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <Meta />
        <Links />
      </head>
      <body>
        {children}
        <ScrollRestoration nonce={nonce} />
        <Scripts nonce={nonce} />
      </body>
    </html>
  );
}
```

در root layout، nonce را از loader data بارگذاری کرده و به کامپوننت‌های `ScrollRestoration` و `Scripts` پاس می‌دهیم. این کامپوننت‌ها attribute nonce را به تگ‌های script خود اضافه می‌کنند و تحت Content Security Policy ما اجرا می‌شوند.

middleware به ترتیب اجرا می‌شود: اول `nonceMiddleware` nonce را تولید می‌کند، سپس `userMiddleware` کاربر را بارگذاری می‌کند. هر دو مقدار در context ذخیره شده و در root loader بارگذاری می‌شوند و برای کل اپلیکیشن در دسترس قرار می‌گیرند.

با وجود این اقدامات امنیتی، اپلیکیشن ما در برابر آسیب‌پذیری‌های رایج وب محافظت شده است. ترکیب httpOnly cookie، sanitization محتوا و security header چندین لایهٔ دفاعی ایجاد می‌کند که با هم برای ایمن نگه داشتن کاربرانمان کار می‌کنند.

## خلاصه {#h1_212}

در این فصل، سیستم کامل authentication و authorization پیاده‌سازی کردیم و اپلیکیشن را در برابر تهدیدات رایج امن‌سازی کردیم. از گسترش API client برای مدیریت خودکار tokenهای authentication شروع کردیم، از جمله refresh خودکار token وقتی access token منقضی می‌شود. این تجربهٔ کاربری seamless ایجاد می‌کند که کاربران بدون وقفه وارد باقی می‌مانند.

flowهای ثبت‌نام، ورود و خروج ساختیم که از httpOnly cookie برای ذخیرهٔ امن tokenهای authentication استفاده می‌کنند. این cookie توسط JavaScript قابل دسترسی نیستند و از حملات XSS محافظت می‌کنند. middleware ساختیم تا کاربر فعلی را در هر درخواست بارگذاری کند و کاربر را با hook سادهٔ `useUser` در سراسر اپلیکیشن در دسترس قرار دادیم.

برای محافظت از routeهای حساس، middleware protected پیاده‌سازی کردیم که کاربران احراز هویت‌نشده را به صفحهٔ ورود هدایت می‌کند. با اضافه کردن این middleware به کامپوننت‌های layout، می‌توانیم بخش‌های کامل اپلیکیشنمان را با یک خط کد محافظت کنیم.

authorization policy پیاده‌سازی کردیم تا کنترل کنیم کاربران احراز هویت‌شده چه کارهایی می‌توانند انجام دهند. این policyها تعریف می‌کنند چه کسی می‌تواند منابع مختلف اپلیکیشنمان را ایجاد، ویرایش و حذف کند. hook `useAuthorization` این policyها را در کامپوننت‌ها آسان استفاده می‌کند و به ما اجازه می‌دهد عناصر UI را بر اساس مجوزهای کاربر نمایش یا پنهان کنیم.

فراتر از authentication و authorization، اقدامات امنیتی برای محافظت در برابر حملات پیاده‌سازی کردیم. از DOMPurify برای sanitization محتوای HTML تولیدشده توسط کاربر استفاده کردیم و از حملات XSS جلوگیری کردیم در حالی که به کاربران اجازهٔ ایجاد محتوای غنی markdown را دادیم. security headerهای جامعی از جمله Content Security Policy با nonce اضافه کردیم تا چندین لایهٔ دفاعی در برابر حملات مختلف مبتنی بر مرورگر ایجاد کنیم.

این practiceهای امنیتی فقط خوب هستند — برای هر اپلیکیشن production ضروری‌اند. Authentication به ما می‌گوید کاربران کیستند، authorization کنترل می‌کند چه کارهایی می‌توانند انجام دهند، و اقدامات امنیتی هم کاربران و هم اپلیکیشنمان را در برابر عاملان مخرب محافظت می‌کنند.
