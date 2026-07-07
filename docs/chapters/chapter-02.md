
# فصل ۲: راه‌اندازی و بررسی ساختار پروژه

قبل از نوشتن هر کدی برای featureها، بهتر است پروژه را از ابتدا به‌درستی راه‌اندازی کنیم. انتخاب ابزارهای نادرست و ساختار نامنظم پوشه‌ها از رایج‌ترین دلایلی است که codebaseها در طول زمان به سختی قابل کار می‌شوند. در این فصل، ابزارها و ساختاری را راه‌اندازی می‌کنیم که از این مشکلات جلوگیری می‌کند: یک meta framework، بررسی نوع (type checking)، لینتینگ (linting)، قالب‌بندی (formatting)، بررسی‌های pre-commit و ساختار پروژه مبتنی بر feature. در پایان، بنیان محکمی خواهیم داشت که با اطمینان روی آن بسازیم.

موارد زیر را پوشش خواهیم داد:

- انتخاب meta framework برای پروژه
- مروری بر راه‌اندازی build tool
- مروری بر راه‌اندازی type checking
- مروری بر راه‌اندازی linting
- مروری بر راه‌اندازی formatting
- مروری بر راه‌اندازی بررسی‌های pre-commit
- مروری بر ساختار پروژه
- مروری بر راه‌اندازی environment variables

در پایان این فصل، درک خوبی از ابزارهایی که برای راه‌اندازی پروژه استفاده می‌کنیم و ساختار پروژه مبتنی بر feature برای سازمان‌دهی بهتر کد خواهید داشت.

## پیش‌نیازهای فنی {#h1_48}

پیش از شروع، باید پروژه را راه‌اندازی کنیم. برای توسعهٔ پروژه به ابزارهای زیر روی کامپیوتر خود نیاز داریم:

- [**Node.js**](https://nodejs.org/) نسخهٔ ۲۴ یا بالاتر. نسخهٔ [**npm**](https://www.npmjs.com/) ۱۱ یا بالاتر همراه Node ارائه می‌شود. می‌توانیم با اجرای `node -v` و `npm -v` در ترمینال این را تأیید کنیم. راه‌های مختلفی برای نصب Node.js و npm وجود دارد. این مقالهٔ مفید را ببینید: [https://www.nodejsdesignpatterns.com/blog/5-ways-to-install-node-js](https://www.nodejsdesignpatterns.com/blog/5-ways-to-install-node-js).
- **VS Code** (اختیاری): ویرایشگری محبوب برای [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) و [TypeScript](https://www.typescriptlang.org/). متن‌باز است، پشتیبانی قوی از TypeScript دارد و افزونه‌های زیادی ارائه می‌دهد. از [https://code.visualstudio.com](https://code.visualstudio.com) قابل دانلود است.

کد این کتاب در مخزن (repository) آن موجود است. برای دسترسی به لینک مخزن، مراحل بخش «دانلود فایل‌های کد نمونه» در پیش‌گفتار را دنبال کنید. آن را clone کنید و به ریشهٔ مخزن بروید:

```bash
git clone https://github.com/PacktPublishing/React-Application-Architecture-for-Production-Second-Edition.git
```

مخزن شامل پوشه‌های فصل با کد هر فصل است، همراه با پوشهٔ مشترک `api` که سرور API مورد استفاده در تمام فصل‌ها را در بر می‌گیرد.

ما روی فصل ۲ کار می‌کنیم، پس وارد پوشهٔ `chapter-02` شوید:

```bash
cd React-Application-Architecture-for-Production-Second-Edition/chapter-02
```

سپس dependencyها را نصب کنید:

```bash
npm install
```

همچنین باید متغیرهای محیطی (environment variables) را اضافه کنیم:

```bash
cp .env.example .env
```

در این مرحله، فرانت‌اند باید آماده باشد و روی [http://localhost:5173](http://localhost:5173) اجرا شود.

اکنون کد پروژه آماده است.

برای اطلاعات بیشتر دربارهٔ جزئیات راه‌اندازی، فایل `README.md` را ببینید.

## انتخاب meta framework برای پروژه {#h1_49}

پیش از ورود به راه‌اندازی meta framework، باید بدانیم meta framework چیست و چرا به آن نیاز داریم. در این بخش موارد زیر را پوشش خواهیم داد:

- meta framework چیست و چرا به آن نیاز داریم؟
- چشم‌انداز meta frameworkهای React
- انتخاب درست
- چرا از [React Router](https://reactrouter.com/) استفاده می‌کنیم
- چگونه کار با React Router را شروع کنیم

### meta framework چیست و چرا به آن نیاز داریم؟ {#h2_50}

وقتی شروع به ساخت اپلیکیشن‌های وب با کتابخانه‌هایی مانند [React](https://react.dev/) می‌کنیم، خیلی زود درمی‌یابیم که این ابزارها فقط بخشی از مسئله را حل می‌کنند. React به‌عنوان یک کتابخانهٔ UI، مدیریت componentها و state پایه را به‌خوبی انجام می‌دهد، اما routing چطور؟ **رندر سمت سرور (SSR)** چطور؟ بهینه‌سازی build؟ یکپارچه‌سازی API؟ ناگهان زمان بیشتری صرف پیکربندی ابزارها می‌کنیم تا ساخت قابلیت‌ها.

اینجاست که meta frameworkها وارد میدان می‌شوند. آن‌ها لایه‌ای هستند که روی کتابخانهٔ UI یا framework پایه قرار می‌گیرند و تمام زیرساخت‌های سطح اپلیکیشن را که برای پروژه نیاز داریم فراهم می‌کنند.

### مشکلاتی که حل می‌کنند {#h2_51}

ساخت یک اپلیکیشن وب آمادهٔ production تصمیم‌ها و پیکربندی‌های زیادی دارد که هیچ ارتباطی با business logic واقعی ما ندارد. باید بفهمیم چگونه انواع مختلف رندر (سمت کلاینت، سمت سرور و ایستا) را مدیریت کنیم، routing با code splitting راه‌اندازی کنیم، bundleها را بهینه کنیم، TypeScript را پیکربندی کنیم، linting را راه‌اندازی کنیم، data fetching را انجام دهیم و همه‌چیز را به‌طور قابل اعتماد deploy کنیم.

اگر بخواهیم تمام این‌ها را از صفر راه‌اندازی کنیم، روزها (یا هفته‌ها) فقط صرف پیکربندی ابزارها می‌شود و هنوز یک خط کد محصول هم ننوشته‌ایم. حتی در آن صورت هم احتمالاً setup‌ای خواهیم داشت که نگهداری و به‌روزرسانی آن دشوار است و پر از مشکلات پنهانی است که فقط در production ظاهر می‌شوند.

meta frameworkها این مشکلات را مستقیماً حل می‌کنند. Routing، حالت‌های رندر (rendering modes) (**رندر سمت کلاینت (CSR)**، **SSR** و رندر ایستا)، code splitting و الگوهای data fetching همگی به‌صورت پیش‌فرض پشتیبانی می‌شوند. به‌جای اینکه خودمان این قطعات را به هم متصل کنیم، سیستمی دریافت می‌کنیم که از همان روز اول کار می‌کند و به ما آزادی می‌دهد تا روی آنچه واقعاً اهمیت دارد تمرکز کنیم: ساخت قابلیت‌ها و حل مسائل برای کاربرانمان.

### چشم‌انداز meta frameworkهای React {#h2_52}

از آنجا که این کتاب دربارهٔ React است، بیایید روی گزینه‌های اصلی موجود در اکوسیستم React تمرکز کنیم. این چشم‌انداز در چند سال اخیر به‌طور قابل ملاحظه‌ای بالغ شده و اکنون بسته به نیاز پروژه، چند انتخاب خوب در اختیار داریم.

### [Next.js](https://nextjs.org/) {#h3_53}

**Next.js** محبوب‌ترین انتخاب در دنیای React است. توسط [Vercel](https://vercel.com/) پشتیبانی می‌شود و بزرگ‌ترین جامعهٔ کاربری را دارد؛ یعنی مستندات عالی، آموزش‌های فراوان و استخدام آسان‌تر. امکانات زیادی به‌صورت built-in دارد: بهینه‌سازی خودکار تصاویر، React Server Components، پیش‌رندر جزئی و موارد دیگر.

### React Router (در حالت framework) {#h3_54}

**React Router** مدت‌هاست به‌عنوان کتابخانهٔ روتینگ سمت کلاینت وجود داشته، اما حالت framework نیز دارد که جانشین Remix است. در حالت framework، به‌عنوان یک meta framework کامل با SSR و بارگذاری داده (data loading) built-in عمل می‌کند. روی استانداردهای وب و بهبود تدریجی (progressive enhancement) تمرکز دارد.

### [TanStack Start](https://tanstack.com/start) {#h3_55}

**TanStack Start** جدیدترین framework در این فضاست. روی type safety و تجربهٔ توسعه‌دهنده (developer experience) تأکید دارد و رویکرد type-first را دنبال می‌کند؛ به‌طوری که پارامترهای مسیر (route parameters)، بارگذاری داده و ناوبری (navigation) از همان ابتدا به‌طور کامل typed هستند.

در حالی که هنوز در حال تکامل است، TanStack Start برای تیم‌هایی که یکپارچه‌سازی قوی TypeScript را در اولویت قرار می‌دهند، نویدبخش به نظر می‌رسد.

### انتخاب درست {#h2_56}

هر سه framework گزینه‌های تولیدی خوب و آمادهٔ production با جامعهٔ کاربری قوی هستند. انتخاب درست برای یک پروژه به آشنایی تیم، نیازمندی‌های hosting و میزان abstractionی که راحت باشد بستگی دارد. برای این کتاب، از React Router در حالت framework استفاده می‌کنیم.

### چرا از React Router استفاده می‌کنیم {#h2_57}

React Router را به دو دلیل اصلی انتخاب کردیم:

- **انعطاف‌پذیری در استقرار (deployment)**: React Router در هر محیط hosting به‌طور یکسان کار می‌کند و ترجیحی به هیچ پلتفرم خاصی ندارد؛ این آزادی بیشتری در نحوهٔ استقرار اپلیکیشن به ما می‌دهد.
- **تمرکز روی یادگیری**: مدل بارگذاری دادهٔ React Router نزدیک به نحوهٔ عملکرد بومی پلتفرم وب است. این ویژگی آن را برای کتابی که روی فهم نحوهٔ عملکرد اپلیکیشن‌های React تمرکز دارد، مناسب می‌کند.

هر framework که انتخاب کنیم، مفاهیمی که در این کتاب پوشش می‌دهیم در همهٔ آن‌ها کاربرد دارند.

### چگونه کار با React Router را شروع کنیم {#h2_58}

برای شروع یک پروژهٔ جدید می‌توانیم از CLI `create-react-router` استفاده کنیم و پروژهٔ جدیدی با دستور زیر بسازیم:

```bash
npx create-react-router@latest my-app
```

این دستور لیستی از گزینه‌ها را برای انتخاب نمایش می‌دهد. می‌توانیم گزینه‌های پیش‌فرض را انتخاب کنیم تا پروژه تولید شود.

ساختار اولیهٔ پروژهٔ ما به این صورت است:

```
├── src/
│   └── app/
│       ├── app.css
│       ├── root.tsx
│       ├── routes/
│       │   └── home.tsx
│       ├── routes.ts
│       └── welcome/
│           ├── logo-dark.svg
│           ├── logo-light.svg
│           └── welcome.tsx
├── Dockerfile
├── node_modules/
├── package-lock.json
├── package.json
├── public/
│   └── favicon.ico
├── react-router.config.ts
├── README.md
├── tsconfig.json
└── vite.config.ts
```

ساختار پروژه به این شرح است:

- `src/app`: دایرکتوری اپلیکیشن
- `src/app/root.tsx`: ریشهٔ اپلیکیشن با ساختار HTML و مدیریت خطا
- `src/app/routes.ts`: فایل پیکربندی مسیر که URLها را به componentها نگاشت می‌کند
- `react-router.config.ts`: فایل پیکربندی React Router
- `tsconfig.json`: فایل پیکربندی TypeScript
- `vite.config.ts`: فایل پیکربندی [Vite](https://vitejs.dev/)

نکته‌ای که اگر CLI را اجرا کنیم متوجه می‌شویم این است که پوشهٔ `app` مستقیماً در ریشهٔ پروژه ایجاد می‌شود، نه در دایرکتوری `src`. این به این دلیل است که CLI پروژه را در دایرکتوری فعلی تولید می‌کند. با این حال، ما آن را در دایرکتوری `src` نگه می‌داریم زیرا می‌خواهیم هر چیزی که داخل دایرکتوری `src` است را با alias `@/` مرجع دهیم؛ این موضوع را در بخش‌های بعدی پوشش خواهیم داد.

این فایل‌ها را در طول کتاب پوشش می‌دهیم، پس فعلاً نگران آن‌ها نباشید.

اکنون که meta framework را راه‌اندازی کردیم، بیایید build tool‌ای که آن را تقویت می‌کند و نحوهٔ پیکربندی آن بر اساس نیازهایمان را بشناسیم.

## مروری بر راه‌اندازی build tool {#h1_59}

React Router در حالت framework از **Vite** به‌عنوان build tool استفاده می‌کند که توسعهٔ سریع و buildهای بهینهٔ production را فراهم می‌کند. در این بخش موارد زیر را پوشش خواهیم داد:

- Vite چیست و چرا به آن نیاز داریم؟
- Vite چگونه با React Router کار می‌کند
- پیکربندی Vite ما
- buildهای development در مقابل production

### Vite چیست و چرا به آن نیاز داریم؟ {#h2_60}

Vite یک build tool مدرن است که موارد زیر را فراهم می‌کند:

- **توسعهٔ سریع**: **جایگزینی ماژول داغ (HMR)** برای به‌روزرسانی‌های آنی در حین توسعه
- **buildهای بهینه**: Tree shaking، code splitting و بهینه‌سازی دارایی‌ها برای production
- **بی‌طرفی نسبت به framework**: با React، Vue، Svelte و غیره کار می‌کند
- **استانداردهای مدرن**: ES moduleهای بومی در محیط توسعه؛ bundleهای بهینه برای production

وقتی اپلیکیشن‌های React می‌سازیم، به ابزاری نیاز داریم که بتواند:

- کد ما را به‌طور کارآمد bundle کند
- انواع مختلف فایل (TypeScript، JSX، [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS) و تصاویر) را مدیریت کند
- تجربهٔ توسعهٔ سریعی فراهم کند
- برای استقرار production بهینه شود

Vite این مشکلات را با رویکردی مدرن حل می‌کند که سریع‌تر از bundlerهای سنتی مانند [webpack](https://webpack.js.org/) است.

### Vite چگونه با React Router کار می‌کند {#h2_61}

React Router روی Vite ساخته شده است، که به این معناست:

- **سرور توسعه** روی سرور توسعهٔ Vite با HMR اجرا می‌شود
- **فرایند build** از بهینه‌سازی‌های Vite برای production استفاده می‌کند
- **متغیرهای محیطی** توسط سیستم Vite مدیریت می‌شوند

این یکپارچه‌سازی بهترین‌های هر دو دنیا را به ما می‌دهد: قابلیت‌های روتینگ و بارگذاری دادهٔ React Router با سیستم build سریع Vite.

### پیکربندی Vite ما {#h2_62}

از Vite برای مدیریت فرایند build استفاده می‌کنیم و پیکربندی ما در فایل `vite.config.ts` تنظیم شده است:

```ts
// vite.config.ts
import { reactRouter } from "@react-router/dev/vite";
import tailwindcss from "@tailwindcss/vite";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [tailwindcss(), reactRouter(), tsconfigPaths()],
});
```

تنظیمات اینجا به شرح زیر است:

- پلاگین `tailwindcss()`: پلاگین [Tailwind CSS](https://tailwindcss.com/) که لایهٔ CSS اپلیکیشن را مدیریت می‌کند.
- پلاگین `reactRouter()`: پلاگین React Router که روتینگ، SSR و ویژگی‌های مخصوص framework را مدیریت می‌کند.
- پلاگین `tsconfigPaths()`: این پلاگین به ما اجازه می‌دهد از `@/` به‌عنوان alias برای دایرکتوری `src` در فایل‌های TypeScript استفاده کنیم، تا بتوانیم به‌جای `../../../components/button` بنویسیم `@/components/button`. فعلاً نگران TypeScript نباشید؛ آن را در بخش بعدی پوشش خواهیم داد.

## مروری بر راه‌اندازی type checking {#h1_63}

اکنون که build tool را شناختیم، باید TypeScript را برای type checking راه‌اندازی کنیم. در این بخش موارد زیر را پوشش خواهیم داد:

- TypeScript چیست و چرا به آن نیاز داریم؟
- TypeScript چگونه کار می‌کند
- پیکربندی TypeScript ما
- اجرای بررسی‌های TypeScript

TypeScript ابزار بسیار ارزشمندی در توسعهٔ مدرن React است. بیایید ببینیم چرا و چگونه آن را بر اساس نیازهایمان پیکربندی کنیم.

### TypeScript چیست و چرا به آن نیاز داریم؟ {#h2_64}

TypeScript زبان برنامه‌نویسی است که JavaScript را با اضافه کردن تعریف نوع ایستا (static type) گسترش می‌دهد. آن را مثل JavaScript با یک شبکهٔ ایمنی در نظر بگیرید — به ما کمک می‌کند خطاها را قبل از وقوع در production پیدا کنیم.

وقتی یک اپلیکیشن React یا به‌طور کلی هر اپلیکیشن JavaScript دیگری می‌سازیم، اغلب با مشکلاتی مانند موارد زیر مواجه می‌شویم:

- ارسال string در جایی که عدد انتظار داشتیم
- فراخوانی تابعی که وجود ندارد یا با آرگومان‌های اشتباه
- دسترسی به propertیهای object که ممکن است undefined باشند یا وجود نداشته باشند
- فراموش کردن مدیریت کردن تمام حالت‌های ممکن در کدمان

TypeScript این مشکلات را در compile time، قبل از اینکه کاربران آن‌ها را ببینند، شناسایی می‌کند.

TypeScript به‌ویژه برای اپلیکیشن‌های بزرگ ساخته‌شده توسط تیم‌های بزرگ مفید است. کد نوشته‌شده در TypeScript بسیار بهتر از کد نوشته‌شده در JavaScript خالص مستند شده است. با نگاه کردن به تعریف نوع‌ها می‌توانیم بفهمیم یک قطعه کد چگونه باید کار کند.

دلیل دیگر این است که TypeScript بازآفرینی (refactoring) را بسیار آسان‌تر می‌کند زیرا بیشتر مشکلات قبل از اجرای اپلیکیشن قابل شناسایی هستند.

TypeScript همچنین در IDE با ویژگی‌هایی مانند تکمیل خودکار (autocomplete)، اطلاعات hover و اطلاعات امضا به ما کمک می‌کند که بهره‌وری ما را افزایش می‌دهد.

### TypeScript چگونه کار می‌کند {#h2_65}

TypeScript با تحلیل کد و فهمیدن نوع داده‌هایی که با آن‌ها کار می‌کنیم عمل می‌کند. در اینجا یک مثال آورده شده است:

```ts
// تابع انتظار عدد دارد
function double(value: number) {
  return value * 2;
}

// TypeScript راضی است - عدد فرستادیم
double(21); // درست کار می‌کند، 42 برمی‌گرداند

// TypeScript خطا را می‌گیرد - string فرستادیم
double("21"); // خطای TypeScript: Argument of type 'string' is not assignable to parameter of type 'number'
```

نکتهٔ کلیدی این است که TypeScript این اشتباهات را در compile time، قبل از اینکه کاربران آن‌ها را ببینند، شناسایی می‌کند.

### پیکربندی TypeScript ما {#h2_66}

از TypeScript استفاده می‌کنیم تا خطاها را قبل از رسیدن به production بگیریم و setup ما طوری پیکربندی شده که strict اما عملی (practical) باشد. بیایید ببینیم آن را چگونه در فایل `tsconfig.json` پیکربندی کرده‌ایم:

```json
{
  "include": [
    "**/*",
    "**/.server/**/*",
    "**/.client/**/*",
    ".react-router/types/**/*"
  ],
  "compilerOptions": {
    "lib": ["DOM", "DOM.Iterable", "ES2022"],
    "types": ["node", "vite/client"],
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "rootDirs": [".", "./.react-router/types"],
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "esModuleInterop": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,
    "resolveJsonModule": true,
    "skipLibCheck": true,
    "strict": true
  }
}
```

مهم‌ترین بخش‌های پیکربندی فوق به شرح زیر است:

- `"strict": true`: تمام گزینه‌های strict type-checking را فعال می‌کند. ممکن است تهاجمی به نظر برسد، اما باگ‌های بالقوهٔ زیادی را زود شناسایی می‌کند.
- `"jsx": "react-jsx"`: به TypeScript می‌گوید چگونه componentهای React را مدیریت کند. دیگر نیازی به import کردن React در هر فایل نیست.
- `"noEmit": true`: اجازه می‌دهیم Vite کامپایل واقعی را انجام دهد، پس TypeScript فقط type checking انجام می‌دهد.
- `"paths": { "@/*": ["./src/*"] }`: به ما اجازه می‌دهد از `@/` به‌عنوان alias برای دایرکتوری `src` استفاده کنیم تا بتوانیم مسیرهای مطلق به فایل‌هایمان بنویسیم، مانند `@/components/button` به‌جای `../../../components/button`. `tsconfigPaths()` را به یاد دارید؟ به Vite اجازه می‌دهد مسیرهایی را که اینجا تعریف شده‌اند resolve کند.

ما همچنین یک اسکریپت `typecheck` در فایل `package.json` خود داریم که هم تولید نوع React Router و هم کامپایل TypeScript را اجرا می‌کند:

```json
"typecheck": "react-router typegen && tsc"
```

این تضمین می‌کند که نوع‌های ما قبل از اجرای type checker همیشه با مسیرهای ما به‌روز باشند.

### تولید نوع React Router {#h2_67}

دستور `react-router typegen` به React Router اجازه می‌دهد نوع‌های TypeScript را برای مسیرهای ما بر اساس ساختار فایل و exportها در فایل‌های مسیر تولید کند. این موارد زیر را به ما می‌دهد:

- **پارامترهای مسیر type-safe**: autocomplete و type checking برای پارامترهای مسیر دریافت می‌کنیم
- **Loader و action type-safe**: توابع بارگذاری دادهٔ ما کاملاً typed هستند
- **ناوبری (navigation) type-safe**: هنگام ناوبری بین مسیرها، type safety برای params و search parameterها دریافت می‌کنیم

نوع‌های تولیدشده در دایرکتوری `.react-router/types` (که در `tsconfig.json` ما included است) قرار می‌گیرند و هر بار که دستور `typecheck` را اجرا می‌کنیم دوباره تولید می‌شوند. این یعنی نوع‌های مسیر ما همیشه با فایل‌های مسیر واقعی ما هماهنگ هستند.

### اجرای بررسی‌های TypeScript {#h2_68}

می‌توانیم type checking را با اسکریپت‌های پیکربندی‌شده اجرا کنیم:

```bash
# اجرای type checking یک بار (شامل تولید نوع React Router)
npm run typecheck

# اجرای type checking در حالت watch (با تغییر فایل‌ها دوباره اجرا می‌شود)
npx tsc --watch

# اجرای type checking برای فایل‌های خاص
npx tsc --noEmit src/features/ideas/api/create-idea.ts
```

TypeScript خطای نوع را می‌گیرد، اما مشکلات دیگری هم می‌توانند رخ دهند. اینجاست که linting وارد میدان می‌شود.

## مروری بر راه‌اندازی linting {#h1_69}

سیستم نوع TypeScript برای شناسایی خطاها در compile time بسیار قدرتمند است، اما استانداردهای کدنویسی را اعمال نمی‌کند یا تمام مشکلات بالقوه را شناسایی نمی‌کند. به‌عنوان مثال، TypeScript اگر متغیرهایی اعلام کنیم که هرگز استفاده نمی‌کنیم، توابع بیش‌ازحد پیچیده بنویسیم یا قالب‌بندی ناهمگون داشته باشیم، خطا نمی‌گیرد. برای حفظ codebase تمیز، یکدست و باکیفیت در سراسر تیم به linter نیاز داریم. در این بخش موارد زیر را پوشش خواهیم داد:

- linting چیست و چرا به آن نیاز داریم؟
- مروری بر [ESLint](https://eslint.org/)
- پیکربندی ESLint ما
- اجرای بررسی‌های ESLint

## linting چیست و چرا به آن نیاز داریم؟ {#h2_70}

linting فرایند تحلیل کد منبع برای یافتن خطاها، باگها، خطاهای استایلی و ساختارهای مشکوک است. آن را مثل یک code review خودکار در نظر بگیرید که هر بار که کد می‌نویسیم اجرا می‌شود.

وقتی در تیم کار می‌کنیم، هر کسی style و اولویت‌های کدنویسی متفاوتی دارد. بعضی‌ها quote تکی ترجیح می‌دهند و بعضی‌ها quote دوتایی. بعضی‌ها فراموش می‌کنند edge caseها را مدیریت کنند یا کدی می‌نویسند که خواندنش دشوار است.

linter به ما کمک می‌کند با:

- **شناسایی زودهنگام باگها**: یافتن مشکلات بالقوه قبل از اینکه مشکل‌ساز شوند
- **اعمال style یکدست**: مطمئن شدن اینکه تمام کد ما یک‌سان به نظر برسد
- **آموزش بهترین شیوه‌ها (best practices)**: نشان دادن راه‌های بهتر کدنویسی
- **جلوگیری از اشتباهات رایج**: شناسایی چیزهایی که ممکن است از قلم بیفتند

## مروری بر ESLint {#h2_71}

ESLint محبوب‌ترین linter برای JavaScript/TypeScript است. با parse کردن کد و سپس اجرای ruleها روی آن برای یافتن مشکلات کار می‌کند.

به‌عنوان مثال، ممکن است کد زیر را بنویسیم:

```js
const unusedVariable = "hello";
console.log("This is fine");
```

ESLint می‌تواند تشخیص دهد که `unusedVariable` declare شده اما هرگز استفاده نشده و دربارهٔ آن به ما هشدار دهد.

## پیکربندی ESLint ما {#h2_72}

پیکربندی ESLint ما کاملاً جامع است و به ما کمک می‌کند کیفیت کد را در سراسر تیم حفظ کنیم.

بیایید آنچه را در `eslint.config.js` راه‌اندازی کرده‌ایم بررسی کنیم:

```js
import js from '@eslint/js';
import typescript from '@typescript-eslint/eslint-plugin';
import typescriptParser from '@typescript-eslint/parser';
import { defineConfig } from 'eslint/config';
import prettierConfig from 'eslint-config-prettier';
import checkFile from 'eslint-plugin-check-file';
import importPlugin from 'eslint-plugin-import';
import prettier from 'eslint-plugin-prettier';
import react from 'eslint-plugin-react';
import reactHooks from 'eslint-plugin-react-hooks';
import globals from 'globals';

... بقیهٔ پیکربندی
```

از فرمت flat config (سبک جدید پیکربندی ESLint) استفاده می‌کنیم و چند پلاگین کلیدی ارزش ذکر دارند:

- `@typescript-eslint`: ruleهای linting مخصوص TypeScript
- `eslint-plugin-react`: ruleهای مخصوص React
- `eslint-plugin-react-hooks`: ruleها برای hookهای React
- `eslint-plugin-prettier`: [Prettier](https://prettier.io/) را با ESLint یکپارچه می‌کند که لحظه‌ای پوشش خواهیم داد
- `eslint-plugin-import`: linting import/export
- `eslint-plugin-check-file`: قرارداد نام‌گذاری فایل (naming convention)

چند rule مهمی که پیکربندی کرده‌ایم به شرح زیر است:

```js
rules: {
  'prettier/prettier': 'error',
  'react/react-in-jsx-scope': 'off',
  'react/prop-types': 'off',
  '@typescript-eslint/no-unused-vars': 'warn',
  '@typescript-eslint/no-explicit-any': 'warn',
  'import/order': [
    'error',
    {
      groups: ['builtin', 'external', 'internal', 'parent', 'sibling', 'index'],
      'newlines-between': 'always',
      alphabetize: { order: 'asc', caseInsensitive: true },
    },
  ],
  'import/no-cycle': ['error', { maxDepth: Infinity }],
  'check-file/filename-naming-convention': [
    'error',
    {
      '**/*.{js,jsx,ts,tsx}': 'KEBAB_CASE',
    },
  ],
}
```

هر rule چه کاری انجام می‌دهد:

- `prettier/prettier`: هر مشکل قالب‌بندی Prettier را به‌عنوان خطای ESLint در نظر می‌گیرد، بنابراین مشکلات قالب‌بندی در کنار مشکلات linting در یک مرحله ظاهر می‌شوند.
- `react/react-in-jsx-scope`: در نسخه‌های قدیمی‌تر React، مجبور بودیم React را در بالای هر فایل JSX import کنیم. React مدرن دیگر این را نیاز ندارد، بنابراین این rule را غیرفعال می‌کنیم.
- `react/prop-types`: در پروژه‌های JavaScript، `prop-types` برای اعتبارسنجی propهای component در runtime استفاده می‌شد. از آنجا که از TypeScript استفاده می‌کنیم، propهای ما قبلاً statically typed هستند، بنابراین این rule غیرضروری است.
- `@typescript-eslint/no-unused-vars`: وقتی متغیرهایی declare می‌کنیم که هرگز استفاده نمی‌شوند، به ما هشدار می‌دهد و کد را تمیز و بدون سردرگمی نگه می‌دارد.
- `@typescript-eslint/no-explicit-any`: وقتی از نوع `any` استفاده می‌کنیم به ما هشدار می‌دهد. استفاده از `any` هدف type safety TypeScript را خنثی می‌کند، بنابراین این ما را مجبور می‌کند به‌جای آن از نوع‌های مناسب استفاده کنیم.
- `import/order`: ترتیب یکدستی برای importها اعمال می‌کند: ماژول‌های built-in اول، سپس بسته‌های external، سپس importهای internal، با جدا شدن هر گروه با یک خط خالی و مرتب‌شده به‌صورت الفبایی. این importها را آسان‌تر اسکن می‌کند.
- `import/no-cycle`: از وابستگی‌های circular بین فایل‌ها جلوگیری می‌کند. importهای circular می‌توانند مشکلات سخت‌دیباگ ایجاد کنند و معمولاً نشانهٔ مشکل معماری هستند.
- `check-file/filename-naming-convention`: نام‌گذاری kebab-case را برای تمام فایل‌های JavaScript/TypeScript اعمال می‌کند (مثلاً `my-component.tsx` به‌جای `MyComponent.tsx`). این نام‌گذاری فایل‌ها را در سراسر پروژه یکدست نگه می‌دارد بدون اینکه هر بار که فایل جدیدی می‌سازیم به آن فکر کنیم.

چند rule سفارشی import هم داریم که به اعمال ساختار پروژه کمک می‌کنند؛ آن‌ها را وقتی دربارهٔ سازمان‌دهی پروژه صحبت می‌کنیم به‌طور دقیق پوشش خواهیم داد.

Ruleهایی که استفاده می‌کنیم چیزی است که باید توسط کل تیم یا سازمان تصمیم گرفته شود. پس از تصمیم، باید آن‌ها را به فایل پیکربندی ESLint اضافه کنیم و به آن‌ها پایبند بمانیم.

## اجرای ESLint {#h2_73}

می‌توانیم linting را با اسکریپت‌های پیکربندی‌شده اجرا کنیم:

```bash
# اجرای linting روی تمام فایل‌ها
npm run lint

# اجرای linting و اصلاح خودکار آنچه قابل اصلاح است
npm run lint:fix

# اجرای linting روی فایل‌ها یا دایرکتوری‌های خاص
npx eslint src/features/ideas/
npx eslint src/components/button.tsx
```

ESLint به ما خطا و هشدار نشان می‌دهد و با flag `--fix` می‌تواند مشکلات زیادی مانند قالب‌بندی، importهای unused و مشکلات سادهٔ code style را به‌طور خودکار اصلاح کند.

## مروری بر راه‌اندازی formatting {#h1_74}

اکنون که linting را پوشش دادیم، بیایید قالب‌بندی کد را ببینیم. در این بخش موارد زیر را پوشش خواهیم داد:

- قالب‌بندی کد چیست و چرا به آن نیاز داریم؟
- Prettier چگونه کار می‌کند
- پیکربندی Prettier
- اجرای بررسی‌های Prettier

## قالب‌بندی کد چیست و چرا به آن نیاز داریم؟ {#h2_75}

قالب‌بندی کد دربارهٔ یکدست و خوانا کردن کد است. وقتی در تیم کار می‌کنیم، همه اولویت‌های متفاوتی برای نحوهٔ ظاهر شدن کد داریم — بعضی‌ها سمیکالن می‌خواهند و بعضی‌ها نه؛ بعضی‌ها quote تکی ترجیح می‌دهند و بعضی‌ها quote دوتایی.

مشکل این است که این تفاوت‌ها code review را دشوارتر می‌کنند و diff و conflictهای غیرضروری ایجاد می‌کنند. وقتی روی logic کدمان تمرکز داریم، نمی‌خواهیم با ناهمگونی قالب‌بندی پرت شویم.

## Prettier چگونه کار می‌کند {#h2_76}

**Prettier** یک قالب‌بند خودکار کد است که کد ما را بر اساس مجموعه‌ای از قوانین فرمت می‌کند. عالی است زیرا نیازی نیست دربارهٔ سمیکالن بودن یا نبودن بحث کنیم؛ Prettier این کار را برای ما انجام می‌دهد.

Prettier به این صورت کار می‌کند:

1. **Parse کردن کد ما**: فهمیدن ساختار JavaScript، TypeScript، CSS و غیره
2. **اعمال قوانین قالب‌بندی**: یکدست کردن ظاهر کد
3. **خروجی دادن کد فرمت‌شده**: بازگرداندن کد تمیز و خوانا

زیبایی Prettier این است که در سراسر تیم یکسان عمل می‌کند. کد همه یک‌سان به نظر می‌رسد، صرف‌نظر از اولویت‌های شخصی‌شان.

## پیکربندی Prettier {#h2_77}

Prettier پیش‌فرض‌های نظری خودش را دارد، اما می‌توانیم آن را بر اساس نیازهایمان پیکربندی کنیم. فایل پیکربندی در `.prettierrc` است که از قبل برای ما راه‌اندازی شده:

```json
{
  "semi": true,
  "trailingComma": "all",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false
}
```

تنظیمات کلیدی اینجا به شرح زیر است:

- `"trailingComma": "all"`: trailing comma را همه‌جا درج می‌کند که diffهای [Git](https://git-scm.com/) را هنگام اضافه کردن آیتم‌های جدید به آرایه‌ها یا objectها تمیزتر می‌کند.
- `"singleQuote": true`: از quote تکی برای stringها استفاده می‌کنیم که اولویت رایج در جامعهٔ React است.
- `"printWidth": 80`: خطوط را زیر ۸۰ کاراکتر نگه می‌داریم که کد را روی صفحه‌نمایش کوچک‌تر خواناتر می‌کند.

Prettier از طریق پلاگین `eslint-plugin-prettier` با ESLint یکپارچه شده، بنابراین مشکلات قالب‌بندی به‌عنوان خطای linting ظاهر می‌شوند. این تضمین می‌کند کد ما هم به‌درستی قالب‌بندی شده و هم از قوانین linting ما پیروی می‌کند.

Prettier به‌طور خودکار کد ما را بر اساس پیکربندی‌مان فرمت می‌کند و چون با ESLint یکپارچه شده، مشکلات قالب‌بندی هم وقتی `npm run lint` را اجرا می‌کنیم ظاهر می‌شوند.

با type checking، linting و ابزارهای formatting راه‌اندازی‌شده، باید مطمئن شویم که واقعاً قبل از ورود کد به repository اجرا می‌شوند. نحوهٔ آن را در بخش بعدی خواهیم دید.

## مروری بر راه‌اندازی بررسی‌های pre-commit {#h1_78}

داشتن ابزارهای static code analysis مانند TypeScript، ESLint و Prettier عالی است؛ آن‌ها را پیکربندی کرده‌ایم و می‌توانیم هر وقت تغییری ایجاد می‌کنیم اسکریپت‌های جداگانه را اجرا کنیم تا مطمئن شویم همه‌چیز در بهترین حالت است.

اما معایبی وجود دارد. توسعه‌دهندگان ممکن است فراموش کنند تمام بررسی‌ها را قبل از commit به repo اجرا کنند، که می‌تواند کد مشکل‌دار و ناهمگون را به production برساند.

خوشبختانه راه‌حلی وجود دارد که این مشکل را حل کند: هر بار که سعی می‌کنیم به repository commit کنیم، تمام بررسی‌ها را به‌طور خودکار اجرا کنیم. اینجاست که pre-commit hook وارد میدان می‌شود.

## pre-commit hook چیست و چرا به آن نیاز داریم؟ {#h2_79}

pre-commit hook اسکریپت‌هایی هستند که به‌طور خودکار قبل از اینکه بتوانیم کدمان را به repository commit کنیم اجرا می‌شوند. آن‌ها به‌عنوان یک شبکهٔ ایمنی عمل می‌کنند و تضمین می‌کنند فقط کد باکیفیت وارد codebase ما شود.

بدون pre-commit hook، ممکن است به‌طور تصادفی موارد زیر را commit کنیم:

- کدی که کامپایل نمی‌شود
- کدی که خطای linting دارد
- کدی که به‌درستی قالب‌بندی نشده
- کدی که testها را خراب می‌کند

pre-commit hook این مشکلات را قبل از اینکه به pull request یا حتی بدتر، به production برسند، شناسایی می‌کند.

## pre-commit hook چگونه کار می‌کند {#h2_80}

Git hook اسکریپت‌هایی هستند که Git در نقاط خاصی از فرایند Git اجرا می‌کند. pre-commit hook درست قبل از ایجاد commit اجرا می‌شود و اگر با status غیرصفر خارج شود، commit مسدود می‌شود.

شکل زیر را بررسی کنید تا بفهمید pre-commit hook چگونه کار می‌کند:

![Figure 2.1 – نحوهٔ کار pre-commit hook](/images/B31385_2_1.png)

**Figure 2.1 — نحوهٔ کار pre-commit hook**

این یعنی نمی‌توانیم کد بد commit کنیم؛ اول باید مشکلات را حل کنیم. ممکن است در ابتدا آزاردهنده به نظر برسد، اما از شکستن تصادفی build یا معرفی باگ یا ناهمگونی جلوگیری می‌کند.

## workflow pre-commit ما {#h2_81}

همان‌طور که می‌بینیم، هر بار که سعی می‌کنیم به repository commit کنیم، Git pre-commit hook اجرا می‌شود و اسکریپت‌هایی را اجرا می‌کند که بررسی را انجام می‌دهند. اگر تمام بررسی‌ها pass شوند، تغییرات به repository commit می‌شوند؛ در غیر این صورت باید مشکلات را حل کنیم و دوباره امتحان کنیم.

برای فعال کردن این جریان از `husky` و `lint-staged` استفاده می‌کنیم:

- [husky](https://typicode.github.io/husky/) ابزاری است که به ما اجازه می‌دهد Git hook اجرا کنیم. می‌خواهیم pre-commit hook را اجرا کنیم تا بررسی‌ها را قبل از commit کردن تغییرات انجام دهیم.
- [lint-staged](https://github.com/lint-staged/lint-staged) ابزاری است که به ما اجازه می‌دهد آن بررسی‌ها را فقط روی فایل‌هایی که در staging area Git هستند اجرا کنیم. این سرعت بررسی کد را بهبود می‌دهد زیرا انجام آن روی کل codebase ممکن است بسیار کند باشد.

قبل از شروع، باید مطمئن شویم repository Git اگر از قبل مقداردهی اولیه نشده، مقداردهی اولیه شود. سپس می‌توانیم `husky` و `lint-staged` را با دستورات زیر نصب کنیم:

```bash
npm install --save-dev husky lint-staged
```

سپس باید `husky` را برای فعال کردن Git hook مقداردهی اولیه کنیم:

```bash
npx husky init
```

پوشهٔ جدیدی به نام `.husky` با فایلی به نام `pre-commit` ایجاد می‌شود که محتوای زیر را دارد:

```bash
npm test
```

بیایید فایل را طوری تغییر دهیم که اسکریپت `lint-staged` را اجرا کند:

```bash
npm run lint-staged
```

pre-commit hook `husky` اسکریپت `lint-staged` را اجرا می‌کند، بنابراین باید تعریف کنیم `lint-staged` چه دستوراتی باید در فایل `lint-staged.config.js` اجرا کند:

```js
// infra/lint-staged.config.js
export default {
  '*.{js,jsx,ts,tsx}': ['eslint --fix'],
  '*.{json,md,css,scss,html}': ['prettier --write'],
  '*.{ts,tsx}': ['bash -c "npm run typecheck"'],
};
```

این به این معناست:

- فایل‌های JavaScript/TypeScript linted و اصلاح می‌شوند
- فایل‌های دیگر فقط با Prettier قالب‌بندی می‌شوند
- فایل‌های TypeScript همچنین type-checked می‌شوند

زیبایی این setup این است که سریع است (فقط فایل‌های تغییریافته را بررسی می‌کند) و جامع است (مشکلات قالب‌بندی، linting و نوع را شناسایی می‌کند).

## اجرای بررسی‌های pre-commit {#h2_82}

می‌توانیم بررسی‌های pre-commit را به‌صورت دستی اجرا کنیم:

```bash
# اجرای تمام بررسی‌های pre-commit (linting، formatting، type checking)
npm run pre-commit

# فقط اجرای lint-staged (بررسی فقط فایل‌های تغییریافته)
npm run lint-staged

# اجرای بررسی‌های جداگانه
npm run lint
npm run format
npm run typecheck
```

این بررسی‌ها همچنین به‌طور خودکار وقتی سعی می‌کنیم کد commit کنیم اجرا می‌شوند، اما می‌توانیم آن‌ها را به‌صورت دستی اجرا کنیم تا قبل از commit مشکلات را شناسایی کنیم.

ارزش ذکر دارد که چون اکنون در دایرکتوری `chapters/chapter-02` هستیم، pre-commit hook اگر از setup فعلی اجرا شود کار نمی‌کند. پوشهٔ `.husky` باید در ریشهٔ repository باشد تا کار کند.

با ابزارهای توسعهٔ ما در جای خود، بیایید ببینیم چگونه فایل‌ها و پوشه‌های کد را در پروژه سازمان‌دهی می‌کنیم تا codebase را مقیاس‌پذیر و قابل نگهداری نگه داریم.

## مروری بر ساختار پروژه {#h1_83}

نحوهٔ سازمان‌دهی فایل‌ها و پوشه‌هایمان تأثیر عظیمی روی سهولت پیمایش (navigation) و نگهداری codebase ما دارد. بیایید رویکردهای مختلف را بررسی کنیم و بفهمیم چرا ساختار مبتنی بر feature را برای اپلیکیشنمان انتخاب کردیم.

## ساختار پروژه چیست و چرا مهم است؟ {#h2_84}

وقتی یک اپلیکیشن React می‌سازیم، باید تصمیم بگیریم موارد زیر را چگونه سازمان‌دهی کنیم:

- Componentها (تکه‌های UI قابل استفادهٔ مجدد)
- صفحات (صفحات کامل)
- API callها (data fetching)
- مدیریت state (دادهٔ اپلیکیشن)
- ابزارهای کمکی (utility functions)
- تستها (تضمین کیفیت)

ساختار اشتباه می‌تواند codebase را دشوار پیمایش کند و منجر به موارد زیر شود:

- دشواری یافتن کد مرتبط
- وابستگی‌های نامشخص بین بخش‌های مختلف
- code review دشوارتر
- توسعهٔ کندتر با رشد پروژه

## رویکردهای مختلف به ساختار پروژه {#h2_85}

چند رویکرد رایج برای سازمان‌دهی اپلیکیشن‌های React وجود دارد که هر کدام trade-offهای خود را دارند. در ادامه هر کدام را بررسی می‌کنیم.

### ساختار بر اساس نوع فایل {#h3_86}

ساختار مبتنی بر نوع فایل معمولاً در پروژه به این صورت به نظر می‌رسد:

```
src/
├── components/
├── pages/
├── hooks/
├── utils/
└── types/
```

مزایای این رویکرد:

- ساده برای شروع کردن
- آسان برای یافتن تمام componentها در یک جا
- جداسازی واضح بر اساس نوع فایل (component، page، hook، utility و type)

معایب این رویکرد:

- دشواری یافتن کد مرتبط وقتی روی یک feature کار می‌کنیم
- componentها بین featureها به‌شدت coupled می‌شوند
- دشواری در scale شدن با رشد اپلیکیشن زیرا codebase بیش از حد بزرگ و دشوار پیمایش می‌شود
- code review دشوارتر می‌شود (باید در چندین پوشه جستجو کنیم)
- دشواری حذف یک feature وقتی دیگر استفاده نمی‌شود زیرا همه‌جا پراکنده است

### ساختار مبتنی بر feature {#h3_87}

ساختار مبتنی بر feature معمولاً در پروژه به این صورت به نظر می‌رسد:

```
src/
├── app/
├── components/
├── features/
│   ├── auth/
│   ├── ideas/
│   ├── profile/
│   └── reviews/
└── lib/
```

مزایای این رویکرد:

- آسان برای یافتن تمام کد مرتبط با یک feature
- featureها می‌توانند به‌صورت مستقل develop شوند
- مرزهای واضح از coupling شدید جلوگیری می‌کنند
- با رشد اپلیکیشن خوب scale می‌شود
- اعضای تیم می‌توانند روی featureهای مختلف بدون conflict کار کنند

معایب این رویکرد:

- راه‌اندازی اولیه کمی پیچیده‌تر
- باید به این فکر کنیم کد مشترک کجا قرار گیرد
- نیاز به نظم برای حفظ مرزها دارد، اما این با قوانین ESLint حل می‌شود

## چرا ساختار مبتنی بر feature را انتخاب کردیم {#h2_88}

دلایلی که این انتخاب درستی برای اپلیکیشن ماست:

- **مرزهای واضح feature**: اپ ما featureهای متمایزی دارد، مانند احراز هویت (authentication)، مدیریت ideas، پروفایل کاربر و reviews. هر feature componentها، API call و logic خودش را دارد.
- **جریان کد واضح**: جریان کد واضح و آسان‌برای فهم است.
- **همکاری تیمی**: توسعه‌دهندگان مختلف می‌توانند روی featureهای مختلف (auth، ideas و reviews) بدون تداخل کار کنند.
- **مقیاس‌پذیری**: با اضافه کردن featureهای جدید، می‌توانیم آن‌ها را به‌عنوان پوشه‌های feature جدید اضافه کنیم بدون تأثیر روی کد موجود.
- **Code review**: هنگام بررسی یک feature، فقط باید فایل‌های آن پوشهٔ feature را ببینیم که بررسی‌ها را متمرکزتر و کارآمدتر می‌کند.
- **استقرار (deployment)**: در آینده می‌توانیم featureها را به repoهای جداگانه یا micro-frontends تقسیم کنیم یا آن‌ها را به‌صورت مستقل deploy کنیم.
- **سهولت حذف**: اگر feature دیگر استفاده نشود، می‌تواند به‌راحتی بدون تأثیر روی بقیهٔ codebase حذف شود.

پیچیدگی جزئی راه‌اندازی مرزها ارزشش را دارد زیرا با رشد اپلیکیشن و گسترش تیممان بازدهی دارد.

دایرکتوری `src` ما به این شکل ساختاربندی می‌شود:

```
src/
├── app/              # راه‌اندازی اصلی اپلیکیشن و entry point
├── components/       # componentهای UI قابل استفادهٔ مجدد (مخصوص feature نیستند)
├── config/           # فایل‌های پیکربندی
├── features/         # ماژول‌های مبتنی بر feature
│   ├── auth/        # feature احراز هویت
│   ├── ideas/       # feature مدیریت ideas
│   ├── profile/     # feature پروفایل کاربر
│   └── reviews/     # feature reviews
├── hooks/           # hookهای سفارشی React (مشترک بین featureها)
├── lib/             # توابع ابزاری و یکپارچه‌سازی‌های third-party
├── stores/          # مدیریت state سراسری
├── types/           # تعریف نوع‌های TypeScript
```

هر پوشهٔ feature شامل تمام چیزهای مرتبط با آن feature است:

```
features/ideas/
├── api/              # API callها برای این feature
├── components/       # componentهای مخصوص این feature
├── config/           # پیکربندی feature (query key و...)
├── hooks/            # hookهای مخصوص این feature
└── locales/          # بین‌المللی‌سازی برای این feature
```

این ساختار مزایای زیادی دارد:

- **یافتن آسان کد**: وقتی باید روی ideas کار کنیم، دقیقاً می‌دانیم کجا بگردیم
- **مرزهای واضح**: هر feature خودبسنده با componentها، API call و logic خودش است
- **مقیاس‌پذیری**: اضافه کردن featureهای جدید روی existingها تأثیر نمی‌گذارد
- **دوستدار تیم**: توسعه‌دهندگان مختلف می‌توانند روی featureهای مختلف بدون conflict کار کنند
- **ماژولار**: featureها می‌توانند به‌صورت مستقل develop، test و deploy شوند

## جریان کد {#h2_89}

بیایید ببینیم جریان کد در اپلیکیشن ما چگونه است.

![Figure 2.2 – جریان کد در codebase ما](/images/B31385_2_2.png)

**Figure 2.2 — جریان کد در codebase ما**

کد به‌صورت واضح و یک‌طرفه جریان می‌یابد. هر سطح را در ادامه بررسی می‌کنیم.

### اپلیکیشن (سطح بالا) {#h3_90}

دایرکتوری `app` در بالای سلسله‌مراتب codebase ما قرار دارد.

- می‌تواند از هر جا import کند: feature، component، hook، lib، store و type
- مثال: یک مسیر در `app/routes/dashboard/ideas/ideas.tsx` می‌تواند از `features/ideas/components/idea-list.tsx` import کند
- اینجاست که بیشتر کد برای تشکیل اپلیکیشن با هم ترکیب می‌شود

### Featureها (سطح میانی) {#h3_91}

Featureها در وسط سلسله‌مراتب قرار دارند.

- می‌توانند از ابزارهای مشترک (component، hook، lib، store و type) import کنند
- فقط اگر به‌طور صریح مجاز باشند می‌توانند از featureهای دیگر import کنند
- نمی‌توانند از دایرکتوری `app` import کنند

- مثال: feature `ideas` می‌تواند از `features/auth/hooks/use-user` import کند زیرا `auth` یک وابستگی مجاز است

- مثال: feature `ideas` نمی‌تواند از `features/profile` یا `features/reviews` import کند

### ابزارهای مشترک (سطح پایین) {#h3_92}

ابزارهای مشترک در پایین سلسله‌مراتب قرار دارند.

- فقط می‌توانند از یکدیگر import کنند
- نمی‌توانند از feature یا app import کنند
- مثال: یک component در `components` می‌تواند از hook در `hooks` یا utility در `lib` استفاده کند
- مثال: یک hook در `hooks` نمی‌تواند از `features/auth` import کند

### چرا این مهم است {#h3_93}

بیایید یک مثال واقعی ببینیم. فرض کنیم داریم لیستی از ideas را نمایش می‌دهیم:

```tsx
// مجاز: App از feature import می‌کند
// app/routes/dashboard/ideas/ideas.tsx
import { IdeaList } from '@/features/ideas/components/idea-list';

// مجاز: Feature از feature auth import می‌کند (وابستگی صریح)
// features/ideas/components/idea-list.tsx
import { useUser } from '@/features/auth/hooks/use-user';

// مجاز: Feature از ابزارهای مشترک import می‌کند
// features/ideas/components/idea-list.tsx
import { Button } from '@/components/ui/button';
import { formatDate } from '@/lib/date';

// ممنوع: Feature نمی‌تواند از app import کند
// features/ideas/api/get-ideas.ts
import { loader } from '@/app/routes/dashboard/ideas/ideas'; // خطای ESLint!

// ممنوع: Feature نمی‌تواند از feature غیرمجاز import کند
// features/ideas/components/idea-list.tsx
import { ProfileCard } from '@/features/profile/components/profile-card'; // خطای ESLint!

// ممنوع: ابزار مشترک نمی‌تواند از feature import کند
// components/idea-card.tsx
import { useUser } from '@/features/auth/hooks/use-user'; // خطای ESLint!
```

این جریان یک‌طرفه تضمین می‌کند:

- ابزارهای مشترک واقعاً قابل استفادهٔ مجدد باقی می‌مانند و وابستگی‌های پنهان ندارند
- Featureها مستقل می‌مانند و می‌توانند به‌صورت جداگانه develop/test شوند
- لایهٔ app تمام اجزا را هماهنگ می‌کند بدون اینکه توسط لایه‌های پایین‌تر import شود
- وابستگی‌ها صریح و کنترل‌شده هستند

## اعمال ساختار پروژه با ESLint {#h2_94}

داشتن چنین استانداردی عالی است و جریان کد واضح است، اما چگونه آن را اعمال کنیم؟ اگر آن را به توسعه‌دهندگانمان واگذار کنیم که از قوانین پیروی کنند، باید به آن‌ها اعتماد کنیم که کار درست را انجام دهند. انسان‌ها اشتباه می‌کنند. نباید حین ساخت featureها و حل مسائل واقعی نگران این نوع چیزها باشند. اینجاست که ESLint وارد تصویر می‌شود.

از `import/no-restricted-paths` از `eslint-plugin-import` برای تنظیم این محدودیت‌ها استفاده می‌کنیم.

برای تمیز نگه داشتن فایل `eslint.config.js` از فایل `infra/eslint-import-rules.js` برای تولید قوانین محدودیت import استفاده می‌کنیم.

اگر فایل `eslint.config.js` را باز کنیم، پیکربندی زیر را می‌بینیم:

```js
// eslint.config.js
import { importRules } from './infra/eslint-import-rules.js';

export default [
  // ... پیکربندی دیگر
  {
    rules: {
      'import/no-restricted-paths': [
        'error',
        {
          zones: importRules, // قوانین سفارشی ما اینجا اعمال می‌شوند
        },
      ],
    },
  },
];
```

هر entry در `zones` یک محدودیت تعریف می‌کند: یک `target` (فایل‌هایی که محافظت می‌شوند) و یک `from` (فایل‌هایی که اجازهٔ import از آن‌ها را ندارند).

بیایید هر محدودیت تعریف‌شده در `infra/eslint-import-rules.js` را بررسی کنیم.

### محدودیت ۱: ابزارهای مشترک باید مستقل باقی بمانند {#h3_95}

برای اعمال این محدودیت، قانون زیر را تعریف می‌کنیم:

```js
// infra/eslint-import-rules.js
zones.push({
  target: [
    './src/components/**',
    './src/config/**',
    './src/hooks/**',
    './src/lib/**',
    './src/stores/**',
    './src/types/**',
    './src/utils/**',
  ],
  from: ['./src/features/**', './src/app/**'],
  message:
    'Shared utilities should not import from features or app directories.',
});
```

این قانون تضمین می‌کند پوشه‌های مشترک (components، config، hooks، lib، stores، types و utils) نمی‌توانند از featureها یا app import کنند. این کد مشترک ما را واقعاً قابل استفادهٔ مجدد نگه می‌دارد. اگر ابزاری به logic مخصوص feature نیاز دارد، باید آن logic را به‌عنوان پارامتر بپذیرد یا به feature منتقل شود.

### محدودیت ۲: Featureها نمی‌توانند از app import کنند {#h3_96}

برای حفظ این مرز، قانون دیگری تعریف می‌کنیم:

```js
zones.push({
  target: `./src/features/**/**`,
  from: `./src/app/**/**`,
  message: `Features should not import from app directory.`,
});
```

این جریان یک‌طرفه را حفظ می‌کند که featureها نمی‌توانند از `app` import کنند. این featureها را مستقل از نحوهٔ استفاده‌شان در مسیرها نگه می‌دارد و به آن‌ها اجازه می‌دهد در سراسر اپلیکیشن مجدداً استفاده شوند.

### محدودیت ۳: وابستگی‌های feature صریح و کنترل‌شده هستند {#h3_97}

این با قانون زیر پیکربندی می‌شود:

```js
// infra/eslint-import-rules.js
const features = [
  { name: 'auth', allowedFeatures: [] },
  { name: 'ideas', allowedFeatures: ['auth'] },
  { name: 'profile', allowedFeatures: ['auth'] },
  { name: 'reviews', allowedFeatures: ['auth'] },
];
```

این پیکربندی وابستگی‌ای ایجاد می‌کند که:

- `auth` بنیاد است؛ نمی‌تواند از featureهای دیگر import کند
- `ideas`، `profile` و `reviews` فقط می‌توانند از `auth` import کنند
- هیچ feature نمی‌تواند از featureهای هم‌سطح import کند (مثلاً `ideas` نمی‌تواند از `profile` import کند)

این از وابستگی‌های circular جلوگیری می‌کند و featureها را مستقل نگه می‌دارد. اگر به کد مشترکی بین featureها نیاز داریم، آن را به پوشهٔ `components` یا `lib` منتقل می‌کنیم.

### چرا این مهم است {#h3_98}

این قوانین ESLint از مشکلات رایج در codebaseهای بزرگ جلوگیری می‌کنند:

- **جلوگیری از وابستگی‌های circular**: سلسله‌مراتب واضح importهای circular را غیرممکن می‌کند
- **کاهش coupling شدید**: Featureها با interfaceهای تعریف‌شده مستقل باقی می‌مانند
- **جلوگیری از لغزش معماری**: نقض‌ها فوراً شناسایی می‌شوند و از میانبرها جلوگیری می‌شود
- **صریح کردن وابستگی‌ها**: وابستگی‌های پنهانی در ابزارهای مشترک وجود ندارد
- **فعال کردن تست مستقل**: هر feature می‌تواند به‌صورت جداگانه tested شود
- **بهبود onboarding**: معماری از طریق پیکربندی ESLint خودمستند است

## ESLint در عمل {#h2_99}

وقتی این قوانین را نقض می‌کنیم، ESLint پیام خطای واضحی به ما می‌دهد:

```tsx
// src/features/profile/components/profile-stats.tsx
import { IdeaCard } from '@/features/ideas/components/idea-card';
// خطای ESLint: feature profile نباید از feature ideas import کند.
```

راه‌حل بستگی به use case ما دارد:

- **انتقال به مشترک**: اگر چندین feature از component یا hook یا هر چیزی استفاده کنند که باید در featureهای مختلف استفاده شود، آن را به یکی از پوشه‌های مشترک مانند `src/components` یا `src/lib` منتقل کنید.
  - مثال: یک component عمومی `Card` که توسط `ideas` و `profile` استفاده می‌شود
- **ارسال از طریق props**: اگر component مخصوص یک feature است، آن را همان‌جا نگه دارید و داده را از طریق props منتقل کنید.
  - مثال: پروفایلی که تعداد ideas را از طریق prop دریافت می‌کند
- **اضافه کردن dependency**: فقط اگر دلیل معماری مشروعی برای وابستگی یک feature به feature دیگر وجود دارد، `allowedFeatures` را در `infra/eslint-import-rules.js` به‌روز کنید.
  - مثال: `profile` نیاز مشروعی به استفاده از بررسی session `auth` دارد

این قوانین به‌طور خودکار بررسی می‌کنند که از معماری پیروی می‌کنیم. وقتی سعی می‌کنیم از جای اشتباه import کنیم، ESLint فوراً به ما می‌گوید. این featureهای ما را مستقل و کدمان را آسان‌برای فهم نگه می‌دارد.

اکنون که ساختار پروژه و ابزارهای کیفیت کد را در جای خود داریم، یک بخش نهایی از setup ما باقی مانده: مدیریت پیکربندی‌ای که بین محیط‌ها تغییر می‌کند. متغیرهای محیطی به ما اجازه می‌دهند اپلیکیشن را بدون hardcoded کردن مقدارها پیکربندی کنیم.

## مروری بر راه‌اندازی متغیرهای محیطی {#h1_100}

اپلیکیشن ما به پیکربندی‌های مختلفی برای development، staging و production نیاز دارد. متغیرهای محیطی به ما اجازه می‌دهند این پیکربندی‌ها را بدون تغییر کد مدیریت کنیم. بیایید ببینیم چگونه آن‌ها را با type safety و validation به‌درستی راه‌اندازی کنیم.

## متغیرهای محیطی چیست و چرا به آن نیاز داریم؟ {#h2_101}

بدون متغیرهای محیطی، مجبور بودیم مقادیر پیکربندی را در کدمان hardcoded کنیم:

```ts
// بد: مقادیر hardcoded
const API_URL = "https://api.production.com";
const API_KEY = "1234567890";
```

این مشکلات زیادی ایجاد می‌کند:

- نمی‌توانیم از APIهای مختلف برای محیط‌های مختلف استفاده کنیم
- به‌طور تصادفی اطلاعات حساس را به repository می‌فرستیم
- مجبوریم هر بار که می‌خواهیم محیط را عوض کنیم کد را تغییر دهیم

## سیستم متغیرهای محیطی ما {#h2_102}

سیستم متغیرهای محیطی قوی‌ای راه‌اندازی کرده‌ایم که پیکربندی ما را هنگام راه‌اندازی validate می‌کند. نحوهٔ کار آن به شرح زیر است:

```ts
// src/config/env.ts
const envMapping = {
  API_URL: 'VITE_API_URL',
} as const;

export const envSchema = z.object({
  API_URL: z.url('API_URL must be a valid URL'),
});

const parseEnv = () => {
  const rawEnv: Record<string, string | undefined> = {};
  
  for (const [cleanKey, viteKey] of Object.entries(envMapping)) {
    rawEnv[cleanKey] = import.meta.env[viteKey];
  }
  
  try {
    return envSchema.parse(rawEnv);
  } catch (error) {
    // ... error handling
  }
};

export const env = parseEnv();
```

چرا آن را این‌گونه راه‌اندازی کرده‌ایم:

- **نام‌های تمیز متغیرها**: از `envMapping` استفاده می‌کنیم تا نام متغیرهای محیطی را به نام‌های تمیز نگاشت کنیم، مانند `API_URL` به‌جای `VITE_API_URL` به‌صورت داخلی. باید این کار را انجام دهیم زیرا Vite پیشوند `VITE_` را به نام متغیرهای محیطی اضافه می‌کند.
- **Type safety**: [Zod](https://zod.dev/) validate می‌کند که متغیرهای محیطی نوع صحیح دارند.
- **پیام‌های خطای واضح**: اگر متغیری ناموجود یا نامعتبر باشد، پیام خطای مفیدی دریافت می‌کنیم.

توجه کنید که فقط متغیرهای محیطی را نمی‌خوانیم؛ آن‌ها را validate می‌کنیم و رابط تمیز و typed برای بقیهٔ اپلیکیشن فراهم می‌کنیم.

اکنون می‌توانیم متغیرهای محیطی را import و استفاده کنیم:

```ts
import { env } from '@/config/env';

console.log(env.API_URL);
```

این رویکرد از خطاهای runtime که توسط متغیرهای محیطی ناموجود یا نامعتبر ایجاد می‌شود جلوگیری می‌کند و مشخص می‌کند اپلیکیشن ما برای اجرا به چه پیکربندی‌ای نیاز دارد، زیرا کل setup متمرکز شده است.

## راه‌اندازی متغیرهای محیطی {#h2_103}

در پروژهٔ ما فایل `.env.example` وجود دارد که تمام متغیرهای محیطی مورد نیاز اپلیکیشن را فهرست می‌کند:

```
# .env.example
VITE_API_URL=http://localhost:9999
```

برای راه‌اندازی محیط توسعهٔ محلی، این فایل را به `.env` کپی می‌کنیم:

```bash
cp .env.example .env
```

سپس مقادیر `.env` را با setup محلی‌مان به‌روز می‌کنیم. فایل `.env` در git ignored است، بنابراین می‌توانیم با خیال راحت مقادیر خودمان را بدون commit کردن به repository اضافه کنیم.

اگر سعی کنیم اپلیکیشن را بدون تنظیم متغیرهای محیطی اجرا کنیم، خطای زیر را دریافت می‌کنیم:

```
3:10:35 PM [vite] Internal server error: Environment validation failed:
API_URL (VITE_API_URL): API_URL must be a valid URL

Please check your .env file and ensure all required variables are set.
```

این عالی است زیرا از استقرار اپلیکیشن با متغیرهای محیطی ناموجود جلوگیری می‌کند. پیام خطا به‌وضوح نشان می‌دهد کدام متغیر ناموجود است و فرمت مورد انتظار چیست.

با داشتن تمام این ابزارها و ساختارها — meta framework، TypeScript، linting، formatting، pre-commit hook، سازمان‌دهی پروژه و پیکربندی متغیرهای محیطی — بنیان محکمی برای ساخت اپلیکیشن‌های React قابل نگهداری داریم.

## خلاصه {#h1_104}

در این فصل بنیان ساخت یک اپلیکیشن React آمادهٔ production را ایجاد کردیم.

با انتخاب **React Router** به‌عنوان meta framework به‌خاطر انعطاف‌پذیری‌اش در پشتیبانی از CSR و SSR بدون قفل شدن در ارائه‌دهندهٔ hosting خاص شروع کردیم.

**Vite** را به‌عنوان build tool و نحوهٔ پیکربندی آن بر اساس نیازهایمان پوشش دادیم.

سپس workflow توسعه را با **TypeScript** برای type safety، **ESLint** برای کیفیت کد، **Prettier** برای قالب‌بندی یکدست و **pre-commit hook** برای شناسایی خودکار مشکلات قبل از رسیدن به repository راه‌اندازی کردیم.

مهم‌ترین تصمیم معماری، پذیرش ساختار پروژه مبتنی بر feature بود. به‌جای گروه‌بندی فایل‌ها بر اساس نوع (component، hook و utility)، آن‌ها را بر اساس feature (auth، ideas، profile و reviews) سازمان‌دهی کردیم. مرزهای feature را با قوانین ESLint اعمال کردیم که از وابستگی‌های circular جلوگیری می‌کند و codebase را با رشد ماژولار نگه می‌دارد.

در پایان، validation متغیرهای محیطی را با **Zod** پیاده‌سازی کردیم تا مطمئن شویم پیکربندی ما قبل از شروع اپلیکیشن صحیح است و خطاهای پیکربندی را زود شناسایی کنیم.

با داشتن این ابزارها و الگوها، تمام چیزی را که برای ساخت اپلیکیشن‌های React قابل نگهداری و مقیاس‌پذیر نیاز داریم در اختیار داریم.
