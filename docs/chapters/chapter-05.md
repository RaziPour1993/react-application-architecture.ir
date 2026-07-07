# فصل ۵: ارتباط با API {#h1_167 .chapterTitle}

هر اپلیکیشن مدرنی برای دریافت داده، ارسال فرم‌ها و به‌روز نگه داشتن محتوا نیاز به ارتباط با یک API backend دارد. در حالی که routing کاربران را به صفحهٔ درست هدایت می‌کند، لایهٔ API محتوای واقعی آن صفحات را فراهم می‌کند. پیاده‌سازی این لایه با type safety، caching و error handling اپلیکیشن را توسعه و نگهداری آسان‌تر می‌کند.

موارد زیر را پوشش می‌دهیم:

- ایجاد API client
- تولید [TypeScript](https://www.typescriptlang.org/) type و validation schema از مشخصات [OpenAPI](https://www.openapis.org/)
- راه‌اندازی [React Query](https://tanstack.com/query/latest)
- ایجاد لایهٔ API برای اپلیکیشن
- یکپارچه‌سازی با اپلیکیشن

در پایان این فصل، یک لایهٔ API کامل با type safety خودکار، caching و یکپارچه‌سازی روان با رندر سمت سرور و سمت کلاینت خواهیم داشت.

## پیش‌نیازهای فنی {#h1_168}

قبل از شروع، باید پروژه را راه‌اندازی کنیم. برای توسعهٔ پروژه به ابزارهای زیر روی کامپیوتر نیاز داریم:

- [Node.js](https://nodejs.org/) نسخه ۲۴ یا بالاتر. نسخه ۱۱ [npm](https://www.npmjs.com/) همراه Node عرضه می‌شود. می‌توانیم با اجرای `node -v` و `npm -v` در ترمینال این را تأیید کنیم. راه‌های مختلفی برای نصب Node.js و npm وجود دارد. این مقالهٔ مفید جزئیات بیشتری ارائه می‌دهد: https://www.nodejsdesignpatterns.com/blog/5-ways-to-install-node-js .
- [VS Code](https://code.visualstudio.com/) (اختیاری)، یک ویرایشگر محبوب برای [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) و TypeScript. متن‌باز است، پشتیبانی TypeScript خوبی دارد و افزونه‌های زیادی ارائه می‌دهد. از https://code.visualstudio.com قابل دانلود است.

کد این کتاب در مخزن کتاب موجود است. برای دسترسی به لینک مخزن، مراحل بخش «*دانلود فایل‌های کد نمونه*» در «*پیش‌گفتار*» را دنبال کنید. آن را کلون کنید و وارد ریشهٔ مخزن شوید:

```sh
git clone https://github.com/PacktPublishing/React-Application-Architecture-for-Production-Second-Edition.git
```

مخزن شامل پوشه‌های فصل با کد هر فصل به همراه پوشهٔ مشترک `api` است که سرور API مورد استفاده در همهٔ فصل‌ها را شامل می‌شود.

ما روی **فصل ۵** کار می‌کنیم، پس وارد پوشهٔ `chapter-05` شوید:

```sh
cd React-Application-Architecture-for-Production-Second-Edition/chapter-05
```

سپس dependencyها را نصب کنید:

```sh
npm install
```

همچنین باید متغیرهای محیطی را فراهم کنیم:

```sh
cp .env.example .env
```

در این مرحله، frontend باید آماده باشد و روی http://localhost:5173 اجرا شود.

همچنین باید سرور API را اجرا کنیم.

پنجرهٔ ترمینال جدیدی باز کنید و وارد پوشهٔ `api` شوید:

```sh
cd React-Application-Architecture-for-Production-Second-Edition/api
```

اسکریپت setup برای **فصل ۵** را اجرا کنید تا همه‌چیز پیکربندی شود:

```sh
npm run setup 05
```

سپس سرور API را استارت کنید:

```sh
npm run dev
```

سرور API حالا باید روی http://localhost:9999 در حال اجرا باشد.

برای اطلاعات بیشتر دربارهٔ جزئیات setup، فایل `README.md` را بررسی کنید.

## ایجاد API client {#h1_169}

هر اپلیکیشنی نیاز به راهی برای ارتباط با backend API خود دارد. در مرورگرهای مدرن می‌توانیم از `fetch` API برای ارسال درخواست‌های HTTP استفاده کنیم، و اگرچه می‌توانیم مستقیماً از `fetch` استفاده کنیم، این یعنی تعریف base URL، header، error handling و request formatting در هر درخواست. به‌جای این کار، یک API client متمرکز می‌سازیم که این موارد را یکبار تعریف کند و رابط تمیزی برای استفاده در سراسر اپلیکیشن فراهم کند.

API client فقط یک wrapper دور `fetch` API است که رابط یکپارچه‌ای برای ارسال درخواست‌های HTTP فراهم می‌کند. چیزهای تکراری‌ای را که برای هر درخواست لازم داریم مانند تنظیم base URL، header و error handling را انتزاع می‌کند.

بیایید با تعریف پارامترهایی که باید به API client پاس دهیم شروع کنیم:

```ts
// src/lib/api.ts

type RequestOptions<TBody = unknown> = {
  method?: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';
  headers?: Record<string, string>;
  body?: TBody;
  params?: Record<string, string | number | boolean | undefined | null>;
};
```

wrapper منتظر URL درخواست و آبجکت options است که پارامترهای زیر را شامل می‌شود:

- متد درخواست که می‌تواند یکی از `GET`، `POST`، `PUT`، `PATCH` یا `DELETE` باشد.
- headerهای درخواست که می‌تواند record از key-value pair باشد.
- body درخواست.
- پارامترهای درخواست که می‌تواند record از key-value pair باشد. این‌ها query parameterها هستند (مثلاً `?page=1&limit=10`).

حالا ببینیم wrapper ما چه شکلی است:

```ts
// src/lib/api.ts

import { env } from '@/config/env';

async function fetchApi<T, TBody = unknown>(
  url: string,
  options: RequestOptions<TBody> = {},
): Promise<T> {
  const { method = 'GET', headers = {}, body, params } = options;

  const fullUrl = new URL(url, env.API_URL);
  if (params) {
    Object.entries(params).forEach(([key, value]) => {
      if (value != null) {
        fullUrl.searchParams.set(key, String(value));
      }
    });
  }

  const makeRequest = async (): Promise<Response> => {
    try {
      return await fetch(fullUrl, {
        method,
        headers: {
          Accept: 'application/json',
          ...(body ? { 'Content-Type': 'application/json' } : {}),
          ...headers,
        },
        body: body ? JSON.stringify(body) : undefined,
      });
    } catch (error) {
      if (error instanceof TypeError) {
        const networkError = new Error(
          'Network error. Please check your connection.',
        );

        throw networkError;
      }
      throw error;
    }
  };

  let response = await makeRequest();

  // ...
}
```

ارسال درخواست کاملاً ساده است. از params (اگر ارائه شده باشد) برای تشکیل URL کامل درخواست با افزودن query parameterها به base URL استفاده می‌کنیم. همچنین بعضی headerهای پیش‌فرض مانند `Accept: application/json` و اگر body وجود داشته باشد `Content-Type: application/json` را تنظیم می‌کنیم. همچنین مطمئن می‌شویم body قبل از ارسال به API به صورت string درآید چون سرور API انتظار JSON object دارد.

حالا ببینیم پاسخ را چگونه handle می‌کنیم:

```ts
// src/lib/api.ts

// ...

async function fetchApi<T, TBody = unknown>(
  url: string,
  options: RequestOptions<TBody> = {},
): Promise<T> {

  // ...

  let response = await makeRequest();

  if (!response.ok) {
    let message = response.statusText;
    try {
      const errorData = await response.json();
      message = errorData.message || message;
    } catch {
  // response body may not be JSON, use the statusText as the message instead
    }

    throw new Error(message);
  }

  try {
    return await response.json();
  } catch {
    throw new Error('Invalid response from server');
  }
}
```

اگر پاسخ موفقیت‌آمیز نباشد، آن را به‌درستی handle می‌کنیم. اگر همه‌چیز درست باشد، پاسخ JSON را برمی‌گردانیم.

تابع `fetchApi` هستهٔ API client ماست. درخواست و پاسخ HTTP واقعی، error handling و غیره را handle می‌کند، اما مستقیماً در اپلیکیشن از آن استفاده نمی‌کنیم. در عوض از رابط مناسب‌تری استفاده می‌کنیم که استفاده و درک آسان‌تری دارد.

```ts
// src/lib/api.ts

export const api = {
  get<T>(url: string, options?: RequestOptions): Promise<T> {
    return fetchApi<T>(url, { ...options, method: 'GET' });
  },
  post<T, TBody = unknown>(
    url: string,
    body?: TBody,
    options?: RequestOptions,
  ): Promise<T> {
    return fetchApi<T, TBody>(url, { ...options, method: 'POST', body });
  },
  put<T, TBody = unknown>(
    url: string,
    body?: TBody,
    options?: RequestOptions,
  ): Promise<T> {
    return fetchApi<T, TBody>(url, { ...options, method: 'PUT', body });
  },
  patch<T, TBody = unknown>(
    url: string,
    body?: TBody,
    options?: RequestOptions,
  ): Promise<T> {
    return fetchApi<T, TBody>(url, { ...options, method: 'PATCH', body });
  },
  delete<T>(url: string, options?: RequestOptions): Promise<T> {
    return fetchApi<T>(url, { ...options, method: 'DELETE' });
  },
};
```

این آبجکت رابط ساده‌ای با متدهایی برای هر HTTP verb فراهم می‌کند: `get`، `post`، `put`، `patch` و `delete`. هر متد تابع `fetchApi` را wrap می‌کند و متد HTTP مناسب را پاس می‌دهد. این کار API callهای ما را در سراسر اپلیکیشن تمیزتر و خواناتر می‌کند.

حالا اگر بخواهیم از API client در اپلیکیشن استفاده کنیم، این‌طور می‌توانیم انجام دهیم:

```ts
import { api } from '@/lib/api';

const idea = await api.get<Idea>('/ideas/1');

const newIdea = await api.post<Idea>('/ideas', { title: 'New Idea' });
```

نکتهٔ عالی این رویکرد این است که می‌توانیم کنترل کنیم چه پارامترهایی به `fetchApi` برای هر متد پاس دهیم. مثلاً point زیادی در پاس دادن body به متد `get` نیست چون نباید body داشته باشد. به این شکل، هر متد type-safe است و نمی‌توانیم پارامترهای اشتباه پاس دهیم.

با API client در جای خود، باید مطمئن شویم داده‌ای که ارسال و دریافت می‌کنیم با آنچه backend انتظار دارد مطابقت دارد. ببینیم چگونه می‌توانیم TypeScript type را از مشخصات API تولید کنیم تا همه‌چیز هماهنگ بماند.

## تولید TypeScript type و validation schema از مشخصات OpenAPI {#h1_170}

وقتی دادهٔ ما از API می‌آید، TypeScript هیچ راهی برای دانستن نوع داده ندارد. اگرچه می‌توانیم برای آنچه از API برمی‌گردد type تعریف کنیم، تضمینی وجود ندارد که داده همیشه صحیح باشد. مثلاً اگر property به یک آبجکت پاسخ API اضافه یا حذف شود، اپلیکیشن frontend ما از آن خبر ندارد و ممکن است اپلیکیشن خراب شود.

برای type-safe کردن API callها، باید typeهای مناسبی برای دادهٔ درخواست و پاسخ فراهم کنیم.

### OpenAPI چیست و چرا type تولید می‌کنیم؟ {#h2_171}

OpenAPI یک فرمت استاندارد برای توصیف APIهای REST است. backend ما یک مشخصات OpenAPI در نقطهٔ انتهایی `${API_URL}/doc` فراهم می‌کند که همهٔ endpointها، داده‌ای که می‌پذیرند و آنچه برمی‌گردانند را لیست می‌کند.

به‌جای نوشتن و نگهداری دستی type برای هر endpoint، آن‌ها را از مشخصات OpenAPI تولید می‌کنیم. این کار typeهای frontend ما را به‌صورت خودکار با backend هماهنگ نگه می‌دارد. وقتی API تغییر می‌کند، typeها را دوباره تولید می‌کنیم و TypeScript دقیقاً به ما می‌گوید چه چیزی نیاز به به‌روزرسانی دارد.

این قرارداد فقط به اندازهٔ مشخصات خودش قابل اعتماد است. مهم است که backend مشخصات OpenAPI را دقیق و به‌روز نگه دارد وگرنه typeهای تولیدشده گمراه‌کننده خواهند بود.

### پیکربندی تولید type {#h2_172}

برای تولید type از مشخصات OpenAPI، از پکیج `@hey-api/openapi-ts` استفاده می‌کنیم. ابتدا پیکربندی ابزار تولیدکننده را در `openapi-ts.config.ts` تنظیم کنیم:

```ts
// openapi-ts.config.ts

export default {
  input: `${process.env.VITE_API_URL}/doc`,
  output: {
    format: 'prettier', // Run prettier to format the generated code.
    path: './src/types/generated', // Output directory for the generated types.
  },
  plugins: [
    {
      name: '@hey-api/typescript', // Generate TypeScript types.
      exportFromIndex: false,
    },
    'zod', // We can also generate runtime validation schemas with Zod.
  ],
};
```

برای اجرای تولیدکننده، می‌توانیم از دستور زیر استفاده کنیم:

```sh
npm run generate:openapi
```

این در فایل `package.json` به این شکل تعریف شده است:

```json
// package.json
"generate:openapi": "dotenv -e .env -- openapi-ts"
```

بعد از اجرای دستور، می‌توانیم TypeScript typeهای تولیدشده را در فایل `src/types/generated/types.gen.ts` پیدا کنیم:

```ts
// src/types/generated/types.gen.ts

// ...

export type User = {
  id: string;
  email: string;
  username: string;
  bio: string;
  createdAt: string;
  updatedAt: string;
};

export type Idea = {
  id: string;
  title: string;
  shortDescription: string;
  description: string;
  tags: Array<string>;
  authorId: string;
  author: UserSummary;
  reviewsCount: number;
  avgRating: number | null;
  createdAt: string;
  updatedAt: string;
};


export type Review = {
  id: string;
  content: string;
  rating: number;
  authorId: string;
  author: UserSummary;
  ideaId: string;
  idea?: IdeaSummary;
  createdAt: string;
  updatedAt: string;
};

// ...
```

این typeهای تولیدشده ساختار داده‌ای را که API ما برمی‌گرداند توصیف می‌کنند. تولیدکننده برای هر request body، پاسخ و مدل دادهٔ تعریف‌شده در مشخصات OpenAPI یک type ایجاد می‌کند.

علاوه بر TypeScript type، می‌توانیم validation schema با [Zod](https://zod.dev/) هم تولید کنیم که می‌تواند برای اعتبارسنجی داده در زمان اجرا استفاده شود.

schemaهای Zod تولیدشده را می‌توانیم در فایل `src/types/generated/zod.gen.ts` پیدا کنیم:

```ts
// src/types/generated/zod.gen.ts

// ...

export const zUser = z.object({
  id: z.string(),
  email: z.string(),
  username: z.string(),
  bio: z.string(),
  createdAt: z.string(),
  updatedAt: z.string(),
});

export const zReview = z.object({
  id: z.string(),
  content: z.string(),
  rating: z.number(),
  authorId: z.string(),
  author: zUserSummary,
  ideaId: z.string(),
  idea: z.optional(zIdeaSummary),
  createdAt: z.string(),
  updatedAt: z.string(),
});

// ...
```

این schemaها آینهٔ typeهای TypeScript هستند اما validation در زمان اجرا فراهم می‌کنند. می‌توانیم از آن‌ها برای تأیید اینکه داده‌ای که از API دریافت می‌کنیم با آنچه انتظار داریم مطابقت دارد استفاده کنیم و مشکلاتی را که TypeScript به‌تنهایی در زمان اجرا نمی‌تواند تشخیص دهد بگیریم.

می‌توانیم برای اعتبارسنجی داده در زمان اجرا این‌طور استفاده کنیم:

```ts
const idea = zIdea.parse(idea);
```

اگر داده معتبر نباشد، schema Zod خطای validation ایجاد می‌کند. فعلاً نیازی نیست نگران این باشیم؛ در ادامه می‌بینیم چگونه از این schemaهای validation هنگام ساخت لایهٔ API و فرم‌ها استفاده می‌شود.

نکته: هنوز باید به یاد داشته باشیم که دستور تولیدکننده را به‌صورت دستی اجرا کنیم تا typeها تولید شوند، چون اگر API تغییر کند و typeها را دوباره تولید نکنیم ممکن است out of sync شوند. می‌توانیم این مشکل را با کمک CI/CD pipeline حل کنیم. CI/CD را در فصل‌های آینده پوشش می‌دهیم، اما اینجا مروری کلی از نحوهٔ حل این مشکل ارائه می‌دهیم:

اگر اپلیکیشن‌های frontend و backend در یک مخزن باشند، می‌توانیم CI/CD pipeline خود را طوری پیکربندی کنیم که هر زمان مشخصات API تغییر کند typeها به‌صورت خودکار تولید شوند. مرحلهٔ typecheck در CI سپس تأیید می‌کند که کد frontend با typeهای جدید سازگار است. اگر تغییرات API کد موجود frontend را خراب کند، typecheck با شکست مواجه می‌شود و مانع merge شده و ما را از لزوم رفع تغییرات breaking آگاه می‌کند. می‌توانیم این فرآیند را این‌طور تصور کنیم:

شکل ۵.۱ – فلو هنگامی که backend و frontend در یک مخزن هستند

اگر در مخزن‌های مختلف باشند، می‌توانیم CI/CD pipeline را در مخزن backend پیکربندی کنیم تا هر زمان مشخصات API تغییر کند، CI/CD در مخزن frontend را trigger کند. این workflow typeهای TypeScript به‌روزرسانی‌شده را تولید و یک pull request ایجاد می‌کند. CI pipeline سپس typechecking و testها را روی این PR اجرا کرده و هر تغییر breaking را می‌گیرد. می‌توانیم PR را بررسی کنیم و تغییرات را ببینیم و قبل از merge هر تغییر breaking را رفع کنیم. این‌طور به نظر می‌رسد:

شکل ۵.۲ – فلو هنگامی که backend و frontend در مخزن‌های مختلف هستند

با typeها و schemaهای validation تولیدشده و همیشه هماهنگ با backend، بنیان خوبی برای ساخت API callهای type-safe و validated داریم. مرحلهٔ بعد، راه‌اندازی React Query برای مدیریت نحوهٔ دریافت و caching این داده است.

## راه‌اندازی React Query {#h1_173}

هنگام انجام API call در اپلیکیشن‌های [React](https://react.dev/)، چیزهای زیادی برای handle کردن داریم مانند loading state، error handling، caching، request deduplication و بیشتر.

React Query برای ساده‌سازی فرآیند مدیریت server state وارد می‌شود.

### چرا React Query؟ {#h2_174}

**React Query** کتابخانه‌ای است که state ناهمگام را در اپلیکیشن‌های React مدیریت می‌کند. بدون آن، باید به‌صورت دستی loading state را ردیابی کنیم، خطاها را handle کنیم، caching پیاده‌سازی کنیم، درخواست‌های duplicate جلوگیری کنیم و تصمیم بگیریم چه زمانی داده را دوباره دریافت کنیم. React Query همهٔ این‌ها را به‌صورت خودکار برای ما انجام می‌دهد و اجازه می‌دهد روی ساخت feature تمرکز کنیم به‌جای اختراع مجدد چرخ با راه‌حل خودمان.

### پیکربندی React Query برای اپلیکیشن {#h2_175}

React Query به یک نمونهٔ `QueryClient` برای مدیریت query نیاز دارد. تابعی می‌سازیم که client را با پیش‌فرض‌های مناسب برای اپلیکیشن پیکربندی کند:

```ts
// src/lib/react-query.ts

import { QueryClient, type QueryClientConfig } from '@tanstack/react-query';

export function createQueryClient(config: Partial<QueryClientConfig> = {}) {
  return new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 60 * 1000,
      },
    },
    ...config,
  });
}
```

زمان stale را ۶۰ ثانیه تنظیم می‌کنیم، یعنی داده یک دقیقه تازه می‌ماند قبل از اینکه React Query دوباره دریافت آن را در نظر بگیرد.

حالا باید QueryClient را برای کل اپلیکیشن در دسترس قرار دهیم و routeهای خود را در `QueryClientProvider` wrap کنیم:

```tsx
// src/app/root.tsx

import { QueryClientProvider } from '@tanstack/react-query';
import { useState } from 'react';
import { Outlet } from 'react-router';

import { createQueryClient } from '@/lib/react-query';

export default function App() {
  const [queryClient] = useState(() => createQueryClient());
  return (
    <QueryClientProvider client={queryClient}>
      <Outlet />
    </QueryClientProvider>
  );
}
```

نمونهٔ QueryClient را یکبار با hook `useState` و initializer تنبل ایجاد می‌کنیم. این تضمین می‌کند client فقط یکبار ایجاد شود و در طول عمر اپلیکیشن باقی بماند. `QueryClientProvider` client را از طریق hook `useQueryClient` در اپلیکیشن در دسترس قرار می‌دهد.

با React Query پیکربندی‌شده، اپلیکیشن ما حالا زیرساخت لازم برای مدیریت کارآمد server state را دارد. مرحلهٔ بعد ساخت لایهٔ API ماست: توابع و hookهایی که داده را در سراسر اپلیکیشن دریافت و تغییر می‌دهند.

## ایجاد لایهٔ API برای اپلیکیشن {#h1_176}

حالا که API client و React Query را راه‌اندازی کردیم، می‌توانیم لایهٔ API واقعی را بسازیم که اپلیکیشن از آن استفاده می‌کند.

قبل از شروع ساخت لایهٔ API، باید query keyها را سازمان‌دهی کنیم.

### سازمان‌دهی query key {#h2_177}

React Query از query key برای شناسایی و caching query استفاده می‌کند. هر کلید یکتا نشان‌دهندهٔ یک قطعه دادهٔ یکتا در cache است.

می‌توانیم این کلیدها را inline در هر query تعریف کنیم، اما سازمان‌دهی آن‌ها در یک جا مدیریت و استفادهٔ مجدد از آن‌ها را آسان‌تر می‌کند. query keyهای feature ایده‌ها را ببینیم:

```ts
// src/features/ideas/config/query-keys.ts

import type { GetAllIdeasData } from '@/types/generated/types.gen';

export const ideasQueryKeys = {
  all: ['ideas'] as const,
  lists: () => [...ideasQueryKeys.all, 'list'] as const,
  list: (params?: GetAllIdeasData['query']) =>
    [...ideasQueryKeys.lists(), params] as const,
  details: () => [...ideasQueryKeys.all, 'detail'] as const,
  detail: (id: string) => [...ideasQueryKeys.details(), id] as const,
  current: () => [...ideasQueryKeys.all, 'current'] as const,
  byUser: (username: string) =>
    [...ideasQueryKeys.all, 'user', username] as const,
  tags: () => [...ideasQueryKeys.all, 'tags'] as const,
} as const;
```

این ساختار الگویی را دنبال می‌کند که از یک کلید ریشه (`ideas`) شروع می‌شود و کلیدهای دقیق‌تری برای انواع مختلف query می‌سازد. مثلاً `detail(id)` کلیدی مانند `['ideas', 'detail', '123']` ایجاد می‌کند. این کار queryهای مرتبط را invalidate کردن آسان می‌کند؛ می‌توانیم همهٔ list ایده‌ها را با هدف قرار دادن `ideasQueryKeys.lists()` یا فقط یک جزئیات خاص را با هدف قرار دادن `ideasQueryKeys.detail(id)` invalidate کنیم.

حالا آمادهٔ تعریف queryهایمان هستیم.

### تعریف query {#h2_178}

هر query الگوی یکسانی دارد: تابع fetcher که API call را انجام می‌دهد، query option که query را پیکربندی می‌کند و custom hook که استفاده از آن را آسان می‌کند. این الگو را در عمل با دریافت یک ایده بر اساس ID آن ببینیم:

```ts
// src/features/ideas/api/get-idea-by-id.ts
import { queryOptions, useQuery } from '@tanstack/react-query';
import { api } from '@/lib/api';
import type { GetIdeaByIdResponse } from '@/types/generated/types.gen';
import {
  zGetIdeaByIdData,
  zGetIdeaByIdResponse,
} from '@/types/generated/zod.gen';
import { ideasQueryKeys } from '../config/query-keys';

// Fetcher function for making API call to fetch an idea by its ID
export async function getIdeaById(id: string): Promise<GetIdeaByIdResponse> {
  // Validate input data using the Zod schema
  // In this example, we need to validate the path parameter `id`
  // If it is not provided, the Zod schema will throw a validation error.
  const validatedData = zGetIdeaByIdData.parse({ path: { id } });

  // Make the API request
  const response = await api.get<GetIdeaByIdResponse>(
    `/ideas/${validatedData.path.id}`,
  );

  // Validate response data using the Zod schema
  // In this example, we need to validate the response data
  return zGetIdeaByIdResponse.parse(response);
}

// Query options factory for the query
export function getIdeaByIdQueryOptions(id: string) {
  return queryOptions({
    queryKey: ideasQueryKeys.detail(id), // Use the query key from the query keys
    queryFn: () => getIdeaById(id), // Use the fetcher function to fetch the data
  });
}

// Custom hook for fetching an idea by ID that returns the query
export function useIdeaByIdQuery({
  id,
  options,
}: {
  id: string;
  options?: Omit<
    ReturnType<typeof getIdeaByIdQueryOptions>,
    'queryKey' | 'queryFn'
  >;
}) {
  return useQuery({
    ...getIdeaByIdQueryOptions(id), // Using the query options factory to create the query
    ...options, // Allowing to override the query options
  });
}
```

این الگو سه چیز به ما می‌دهد: تابع fetcher که API call و validation را handle می‌کند، query options factory که query را پیکربندی می‌کند و custom hook که اجازه می‌دهد query را در component استفاده کنیم. پارامتر options به componentها اجازه می‌دهد تنظیمات خاص مانند enable یا disable کردن query را override کنند.

حالا ببینیم mutation برای عملیاتی که داده را روی سرور تغییر می‌دهند چگونه کار می‌کند.

### تعریف mutation {#h2_179}

Mutation برای عملیاتی است که داده را روی سرور تغییر می‌دهند. همان الگوی query را دنبال می‌کنند: تابع fetcher، mutation option و custom hook. نحوهٔ ایجاد یک ایدهٔ جدید این‌طور است:

```ts
// src/features/ideas/api/create-idea.ts

import {
  mutationOptions,
  useMutation,
  type UseMutationOptions,
} from '@tanstack/react-query';

import { api } from '@/lib/api';
import type {
  CreateIdeaData,
  CreateIdeaResponse,
} from '@/types/generated/types.gen';
import {
  zCreateIdeaData,
  zCreateIdeaResponse,
} from '@/types/generated/zod.gen';

// Fetcher function for making API call to create an idea
export async function createIdea(
  data: CreateIdeaData['body'],
): Promise<CreateIdeaResponse> {
  // Validate input data using the Zod schema
  // In this example, we need to validate the body of the request
  // If it is not provided, the Zod schema will throw a validation error.
  const validatedData = zCreateIdeaData.parse({ body: data });

  // Make the API request
  const response = await api.post<CreateIdeaResponse>(
    '/ideas',
    validatedData.body,
  );

  // Validate response data using the Zod schema
  // In this example, we need to validate the response data
  return zCreateIdeaResponse.parse(response);
}

// Mutation options factory for the mutation
export function getCreateIdeaMutationOptions() {
  return mutationOptions({
    mutationFn: createIdea,
  });
}

// Custom hook for creating an idea that returns the mutation
export function useCreateIdeaMutation({
  options,
}: {
  options?: Omit<
    UseMutationOptions<CreateIdeaResponse, Error, CreateIdeaData['body']>,
    'mutationFn'
  >;
}) {
  return useMutation({
    ...getCreateIdeaMutationOptions(), // Using the mutation options factory to create the mutation
    ...options, // Allowing to override the mutation options
  });
}
```

الگوی mutation همان ساختار query را دنبال می‌کند: تابع fetcher، mutation options factory و custom hook. تفاوت کلیدی این است که mutation برای عملیاتی است که داده را تغییر می‌دهند مانند ایجاد، به‌روزرسانی یا حذف resource. برخلاف query، mutation به‌صورت خودکار اجرا نمی‌شوند. آن‌ها را با فراخوانی متد `mutate` trigger می‌کنیم که در ادامه خواهیم دید.

حالا که لایهٔ API تعریف شده، می‌توانیم آن را در اپلیکیشن یکپارچه کنیم.

## یکپارچه‌سازی با اپلیکیشن {#h1_180}

همهٔ بخش‌ها را ساخته‌ایم: API client، typeهای تولیدشده، پیکربندی React Query و لایهٔ API. حالا وقتش است همه را کنار هم بگذاریم و ببینیم این بخش‌ها در سناریوهای مختلف رندر چگونه کار می‌کنند.

### Query {#h2_181}

می‌توانیم لایهٔ API را در سمت کلاینت یا سمت سرور استفاده کنیم. ببینیم چطور.

#### استفاده از سمت کلاینت {#h3_182}

ساده‌ترین راه استفاده از React Query در سمت کلاینت است. custom hook را در component فراخوانی می‌کنیم و React Query داده را دریافت کرده، cache را مدیریت کرده و state loading و error را فراهم می‌کند. نحوهٔ دریافت ایده‌های کاربر فعلی این‌طور است:

```tsx
// src/app/routes/dashboard/ideas/ideas.tsx

import { useCurrentUserIdeasQuery } from '@/features/ideas/api/get-current-user-ideas';
import { IdeasList } from '@/features/ideas/components/ideas-list';


export default function MyIdeasPage() {
  const ideasQuery = useCurrentUserIdeasQuery(); // Use the custom hook to use the query
  const ideas = ideasQuery.data?.data;

  return (
    <div className="container mx-auto px-4 py-8">
      <div className="max-w-4xl mx-auto space-y-6">
        {/* More UI code here */}
        <IdeasList
          ideas={ideas}
          isLoading={ideasQuery.isLoading} // Show loading state while the query is loading
          emptyMessage="You haven't created any ideas yet."
          error={ideasQuery.error} // Show error state if the query fails
        />
      </div>
    </div>
  );
}
```

hook آبجکت query را برمی‌گرداند که propertyهای `data`، `isLoading` و `error` را شامل می‌شود. React Query همهٔ پیچیدگی را handle می‌کند، داده را دریافت می‌کند، loading state را مدیریت می‌کند، نتیجه را cache می‌کند و خطاها را handle می‌کند. فقط باید داده را در component نمایش دهیم.

این برای data fetching سمت کلاینت عالی کار می‌کند. اما اگر نیاز داشته باشیم داده را روی سرور دریافت کرده و برای performance و SEO بهتر به کلاینت بفرستیم چطور؟

#### استفاده از سمت سرور {#h3_183}

برای رندر سمت سرور، داده را در تابع loader دریافت کرده و به query به‌عنوان initial data پاس می‌دهیم. این مزایای SSR را به ما می‌دهد در حالی که همچنان از React Query استفاده می‌کنیم. نحوهٔ دریافت یک ایده و نقدهای آن روی سرور این‌طور است:

```tsx
// src/app/routes/ideas/idea.tsx

import {
  getReviewsByIdea,
  useReviewsByIdeaQuery,
} from '@/features/reviews/api/get-reviews-by-idea';


import type { Route } from './+types/idea';

// We are using the loader function to fetch the data on the server side
export async function loader({ params }: Route.LoaderArgs) {
  const [idea, reviews] = await Promise.all([
    getIdeaById(params.id),
    getReviewsByIdea(params.id),
  ]);
  return routerData({
    idea,
    reviews,
    error: null,
    meta: {
      title: idea.title,
      description: idea.shortDescription,
    },
  });
}

export default function IdeaDetailPage({
  params,
  loaderData, // We are receiving the data from the loader function
}: Route.ComponentProps) {
  const ideaId = params.id as string;
  const user = useUser();

  const ideaQuery = useIdeaByIdQuery({
    id: ideaId,
    options: {
       // Use the data from the loader function as initial data
      ...(loaderData?.idea && { initialData: loaderData.idea }),
    },
  });
  const idea = ideaQuery?.data;

  const reviewsQuery = useReviewsByIdeaQuery({
    id: ideaId,
    options: {
       // Use the data from the loader function as initial data
      ...(loaderData?.reviews && { initialData: loaderData.reviews }),
    },
  });

  // more code here...
}

export function ErrorBoundary({ error }: { error: Error }) {
  // ...
}
```

در این رویکرد، داده را در تابع loader دریافت کرده و به‌عنوان `initialData` به query پاس می‌دهیم. این رندر سمت سرور را در حالی فراهم می‌کند که همچنان از React Query برای تمام مزایای آن استفاده می‌کنیم. query از initial data فوراً استفاده می‌کند، سپس React Query هر refetch یا update را بر عهده می‌گیرد. اگر هر یک از queryها با خطا مواجه شوند، می‌توانیم خطاها را در component `ErrorBoundary` handle کنیم.

این وقتی که query مستقیماً در component صفحه استفاده می‌شود خوب کار می‌کند. اما اگر componentی عمیق در درخت به داده نیاز داشته باشد چطور؟ می‌توانیم آن را از طریق props پایین بفرستیم، اما این منجر به prop drilling می‌شود. راه بهتری وجود دارد.

#### استفاده از سمت سرور از طریق HydrationBoundary {#h3_184}

React Query کامپوننت `HydrationBoundary` را فراهم می‌کند که مشکل prop drilling را حل می‌کند. به‌جای پاس دادن داده به‌عنوان props، queryها را روی سرور prefetch می‌کنیم، cache را dehydrate کرده و روی کلاینت rehydrate می‌کنیم. هر component سپس می‌تواند از queryها استفاده کرده و به دادهٔ prefetch شده دسترسی داشته باشد.

نحوهٔ کار این‌طور است:

```tsx
// src/app/routes/ideas/ideas.tsx

import { dehydrate, HydrationBoundary } from '@tanstack/react-query';

import {
  getIdeasQueryOptions,
  useIdeasQuery,
} from '@/features/ideas/api/get-ideas';
import { createQueryClient } from '@/lib/react-query';

import type { Route } from './+types/ideas';

// We are using the loader function to prefetch the query on the server side 
// and return the dehydrated state to the component
export async function loader() {
  const queryClient = createQueryClient();

  // But we are not fetching the data directly.
  // Instead, we are prefetching the query using the query options factory.
  // This will populate the cache with the data from the server side.
  await queryClient.prefetchQuery(getIdeasQueryOptions());

  return routerData({
    meta: {
      title: 'AIdeas - Discover AI Ideas',
      description:
        'AIdeas - Browse and explore innovative AI application ideas from the community',
    },
    // Dehydrate the query client state which will be used to hydrate the component on the client side
    dehydratedState: dehydrate(queryClient), 
  });
}

// Component that uses the hydrated data
export default function IdeasPage({ loaderData }: Route.ComponentProps) {
  return (
    // Wrap the component in the HydrationBoundary component to hydrate the component on the client side
    <HydrationBoundary state={loaderData.dehydratedState}>
      <Ideas />
    </HydrationBoundary>
  );
}


// This component could be rendered anywhere in the application
// Queries used here will always have access to the initial data 
// from the server side because of dehydrated state
function Ideas() {
  const ideasQuery = useIdeasQuery({});

  const allIdeas = ideasQuery.data?.data || [];

  // more code here...
}
export function ErrorBoundary({ error }: { error: Error }) {
  // ...
}
```

نحوهٔ کار این‌طور است:

1. تابع loader query را روی سرور با استفاده از `queryClient.prefetchQuery()` prefetch می‌کند. این cache React Query را با داده از سرور پر می‌کند.
2. state query client را با استفاده از تابع `dehydrate` dehydrate می‌کنیم. این cache را به فرمتی serialize می‌کند که می‌تواند به کلاینت فرستاده شود.
3. کامپوننت صفحه محتوای خود را در کامپوننت `HydrationBoundary` wrap کرده و state dehydrated شده را پاس می‌دهد.
4. روی کلاینت، `HydrationBoundary` cache را با دادهٔ سرور rehydrate می‌کند. هر component که از همان query استفاده کند فوراً به آن داده دسترسی خواهد داشت.

این رویکرد قدرتمند است چون componentها در هر جای درخت می‌توانند بدون prop drilling از query استفاده کنند. کامپوننت `Ideas` نیازی ندارد داده را به‌عنوان props دریافت کند — فقط از query hook استفاده می‌کند و دادهٔ prefetch شده توسط سرور را از cache دریافت می‌کند.

حالا ببینیم mutation در اپلیکیشن ما چگونه کار می‌کند.

### Mutation {#h2_185}

حالا ببینیم چگونه می‌توانیم از hook `useCreateIdeaMutation` برای ایجاد یک ایدهٔ جدید استفاده کنیم:

```tsx
// src/app/routes/dashboard/ideas/new.tsx

import { useQueryClient } from '@tanstack/react-query';
import { useNavigate } from 'react-router';

import { ErrorMessage } from '@/components/error-message';
import { useCreateIdeaMutation } from '@/features/ideas/api/create-idea';
import { IdeaForm } from '@/features/ideas/components/idea-form';
import { ideasQueryKeys } from '@/features/ideas/config/query-keys';

export default function NewIdeaPage() {
  const navigate = useNavigate();
  const queryClient = useQueryClient();

  // We are using the mutation hook to get the mutation object:
  const createIdeaMutation = useCreateIdeaMutation({
    options: {
      onSuccess: () => {
        // We are using the query keys to invalidate the queries and refresh the data
        queryClient.invalidateQueries({ queryKey: ideasQueryKeys.lists() });
        queryClient.invalidateQueries({ queryKey: ideasQueryKeys.current() });
        navigate('/dashboard/ideas');
      },
    },
  });

  return (
    <div>
      <div className="space-y-6">
        <div>
          <h1 className="text-3xl font-bold mb-2">Create New Idea</h1>
          <p className="text-muted-foreground">
            Share your AI idea with the community
          </p>
        </div>

        {/* We are displaying the error state if the mutation fails */}
        {createIdeaMutation.error && (
          <ErrorMessage error={createIdeaMutation.error} />
        )}
        <IdeaForm
          // We are using mutate method to trigger the mutation
          onSubmit={createIdeaMutation.mutate} 
          onCancel={() => navigate('/dashboard/ideas')}
          // We are showing the loading state if the mutation is pending
          isSubmitting={createIdeaMutation.isPending} 
        />
      </div>
    </div>
  );
}
```

hook mutation متد `mutate` را فراهم می‌کند که آن را به `onSubmit` فرم پاس می‌دهیم. وقتی فرم ارسال می‌شود، mutation اجرا می‌شود. از `isPending` برای نمایش loading state و از `error` برای نمایش خطاها استفاده می‌کنیم. وقتی mutation موفق شد، queryهای مرتبط را invalidate می‌کنیم تا با دادهٔ به‌روزرسانی‌شده دوباره دریافت شوند و سپس به لیست ایده‌ها ناوبری می‌کنیم.

این الگو logic mutation را تمیز و جدا از UI نگه می‌دارد. فرم فقط با داده `mutate` را فراخوانی می‌کند و React Query بقیه را handle می‌کند.

## خلاصه {#h1_186}

در این فصل، یک لایهٔ API کامل برای اپلیکیشن خود ساختیم.

با ایجاد API client شروع کردیم که `fetch` API را با error handling و پیکربندی یکپارچه wrap می‌کرد. این یک مکان واحد برای مدیریت نحوهٔ ارتباط با backend فراهم کرد.

سپس تولید type را از مشخصات OpenAPI راه‌اندازی کردیم. حالا می‌توانیم دستوری اجرا کنیم تا TypeScript type و validation schema Zod مطابق با backend تولید شوند. این تغییرات breaking را در compile time می‌گیرد و validation در زمان اجرا فراهم می‌کند.

سپس React Query را برای مدیریت server state پیکربندی کردیم. React Query caching، loading state، error handling و request deduplication را به‌صورت خودکار handle می‌کند، پس نیازی به مدیریت دستی این‌ها نداریم.

با آماده شدن بنیان، لایهٔ API خود را با الگوی یکپارچه ساختیم: تابع fetcher، options factory و custom hook. هر تابع API ورودی و خروجی را با schemaهای Zod برای type safety سرتاسری اعتبارسنجی می‌کند.

در نهایت، سه راه یکپارچه‌سازی query را بررسی کردیم: دریافت سمت کلاینت با hook، دریافت سمت سرور با initial data و prefetch سمت سرور با hydration boundary. هر رویکرد برای use caseهای مختلف مناسب است و همه با React Query به‌صورت یکپارچه کار می‌کنند.

لایهٔ API ما تضمین می‌کند داده type-safe باشد، به‌صورت کارآمد cache شود و همیشه به‌روز باشد، چه روی سرور رندر کنیم چه روی کلایント.
