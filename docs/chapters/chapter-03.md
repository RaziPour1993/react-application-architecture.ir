# فصل ۳: ساخت و مستندسازی کامپوننت‌ها

در [React](https://react.dev/)، همه‌چیز یک **component** است. این paradigma به ما اجازه می‌دهد رابط کاربری را به بخش‌های کوچک‌تر تقسیم کنیم و توسعهٔ رابط کاربری و اپلیکیشن‌های بزرگ‌تر را ساده‌تر کنیم. همچنین با فعال کردن قابلیت استفادهٔ مجدد از componentها، از اصل DRY (خودتان را تکرار نکنید) پیروی می‌کند؛ چون می‌توانیم همان componentها را در چندین نقطه استفاده کنیم.

در این فصل یاد می‌گیریم چگونه componentهای بنیادی برای design system اپلیکیشنمان بسازیم. این کار باعث می‌شود UI اپلیکیشن یکپارچه‌تر و درک و نگهداری آن آسان‌تر شود. همچنین یاد می‌گیریم چگونه این componentها را با [**Storybook**](https://storybook.js.org/) مستند کنیم — ابزاری عالی که به‌عنوان کاتالوگی برای تمام elementهای UI قابل استفادهٔ مجدد ما عمل می‌کند.

موارد زیر را پوشش خواهیم داد:

- آناتومی یک component
- ساخت کتابخانهٔ component
- مستندسازی componentها

در پایان این فصل، درک عمیقی از معماری component و کتابخانه‌ای از componentهای قابل استفادهٔ مجدد خواهیم داشت که با Storybook مستند شده و در سراسر اپلیکیشن قابل استفاده است.

## پیش‌نیازهای فنی {#h1_108}

پیش از شروع، باید پروژه را راه‌اندازی کنیم. برای توسعهٔ پروژه به ابزارهای زیر روی کامپیوتر خود نیاز دارید:

- [**Node.js**](https://nodejs.org/) نسخهٔ ۲۴ یا بالاتر. نسخهٔ [**npm**](https://www.npmjs.com/) ۱۱ یا بالاتر همراه Node ارائه می‌شود. می‌توانید با اجرای `node -v` و `npm -v` در terminal این را تأیید کنید. راه‌های مختلفی برای نصب Node.js و npm وجود دارد. این مقالهٔ مفید را ببینید: [https://www.nodejsdesignpatterns.com/blog/5-ways-to-install-node-js](https://www.nodejsdesignpatterns.com/blog/5-ways-to-install-node-js).
- [**VS Code**](https://code.visualstudio.com/) (اختیاری)، یک ویرایشگر محبوب برای [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) و [TypeScript](https://www.typescriptlang.org/). نرم‌افزاری open source با پشتیبانی قوی از TypeScript و انبوهی از extensionهاست. می‌توانید آن را از [https://code.visualstudio.com](https://code.visualstudio.com) دانلود کنید.

کد این کتاب در مخزن (repository) کتاب موجود است. برای دسترسی به لینک مخزن، مراحل بخش «دانلود فایل‌های کد نمونه» در پیش‌گفتار را دنبال کنید. آن را clone کنید و وارد ریشهٔ مخزن شوید:

```bash
git clone https://github.com/PacktPublishing/React-Application-Architecture-for-Production-Second-Edition.git
```

مخزن شامل پوشه‌های فصل با کد هر فصل است، همراه با پوشهٔ مشترک `api` که سرور API مورد استفاده در تمام فصل‌ها را در بر می‌گیرد.

ما روی فصل ۳ کار می‌کنیم، بنابراین وارد پوشهٔ `chapter-03` شوید:

```bash
cd React-Application-Architecture-for-Production-Second-Edition/chapter-03
```

سپس وابستگی‌ها (dependencies) را نصب کنید:

```bash
npm install
```

همچنین باید متغیرهای محیطی (environment variables) را تنظیم کنید:

```bash
cp .env.example .env
```

در این مرحله، frontend باید آماده باشد و روی [http://localhost:5173](http://localhost:5173) اجرا شود.

حالا کد پروژه باید آماده باشد.

برای اطلاعات بیشتر دربارهٔ جزئیات راه‌اندازی، فایل `README.md` را مشاهده کنید.

## آناتومی یک component {#h1_109}

پیش از شروع ساخت کتابخانهٔ component، باید بفهمیم component چیست و چگونه کار می‌کند. در این بخش موارد زیر را پوشش می‌دهیم:

- component چیست؟
- propهای component
- state component
- event handlerها
- TypeScript و component

بیایید هر یک از این مفاهیم را با یک مثال عملی بررسی کنیم.

### component چیست؟ {#h2_110}

component تکه‌ای خودبسنده و قابل استفادهٔ مجدد از UI است که ساختار، رفتار و styling خود را در بر می‌گیرد. componentها با هم ترکیب می‌شوند تا رابط کاربری کاملی را بسازند.

در اینجا یک مثال ساده آمده است:

```tsx
// src/components/counter.tsx
import { useState } from 'react';

export type CounterProps = {
  initialValue: number;
  label: string;
  onIncrement?: (newValue: number) => void;
  onDecrement?: (newValue: number) => void;
};

export function Counter(props: CounterProps) {
  const [value, setValue] = useState(props.initialValue);

  const handleIncrement = () => {
    const newValue = value + 1;
    setValue(newValue);
    props.onIncrement?.(newValue);
  };

  const handleDecrement = () => {
    const newValue = value - 1;
    setValue(newValue);
    props.onDecrement?.(newValue);
  };

  return (
    <div>
      <label>{props.label}</label>
      <div className="flex items-center gap-2">
        <button onClick={handleDecrement}>-</button>
        <p>{value}</p>
        <button onClick={handleIncrement}>+</button>
      </div>
    </div>
  );
}
```

این component بخش‌های کلیدی یک component در React را نشان می‌دهد:

- **تابع component**: در هستهٔ خود، یک component فقط یک تابع JavaScript است که JSX (JavaScript XML) برمی‌گرداند؛ چیزی که شبیه HTML به نظر می‌رسد اما در واقع JavaScript است. این تابع توصیف می‌کند چه چیزی باید روی صفحه نمایش داده شود. در پشت صحنه، کامپایلر JSX markup را به فراخوانی‌های سادهٔ `React.createElement` تبدیل می‌کند؛ بنابراین `<button onClick={fn}>Click</button>` تبدیل می‌شود به: `React.createElement('button', { onClick: fn }, 'Click')`.
- **Props (ویژگی‌ها)**: پارامتر `props` روش ارسال داده به componentهاست. Props می‌تواند هر چیزی باشد — string، number، function، object، array یا حتی componentهای دیگر. Props به componentها پویایی و قابلیت استفادهٔ مجدد می‌بخشد.
- **TypeScript type**: تعریف type `CounterProps` را ببینید. TypeScript از ارسال string در جایی که عدد نیاز داریم یا فراموش کردن propهای ضروری جلوگیری می‌کند. همچنین IntelliSense عالی در اختیار IDE ما قرار می‌دهد.
- **State**: هوک `useState` به component اجازه می‌دهد مقدارها را بین renderها به یاد بسپارد. وقتی `setValue` را فراخوانی می‌کنیم، React component را با مقدار جدید دوباره render می‌کند و آنچه کاربران روی صفحه می‌بینند به‌روز می‌شود.
- **Event handler**: توابعی مانند `handleIncrement` به تعاملات کاربر پاسخ می‌دهند. وقتی کسی روی دکمهٔ افزایش کلیک می‌کند، state را به‌روز کرده و در صورت نیاز callback prop را فراخوانی می‌کنیم.

### استفاده از component {#h2_111}

وقتی componentی را تعریف کردیم، می‌توانیم آن را در هر نقطه‌ای از اپلیکیشن استفاده کنیم:

```tsx
// src/app/routes/home.tsx
import { Counter } from '@/components/counter';

export default function HomePage() {
  return (
    <div>
      <h1>Home</h1>
      <Counter
        initialValue={0}
        label="Click Counter"
        onIncrement={(newValue) => {
          console.log(`Incremented to ${newValue}`);
        }}
        onDecrement={(newValue) => {
          console.log(`Decremented to ${newValue}`);
        }}
      />
    </div>
  );
}
```

هنگام render شدن، کاربران برچسب **Click Counter**، مقدار فعلی شمارنده و دو دکمه را مشاهده می‌کنند. کلیک روی دکمه‌ها عدد نمایش‌داده‌شده را به‌روز کرده و پیام‌ها را در console ثبت می‌کند.

می‌توانیم همان component `Counter` را چندین بار در یک صفحه با propهای مختلف `initialValue` و `label` استفاده کنیم و هر کدام به‌صورت مستقل عمل خواهند کرد.

حالا که مبانی component را فهمیدیم، بیایید کتابخانه‌ای از componentها بسازیم که پایهٔ اپلیکیشن ما را تشکیل دهد.

## ساخت کتابخانهٔ component {#h1_112}

در توسعهٔ یک اپلیکیشن production، به مجموعه‌ای یکپارچه از componentهای UI نیاز داریم — دکمه‌ها، inputها، dialogها، فرم‌ها و موارد دیگر. چندین رویکرد برای تهیهٔ این componentها وجود دارد. در این بخش موارد زیر را پوشش می‌دهیم:

- کتابخانهٔ component چیست و چرا به آن نیاز داریم؟
- رویکردهای مختلف به کتابخانهٔ component
- [Shadcn UI](https://ui.shadcn.com/) چیست و چرا آن را انتخاب کردیم؟
- نحوهٔ راه‌اندازی Shadcn UI

بیایید با درک اینکه کتابخانهٔ component چیست و چرا به آن نیاز داریم شروع کنیم.

### کتابخانهٔ component چیست و چرا به آن نیاز داریم؟ {#h2_113}

بدون کتابخانهٔ component، هر بار که به دکمه، input یا modal نیاز داشته باشیم باید آن را از صفر بسازیم. این موضوع منجر به موارد زیر می‌شود:

- **UI ناهمگون**: دکمه‌های مختلف با استایل‌های متفاوت در سراسر اپلیکیشن
- **تکرار کد**: ساخت همان componentها به‌صورت چندباره
- **نگهداری دشوارتر**: نیاز به به‌روزرسانی همان component در چندین نقطه
- **مشکلات accessibility**: فراموش کردن آسان keyboard navigation، screen reader و attributeهای ARIA
- **توسعهٔ کندتر**: ساخت همه‌چیز از صفر زمان‌بر است

کتابخانهٔ component این مشکلات را با فراهم کردن مجموعه‌ای از componentهای از پیش ساخته‌شده، tested و accessible حل می‌کند که می‌توانیم در سراسر اپلیکیشن از آن‌ها استفاده کنیم.

### رویکردهای مختلف به کتابخانهٔ component {#h2_114}

چندین روش برای تهیهٔ کتابخانهٔ component در اپلیکیشن React ما وجود دارد:

### گزینه ۱: نصب یک بسته (Material-UI، Ant Design، Chakra UI، Mantine) {#h3_115}

یک کتابخانه را از طریق `npm` نصب کرده و componentها را import می‌کنیم:

```tsx
import Button from '@mui/material/Button';

<Button variant="contained">Click me</Button>
```

#### مزایا: {#h4_116}

- شروع سریع
- تست و نگهداری خوب
- مستندات و جامعهٔ فعال

#### معایب: {#h4_117}

- حجم bundle بزرگ (حتی اگر فقط از چند component استفاده کنیم، کل کتابخانه ارسال می‌شود)
- سفارشی‌سازی محدود (پیروی از design system کتابخانه)
- به‌روزرسانی‌ها ممکن است مشکلاتی ایجاد کنند

در سمت دیگر طیف، گزینهٔ ساخت همه‌چیز توسط خودمان قرار دارد.

### گزینه ۲: ساخت از صفر {#h3_118}

تمام componentها را خودمان از elementهای HTML پایه می‌سازیم.

#### مزایا: {#h4_119}

- کنترل کامل روی همه‌چیز
- وابستگی اضافی نداریم
- دقیقاً همان چیزی که نیاز داریم

#### معایب: {#h4_120}

- ساخت زمان‌بر
- باید accessibility را خودمان مدیریت کنیم
- باید همه‌چیز را تست کنیم
- اختراع مجدد چرخ

گزینهٔ میانه‌ای نیز وجود دارد که مزایای هر دو رویکرد را ترکیب می‌کند.

### گزینه ۳: Shadcn UI (componentهای copy-paste) {#h3_121}

Shadcn UI رویکرد متفاوتی دارد — این یک بستهٔ npm نیست. در عوض، مجموعه‌ای از componentهای accessible است که مستقیماً در پروژهٔ ما کپی می‌شوند. این componentها با [Radix UI](https://www.radix-ui.com/) یا [Base UI](https://mui.com/base-ui/) (کتابخانه‌های component headless برای accessibility و رفتار) ساخته شده و با [Tailwind CSS](https://tailwindcss.com/) استایل‌دهی می‌شوند. ما برای پروژهٔ خود از Base UI استفاده می‌کنیم، زیرا در زمان نگارش این کتاب نگهداری فعال‌تری دارد — موضوعی که همیشه باید هنگام انتخاب کتابخانهٔ component در نظر گرفته شود.

#### مزایا: {#h4_122}

- componentها در codebase ما قرار می‌گیرند (کنترل و سفارشی‌سازی کامل)
- وابستگی runtime برای مدیریت نداریم
- حجم bundle کوچک‌تر (فقط آنچه استفاده می‌کنیم نصب و ارسال می‌شود)
- بر پایهٔ Base UI برای functionality و accessibility ساخته شده
- به‌سادگی قابل تغییر و گسترش

#### معایب: {#h4_123}

- اگر Shadcn UI بهبودهایی منتشر کند یا به‌روزرسانی‌هایی در Base UI یا Tailwind اتفاق بیفتد، باید componentها را به‌صورت دستی به‌روز کنیم
- نیاز به راه‌اندازی اولیه دارد

برای این پروژه از Shadcn UI استفاده خواهیم کرد. بیایید ببینیم چرا این انتخاب با نیازهای ما همخوانی دارد.

### چرا Shadcn UI را انتخاب کردیم {#h2_124}

برای این پروژه، Shadcn UI را به دلایل زیر انتخاب کردیم:

- **مالکیت**: componentها بخشی از codebase ما می‌شوند؛ بنابراین می‌توانیم آن‌ها را هر طور که نیاز داریم سفارشی‌سازی کنیم، بدون نگرانی از محدودیت‌های کتابخانه یا تغییرات مخرب (breaking changes) از upstream
- **stack مدرن**: از Tailwind CSS برای styling استفاده می‌کند که با شیوه‌های مدرن توسعهٔ React هماهنگ است و styling را intuitive و سریع می‌کند
- **Accessibility**: بر پایهٔ Base UI ساخته شده که نیازمندی‌های پیچیدهٔ accessibility (keyboard navigation، focus management، attributeهای ARIA) را مدیریت می‌کند — کاری که پیاده‌سازی صحیح آن توسط خودمان زمان قابل توجهی می‌برد
- **حجم Bundle**: فقط componentهایی را شامل می‌کنیم که واقعاً استفاده می‌کنیم و اپلیکیشن را سبک نگه می‌داریم
- **تجربهٔ توسعه‌دهنده**: CLI اضافه کردن componentها را ساده می‌کند و چون آن‌ها در codebase ما هستند، دقیقاً می‌توانیم ببینیم چگونه کار می‌کنند

### پیکربندی Shadcn UI {#h2_125}

componentهای Shadcn UI را می‌توان با CLI یا به‌صورت دستی به پروژه اضافه کرد. بیایید هر دو رویکرد را بررسی کنیم.

#### استفاده از CLI {#h3_126}

CLI Shadcn با اجرای دستور زیر، تمام فایل‌ها و پیکربندی‌های لازم را برای ما راه‌اندازی می‌کند:

```bash
npx shadcn@latest init --base=base --preset=vega
```

Base UI را به‌عنوان کتابخانهٔ component و Vega را به‌عنوان preset انتخاب می‌کنیم. این دستور موارد زیر را انجام می‌دهد:

- نصب وابستگی‌های لازم
- ایجاد فایل‌های پیکربندی
- راه‌اندازی styleهای Tailwind CSS
- پیکربندی path aliasها

CLI فایل `components.json` را تولید می‌کند که شامل پیکربندی تولید componentهاست:

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "base-vega",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "src/app/app.css",
    "baseColor": "neutral",
    "cssVariables": true,
    "prefix": ""
  },
  "iconLibrary": "lucide",
  "rtl": false,
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  },
  "menuColor": "default",
  "menuAccent": "subtle",
  "registries": {}
}
```

این پیکربندی چه کاری انجام می‌دهد:

- `style`: استایل طراحی مورد استفاده (`base-vega` برای استفاده از Base UI به‌عنوان کتابخانهٔ component با preset Vega)
- `tailwind.css`: مسیر فایل Tailwind CSS
- `aliases`: نگاشت مسیرهایی که با پیکربندی TypeScript ما مطابقت دارند — Shadcn UI باید از این پوشه‌ها آگاه باشد زیرا componentها و utilityهای لازم را در آن‌ها ذخیره می‌کند
- `iconLibrary`: کتابخانهٔ آیکون مورد استفاده (`lucide-react`)

#### اضافه کردن componentها {#h3_127}

حالا می‌توانیم با CLI component اضافه کنیم:

```bash
npx shadcn@latest add button
```

این دستور component را تولید و به codebase ما اضافه می‌کند. component تولیدشدهٔ button به شکل زیر است:

```tsx
// src/components/ui/button.tsx

import { Button as ButtonPrimitive } from '@base-ui/react/button';
import { cva, type VariantProps } from 'class-variance-authority';

import { cn } from '@/lib/utils';

const buttonVariants = cva(
  "group/button inline-flex shrink-0 items-center justify-center rounded-md border border-transparent bg-clip-padding text-sm font-medium whitespace-nowrap transition-all outline-none select-none focus-visible:border-ring focus-visible:ring-3 focus-visible:ring-ring/50 disabled:pointer-events-none disabled:opacity-50 aria-invalid:border-destructive aria-invalid:ring-3 aria-invalid:ring-destructive/20 dark:aria-invalid:border-destructive/50 dark:aria-invalid:ring-destructive/40 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/80',
        outline:
          'border-border bg-background shadow-xs hover:bg-muted hover:text-foreground aria-expanded:bg-muted aria-expanded:text-foreground dark:border-input dark:bg-input/30 dark:hover:bg-input/50',
        secondary:
          'bg-secondary text-secondary-foreground hover:bg-secondary/80 aria-expanded:bg-secondary aria-expanded:text-secondary-foreground',
        ghost:
          'hover:bg-muted hover:text-foreground aria-expanded:bg-muted aria-expanded:text-foreground dark:hover:bg-muted/50',
        destructive:
          'bg-destructive/10 text-destructive hover:bg-destructive/20 focus-visible:border-destructive/40 focus-visible:ring-destructive/20 dark:bg-destructive/20 dark:hover:bg-destructive/30 dark:focus-visible:ring-destructive/40',
        link: 'text-primary underline-offset-4 hover:underline',
      },
      size: {
        default:
          'h-9 gap-1.5 px-2.5 in-data-[slot=button-group]:rounded-md has-data-[icon=inline-end]:pr-2 has-data-[icon=inline-start]:pl-2',
        xs: "h-6 gap-1 rounded-[min(var(--radius-md),8px)] px-2 text-xs in-data-[slot=button-group]:rounded-md has-data-[icon=inline-end]:pr-1.5 has-data-[icon=inline-start]:pl-1.5 [&_svg:not([class*='size-'])]:size-3",
        sm: 'h-8 gap-1 rounded-[min(var(--radius-md),10px)] px-2.5 in-data-[slot=button-group]:rounded-md has-data-[icon=inline-end]:pr-1.5 has-data-[icon=inline-start]:pl-1.5',
        lg: 'h-10 gap-1.5 px-2.5 has-data-[icon=inline-end]:pr-3 has-data-[icon=inline-start]:pl-3',
        icon: 'size-9',
        'icon-xs':
          "size-6 rounded-[min(var(--radius-md),8px)] in-data-[slot=button-group]:rounded-md [&_svg:not([class*='size-'])]:size-3",
        'icon-sm':
          'size-8 rounded-[min(var(--radius-md),10px)] in-data-[slot=button-group]:rounded-md',
        'icon-lg': 'size-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  },
);

function Button({
  className,
  variant = 'default',
  size = 'default',
  ...props
}: ButtonPrimitive.Props&VariantProps<typeof buttonVariants>) {
  return (
    <ButtonPrimitive
      data-slot="button"
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  );
}

export { Button, buttonVariants };
```

ویژگی‌های کلیدی component تولیدشده:

- **Type-safe**: از TypeScript برای اعتبارسنجی propها استفاده می‌کند
- **Variants**: استایل‌های بصری مختلف (`default`، `destructive`، `outline`، `secondary`، `ghost`، `link`)
- **Sizes**: اندازه‌های مختلف (`default, sm, lg, icon`)
- قابل گسترش: می‌توانیم styleها را با prop className override کنیم
- **Accessible**: بر پایهٔ primitiveهای Base UI ساخته شده

می‌توانیم چندین component را یکجا با ارسال نام چندین component به دستور `add` تولید کنیم:

```bash
npx shadcn@latest add button card
```

### ساخت component به‌صورت دستی {#h2_128}

همچنین می‌توانیم با مراجعه به وبسایت Shadcn UI، componentها را به‌صورت دستی اضافه کنیم:

![Figure 3.1 – Installing Shadcn UI components manually](/images/B31385_3_1.png)

**Figure 3.1 — نصب دستی componentهای Shadcn UI**

فرایند دستی شامل مراحل زیر است:

- ایجاد فایل component در `src/components/ui/`
- کپی کد component از وبسایت به فایل component
- نصب وابستگی‌های لازم به‌صورت دستی

در حالی که CLI سریع‌تر است، رویکرد دستی کنترل بیشتری روی آنچه به پروژه اضافه می‌شود در اختیار ما قرار می‌دهد.

حالا که componentهایمان را داریم، به روشی برای مستند کردن و آزمایش آن‌ها نیاز داریم.

برای آخرین دستورالعمل‌های راه‌اندازی، همیشه مستندات رسمی Shadcn UI را در https://ui.shadcn.com/ مشاهده کنید.

## مستندسازی componentها {#h1_129}

component `Button` ما variantها و اندازه‌های مختلفی دارد. به روشی برای مستند کردن و تست کردن این حالت‌های مختلف نیاز داریم. در این بخش دو رویکرد را بررسی می‌کنیم:

- ایجاد یک route اختصاصی در اپلیکیشن
- استفاده از Storybook برای مستندسازی componentها

می‌توانیم از ساده‌ترین رویکرد شروع کنیم.

### ایجاد route component {#h2_130}

ساده‌ترین راه برای مشاهدهٔ componentها، ایجاد یک route اختصاصی است. می‌توانیم یک route برای components در اپلیکیشن ایجاد کنیم:

```tsx
// src/app/routes/components.tsx
import { Button } from '@/components/ui/button';

export default function Components() {
  return (
    <div className="container mx-auto py-8 space-y-12">
      <div>
        <h1 className="text-3xl font-bold mb-8">Components in Action</h1>
        <section className="space-y-6">
          <h2 className="text-2xl font-semibold">Button Component</h2>

          <div className="space-y-4">
            <h3 className="text-lg font-medium">Variants</h3>
            <div className="flex flex-wrap gap-4">
              <Button variant="default">Default</Button>
              <Button variant="destructive">Destructive</Button>
              <Button variant="outline">Outline</Button>
              <Button variant="secondary">Secondary</Button>
              <Button variant="ghost">Ghost</Button>
              <Button variant="link">Link</Button>
            </div>
          </div>

          <div className="space-y-4">
            <h3 className="text-lg font-medium">Sizes</h3>
            <div className="flex flex-wrap items-center gap-4">
              <Button size="sm">Small</Button>
              <Button size="default">Default</Button>
              <Button size="lg">Large</Button>
              <Button size="icon-sm">⌕</Button>
              <Button size="icon">⌕</Button>
              <Button size="icon-lg">⌕</Button>
            </div>
          </div>
        </section>
      </div>
    </div>
  );
}
```

این کد تمام variantهای button را render می‌کند:

![Figure 3.2 – Documenting components on a dedicated application route](/images/B31385_3_2.png)

**Figure 3.2 — مستندسازی componentها در یک route اختصاصی اپلیکیشن**

این رویکرد بدون نیاز به setup اضافی کار می‌کند، اما چندین مشکل دارد:

- **componentها را در اپلیکیشن expose می‌کند**: Routeهای اپلیکیشن باید روی featureها تمرکز کنند، نه مستندسازی component
- **نگهداری دشوار**: اضافه کردن هر component و variant به یک route، آن را شلوغ می‌کند
- **interactivity ندارد**: نمی‌توانیم به‌سادگی ترکیب‌های مختلف prop را تست کنیم
- **ایزولاسیون ندارد**: componentها به‌صورت جداگانه از اپلیکیشن develop نمی‌شوند

به راه‌حل بهتری نیاز داریم. اینجاست که Storybook وارد میدان می‌شود.

### Storybook چیست؟ {#h2_131}

Storybook ابزاری برای develop و test کردن componentهای UI به‌صورت ایزوله است. این ابزار به‌عنوان کاتالوگی از componentهای اپلیکیشن ما عمل می‌کند.

مزایای استفاده از Storybook:

- **ایزولاسیون component**: develop و test کردن componentها بدون اجرای کل اپلیکیشن
- **تست بصری**: مشاهدهٔ تمام variantهای یک component در یک مکان
- **بازیگاه تعاملی**: آزمایش با ترکیب‌های مختلف prop به‌صورت real-time
- **مستندات زنده**: storyها همراه با کد ما به‌روز می‌مانند
- **همکاری تیمی**: طراحان و product managerها می‌توانند componentها را بدون نیاز به setup فنی بررسی کنند
- **تست regression**: شناسایی باگهای بصری با مقایسهٔ screenshot از storyها. اگر می‌خواهید عمیق‌تر دربارهٔ تست regression با Storybook بدانید، مستندات آن‌ها را در [https://storybook.js.org/docs/writing-tests/visual-testing](https://storybook.js.org/docs/writing-tests/visual-testing) مشاهده کنید.

بیایید ببینیم چگونه Storybook را برای کار با پروژهٔ خود پیکربندی کنیم.

### پیکربندی Storybook {#h2_132}

می‌توانیم از Storybook CLI برای نصب و پیکربندی خودکار تمام موارد استفاده کنیم.

Storybook با دستور زیر در پروژه مقداردهی اولیه می‌شود:

```bash
npm create storybook@latest
```

این دستور موارد زیر را انجام می‌دهد:

- نصب Storybook و وابستگی‌هایش
- ایجاد پوشهٔ `.storybook` با فایل‌های پیکربندی
- اضافه کردن اسکریپت‌های Storybook به `package.json`
- تشخیص setup پروژه (React، Vite، TypeScript) و پیکربندی بر اساس آن

چند نکته هست که باید پیکربندی کنیم تا Storybook با پروژهٔ ما به درستی کار کند.

### پیکربندی Vite برای Storybook {#h2_133}

از آنجا که از React Router با Vite استفاده می‌کنیم، باید تنظیم کوچکی در پیکربندی Vite اعمال کنیم. پلاگین React Router فقط باید هنگام build کردن اپلیکیشن اجرا شود، نه هنگام اجرای Storybook.

باید فایل `vite.config.ts` را به‌روز کنیم:

```ts
// vite.config.ts
import { reactRouter } from "@react-router/dev/vite";
import tailwindcss from "@tailwindcss/vite";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";

const isStorybook = process.env.STORYBOOK === 'true';

export default defineConfig({
  plugins: [tailwindcss(), !isStorybook && reactRouter(), tsconfigPaths()],
});
```

تغییرات اعمال‌شده:

- `const isStorybook = process.env.STORYBOOK === 'true';`: تشخیص اینکه آیا Storybook در حال اجراست
- `!isStorybook && reactRouter()`: پلاگین React Router فقط وقتی بارگذاری می‌شود که Storybook اجرا نشده باشد. در Storybook به آن نیاز نداریم؛ به همین دلیل آن را غیرفعال می‌کنیم

این تنظیم از بروز conflict بین React Router و Storybook در setup Vite جلوگیری می‌کند.

#### پیکربندی preview Storybook {#h3_134}

فایل `.storybook/preview.ts` نحوهٔ نمایش storyهای ما را کنترل می‌کند. باید styleهای اپلیکیشن را import کنیم تا componentها در Storybook دقیقاً همان‌طور به نظر برسند که در اپلیکیشن هستند:

```ts
// .storybook/preview.ts
import type { Preview } from '@storybook/react-vite'
import '../src/app/app.css';

const preview: Preview = {
  parameters: {
    controls: {
      matchers: {
       color: /(background|color)$/i,
       date: /Date$/i,
      },
    },
  },
};

export default preview;
```

این پیکربندی چه کاری انجام می‌دهد:

- `import '../src/app/app.css'`: styleهای Tailwind CSS را import می‌کند تا componentها با styling صحیح render شوند
- `parameters.controls.matchers`: به Storybook می‌گوید color picker برای propهای رنگی و date picker برای propهای تاریخ فراهم کند

### اجرای Storybook {#h2_135}

CLI دو اسکریپت جدید به `package.json` اضافه کرده است:

```json
{
  "scripts": {
    "storybook": "STORYBOOK=true storybook dev -p 6006",
    "build-storybook": "STORYBOOK=true storybook build"
  }
}
```

حالا می‌توانیم Storybook را با دستورات زیر اجرا کنیم:

```bash
# شروع development server Storybook
npm run storybook

# build کردن Storybook به‌صورت static برای deployment
npm run build-storybook
```

دستور `storybook` یک development server روی پورت `6006` راه‌اندازی می‌کند. وقتی [http://localhost:6006](http://localhost:6006) را در مرورگر باز کنیم، رابط Storybook با تمام componentهای مستندشده را مشاهده خواهیم کرد.

### مستندسازی componentها با Storybook {#h2_136}

حالا که Storybook را پیکربندی کردیم، بیایید اولین story خود را بسازیم. Storyها نمونه‌هایی از نحوهٔ استفاده از component با propهای مختلف هستند.

#### ایجاد فایل story {#h3_137}

فایل‌های story از قرارداد نام‌گذاری `ComponentName.stories.tsx` پیروی می‌کنند. از **Component Story Format version 3** (**CSF3**) استفاده می‌کنیم که روش مدرن نوشتن storyهای Storybook است. بیایید یک story برای component `Button` بسازیم:

```tsx
// src/components/ui/button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react-vite';

import { Button } from './button';

const meta = {
  title: 'UI/Button',
  component: Button,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: [
        'default',
        'destructive',
        'outline',
        'secondary',
        'ghost',
        'link',
      ],
    },
    size: {
      control: 'select',
      options: ['default', 'sm', 'lg', 'icon'],
    },
    disabled: {
      control: 'boolean',
    },
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    children: 'Button',
    variant: 'default',
  },
};

export const Destructive: Story = {
  args: {
    children: 'Delete',
    variant: 'destructive',
  },
};

// ... storyهای بیشتر ...
```

بیایید ببینیم هر بخش چه کاری انجام می‌دهد:

#### پیکربندی Meta {#h4_138}

آبجکت `meta` پیکربندی تمام storyهای این component را تعریف می‌کند:

- `title: 'UI/Button'`: story را در sidebar Storybook زیر **UI > Button** سازمان‌دهی می‌کند
- `component: Button`: به Storybook می‌گوید این storyها برای کدام component هستند
- `parameters: { layout: 'centered' }`: component را در مرکز canvas قرار می‌دهد
- `tags: ['autodocs']`: به‌طور خودکار مستنداتی از propها و storyهای component تولید می‌کند
- `argTypes`: controlهای تعاملی برای propهای component در پنل controls Storybook تعریف می‌کند

#### پیکربندی ArgTypes {#h4_139}

بخش `argTypes` controlهای تعاملی در Storybook ایجاد می‌کند:

- `variant`: یک dropdown با تمام گزینه‌های variant ایجاد می‌کند
- `size`: یک dropdown با تمام گزینه‌های size ایجاد می‌کند
- `asChild` و `disabled`: کلیدهای toggle برای propهای boolean ایجاد می‌کنند

#### Storyهای جداگانه {#h4_140}

هر story export‌شده نمایانگر حالت یا variation متفاوتی از component است:

- `Default`: دکمهٔ پیش‌فرض را نشان می‌دهد
- `Destructive`: variant مخرب (برای عملیات حذف) را نشان می‌دهد
- `Outline`، `Secondary`، `Ghost`، `Link`: استایل‌های بصری مختلف را نشان می‌دهند
- `Small` و `Large`: اندازه‌های مختلف را نشان می‌دهند
- `Disabled`: state غیرفعال را نشان می‌دهد

هر story آبجکتی به نام `args` دارد که propهای ارسالی به component را تعریف می‌کند. این مقادیر را می‌توان در پنل controls Storybook تغییر داد تا ببینیم component به ورودی‌های مختلف چگونه پاسخ می‌دهد.

### مشاهدهٔ storyها در Storybook {#h2_141}

وقتی `npm run storybook` را اجرا کرده و [http://localhost:6006](http://localhost:6006) را باز کنیم، رابط Storybook را مشاهده می‌کنیم:

![Figure 3.3 – Viewing documented components in Storybook](/images/B31385_3_3.png)

**Figure 3.3 — مشاهدهٔ componentهای مستندشده در Storybook**

رابط Storybook چندین بخش کلیدی دارد:

- **Sidebar (چپ)**: تمام componentها و storyهای آن‌ها را فهرست می‌کند
- **Canvas (مرکز)**: component render‌شده را نشان می‌دهد
- **Controls panel (پایین)**: controlهای تعاملی برای تغییر propهای component
- **تب Docs**: مستندات خودکار تولیدشده که تمام variantها و توضیحات prop را نمایش می‌دهد

می‌توانیم:

- روی storyهای مختلف کلیک کنیم تا حالت‌های مختلف را ببینیم
- از پنل controls برای تغییر تعاملی propها استفاده کنیم
- به تب **Docs** سوئیچ کنیم تا تمام variantها را یکجا ببینیم
- مثال‌های کد نحوهٔ استفاده از component را کپی کنیم

با داشتن کتابخانهٔ component و سیستم مستندات، می‌توانیم با اطمینان UI اپلیکیشنمان را بسازیم.

## خلاصه {#h1_142}

در این فصل بنیان رابط کاربری اپلیکیشنمان را ساختیم.

با درک مفهوم componentها شروع کردیم — تکه‌های خودبسندهٔ UI که از prop برای ورودی، state برای مدیریت مقدارها، event handlerها برای تعاملات و TypeScript برای type safety استفاده می‌کنند.

سپس رویکردهای مختلف کتابخانهٔ component را ارزیابی کردیم. Shadcn UI را انتخاب کردیم زیرا با کپی مستقیم componentها در codebase، کنترل کاملی به ما می‌دهد و در عین حال از Base UI برای accessibility و Tailwind CSS برای styling بهره‌مند می‌شویم. می‌توانیم از shadcn CLI برای اضافه کردن componentها استفاده کنیم یا کد را به‌صورت دستی کپی کنیم.

در نهایت، Storybook را برای مستندسازی componentها راه‌اندازی کردیم. Storybook را طوری پیکربندی کردیم که با React Router و Vite کار کند، فایل‌های story ساختیم که حالت‌های مختلف component را نمایش می‌دهند و یاد گرفتیم Storybook محیطی ایزوله برای develop و test کردن componentها فراهم می‌کند. این کاتالوگ زنده‌ای در اختیار ما قرار می‌دهد که طراحان، توسعه‌دهندگان و stakeholderها می‌توانند به آن مراجعه کنند.

با داشتن کتابخانهٔ component و سیستم مستندات، آماده‌ایم ساخت featureهای اپلیکیشنمان را آغاز کنیم.
