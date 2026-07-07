# فصل ۹: بین‌المللی‌سازی {#h1_232 .chapterTitle}

اگر می‌خواهیم اپلیکیشنمان به کاربران سراسر جهان برسد، باید از زبان‌های مختلف پشتیبانی کنیم. بین‌المللی‌سازی دقیقاً همین جا وارد می‌شود. شاید این اصطلاح را به اختصار i18n ببینید، چون بین حرف «i» و «n» در کلمهٔ internationalization ۱۸ حرف وجود دارد.

وقتی کاربران بتوانند با اپلیکیشن به زبان خودشان تعامل کنند، راحت‌تر هستند و احتمال استفادهٔ مداومشان بیشتر است. اما بین‌المللی‌سازی فقط ترجمهٔ متن نیست. باید به نحوهٔ قالب‌بندی تاریخ‌ها، نمایش اعداد، pluralization (یک قلم در برابر دو قلم) و حتی جهت جریان متن هم فکر کنیم. بعضی زبان‌ها مثل عربی و عبری از راست به چپ خوانده می‌شوند.

اگر از ابتدا بین‌المللی‌سازی را درست راه‌اندازی کنیم، اضافه کردن زبان‌های جدید در آینده ساده می‌شود. فایل‌های ترجمه اضافه می‌کنیم و اپلیکیشن خودش کار می‌کند.

موارد زیر را پوشش می‌دهیم:

- درک معماری بین‌المللی‌سازی
- راه‌اندازی i18n در اپلیکیشن
- استفاده از ترجمه‌ها در اپلیکیشن
- جابه‌جایی بین زبان‌ها در اپلیکیشن

در پایان این فصل، اپلیکیشنی کاملاً بین‌المللی‌شده خواهیم داشت که از زبان‌های مختلف پشتیبانی می‌کند، با ترجمه‌های type-safe و تجربهٔ کاربری روان هنگام جابه‌جایی بین زبان‌ها.

## الزامات فنی {#h1_233}

قبل از شروع باید پروژه را راه‌اندازی کنیم. برای توسعهٔ پروژه به ابزارهای زیر روی کامپیوتر نیاز داریم:

- [Node.js](https://nodejs.org/) نسخهٔ ۲۴ یا بالاتر. [npm](https://www.npmjs.com/) نسخهٔ ۱۱ یا بالاتر همراه Node عرضه می‌شود. می‌توانیم با اجرای `node ‑v` و `npm ‑v` در ترمینال این را تأیید کنیم. راه‌های مختلفی برای نصب Node.js و npm وجود دارد. این مقالهٔ مفید جزئیات بیشتری توضیح می‌دهد: [https://www.nodejsdesignpatterns.com/blog/5-ways-to-install-node-js](https://www.nodejsdesignpatterns.com/blog/5-ways-to-install-node-js).
- [VS Code](https://code.visualstudio.com/) (اختیاری)، ویرایشگر محبوب برای JavaScript و [TypeScript](https://www.typescriptlang.org/). متن‌باز است، پشتیبانی خوبی از TypeScript دارد و افزونه‌های زیادی ارائه می‌دهد. می‌توان آن را از [https://code.visualstudio.com](https://code.visualstudio.com) دانلود کرد.

کد این کتاب در [https://github.com/PacktPublishing/React-Application-Architecture-for-Production-Second-Edition](https://github.com/PacktPublishing/React-Application-Architecture-for-Production-Second-Edition) در GitHub موجود است. آن را clone کنید و وارد ریشهٔ مخزن شوید:

```shell
git clone https://github.com/PacktPublishing/React-Application-Architecture-for-Production-Second-Edition.git
```

مخزن شامل پوشه‌های فصلی با کد هر فصل و یک پوشهٔ مشترک `api` است که سرور API مورد استفاده در تمام فصل‌ها را شامل می‌شود.

در حال کار روی فصل ۹ هستیم، پس وارد پوشهٔ `chapter‑09` شوید:

```shell
cd React-Application-Architecture-for-Production-Second-Edition/chapter-09
```

سپس dependencyها را نصب کنید:

```shell
npm install
```

همچنین باید متغیرهای محیطی را ارائه دهیم:

```shell
cp .env.example .env
```

در این مرحله frontend باید آماده باشد و روی [http://localhost:5173](http://localhost:5173) اجرا شود.

همچنین باید سرور API را اجرا کنیم.

پنجرهٔ ترمینال جدیدی باز کنید و وارد پوشهٔ `api` شوید:

```shell
cd React-Application-Architecture-for-Production-Second-Edition/api
```

اسکریپت setup را برای فصل ۹ اجرا کنید تا همه‌چیز پیکربندی شود:

```shell
npm run setup 09
```

سپس سرور API را اجرا کنید:

```shell
npm run dev
```

سرور API حالا باید روی [http://localhost:9999](http://localhost:9999) اجرا شود.

برای اطلاعات بیشتر دربارهٔ جزئیات setup، فایل `README.md` را ببینید.

## درک معماری بین‌المللی‌سازی {#h1_234}

ایدهٔ اصلی پشت بین‌المللی‌سازی ساده است: محتوای متنی را از کد جدا می‌کنیم. به‌جای نوشتن «Welcome» مستقیماً در کامپوننت، از کلیدی مثل `welcome` استفاده می‌کنیم و سیستم i18n بر اساس زبان کاربر متن مناسب را پیدا می‌کند.

چرا این کار را می‌کنیم؟ فکر کنید بدون این جداسازی چه اتفاقی می‌افتد. اگر «Welcome» را مستقیماً در کامپوننت hardcode کنیم و بعداً نیاز به پشتیبانی از اسپانیایی باشد، باید هر کامپوننت را بررسی کنیم، هر رشته را پیدا کنیم و logic شرطی اضافه کنیم تا نسخهٔ اسپانیایی نشان داده شود. یا نسخهٔ جداگانهٔ اپلیکیشن برای هر زبان داشته باشیم. این کار به‌هم‌ریخته، پرخطا و اصلاً منطقی نیست.

با i18n، کامپوننت همیشه از کلیدهای ترجمه مثل `t('welcome')` استفاده می‌کند. سیستم ترجمه بقیهٔ کارها را انجام می‌دهد. وقتی اسپانیایی اضافه می‌کنیم، فقط فایل ترجمهٔ اسپانیایی با همان کلیدها می‌سازیم. کامپوننت‌ها اصلاً تغییر نمی‌کنند.

### ذخیرهٔ ترجمه‌ها {#h2_235}

قبل از ساخت سیستم i18n باید تصمیم بگیریم ترجمه‌ها کجا ذخیره شوند. این انتخاب روی نحوهٔ کار مترجمان، نحوهٔ استقرار به‌روزرسانی‌ها و ابزارهایی که می‌توانیم استفاده کنیم تأثیر می‌گذارد.

دو رویکرد اصلی وجود دارد:

- **ترجمه‌های درون codebase**: فایل‌های ترجمه مستقیماً در مخزن کد منبع ذخیره می‌شوند، معمولاً به فرمت JSON، YAML یا TypeScript. این رویکرد ساده است و برای اکثر پروژه‌ها خوب کار می‌کند. ترجمه‌ها کنار کد version controlled می‌شوند، پس وقتی feature جدید اضافه می‌کنید، ترجمه‌های آن را در همان pull request اضافه می‌کنید. تغییرات ترجمه همان فرآیند review کد را طی می‌کنند. محدودیت اصلی این است که به‌روزرسانی ترجمه نیاز به استقرار کد دارد؛ نمی‌توانید یک غلط تایپی را بدون push نسخهٔ جدید اصلاح کنید.
- **سرویس‌های ترجمهٔ خارجی** مثل [Lokalise](https://lokalise.com/)، [Crowdin](https://crowdin.com/) یا [Phrase](https://www.phrase.com/) ترجمه‌ها را از کد جدا می‌کنند. این پلتفرم‌ها رابط وب ارائه می‌دهند که مترجمان بدون دسترسی به codebase کار می‌کنند. برای تیم‌های بزرگ‌تر ارزشمندند، به‌خصوص وقتی با مترجمان حرفه‌ای کار می‌کنید که نباید به codebase دسترسی داشته باشند. بسیاری از آن‌ها ویژگی‌هایی مثل translation memory، پیشنهادهای خودکار و مدیریت workflow ارائه می‌دهند. اما پیچیدگی، هزینه و overhead ادغام اضافه می‌کنند که ممکن است برای پروژه‌های کوچک‌تر توجیه‌پذیر نباشد.

برای اپلیکیشن ما، از ترجمه‌های درون codebase ذخیره‌شده به فرمت TypeScript استفاده می‌کنیم. این type safety (TypeScript می‌تواند کلیدهای ترجمه را اعتبارسنجی کند)، سادگی (بدون dependency خارجی) و ادغام نزدیک با workflow توسعه را فراهم می‌کند. با رشد پروژه، در صورت نیاز می‌توانیم به سرویس خارجی مهاجرت کنیم، اما شروع ساده منطقی است.

### نحوهٔ کار سیستم i18n ما {#h2_236}

حالا بیایید جریان کامل نحوهٔ عملکرد بین‌المللی‌سازی در اپلیکیشن server-rendered [React](https://react.dev/) را درک کنیم. این به شما کمک می‌کند ببینید همهٔ قطعاتی که خواهیم ساخت چگونه با هم ترکیب می‌شوند.

![شکل ۹.۱ – جریان سیستم i18n](/images/B31385_9_1.png)

**شکل ۹.۱ — جریان سیستم i18n**

نمودار نشان می‌دهد ترجمه‌ها چگونه در اپلیکیشن جریان می‌یابند:

### تشخیص و ذخیرهٔ زبان {#h3_237}

وقتی کاربر از اپلیکیشن ما بازدید می‌کند، باید بدانیم چه زبانی نشانش دهیم. تشخیص ساده است: cookie ترجیح زبان را بررسی می‌کنیم. اگر cookie وجود داشت، از آن زبان استفاده می‌کنیم. اگر وجود نداشت، به‌طور پیش‌فرض انگلیسی را انتخاب می‌کنیم.

کاربران به‌طور صریح زبان خود را با استفاده از مبدل زبانی که خواهیم ساخت انتخاب می‌کنند. وقتی انتخاب می‌کنند، آن را در یک cookie امن HTTP-only ذخیره می‌کنیم. این یعنی انتخابشان در نشست‌های مرورگر و بارگذاری‌های مجدد صفحه باقی می‌ماند. هم سرور و هم client این cookie را می‌خوانند و تشخیص یکسان زبان را در هر دو محیط تضمین می‌کنند.

### رندر سمت سرور با ترجمه‌ها {#h3_238}

وقتی سرور درخواستی دریافت می‌کند، قبل از رندر صفحه cookie زبان را می‌خواند. سرور به همهٔ فایل‌های ترجمه دسترسی دارد، پس می‌تواند فوراً کل صفحه را به زبان صحیح رندر کند. این برای SEO مهم است چون موتورهای جستجو محتوای ترجمه‌شده را می‌بینند. همچنین برای performance مهم است چون کاربران محتوا را به زبان خودشان فوراً می‌بینند، بدون اینکه منتظر بارگذاری و اجرای JavaScript بمانند.

سرور HTML کاملاً ترجمه‌شده را به مرورگر ارسال می‌کند. اگر کاربر اسپانیایی‌زبان صفحهٔ اصلی را درخواست کند، HTML با «Bienvenido» از قبل در آن دریافت می‌کند، نه کلیدی مثل `welcome` که باید در سمت client ترجمه شود.

### hydration سمت کلاینت {#h3_239}

وقتی JavaScript در مرورگر بارگذاری می‌شود، React باید «hydrate» کند — event listenerها را متصل کند و صفحه را interactive کند. برای اینکه این بدون flash یا re-render کار کند، client به همان ترجمه‌هایی نیاز دارد که سرور استفاده کرده.

ترجمه‌هایی که بیشترین استفاده را دارند مستقیماً در JavaScript bundle می‌کنیم. این‌ها ترجمه‌هایی هستند که در هر صفحه ظاهر می‌شوند، مثل navigation، common و غیره. آن‌ها فوراً در دسترسند وقتی JavaScript مقداردهی اولیه می‌شود.

Client زبان فعلی را از attribute `lang` تشخیص می‌دهد که توسط سرور تنظیم شده و آن ترجمه‌های bundle شده را بارگذاری می‌کند. حالا وقتی React hydrate می‌شود، از همان کلیدهای ترجمه استفاده می‌کند و همان متنی را دریافت می‌کند که سرور رندر کرده. بدون flash یا جابه‌جایی layout.

### بارگذاری پویای ترجمه‌ها {#h3_240}

همهٔ ترجمه‌ها با JavaScript bundle نمی‌شوند. اگر همهٔ ترجمه‌ها برای همهٔ featureها در هر زبان bundle می‌کردیم، بارگذاری اولیه بسیار سنگین می‌شد. بیشتر کاربران هرگز هر صفحه را بازدید نمی‌کنند، پس ترجمه‌هایی را دانلود می‌کردند که هرگز استفاده نمی‌کنند.

در عوض، ترجمه‌ها را on-demand بارگذاری می‌کنیم. وقتی کاربر به صفحات `auth` ناوبری می‌کند، client namespace مربوط به auth را از API ما دریافت می‌کند. وقتی بخش `ideas` را بازدید می‌کند، namespace ideas را دریافت می‌کند. این به‌صورت خودکار اتفاق می‌افتد. پیکربندی می‌کنیم هر route به چه namespaceهایی نیاز دارد و [react-i18next](https://react.i18next.com/) دریافت را انجام می‌دهد.

فایل‌های ترجمه به‌شدت cache می‌شوند، پس بعد از بازدید اول از یک بخش، بازدیدهای بعدی فوری هستند.

### جابه‌جایی بین زبان‌ها {#h3_241}

وقتی کاربر زبان را عوض می‌کند، cookie را از طریق API call به‌روزرسانی می‌کنیم و به سیستم i18n سمت client می‌گوییم زبان را تغییر دهد. اگر ترجمه‌های زبان جدید هنوز بارگذاری نشده باشند، از API دریافت می‌شوند. وقتی همه‌چیز آماده شد، کل UI به زبان جدید re-render می‌شود. cookie تضمین می‌کند اگر کاربر صفحه را reload کند یا بعداً برگردد، همان زبان را ببیند.

این معماری بهترین هر دو دنیا را می‌دهد: صفحات سریع رندر شده در سرور با ترجمه‌های کامل برای SEO و بارگذاری اولیه، و به‌روزرسانی‌های کارآمد سمت client که فقط آنچه لازم است را بارگذاری می‌کند. همان کامپوننت‌ها در هر دو محیط کار می‌کنند چون همیشه از کلیدهای ترجمه استفاده می‌کنند، هرگز رشته‌های hard-coded.

حالا بیایید این سیستم را قطعه‌به‌قطعه بسازیم.

## راه‌اندازی i18n در اپلیکیشن {#h1_242}

حالا که معماری را درک کردیم، بیایید آن را پیاده‌سازی کنیم. این شامل سازمان‌دهی ترجمه‌ها با namespace، پیکربندی react-i18next، سرو ترجمه‌ها از طریق API، اضافه کردن type safety و راه‌اندازی attributeهای زبان و جهت می‌شود.

### سازمان‌دهی ترجمه‌ها با namespace {#h2_243}

با اضافه کردن featureهای بیشتر، فایل‌های ترجمه به‌راحتی به هزاران خط رشد می‌کنند. پیدا کردن و به‌روزرسانی ترجمه‌ها دشوار می‌شود. namespace این مشکل را حل می‌کند و اجازه می‌دهد ترجمه‌ها را به گروه‌های منطقی تقسیم کنیم.

به‌جای یک فایل عظیم شامل همهٔ ترجمه‌ها، فایل‌های کوچک‌تری بر اساس feature یا صفحه ایجاد می‌کنیم. صفحهٔ اصلی ترجمه‌های خودش را دارد، احراز هویت ترجمه‌های خودش را دارد، و به همین ترتیب. این پیدا کردن و نگهداری ترجمه‌ها را آسان‌تر می‌کند. همچنین به performance کمک می‌کند چون فقط ترجمه‌هایی را بارگذاری می‌کنیم که واقعاً برای صفحهٔ فعلی لازم داریم.

ترجمه‌ها را در دو سطح سازمان‌دهی می‌کنیم: ترجمه‌های سطح اپلیکیشن برای چیزهایی که همه‌جا استفاده می‌شوند و ترجمه‌های feature-scoped که کنار feature خودشان قرار دارند.

### ترجمه‌های سطح اپلیکیشن {#h3_244}

برای ترجمه‌هایی که در کل اپلیکیشن استفاده می‌شوند، فایل‌ها را در پوشهٔ `app/locales` ایجاد می‌کنیم:

```typescript
// src/app/locales/en/common.ts

export default {
  cancel: 'Cancel',
  delete: 'Delete',
  deleting: 'Deleting...',
  // ...
};
```

این namespace `common` شامل ترجمه‌هایی است که در صفحات زیادی ظاهر می‌شوند.

همچنین می‌توانیم فایل‌های namespace برای صفحات خاص ایجاد کنیم:

```typescript
// src/app/locales/en/home.ts

export default {
  meta: {
    description: 'A community platform for sharing and discovering AI ideas',
    title: 'AIdeas - Share and Discover AI Ideas',
  },
  subtitle: 'A community platform for sharing and discovering AI ideas',
  title: 'AIdeas - Share and Discover AI Ideas',
  // ...
};
```

ترجمه‌های مرتبط را تو در تو قرار می‌دهیم تا گروه‌بندی شوند. شیء `meta` همهٔ ترجمه‌های مربوط به metadata را گروه‌بندی می‌کند. وقتی نیاز به پیدا کردن ترجمه داریم، تو در تو بودن مشخص می‌کند کجا دنبالش بگردیم.

### ترجمه‌های feature-scoped {#h3_245}

برای featureهای بزرگ‌تر، ترجمه‌ها را نزدیک کد خود feature نگه می‌داریم. این همان اصلی است که برای سازمان‌دهی کامپوننت‌ها و hookها استفاده می‌کنیم، جایی که چیزهای مرتبط را کنار هم نگه می‌داریم:

```typescript
// src/features/auth/locales/en.ts

export default {
  alreadyHaveAccount: 'Already have an account?',
  creatingAccount: 'Creating account...',
  dontHaveAccount: "Don't have an account?",
  // ...
};
```

با قرار دادن ترجمه‌های feature در `features/auth/locales`، feature auth را self-contained می‌کنیم. اگر هر زمانی نیاز باشد این feature را به package جداگانه‌ای استخراج کنیم یا در جای دیگری استفاده کنیم، همهٔ ترجمه‌هایش همراهش می‌آیند.

### ترکیب ترجمه‌ها {#h3_246}

همهٔ فایل‌های ترجمهٔ جداگانه باید کنار هم بیایند تا سیستم i18n بتواند به آن‌ها دسترسی داشته باشد. فایل index ایجاد می‌کنیم که همه‌چیز را ترکیب می‌کند:

```typescript
// src/app/locales/en/index.ts

import type { ResourceLanguage } from 'i18next';

import authTranslations from '@/features/auth/locales/en';
import ideasTranslations from '@/features/ideas/locales/en';
import profileTranslations from '@/features/profile/locales/en';
import reviewsTranslations from '@/features/reviews/locales/en';

import about from './about';
import common from './common';
import components from './components';
import dashboard from './dashboard';
import home from './home';
import navigation from './navigation';
import notFound from './not-found';

export default {
  common,
  notFound,
  home,
  about,
  dashboard,
  navigation,
  components,
  auth: authTranslations,
  ideas: ideasTranslations,
  reviews: reviewsTranslations,
  profile: profileTranslations,
} satisfies ResourceLanguage;
```

این namespaceهای مشخصی به ما می‌دهد: `common`، `home`، `auth`، `ideas` و غیره. وقتی از ترجمه‌ها در کامپوننت‌ها استفاده می‌کنیم، آن‌ها را به صورت `common:cancel` یا `auth:loginTitle` مرجع می‌دهیم. پیشوند namespace مشخص می‌کند هر ترجمه از کجا می‌آید.

## پیکربندی react-i18next {#h2_247}

react-i18next ادغام React برای [i18next](https://www.i18next.com/) است که یک کتابخانهٔ بسیار محبوب i18n در اکوسیستم JavaScript است. hookها و کامپوننت‌هایی به ما می‌دهد که کار با ترجمه‌ها در React را ساده می‌کنند.

برای شروع، فایل پیکربندی متمرکزی ایجاد می‌کنیم که زبان‌های پشتیبانی‌شده و نحوهٔ عملکرد سیستم را تعریف می‌کند:

```typescript
// src/config/i18n.ts

export const languages = {
  en: 'English',
  es: 'Español',
} as const;

export type Language = keyof typeof languages;
const supportedLanguages = Object.keys(languages) as Language[];

export const i18nConfig = {
  defaultNS: 'common' as const,
  fallbackLng: 'en' as const,,
  supportedLanguages,
  backend: {
    loadPath: '/api/locales/{{lng}}/{{ns}}',
  },
  detection: {
    order: ['htmlTag'],
    caches: [],
  },
  cookieName: 'lng' as const,
};
```

بیایید هر تنظیم را بررسی کنیم:

- **languages** — کدهای زبان را به نام‌های نمایشی نگاشت می‌کند. `as const` باعث می‌شود TypeScript آن‌ها را به‌عنوان نوع literal برای type safety بهتر در نظر بگیرد.
- **supportedLanguages** — لیست کدهای زبانی که پشتیبانی می‌کنیم، استخراج‌شده از شیء languages.
- **defaultNS** — namespace پیش‌فرض وقتی namespace مشخصی تعیین نمی‌کنیم. از آنجا که `common` شامل ترجمه‌های پرکاربرد است، انتخاب پیش‌فرض منطقی است.
- **fallbackLng** — زبانی که اگر زبان ترجیحی کاربر در دسترس نباشد استفاده می‌شود. پیش‌فرض انگلیسی است.
- **backend.loadPath** — الگوی URL برای بارگذاری ترجمه‌هایی که bundle نشده‌اند. placeholderهای `{{ lng }}` و `{{ns}}` با زبان و namespace جایگزین می‌شوند. به‌زودی این endpoint API را ایجاد خواهیم کرد.
- **detection.order** — نحوهٔ تشخیص زبان کاربر توسط client. پیکربندی می‌کنیم از attribute `lang` HTML بخواند که سرور بر اساس cookie تنظیم کرده.
- **cookieName** — نام cookie که ترجیح زبان کاربر را ذخیره می‌کند.

حالا بیایید این پیکربندی را در اپلیکیشن استفاده کنیم.

### ایجاد middleware i18next {#h3_248}

از آنجا که از رندر سمت سرور استفاده می‌کنیم، به middleware نیاز داریم که قبل از رندر صفحات i18n را راه‌اندازی کند:

```typescript
// src/app/middleware/i18next.ts

import { createCookie } from 'react-router';
import { createI18nextMiddleware } from 'remix-i18next/middleware';

import { resources } from '@/app/locales';
import { i18nConfig } from '@/config/i18n';

export const localeCookie = createCookie(i18nConfig.cookieName, {
  path: '/',
  sameSite: 'lax',
  secure: process.env.NODE_ENV === 'production',
  httpOnly: true,
});

export const [i18nextMiddleware, getLocale, getInstance] =
  createI18nextMiddleware({
    detection: {
      supportedLanguages: i18nConfig.supportedLanguages,
      fallbackLanguage: i18nConfig.fallbackLng,
      cookie: localeCookie,
    },
    i18next: {
      resources,
      defaultNS: i18nConfig.defaultNS,
    },
  });
```

این middleware سه کار مهم انجام می‌دهد. cookie امنی ایجاد می‌کند تا ترجیح زبان کاربر را به یاد بیاورد. زبان کاربر را از cookie تشخیص می‌دهد. نمونهٔ i18next را با همهٔ ترجمه‌هایمان مقداردهی اولیه می‌کند تا بتوانیم صفحات را به زبان صحیح در سرور رندر کنیم.

### مقداردهی اولیه در سرور {#h3_249}

سرور از طریق middleware که تازه پیکربندی کردیم به همهٔ فایل‌های ترجمه دسترسی دارد. middleware cookie زبان را می‌خواند و قبل از رندر، i18next را با زبان مناسب مقداردهی اولیه می‌کند.

در نقطهٔ ورود سرور، اپلیکیشن را با provider i18next دربر می‌گیریم تا همهٔ کامپوننت‌ها به ترجمه‌ها دسترسی داشته باشند:

```tsx
// src/app/entry.server.tsx

import { I18nextProvider } from 'react-i18next';
import { ServerRouter } from 'react-router';
import type { EntryContext } from 'react-router';

import { getInstance } from './middleware/i18next';

export default function handleRequest(
  request: Request,
  responseStatusCode: number,
  responseHeaders: Headers,
  routerContext: EntryContext,
  loadContext: RouterContextProvider,
) {
  // ...

  const { pipe, abort } = renderToPipeableStream(
    <I18nextProvider i18n={getInstance(loadContext)}>
      <ServerRouter context={routerContext} url={request.url} nonce={nonce} />
    </I18nextProvider>,
    {
      // ... 
    },
  );

  // ... 
}
```

`I18nextProvider` نمونهٔ i18next را در اختیار همهٔ کامپوننت‌های درخت قرار می‌دهد، مشابه نحوهٔ کار React Context.

همچنین باید attributeهای `lang` و `dir` را روی عنصر HTML در حین رندر سرور تنظیم کنیم. این attributeها مهم‌اند چون مرورگرها، موتورهای جستجو و screen readerها همه به آن‌ها تکیه می‌کنند. attribute `lang` به screen readerها می‌گوید چگونه متن را تلفظ کنند، به موتورهای جستجو کمک می‌کند محتوا را به کاربران صحیح نشان دهند و مرورگرها را دربارهٔ گزینه‌های ترجمه آگاه می‌کند. attribute dir جهت متن را برای زبان‌های راست به چپ مثل عربی و عبری کنترل می‌کند.

هر دو attribute را در کامپوننت layout ریشه تنظیم می‌کنیم که در سرور رندر می‌شود:

```tsx
// src/app/root.tsx

import { useTranslation } from 'react-i18next';
import { useLoaderData } from 'react-router';

import { i18nextMiddleware } from '@/app/middleware/i18next';

export const middleware = [nonceMiddleware, userMiddleware, i18nextMiddleware];

export async function loader({ context }: Route.LoaderArgs) {
  // ...
}

export function Layout({ children }: { children: React.ReactNode }) {
  const { nonce } = useLoaderData<typeof loader>();
  const { i18n } = useTranslation();

  return (
    <html lang={i18n.language} dir={i18n.dir(i18n.language)}>
      {/* ... */}
    </html>
  );
}
```

بخش‌های کلیدی اینجا عبارت‌اند از:

- آرایهٔ `middleware` — شامل `i18nextMiddleware` است تا در هر درخواست اجرا شود و قبل از رندر i18n را راه‌اندازی کند.
- کامپوننت `Layout` — نمونهٔ i18n را از `useTranslation()` دریافت می‌کند و از آن برای تنظیم `i18n.language` روی attribute `lang` و `i18n.dir()` برای attribute `dir` استفاده می‌کند. middleware قبلاً i18next را با زبان صحیح از cookie مقداردهی اولیه کرده.

روش `i18n.dir()` برای زبان‌های چپ‌به‌راست مثل انگلیسی `'ltr'` و برای زبان‌های راست‌به‌چپ مثل عربی `'rtl'` برمی‌گرداند. وقتی `dir="rtl"` تنظیم می‌کنیم، مرورگر بیشتر تغییرات layout را خودکار انجام می‌دهد. CSS دیگر نیازی به نوشتن rules اختصاصی برای rtl ندارد — مرورگر این کار را برای padding، margin، flexbox alignment و جهت متن انجام می‌دهد. بعضی چیزهای ظریف وجود دارد (آیکون‌ها، تصاویر جهت‌دار) اما مرورگر کار سنگین را انجام می‌دهد.

با پیکربندی سرور، middleware در هر درخواست اجرا می‌شود، زبان را از cookie تشخیص می‌دهد و HTML کاملاً ترجمه‌شده با attributeهای lang و `dir` صحیح رندر می‌کند.

### مقداردهی اولیه در کلاینت {#h3_250}

حالا که سرور صفحه را با زبان صحیح رندر کرده و attribute `lang` را تنظیم کرده، client باید i18next را مطابق آن مقداردهی اولیه کند. client زبان را از attribute `lang` می‌خواند و از pluginها برای بارگذاری ترجمه‌های اضافی on-demand استفاده می‌کند.

همان‌طور که در مرور بحث کردیم، از partial bundling استفاده می‌کنیم: فقط namespaceهای `common` و `navigation` (در هر صفحه استفاده می‌شوند) را bundle می‌کنیم و namespaceهای دیگر را on-demand دریافت می‌کنیم. نحوهٔ پیکربندی این به این صورت است:

```tsx
// src/app/entry.client.tsx

import i18next from 'i18next';
import I18nextBrowserLanguageDetector from 'i18next-browser-languagedetector';
import Fetch from 'i18next-fetch-backend';
import { startTransition, StrictMode } from 'react';
import { hydrateRoot } from 'react-dom/client';
import { I18nextProvider, initReactI18next } from 'react-i18next';
import { HydratedRouter } from 'react-router/dom';

import { i18nConfig } from '@/config/i18n';

import { resources } from './locales';

async function main() {
  await i18next
    .use(initReactI18next)
    .use(Fetch)
    .use(I18nextBrowserLanguageDetector)
    .init({
      defaultNS: i18nConfig.defaultNS, // common
      partialBundledLanguages: true,
      resources: {
        en: {
          common: resources.en.common,
          navigation: resources.en.navigation,
        },
        es: {
          common: resources.es.common,
          navigation: resources.es.navigation,
        },
      },
      ns: ['common', 'navigation'],
      fallbackLng: i18nConfig.fallbackLng, // en
      detection: i18nConfig.detection, // order: ['htmlTag'], caches: []
      backend: i18nConfig.backend, // loadPath: '/api/locales/{{lng}}/{{ns}}'
    });

  startTransition(() => {
    hydrateRoot(
      document,
      <I18nextProvider i18n={i18next}>
        <StrictMode>
          <HydratedRouter />
        </StrictMode>
      </I18nextProvider>,
    );
  });
}

main().catch((error) => console.error(error));
```

بیایید بررسی کنیم چه اتفاقی می‌افتد:

- `use(initReactI18next)` — i18next را به React متصل می‌کند تا hook `useTranslation()` کار کند.
- `use(Fetch)` — بارگذاری ترجمه‌ها از API را در صورت نیاز فعال می‌کند.
- `use(I18nextBrowserLanguageDetector)` — مکانیزم تشخیص را فراهم می‌کند. پیکربندی می‌کنیم از attribute `lang` HTML بخواند.
- `partialBundledLanguages: true` — به i18next می‌گوید بعضی ترجمه‌ها bundle شده‌اند و بقیه بعداً بارگذاری می‌شوند. بدون این flag، i18next فکر می‌کرد هر namespaceای که در bundle نیست گم شده.
- `resources` — فقط namespaceهای `common` و `navigation` را شامل می‌شود. این‌ها در هر صفحه ظاهر می‌شوند، پس آن‌ها را bundle می‌کنیم. namespaceهای دیگر در صورت نیاز بارگذاری می‌شوند.
- `ns: ['common', 'navigation']` — این namespaceها را در شروع preload می‌کند.
- `fallbackLng` — زبانی که اگر زبان درخواستی در دسترس نباشد استفاده می‌شود. روی `'en'` به‌عنوان پیش‌فرض تنظیم شده.
- `detection` — نحوهٔ تشخیص زبان کاربر را پیکربندی می‌کند. از attribute `lang` HTML (تنظیم‌شده توسط سرور) می‌خوانیم و نتیجهٔ تشخیص را cache نمی‌کنیم چون به cookie تنظیم‌شده توسط سرور تکیه داریم.
- `backend` — نحوهٔ دریافت namespaceهایی که bundle نشده‌اند را پیکربندی می‌کند. به endpoint API ما در `/api/locales/{{lng}}/{{ns}}` اشاره می‌کند.

i18next را قبل از hydrate React مقداردهی اولیه می‌کنیم تا مطمئن شویم ترجمه‌ها وقتی کامپوننت‌ها رندر می‌شوند آماده‌اند. اما قبل از آن، باید endpoint API ایجاد کنیم که ترجمه‌ها را on-demand سرو کند.

## سرو ترجمه‌ها از طریق API {#h2_251}

برای اینکه بارگذاری پویا کار کند، به endpoint API نیاز داریم که ترجمه‌ها را on-demand سرو کند:

```typescript
// src/app/routes/api/locales.ts

import { cacheHeader } from 'pretty-cache-header';
import { data } from 'react-router';
import { z } from 'zod';

import { resources } from '@/app/locales';
import type { Language } from '@/config/i18n';

import type { Route } from './+types/locales';

export async function loader({ params }: Route.LoaderArgs) {
  const lng = z
    .enum(Object.keys(resources) as Array<Language>)
    .safeParse(params.lng);

  if (lng.error) return data({ error: lng.error }, { status: 400 });

  const namespaces = resources[lng.data];

  const ns = z
    .enum(Object.keys(namespaces) as Array<keyof typeof namespaces>)
    .safeParse(params.ns);

  if (ns.error) return data({ error: ns.error }, { status: 400 });

  const headers = new Headers();

  if (process.env.NODE_ENV === 'production') {
    headers.set(
      'Cache-Control',
      cacheHeader({
        maxAge: '5m',
        sMaxage: '1d',
        staleWhileRevalidate: '7d',
        staleIfError: '7d',
      }),
    );
  }

  return data(namespaces[ns.data], { headers });
}
```

این endpoint با الگوی URL `/api/locales/{{lng}}/{{ns}}` که قبلاً پیکربندی کردیم مطابقت دارد. هر دو پارامتر را با Zod اعتبارسنجی می‌کند و ترجمه‌های درخواستی را به فرمت JSON برمی‌گرداند.

از آنجا که ترجمه‌ها زیاد تغییر نمی‌کنند، آن‌ها را به‌شدت cache می‌کنیم. این یعنی کاربران به‌ندرت بعد از بارگذاری اول منتظر درخواست ترجمه می‌مانند.

## اضافه کردن type safety به کلیدهای ترجمه {#h2_252}

کلیدهای ترجمه رشته‌اند، یعنی TypeScript به‌طور پیش‌فرض نمی‌تواند غلط‌های تایپی را تشخیص دهد. اگر `t('welcom')` به‌جای `t('welcome')` بنویسیم، تا زمانی که ترجمهٔ خراب را در مرورگر نبینیم متوجه نمی‌شویم.

می‌توانیم این را با اطلاع دادن به TypeScript دربارهٔ ساختار ترجمه‌ها حل کنیم:

```typescript
// src/app/types/i18next.d.ts

import 'i18next';

import type { resources } from '@/app/locales';
import { i18nConfig } from '@/config/i18n';

declare module 'i18next' {
  interface CustomTypeOptions {
    defaultNS: typeof i18nConfig.defaultNS;
    resources: typeof resources.en;
  }
}
```

این به TypeScript می‌گوید ترجمه‌های ما از ساختار `resources.en` پیروی می‌کنند. حالا وقتی `t('` تایپ می‌کنیم، ویرایشگر همهٔ کلیدهای موجود را نشان می‌دهد. غلط‌های تایپی در compile time با خط قرمز مشخص می‌شوند، نه در runtime وقتی کاربر باگ را گزارش می‌کند.

از ترجمه‌های انگلیسی به‌عنوان source of truth استفاده می‌کنیم. اگر کلیدی در انگلیسی وجود داشته باشد، TypeScript انتظارش را دارد. اگر به کلیدی ارجاع دهیم که وجود ندارد، TypeScript به ما هشدار می‌دهد. همچنین این تضمین می‌کند ترجمه‌های گمشده در زبان‌های دیگر نداشته باشیم؛ وگرنه TypeScript به ما هشدار می‌دهد.

با پیکربندی کامل سیستم i18n، آماده‌ایم از ترجمه‌ها در کامپوننت‌های اپلیکیشن استفاده کنیم.

## استفاده از ترجمه‌ها در اپلیکیشن {#h1_253}

react-i18next hook `useTranslation` را برای دسترسی به ترجمه‌ها فراهم می‌کند. این نحوهٔ جایگزینی رشته‌های hard-coded با متن ترجمه‌شده است.

بیایید مثال ساده‌ای از صفحهٔ اصلی ببینیم:

```tsx
// src/app/routes/home.tsx

import { useTranslation } from 'react-i18next';
import { Link } from 'react-router';

import { Button } from '@/components/ui/button';

export default function HomePage() {
  const { t } = useTranslation(['home']);

  return (
    <div className="container mx-auto px-4 py-16">
      <Seo
        title={t('meta.title')}
        description={t('meta.description')}
      />
      <div className="text-center mb-16">
        <h1 className="text-4xl md:text-6xl font-bold mb-6">
          {t('title')}
        </h1>
        <p className="text-xl text-muted-foreground mb-8 max-w-2xl mx-auto">
          {t('subtitle')}
        </p>
        {/* ... */}
      </div>
      {/* ... */}
    </div>
  );
}
```

`['home']` را به `useTranslation` پاس می‌دهیم تا بگوییم کدام namespace بارگذاری شود. hook تابع t را برمی‌گرداند که برای دریافت ترجمه‌ها استفاده می‌کنیم.

وقتی `t('home:title')` فراخوانی می‌کنیم، از namespace `home` کلید `title` را درخواست می‌دهیم. دونقطه namespace را از کلید جدا می‌کند. برای ترجمه‌های تو در تو مثل `t('home:meta.title')`، از dot notation برای ورود به آبجکت `meta` استفاده می‌کنیم.

### استفاده از namespace پیش‌فرض {#h2_254}

از آنجا که `common` را به‌عنوان namespace پیش‌فرض پیکربندی کردیم، می‌توانیم بدون پیشوند namespace به ترجمه‌هایش دسترسی داشته باشیم:

```tsx
import { useTranslation } from 'react-i18next';

function DeleteButton() {
  const { t } = useTranslation();

  return <button>{t('delete')}</button>;
}
```

اینجا، `t('delete')` خودکار کلید `delete` را در namespace `common` پیدا می‌کند. این برای ترجمه‌هایی که زیاد استفاده می‌کنیم تایپ کردن را کم می‌کند.

### interpolation {#h2_255}

ترجمه‌های استاتیک فقط تا حدی کارآمدند. اپلیکیشن‌های واقعی نیاز دارند مقادیر پویا مثل نام کاربران و تعدادها را درج کنند.

interpolation اجازه می‌دهد متغیرها را در رشته‌های ترجمه درج کنیم. از brace دوتایی به‌عنوان placeholder استفاده می‌کنیم:

```typescript
// src/app/locales/en/common.ts

export default {
  welcome: 'Welcome, {{username}}!',
  // ...
};
```

برای استفاده از آن، شیء‌ای با مقادیر متغیر پاس می‌دهیم:

```tsx
function WelcomeMessage({ username }: { username: string }) {
  const { t } = useTranslation();

  return <h1>{t('welcome', { username })}</h1>;
}
```

وقتی با `username="John"` رندر شود، این `"Welcome, John!"` را نمایش می‌دهد. placeholder `{{username}}` با هر مقداری که پاس دهیم جایگزین می‌شود.

این قدرتمند است چون مترجمان می‌توانند متغیر را در هر جای جمله قرار دهند. در بعضی زبان‌ها، احتمالاً ساختار سلام متفاوت است، مثل `"John, welcome!"` مترجم ترتیب کلمات را کنترل می‌کند و توسعه‌دهنده فقط مقادیر را فراهم می‌کند.

### pluralization {#h2_256}

مقادیر مختلف اغلب به عبارت‌بندی متفاوت نیاز دارند. «1 idea» در برابر «2 ideas» در انگلیسی. react-i18next این را با پسوندهای `_one` و `_other` مدیریت می‌کند:

```typescript
// src/features/ideas/locales/en.ts

export default {
  ideasBy_one: '{{count}} Idea by {{username}}',
  ideasBy_other: '{{count}} Ideas by {{username}}',
  // ...
};
```

وقتی ترجمه را با متغیر `count` فراخوانی می‌کنیم، i18next خودکار فرم صحیح را انتخاب می‌کند:

```tsx
function UserIdeas({ count, username }: { count: number; username: string }) {
  const { t } = useTranslation(['ideas']);

  return <p>{t('ideas:ideasBy', { count, username })}</p>;
}
```

اگر `count` برابر ۱ باشد، i18next از `ideasBy_one` استفاده می‌کند و `1 Idea by John` را نشان می‌دهد. برای هر عدد دیگری از `ideasBy_other` استفاده می‌کند و `5 Ideas by John` را نشان می‌دهد.

برای اسپانیایی:

```typescript
// src/features/ideas/locales/es.ts

export default {
  ideasBy_one: '{{count}} Idea de {{username}}',
  ideasBy_other: '{{count}} Ideas de {{username}}',
  // ...
};
```

کد یکسان می‌ماند؛ فقط فایل‌های ترجمه متفاوت‌اند.

### قالب‌بندی تاریخ‌ها {#h2_257}

تاریخ‌ها پیچیده‌اند چون مناطق مختلف آن‌ها را متفاوت قالب‌بندی می‌کنند. ۱۵ ژانویهٔ ۲۰۲۴ در آمریکا در بسیاری از کشورهای اروپایی به ۱۵/۰۱/۲۰۲۴ تبدیل می‌شود. حتی نام ماه‌ها تغییر می‌کند؛ «January» در انگلیسی در اسپانیایی «enero» است.

JavaScript پشتیبانی داخلی از این طریق API `Intl.DateTimeFormat` دارد:

```typescript
// src/lib/date.ts

export function formatDate(date: string | Date, locale: string = 'en'): string {
  const d = date instanceof Date ? date : new Date(date);

  if (isNaN(d.getTime())) {
    console.error(`Invalid date: "${date}"`);
    return '';
  }

  return new Intl.DateTimeFormat(locale, {
    year: 'numeric',
    month: 'short',
    day: 'numeric',
    timeZone: 'UTC',
  }).format(d);
}
```

این تابع تاریخ‌ها را بر اساس locale قالب‌بندی می‌کند. برای انگلیسی ممکن است `Jan 15, 2024` دریافت کنیم. برای اسپانیایی ممکن است `15 ene 2024` دریافت کنیم.

برای استفاده از آن در کامپوننت‌ها، زبان فعلی را از نمونهٔ i18n دریافت می‌کنیم:

```tsx
// src/features/ideas/components/idea-card.tsx

import { useTranslation } from 'react-i18next';

import { formatDate } from '@/lib/date';

export function IdeaCard({ idea }: IdeaCardProps) {
  const { i18n } = useTranslation();
  
  return (
    <div>
      <h2>{idea.title}</h2>
      <p>Created: {formatDate(idea.createdAt, i18n.language)}</p>
    </div>
  );
}
```

hook `useTranslation()` هم تابع t و هم نمونهٔ `i18n` را به ما می‌دهد. `i18n.language` را به تابع `formatDate()` پاس می‌دهیم و تاریخ‌ها با فرمت صحیح برای زبان کاربر نمایش داده می‌شوند.

## جابه‌جایی بین زبان‌ها در اپلیکیشن {#h1_258}

کاربران نیاز به راهی برای تغییر ترجیح زبان خود دارند. این هم به یک کامپوننت UI برای انتخاب زبان و هم به یک endpoint API برای ذخیرهٔ این انتخاب نیاز دارد. بیایید هر دو قطعه را بسازیم.

### کامپوننت مبدل زبان {#h2_259}

مبدل dropdown ای است که زبان‌های موجود را نشان می‌دهد و به کاربران اجازه می‌دهد یکی را انتخاب کنند:

```tsx
// src/components/language-switcher.tsx

import { Check, Globe } from 'lucide-react';
import { useTranslation } from 'react-i18next';

import { Button } from '@/components/ui/button';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { i18nConfig, languages, type Language } from '@/config/i18n';
import { useNotificationActions } from '@/stores/notifications';

export function LanguageSwitcher() {
  const { t, i18n } = useTranslation(['components', 'common']);
  const { showNotification } = useNotificationActions();

  const handleLanguageChange = async (language: Language) => {
    const formData = new FormData();
    formData.append(i18nConfig.cookieName, language);

    try {
      const response = await fetch('/api/set-language', {
        method: 'POST',
        body: formData,
      });

      if (response.ok) {
        await i18n.changeLanguage(language);
        showNotification({
          type: 'success',
          title: t('common:languageChanged', {
            lng: language,
          }),
        });
      }
    } catch (error) {
      console.error('Failed to change language:', error);
    }
  };

  return (
    <DropdownMenu>
      <DropdownMenuTrigger
        render={
          <Button variant="ghost" size="sm" className="h-8 w-8 p-0">
            <Globe className="h-4 w-4" />
            <span className="sr-only">
              {t('components:languageSwitcher.switchLanguage')}
            </span>
          </Button>
        }
      />
      <DropdownMenuContent align="end">
        {Object.entries(languages).map(([key, value]) => (
          <DropdownMenuItem
            key={key}
            onClick={() => handleLanguageChange(key as Language)}
            className="cursor-pointer"
          >
            <span className="flex items-center gap-2">
              {i18n.language === key && <Check className="h-4 w-4" />}
              <span className={i18n.language === key ? '' : 'ml-6'}>
                {value}
              </span>
            </span>
          </DropdownMenuItem>
        ))}
      </DropdownMenuContent>
    </DropdownMenu>
  );
}
```

تغییر زبان در دو مرحله اتفاق می‌افتد. اول، به سرور می‌گوییم cookie را به‌روزرسانی کند تا ترجیح ذخیره شود. سپس نمونهٔ i18n سمت client را به‌روزرسانی می‌کنیم تا UI فوراً به‌روز شود. بدون هر دو مرحله، تغییر یا در reloadهای صفحه باقی نمی‌ماند یا تا refresh نامرئی می‌ماند.

### endpoint API {#h2_260}

برای اینکه کامپوننت کار کند، به endpoint API نیاز داریم که cookie زبان را تنظیم کند:

```typescript
// src/app/routes/api/set-language.ts

import { data } from 'react-router';
import z from 'zod';

import { localeCookie } from '@/app/middleware/i18next';
import { i18nConfig, languages, type Language } from '@/config/i18n';

import type { Route } from './+types/set-language';

const languageSchema = z.enum(
  Object.keys(languages) as [Language, ...Language[]],
);

export async function action({ request }: Route.ActionArgs) {
  const formData = await request.formData();

  const language = languageSchema.safeParse(
    formData.get(i18nConfig.cookieName),
  );

  if (!language.success) {
    return data({ success: false }, { status: 400 });
  }

  return data(
    { success: true },
    {
      headers: {
        'Set-Cookie': await localeCookie.serialize(language.data),
      },
    },
  );
```

این endpoint اعتبارسنجی می‌کند که زبان درخواستی یکی از زبان‌های پشتیبانی‌شده باشد، سپس cookie را تنظیم می‌کند. دفعهٔ بعدی که کاربر صفحه‌ای را بارگذاری کند، سرور ترجیحش را از این cookie تشخیص می‌دهد و صفحه را به زبان صحیح رندر می‌کند.

## خلاصه {#h1_261}

در این فصل، سیستم بین‌المللی‌سازی کاملی برای اپلیکیشن ساختیم. با درک اینکه چرا جداسازی متن از کد ترجمه‌ها را مدیریت‌پذیر می‌کند شروع کردیم.

ترجمه‌ها را با namespace سازمان‌دهی کردیم تا فایل‌ها کوچک و متمرکز بمانند. namespaceهای سطح اپلیکیشن مثل `common` شامل ترجمه‌های مشترک هستند، در حالی که ترجمه‌های feature-scoped کنار کد feature خودشان قرار دارند. این سازمان‌دهی پیدا کردن و نگهداری ترجمه‌ها را آسان می‌کند.

react-i18next را با مقداردهی اولیهٔ سرور و کلاینت راه‌اندازی کردیم. سرور صفحات را با ترجمه‌های صحیح رندر می‌کند و کلاینت روان hydrate می‌شود. endpoint API برای سرو ترجمه‌ها on-demand ایجاد کردیم که partial bundling را فعال می‌کند و bundle اولیهٔ JavaScript را کوچک نگه می‌دارد.

type safety خطاهای ترجمه را زودگیر می‌کند. با اطلاع دادن به TypeScript دربارهٔ ساختار ترجمه‌ها، autocomplete و بررسی compile time برای کلیدهای ترجمه دریافت می‌کنیم. همچنین attributeهای زبان و جهت را پیکربندی کردیم تا مرورگرها، screen readerها و موتورهای جستجو محتوای ما را بفهمند.

یاد گرفتیم چگونه از hook `useTranslation` برای دسترسی به ترجمه‌ها در کامپوننت‌ها استفاده کنیم و interpolation و pluralization چگونه محتوای پویا را مدیریت می‌کنند. قالب‌بندی تاریخ localized را با API داخلی `Intl` مرورگر پیاده‌سازی کردیم.

در نهایت، مبدل زبانی ساختیم که انتخاب کاربر را در cookie ذخیره می‌کند. ترجیح در نشست‌های مختلف اعمال می‌شود و تجربهٔ یکسانی به کاربران می‌دهد.

با پیاده‌سازی بین‌المللی‌سازی، اپلیکیشن ما می‌تواند به کاربران سراسر جهان به زبان‌های ترجیحیشان سرویس دهد.
