# فصل ۸: بهبود عملکرد اپلیکیشن {#h1_215 .chapterTitle}

Performance یکی از مهم‌ترین جنبه‌های هر اپلیکیشن وب است. وقتی کاربران با اپلیکیشن ما تعامل می‌کنند، انتظار دارند در اسرع وقت پاسخ بگیرند. حتی تأخیرهای چند صد میلی‌ثانیه‌ای می‌تواند احساس کُندی و ناامیدی ایجاد کند.

برای هر اپلیکیشنی، performance روی تجربهٔ کاربر اثر دارد. کاربران بیشتر با محتوا تعامل می‌کنند و actionها را تکمیل می‌کنند وقتی اپلیکیشن سریع پاسخ می‌دهد. اپلیکیشن‌های کُند می‌توانند روی معیارهای کسب‌وکار اثر بگذارند و کاربران را از دست بدهند. حتی تأخیرهای کوچک می‌توانند تأثیر قابل‌توجهی روی نرخ conversion و engagement کاربران داشته باشند.

خوشبختانه، با تصمیم‌های معماری درست می‌توانیم اپلیکیشن‌هایی بسازیم که برای کاربران بسیار سریع‌تر به نظر برسند.

موارد زیر را پوشش خواهیم داد:

- شناسایی مشکلات performance
- بهینه‌سازی componentها
- Code splitting و lazy loading
- Streaming محتوا از سرور
- Debouncing ورودی کاربر
- مدیریت data setهای بزرگ با pagination و virtualization
- به‌روزرسانی‌های خوش‌بینانه (Optimistic UI)

در پایان این فصل، چندین بهینه‌سازی performance پیاده‌سازی کرده‌ایم که اپلیکیشن ما را سریع و واکنش‌گرا نگه می‌دارد، حتی هنگام کار با شبکه‌های کُند یا حجم زیاد داده.

## الزامات فنی {#h1_216 .heading-1}

قبل از شروع، باید پروژه را راه‌اندازی کنیم. برای توسعهٔ پروژه، چیزهای زیر باید روی کامپیوتر نصب باشند:

- [Node.js](https://nodejs.org/) نسخه ۲۴ یا بالاتر، [npm](https://www.npmjs.com/) نسخه ۱۱ یا بالاتر که همراه Node عرضه می‌شود. می‌توانیم با اجرای `node -v` و `npm -v` در ترمینال تأیید کنیم. راه‌های متعددی برای نصب Node.js و npm وجود دارد. این مقالهٔ عالی جزئیات بیشتری دارد: https://www.nodejsdesignpatterns.com/blog/5-ways-to-install-node-js .
- [VS Code](https://code.visualstudio.com/) (اختیاری) ویرایشگر محبوبی برای JavaScript و [TypeScript](https://www.typescriptlang.org/) است: متن‌باز، پشتیبانی خوب از TypeScript، و extensionها. از https://code.visualstudio.com قابل دانلود است.

کد این کتاب در مخزن کتاب موجود است. برای دسترسی به لینک مخزن، مراحل بخش «Download the example code files» در Preface را دنبال کنید. آن را clone کنید و به ریشهٔ مخزن بروید:

```bash
git clone https://github.com/PacktPublishing/React-Application-Architecture-for-Production-Second-Edition.git
```

مخزن شامل پوشه‌های فصل با کد هر فصل، به‌علاوهٔ یک پوشهٔ مشترک `api` با API server مورد استفاده در همهٔ فصل‌هاست.

ما در فصل ۸ هستیم، پس باید به پوشهٔ `chapter-08` برویم:

```bash
cd React-Application-Architecture-for-Production-Second-Edition/chapter-08
```

سپس باید dependencyها را نصب کنیم:

```bash
npm install
```

همچنین باید متغیرهای محیطی را فراهم کنیم:

```bash
cp .env.example .env
```

حالا باید فرانت‌اند در http://localhost:5173 اجرا شده باشد.

همچنین باید API server ما در حال اجرا باشد.

بیایید یک پنجرهٔ ترمینال جدید باز کنیم و به پوشهٔ `api` برویم:

```bash
cd React-Application-Architecture-for-Production-Second-Edition/api
```

حالا باید اسکریپت setup برای فصل ۸ را اجرا کنیم تا همه‌چیز برایمان پیکربندی شود:

```bash
npm run setup 08
```

سپس باید API server را اجرا کنیم:

```bash
npm run dev
```

باید API server را روی http://localhost:9999 ببینیم.

برای اطلاعات بیشتر دربارهٔ جزئیات setup، فایل `README.md` را بررسی کنید.

## شناسایی مشکلات performance {#h1_217 .heading-1}

قبل از اینکه بتوانیم مشکلات performance را حل کنیم، باید آن‌ها را شناسایی کنیم. اپلیکیشن‌های [React](https://react.dev/) ممکن است از re-renderهای غیرضروری رنج ببرند که چرخه‌های CPU را هدر می‌دهند و UI را بی‌واکنش نشان می‌دهند. فهمیدن اینکه کامپوننت‌ها کِی و چرا دوباره رندر می‌شوند برای ساخت اپلیکیشن‌های React سریع حیاتی است.

[React DevTools](https://react.dev/learn/react-developer-tools) بهترین راه برای فهمیدن نحوهٔ رندر اپلیکیشن و شناسایی کامپوننت‌هایی است که ممکن است باعث کندی شوند. این یک extension مرورگر است که تیم React ساخته و بینشی از کارهای React در پشت صحنه به ما می‌دهد.

تب **React DevTools Profiler** به ما اجازه می‌دهد یک session ضبط کنیم و دقیقاً ببینیم کدام کامپوننت‌ها رندر شدند، هر رندر چقدر طول کشید، و چه چیزی رندر را triggered کرد. برای استفاده، تب **Profiler** را باز می‌کنیم، دکمهٔ record را می‌زنیم، با اپلیکیشن تعامل می‌کنیم، و سپس ضبط را متوقف می‌کنیم. profiler یک flame graph از رندرها به ما نشان می‌دهد که در آن نوارهای پهن‌تر زمان رندر طولانی‌تر را نشان می‌دهند.

خروجی profiler به این شکل است:

شکل ۸.۱ — خروجی Profiler

هنگام بررسی خروجی profiler، می‌توانیم ببینیم کدام کامپوننت‌ها رندر شدند، رندر چقدر زمان برد، و چند بار رندر شدند. باید روی سه چیز تمرکز کنیم:

- **کامپوننت‌هایی که مکرراً رندر می‌شوند** — اگر یک کامپوننت بیش از حد دوباره رندر شود، ممکن است کار غیرضروری انجام دهد
- **کامپوننت‌هایی با زمان رندر طولانی** — این‌ها بیشترین تأثیر را در بهینه‌سازی دارند چون UI را برای مدت طولانی‌تری block می‌کنند
- **رندرهای آبشاری (Cascading renders)** — وقتی یک به‌روزرسانی کامپوننت باعث می‌شود بسیاری از کامپوننت‌های فرزند دوباره رندر شوند، ممکن است مشکل مدیریت state داشته باشیم

نکتهٔ کلیدی این است که همهٔ re-renderها بد نیستند. React طوری طراحی شده که به‌طور کارآمد دوباره رندر شود. فقط باید وقتی بهینه‌سازی کنیم که مشکل واقعی performance ببینیم. بهینه‌سازی زودرس می‌تواند کد را سخت‌تر کند بدون اینکه فایدهٔ واقعی ایجاد کند.

## بهینه‌سازی کامپوننت‌ها {#h1_218 .heading-1}

از آنجا که React دربارهٔ کامپوننت‌هاست، باید رویکرد درستی برای ساخت و ترکیب آن‌ها برای performance بهینه داشته باشیم. نحوهٔ ساختاردهی کامپوننت‌ها مستقیماً روی تعداد دفعات re-render و میزان کاری که React باید انجام دهد اثر می‌گذارد.

چند تکنیک مهم برای بهینه‌سازی کامپوننت‌ها برای بهترین performance:

- قرار دادن state در نزدیک‌ترین محل (Colocate state)
- Memoize کردن محاسبات گران‌قیمت
- استفاده از component composition

بیایید یک مثال عملی از نحوهٔ اعمال این تکنیک‌ها ببینیم.

### قرار دادن state در نزدیک‌ترین محل {#h2_219 .heading-2}

**State colocation** یعنی نگه‌داشتن state تا جایی که ممکن است نزدیک محل استفاده‌اش. اگر فقط یک کامپوننت به یک piece of state نیاز دارد، آن state باید در همان کامپوننت باشد، نه در parent. این یکی از تأثیرگذارترین بهینه‌سازی‌های performance است چون مستقیماً کنترل می‌کند کدام کامپوننت‌ها دوباره رندر شوند.

فرض کنید یک داشبورد داریم با بخش user profile و یک لیست ideas. در اینجا یک اشتباه رایج است که همهٔ state در یک کامپوننت زندگی می‌کند:

```tsx
function Dashboard() {
  const [user, setUser] = useState({ name: '', email: '' });
  const [ideas, setIdeas] = useState([
    { id: 1, title: 'Smart City Noise Monitoring' },
    { id: 2, title: 'AI-Powered Personal Assistant' },
  ]);
  const [searchTerm, setSearchTerm] = useState('');

  const filteredIdeas = ideas.filter(idea => 
    idea.title.toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    <div>
      <div>
        <h2>Profile</h2>
        <input 
          value={user.name}
          onChange={e => setUser({ ...user, name: e.target.value })}
          placeholder="Name"
        />
        <input 
          value={user.email}
          onChange={e => setUser({ ...user, email: e.target.value })}
          placeholder="Email"
        />
      </div>

      <div>
        <h2>My Ideas</h2>
        <input
          value={searchTerm}
          onChange={e => setSearchTerm(e.target.value)}
          placeholder="Search ideas..."
        />
        {filteredIdeas.map(idea => (
          <div key={idea.id}>{idea.title}</div>
        ))}
      </div>
    </div>
  );
}
```

مشکل اینجا چیست؟ هر بار که در فیلد نام پروفایل تایپ می‌کنیم، کل کامپوننت دوباره رندر می‌شود، از جمله لیست ideas. وقتی برای ideas جستجو می‌کنیم، فیلدهای profile هم دوباره رندر می‌شوند. همه‌چیز بر همه‌چیز اثر می‌گذارد چون همهٔ state در یک جا زندگی می‌کند.

می‌توانیم این را با تقسیم به کامپوننت‌های متمرکز حل کنیم که هر piece of state در کامپوننتی باشد که از آن استفاده می‌کند. ببینید هر کامپوننت چگونه state خودش را مدیریت می‌کند:

```tsx
function UserProfile() {
  const [user, setUser] = useState({ name: '', email: '' });

  return (
    <div>
      <h2>Profile</h2>
      <input 
        value={user.name}
        onChange={e => setUser({ ...user, name: e.target.value })}
        placeholder="Name"
      />
      <input 
        value={user.email}
        onChange={e => setUser({ ...user, email: e.target.value })}
        placeholder="Email"
      />
    </div>
  );
}

function IdeasList() {
  const [ideas] = useState([
    { id: 1, title: 'Smart City Noise Monitoring' },
    { id: 2, title: 'AI-Powered Personal Assistant' },
  ]);
  const [searchTerm, setSearchTerm] = useState('');

  const filteredIdeas = ideas.filter(idea => 
    idea.title.toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    <div>
      <h2>My Ideas</h2>
      <input
        value={searchTerm}
        onChange={e => setSearchTerm(e.target.value)}
        placeholder="Search ideas..."
      />
      {filteredIdeas.map(idea => (
        <div key={idea.id}>{idea.title}</div>
      ))}
    </div>
  );
}

function Dashboard() {
  return (
    <div>
      <UserProfile />
      <IdeasList />
    </div>
  );
}
```

حالا state به‌درستی colocated شده. بیایید ببینیم چه چیزی تغییر کرده:

- state `user` در `UserProfile` زندگی می‌کند — فقط بخش profile به این داده نیاز دارد، پس state را مالکیت می‌کند
- `ideas` و `searchTerm` در `IdeasList` زندگی می‌کنند — بخش ideas داده و قابلیت جستجوی خودش را مدیریت می‌کند
- `Dashboard` هیچ state ندارد — صرفاً یک کامپوننت composition است که layout را تنظیم می‌کند

نتیجه؟ وقتی در باکس جستجو تایپ می‌کنیم، فقط `IdeasList` دوباره رندر می‌شود. وقتی پروفایل را ویرایش می‌کنیم، فقط `UserProfile` دوباره رندر می‌شود. parent `Dashboard` هرگز دوباره رندر نمی‌شود چون هیچ state ندارد. State colocation مستقیماً از re-renderهای غیرضروری جلوگیری می‌کند با نگه‌داشتن کامپوننت‌ها isolated از تغییرات یکدیگر.

### Memoize کردن محاسبات گران‌قیمت {#h2_220 .heading-2}

با state به‌درستی colocated شده، کامپوننت‌های ما برای بیشتر scenarioها بهینه هستند. اما اگر یک کامپوننت محاسبهٔ گران‌قیمتی انجام دهد که لازم نیست در هر رندر اجرا شود چه؟

فرض کنید لیست ideas ما باید آمار نتایج فیلتر شده را محاسبه کند و همچنین می‌خواهیم به کاربران اجازه دهیم بین نمای list و grid جابجا شوند:

```tsx
function IdeasList() {
  const [ideas] = useState([
    { id: 1, title: 'Smart City Noise Monitoring' },
    { id: 2, title: 'AI-Powered Personal Assistant' },
  ]);
  const [searchTerm, setSearchTerm] = useState('');
  const [viewMode, setViewMode] = useState<'list' | 'grid'>('list');

  const filteredIdeas = ideas.filter(idea => 
    idea.title.toLowerCase().includes(searchTerm.toLowerCase())
  );

  const stats = calculateIdeaStatistics(filteredIdeas);

  return (
    <div>
      <h2>My Ideas</h2>
      <div>
        <input
          value={searchTerm}
          onChange={e => setSearchTerm(e.target.value)}
          placeholder="Search ideas..."
        />
        <button onClick={() => setViewMode(viewMode === 'list' ? 'grid' : 'list')}>
          {viewMode === 'list' ? 'Grid View' : 'List View'}
        </button>
      </div>
      
      <div>
        <p>Total: {stats.total}</p>
        <p>Average length: {stats.averageLength}</p>
      </div>

      <div className={viewMode === 'grid' ? 'grid grid-cols-2 gap-4' : ''}>
        {filteredIdeas.map(idea => (
          <div key={idea.id}>{idea.title}</div>
        ))}
      </div>
    </div>
  );
}
```

مشکل اینجا این است که `calculateIdeaStatistics` در هر رندر اجرا می‌شود، حتی وقتی لازم نیست. وقتی کاربر حالت نمایش را تغییر می‌دهد، کامپوننت دوباره رندر می‌شود، اما ideas فیلتر شده تغییر نکرده‌اند، پس محاسبهٔ مجدد آمار کار هدر رفته است. می‌توانیم این را با memoize کردن محاسبهٔ گران‌قیمت با [`useMemo`](https://react.dev/reference/react/useMemo) حل کنیم:

```typescript
function IdeasList() {
  const [ideas] = useState([
    { id: 1, title: 'Smart City Noise Monitoring' },
    { id: 2, title: 'AI-Powered Personal Assistant' },
  ]);
  const [searchTerm, setSearchTerm] = useState('');
  const [viewMode, setViewMode] = useState<'list' | 'grid'>('list');

  const filteredIdeas = useMemo(() => {
    return ideas.filter(idea => 
      idea.title.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [ideas, searchTerm]);

  const stats = useMemo(() => {
    return calculateIdeaStatistics(filteredIdeas);
  }, [filteredIdeas]);

  // ...
}
```

حالا وقتی کاربر حالت نمایش را تغییر می‌دهد، React می‌بیند که `ideas` و `searchTerm` تغییر نکرده‌اند، پس فیلتر کردن را رد می‌کند و `filteredIdeas` کش‌شده را برمی‌گرداند. به همین ترتیب، از آنجا که `filteredIdeas` مرجع یکسانی دارد، محاسبهٔ گران‌قیمت `calculateIdeaStatistics` هم رد می‌شود.

> نکته: نباید همه‌چیز را memoize کنیم. `useMemo` هزینهٔ اضافی دارد، پس فقط باید برای محاسباتی که واقعاً گران‌قیمت هستند استفاده شود. برای تصمیم‌گیری باید تعیین کنیم آیا piece of state دیگری در کامپوننت داریم که نامربوط به محاسبهٔ گران‌قیمت باشد. در بستر مثال ما، `viewMode` باعث re-render می‌شود بدون اینکه روی مقدار memoize شدهٔ `stats` اثر بگذارد. می‌توانیم از React DevTools Profiler استفاده کنیم تا مشخص کنیم کدام محاسبات باعث re-renderهای کُند می‌شوند قبل از اینکه سراغ `useMemo` برویم.

### استفاده از component composition {#h2_221 .heading-2}

**Component composition** به ما اجازه می‌دهد UIهای پیچیده از قطعات ساده و قابل استفادهٔ مجدد بسازیم در حالی که ویژگی‌های performance عالی حفظ می‌شوند. وقتی به‌درستی انجام شود، composition بهینه‌سازی خودکار از طریق referenceهای پایدار prop به ما می‌دهد.

فرض کنید می‌خواهیم وقتی بخش user profile focused شود، یک نشانگر بصری نشان دهیم. رویکرد رایج اضافه کردن focus tracking مستقیماً به کامپوننت است:

```tsx
function UserProfile() {
  const [user, setUser] = useState({ name: '', email: '' });
  const [isFocused, setIsFocused] = useState(false);

  return (
    <div 
      style={{ border: isFocused ? '2px dashed blue' : 'none' }}
      onFocus={() => setIsFocused(true)}
      onBlur={() => setIsFocused(false)}
    >
      <h2>Profile</h2>
      <input 
        value={user.name}
        onChange={e => setUser({ ...user, name: e.target.value })}
        placeholder="Name"
      />
      <input 
        value={user.email}
        onChange={e => setUser({ ...user, email: e.target.value })}
        placeholder="Email"
      />
    </div>
  );
}
```

مشکل این رویکرد این است که تغییرات focus کل فرم profile را دوباره رندر می‌کند و تغییرات فرم، logic مدیریت focus را دوباره رندر می‌کند. این‌ها نگرانی‌های نامرتبطی هستند که نباید بر هم اثر بگذارند.

می‌توانیم از composition برای جدا کردن این نگرانی‌ها استفاده کنیم:

```tsx
function FocusTracker({ children }) {
  const [isFocused, setIsFocused] = useState(false);

  return (
    <div 
      style={{ border: isFocused ? '2px dashed blue' : 'none' }}
      onFocus={() => setIsFocused(true)}
      onBlur={() => setIsFocused(false)}
    >
      {children}
    </div>
  );
}

function UserProfile() {
  const [user, setUser] = useState({ name: '', email: '' });

  return (
    <div>
      <h2>Profile</h2>
      <input 
        value={user.name}
        onChange={e => setUser({ ...user, name: e.target.value })}
        placeholder="Name"
      />
      <input 
        value={user.email}
        onChange={e => setUser({ ...user, email: e.target.value })}
        placeholder="Email"
      />
    </div>
  );
}

function Dashboard() {
  return (
    <div>
      <FocusTracker>
        <UserProfile />
      </FocusTracker>
      <IdeasList />
    </div>
  );
}
```

حالا وقتی روی یک بخش focus می‌کنیم، فقط `FocusTracker` برای به‌روزرسانی style border خود دوباره رندر می‌شود. از آنجا که کامپوننت `UserProfile` state خودش را مستقیماً مدیریت می‌کند، تغییرات focus در کامپوننت `FocusTracker` باعث re-render در `UserProfile` نمی‌شوند چون دو کامپوننت از به‌روزرسانی‌های state یکدیگر isolated هستند.

این الگوی composition دو مزیت بهینه‌سازی به ما می‌دهد:

- **بهینه‌سازی Props** — prop `children` یک‌بار در parent ساخته می‌شود و به پایین منتقل می‌شود. حتی اگر `FocusTracker` با تغییر focus دوباره رندر شود، React children را دوباره رندر نمی‌کند چون مرجع prop پایدار است.
- **بهینه‌سازی رندر** — هر کامپوننت فقط وقتی دوباره رندر می‌شود که state خودش تغییر کند. به‌روزرسانی‌های focus به `FocusTracker` محدود و تغییرات فرم به `UserProfile` محدود می‌شوند.

وقتی کامپوننت‌ها با state colocated، محاسبات memoize شده و composition هوشمندانه به‌درستی ساختاردهی شوند، طبیعتاً کمتر دوباره رندر می‌شوند. به همین دلیل معماری خوب کامپوننت گام عالی به سمت بهینه‌سازی performance اپلیکیشن است.

این‌ها چند بهینه‌سازی پایهٔ React بودند. حالا بیایید چند تکنیک پیشرفته‌تر که فراتر از React هستند و جنبه‌های مختلف اپلیکیشن ما را بهینه می‌کنند ببینیم.

### Code splitting و lazy loading {#h2_222 .heading-2}

وقتی کاربران برای اولین بار به اپلیکیشن ما مراجعه می‌کنند، مرورگر تمام کد JavaScript را قبل از interactive شدن صفحه دانلود می‌کند. برای اپلیکیشن‌های بزرگ، این می‌تواند چندین مگابایت کد باشد. اما نکته اینجاست: کاربران ممکن است به همهٔ ویژگی‌های اپلیکیشن نیاز نداشته باشند، پس چرا کدی را که هرگز استفاده نمی‌کنند دانلود کنند؟

**Code splitting** به ما اجازه می‌دهد bundle JavaScript خود را به قطعات کوچکتری تقسیم کنیم که به‌صورت on-demand بارگذاری شوند. به‌جای دانلود تمام کد از ابتدا، کاربران فقط آنچه نیاز دارند را دانلود می‌کنند. این به‌ویژه برای اپلیکیشن‌های بزرگ ارزشمند است، جایی که کاربران ممکن است هرگز به برخی از صفحات مراجعه نکنند.

حداقل باید در سطح route code split کنیم تا کد هر صفحه فقط وقتی بارگذاری شود که کاربر به آن ناوبری کند. خبر خوب این است که [React Router](https://reactrouter.com/) در حالت framework به‌طور خودکار routeها را code split می‌کند. با این حال، هنوز باید code splitting و lazy loading کامپوننت‌های سنگین درون صفحات را در نظر بگیریم، مثل نمودارهای پیچیده، ویرایشگرها یا ویژگی‌هایی که بسیاری از کاربران استفاده نمی‌کنند.

React تابع [`lazy`](https://react.dev/reference/react/lazy) را برای بارگذاری کامپوننت‌ها به‌صورت on-demand فراهم می‌کند. این یک مثال ساده است:

```tsx
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('@/components/heavy-component'));

function Ideas() {
  const [showHeavyComponent, setShowHeavyComponent] = useState(false);

  return (
    <div>
      <button onClick={() => setShowHeavyComponent(true)}>View Heavy Component</button>
      <Suspense fallback={<div>Loading...</div>}>
{showHeavyComponent&&<HeavyComponent />}
      </Suspense>
    </div>
  );
}
```

تابع `lazy` یک تابع می‌گیرد که یک dynamic import برمی‌گرداند. وقتی کامپوننت برای اولین بار رندر می‌شود، React فایل JavaScript حاوی `HeavyComponent` را بارگذاری می‌کند. کامپوننت [`Suspense`](https://react.dev/reference/react/Suspense) یک fallback در حین بارگذاری کد نشان می‌دهد.

توجه کنید که فقط باید کامپوننت‌هایی را که واقعاً سنگین هستند code split و lazy load کنیم، چون نمی‌خواهیم درخواست chunk اضافی بدون فایدهٔ قابل‌توجه ایجاد کنیم. برای فهمیدن اندازهٔ chunkها، می‌توانیم از analyzer plugin برای [Vite](https://vitejs.dev/) استفاده کنیم. برای دریافت گزارش، می‌توانیم دستور `analyze` را اجرا کنیم و سپس گزارش را در مرورگر باز کنیم.

```bash
npm run analyze
```

این یک گزارش در مرورگر باز می‌کند.

شکل ۸.۲ — گزارش Bundle analyzer

در اینجا می‌توانیم کشف کنیم کدام بخش‌های اپلیکیشن سنگین هستند و باید code split و lazy load شوند.

## Streaming محتوا از سرور {#h1_223 .heading-1}

**Server-side rendering (SSR)** بارگذاری اولیهٔ صفحه را با رندر کردن HTML روی سرور بهبود می‌دهد. با این حال، SSR سنتی معمولاً یک محدودیت دارد: سرور باید منتظر همهٔ داده‌ها بماند قبل از اینکه چیزی به مرورگر بفرستد. اگر یک درخواست API کُند باشد، کل صفحه تأخیر پیدا می‌کند.

این یک تجربهٔ ناامیدکننده ایجاد می‌کند. تصور کنید صفحه‌ای که یک ایده و نظراتش را نشان می‌دهد. دادهٔ ایده در ۱۰۰ میلی‌ثانیه بارگذاری می‌شود، اما نظرات ۲ ثانیه طول می‌کشند. با SSR سنتی، کاربران ۲ ثانیه منتظر می‌مانند تا چیزی ببینند، حتی اگر ایده تقریباً فوراً آماده بود.

راه‌حل streaming محتوا به‌محض رسیدن است. محتوای سریع فوراً ظاهر می‌شود، در حالی که محتوای کُندتر به‌تدریج در پس‌زمینه بارگذاری می‌شود.

بیایید streaming را برای صفحهٔ جزئیات ایده پیاده‌سازی کنیم. دادهٔ ایده اول بارگذاری می‌شود چون کاربران باید فوراً آن را ببینند. نظرات می‌توانند بعداً بارگذاری شوند چون برای دیدن کاربر حیاتی‌تر نیستند.

```typescript
// src/app/routes/ideas/idea.tsx

export async function loader({ params }: Route.LoaderArgs) {
  const idea = await getIdeaById(params.id);
  const reviewsPromise = getReviewsByIdea(params.id);
  
  return routerData({
    idea,
    reviewsPromise,
  });
}
```

تفاوت نحوهٔ مدیریت دو piece of data را توجه کنید. ما روی idea `await` می‌کنیم، یعنی سرور قبل از ارسال هرچیزی منتظر آن می‌ماند. اما روی reviews `await` نمی‌کنیم؛ promise را مستقیماً به کامپوننت پاس می‌دهیم. `reviewsPromise` به‌عنوان یک promise حل‌نشده به کامپوننت فرستاده می‌شود. رندرر streaming React از طریق مرز Suspense promise در حال انتظار را شناسایی می‌کند و بلافاصله شروع به ارسال HTML موجود به مرورگر می‌کند، بدون اینکه منتظر حل شدن reviews بماند.

در کامپوننت، از `Suspense` React برای مدیریت حالت بارگذاری reviews استفاده می‌کنیم:

```typescript
export default function IdeaDetailPage({
  params,
  loaderData,
}: Route.ComponentProps) {
  const ideaId = params.id as string;

  const ideaQuery = useIdeaByIdQuery({
    id: ideaId,
    options: {
      ...(loaderData?.idea && { initialData: loaderData.idea }),
    },
  });
  const idea = ideaQuery?.data ?? loaderData?.idea;

  return (
    <div className="container mx-auto px-4 py-8">
      <Seo
        title={`${idea?.title || 'Idea'} | AIdeas`}
        description={idea?.shortDescription || 'View idea details'}
      />
      <div className="max-w-4xl mx-auto">
        <IdeaDetails idea={idea} />
        <Suspense fallback={<ReviewsSkeleton />}>
          <IdeaReviews
            idea={idea}
            reviewsPromise={loaderData.reviewsPromise}
          />
        </Suspense>
      </div>
    </div>
  );
}
```

کامپوننت `Suspense` بخش reviews را wrap می‌کند. در حالی که promise reviews هنوز در انتظار است، React `ReviewsSkeleton` fallback را نشان می‌دهد. وقتی داده می‌رسد، skeleton به‌طور یکپارچه با reviews واقعی جایگزین می‌شود.

اما کامپوننت `IdeaReviews` چگونه به داده از promise دسترسی پیدا می‌کند؟ React هوک `use` را دقیقاً برای همین منظور فراهم می‌کند:

```typescript
function IdeaReviews({
  idea,
  reviewsPromise,
}: {
  idea: Idea;
  reviewsPromise: Promise<ReviewListResponse>;
}) {
  const reviewsData = use(reviewsPromise);
  
  // Now we can use reviewsData to render the reviews
  // ...
}
```

Hook `use` از React promise را unwrap می‌کند. وقتی promise حل می‌شود، React کامپوننت را با داده دوباره رندر می‌کند. اگر promise هنوز در انتظار باشد، `use` کامپوننت را suspend می‌کند، به همین دلیل به wrapper `Suspense` بالا نیاز داریم. ارزش دارد که توجه کنیم hook `use` از React 19 در دسترس است. به‌عنوان جایگزین، می‌توانیم از کامپوننت `Await` از React Router استفاده کنیم تا promise را به همین شکل unwrap کنیم:

```typescript
import { Await } from "react-router";

function IdeaReviews({
  idea,
  reviewsPromise,
}: {
  idea: Idea;
  reviewsPromise: Promise<ReviewListResponse>;
}) {
  return (
    <Await promise={reviewsPromise}>
      {({ data }) => (
        // Use data to render the reviews
      )}
    </Await>
  );
}
```

با [streaming SSR](https://react.dev/reference/react-dom/server/renderToPipeableStream)، جزئیات ایده فوراً ظاهر می‌شوند در حالی که skeleton نشان می‌دهد نظرات کجا خواهند آمد. وقتی نظرات بارگذاری تمام می‌شوند، به‌طور یکپارچه skeleton را جایگزین می‌کنند. این باعث می‌شود صفحه سریع‌تر به نظر برسد چون کاربران می‌توانند خواندن ایده را شروع کنند در حالی که نظرات در پس‌زمینه بارگذاری می‌شوند.

## Debouncing ورودی کاربر {#h1_224 .heading-1}

وقتی کاربران در یک فیلد جستجو تایپ می‌کنند، هر ضربهٔ کلید می‌تواند یک درخواست API جدید ایجاد کند. اگر کسی به‌سرعت «Education» را جستجو کند، آن ۹ کاراکتر و احتمالاً ۹ درخواست network جداگانه است. بیشتر این درخواست‌ها هدر می‌روند چون کاربر هنوز تایپ کردن را تمام نکرده.

این مشکل را می‌توان از تب network در مرورگر دید.

شکل ۸.۳ — درخواست‌های network هنگام تایپ بدون debouncing

همان‌طور که می‌بینیم، ۹ درخواست network هنگام تایپ کردن ایجاد می‌شود. این پهنای باند را هدر می‌دهد، بار اضافی روی سرور می‌گذارد، و می‌تواند race condition ایجاد کند که پاسخ‌ها به ترتیب نادرست برسند. تصور کنید اگر پاسخ «Edu» بعد از پاسخ «Education» برسد؛ UI نتایج نادرست نشان می‌دهد که تجربهٔ کاربری بدی است.

Debouncing این مشکل را با صبر کردن برای وقفه در ورودی کاربر قبل از ارسال درخواست حل می‌کند. به‌جای فعال شدن در هر ضربهٔ کلید، منتظر می‌مانیم تا کاربر برای مدت مشخصی (معمولاً ۳۰۰ تا ۵۰۰ میلی‌ثانیه) تایپ کردن را متوقف کند، سپس یک درخواست واحد با مقدار نهایی ارسال می‌کنیم.

برای debounce کردن ورودی، ابتدا یک hook قابل استفادهٔ مجدد `useDebouncedValue` می‌سازیم:

```typescript
// src/hooks/use-debounced-value.ts
import { useEffect, useState } from 'react';

export function useDebouncedValue<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}
```

این hook هر مقداری و یک تأخیر به میلی‌ثانیه می‌گیرد. نحوهٔ کار این است: وقتی مقدار تغییر می‌کند، یک timer شروع می‌شود. اگر مقدار قبل از اتمام timer دوباره تغییر کند، timer reset می‌شود. فقط وقتی مقدار برای کل تأخیر پایدار بماند، مقدار debounced به‌روزرسانی می‌شود.

حالا بیایید این hook را در hook جستجو و فیلترهایمان استفاده کنیم:

```typescript
// src/features/ideas/hooks/use-search-and-filters.ts

export function useSearchAndFilters(debounceMs: number = 500) {
  const [urlSearchTerm, setUrlSearchTerm] = useQueryState(
    'search',
    parseAsString.withDefault(''),
  );

  const debouncedSearchTerm = useDebouncedValue(urlSearchTerm, debounceMs);

  return {
    searchTerm: urlSearchTerm,
    debouncedSearchTerm,
    params: {
      ...(debouncedSearchTerm && { search: debouncedSearchTerm }),
      // ...other params
    },
    setSearchTerm: setUrlSearchTerm,
    // ...other actions
  };
}
```

اینجا دو مقدار مختلف برمی‌گردانیم و این مهم است. `searchTerm` وقتی کاربران تایپ می‌کنند فوراً به‌روزرسانی می‌شود و فیلد ورودی را واکنش‌گرا نگه می‌دارد؛ کاربران کاراکترهایشان را فوراً می‌بینند. `debouncedSearchTerm` فقط بعد از توقف کاربر به‌روزرسانی می‌شود و ما این مقدار debounced را در شیء `params` برای درخواست‌های API استفاده می‌کنیم.

در کامپوننت، فیلد جستجو مستقیماً به `searchTerm` bind می‌شود:

```typescript
// src/features/ideas/components/idea-search-and-filters.tsx

export function IdeaSearchAndFilters() {
  const {
    searchTerm,
    setSearchTerm,
    // ...
  } = useSearchAndFilters();

  return (
    <div className="space-y-4">
      <div className="relative">
        <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-muted-foreground" />
        <Input
          placeholder="Search ideas by title or description..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          className="pl-10"
        />
      </div>
      {/* ... */}
    </div>
  );
}
```

ورودی آنچه کاربران تایپ می‌کنند فوراً نشان می‌دهد، اما شیء `params` که توسط query استفاده می‌شود فقط بعد از توقف تایپ کردن کاربر به مدت ۵۰۰ میلی‌ثانیه به‌روزرسانی می‌شود:

```typescript
// src/app/routes/ideas/ideas.tsx

function Ideas() {
  const { params } = useSearchAndFilters();

  const ideasInfiniteQuery = useIdeasInfiniteQuery({
    params: params as GetAllIdeasData['query'],
  });

  // ...
}
```

بعد از پیاده‌سازی debouncing، می‌توانیم تفاوت را در تب network ببینیم.

شکل ۸.۴ — درخواست‌های network هنگام تایپ با debouncing

همان‌طور که می‌بینیم، فقط یک درخواست network بعد از توقف تایپ کردن کاربر به مدت ۵۰۰ میلی‌ثانیه ایجاد می‌شود. Debouncing یکی از ساده‌ترین بهینه‌سازی‌هایی است که می‌توانیم انجام دهیم، اما تأثیر آن قابل‌توجه است. این تعداد درخواست‌های API را از ۹ به ۱ در هر جستجو کاهش می‌دهد و هم performance client و هم بار سرور را بهبود می‌دهد.

## بهینه‌سازی data setهای بزرگ {#h1_225 .heading-1}

هنگام کار با data setهای بزرگ، performance می‌تواند برای هم سرور و هم client یک bottleneck شود. سرور باید احتمالاً هزاران رکورد را پردازش کند که زمان پاسخ و مصرف پهنای باند را افزایش می‌دهد. client باید همهٔ آن رکوردها را رندر کند که می‌تواند کُند و حافظه‌بر باشد، به‌ویژه در دستگاه‌های موبایل و سطح پایین.

می‌توانیم این چالش‌ها را با دو تکنیک حل کنیم: pagination که مقدار دادهٔ منتقل شده را کاهش می‌دهد، و virtualization که تعداد آیتم‌های لیست رندر شده در صفحه را کاهش می‌دهد.

### Pagination {#h2_226 .heading-2}

به‌جای بازگرداندن همهٔ ideas به‌یکباره، می‌توانیم infinite pagination پیاده‌سازی کنیم تا مجموعه‌ای از ideas را دریافت کنیم، سپس هنگام اسکرول کاربر یا کلیک دکمهٔ «Load More» ideas بیشتری دریافت کنیم. به این ترتیب، کاربران با یک مجموعهٔ کوچک و سریع‌البارگذاری از داده شروع می‌کنند و می‌توانند در صورت نیاز بیشتر بارگذاری کنند.

شکل ۸.۵ — Infinite pagination

[React Query](https://tanstack.com/query/latest) هوک `useInfiniteQuery` را دقیقاً برای این الگو فراهم می‌کند. این hook چندین صفحه از داده را مدیریت می‌کند و می‌داند چه زمانی صفحات بیشتری در دسترس هستند. بیایید API ideas خود را برای پشتیبانی از بارگذاری infinit به‌روزرسانی کنیم.

```typescript
// src/features/ideas/api/get-ideas.ts

export async function getIdeas(
  params?: GetAllIdeasData['query'],
): Promise<GetAllIdeasResponse> {
  const validatedData = zGetAllIdeasData.parse({ query: params });

  const response = await api.get<GetAllIdeasResponse>('/ideas', {
    params: { ...validatedData.query, limit: 10 },
  });

  return zGetAllIdeasResponse.parse(response);
}

export function getIdeasInfiniteQueryOptions(
  params?: GetAllIdeasData['query'],
) {
  return infiniteQueryOptions({
    queryKey: ideasQueryKeys.list(params),
    queryFn: ({ pageParam }) =>
      getIdeas({ ...params, page: pageParam.toString() }),
    getNextPageParam: ({ pagination }) => pagination.nextPage,
    initialPageParam: 1,
  });
}

export function useIdeasInfiniteQuery({
  params,
  options,
}: {
  params?: GetAllIdeasData['query'];
  options?: Omit<
    ReturnType<typeof getIdeasInfiniteQueryOptions>,
    'queryKey' | 'queryFn'
  >;
}) {
  return useInfiniteQuery({
    ...getIdeasInfiniteQueryOptions(params),
    ...options,
  });
}
```

بیایید بخش‌های کلیدی را بررسی کنیم:

- `limit: 10` — فقط ۱۰ آیتم در هر صفحه درخواست می‌کنیم نه همهٔ ideas به‌یکباره. این هر درخواست را سریع و پاسخ را کوچک نگه می‌دارد.
- `getNextPageParam` — این تابع شمارهٔ صفحهٔ بعدی را از پاسخ API استخراج می‌کند. React Query از این استفاده می‌کند تا بداند چه زمانی صفحات بیشتری برای بارگذاری وجود دارد. اگر `undefined` برگرداند، صفحهٔ بیشتری وجود ندارد.
- `initialPageParam: 1` — اولین صفحه‌ای که هنگام اجرای query بارگذاری می‌شود.

حالا بیایید این را در صفحهٔ ideas استفاده کنیم:

```typescript
// src/app/routes/ideas/ideas.tsx

export async function loader({ request }: Route.LoaderArgs) {
  // ...

  await Promise.all([
    queryClient.prefetchInfiniteQuery(
      getIdeasInfiniteQueryOptions(ideasParams),
    ),
    queryClient.prefetchQuery(getTagsQueryOptions()),
  ]);

  return routerData({
    dehydratedState: dehydrate(queryClient),
  });
}

function Ideas() {
  const { params } = useSearchAndFilters();

  const ideasInfiniteQuery = useIdeasInfiniteQuery({
    params: params as GetAllIdeasData['query'],
  });

  const allIdeas =
    ideasInfiniteQuery.data?.pages.flatMap((page) => page.data) || [];

  return (
    <div className="container mx-auto px-4 py-8">
      <Seo
        title="Discover AI Ideas | AIdeas"
        description="Browse and explore innovative AI application ideas from the community"
      />
      <div className="mb-8">
        <h1 className="text-3xl font-bold mb-4">Discover AI Ideas</h1>
        <p className="text-muted-foreground mb-6">
          Browse and explore innovative AI application ideas from the community
        </p>
        <IdeaSearchAndFilters />
      </div>

      <IdeasList
        ideas={allIdeas}
        isLoading={ideasInfiniteQuery.isLoading && allIdeas.length === 0}
        emptyMessage="No ideas available yet"
        error={ideasInfiniteQuery.error}
      />

      {allIdeas.length > 0 && ideasInfiniteQuery.hasNextPage && (
        <div className="flex justify-center mt-8">
          <Button
            onClick={() => ideasInfiniteQuery.fetchNextPage()}
            disabled={ideasInfiniteQuery.isFetchingNextPage}
          >
            {ideasInfiniteQuery.isFetchingNextPage
              ? 'Loading more ideas...'
              : 'Load more ideas'}
          </Button>
        </div>
      )}
    </div>
  );
}
```

آرایهٔ `pages` شامل تمام صفحاتی است که تا الان بارگذاری کرده‌ایم. هر صفحه آرایهٔ `data` خودش از ideas را دارد. از `flatMap` استفاده می‌کنیم تا همهٔ ideas از همهٔ صفحات را در یک آرایهٔ واحد ترکیب کنیم که بتوانیم به کامپوننت لیست پاس دهیم.

وقتی کاربر روی «Load more ideas» کلیک می‌کند، `fetchNextPage` صفحهٔ بعدی را بارگذاری می‌کند و آن را به آرایهٔ `pages` اضافه می‌کند. React Query تمام caching و مدیریت state را برای ما انجام می‌دهد. مقدار boolean `hasNextPage` به ما می‌گوید آیا صفحات بیشتری برای بارگذاری وجود دارد، بنابراین می‌توانیم دکمه را وقتی به انتهای مسیر رسیدیم مخفی کنیم.

Pagination performance سرور و زمان بارگذاری اولیه را بهبود می‌دهد، اما با بارگذاری صفحات بیشتر توسط کاربران، ممکن است صدها آیتم در DOM داشته باشیم. بیایید بررسی کنیم چگونه این را بهینه کنیم.

## List virtualization {#h1_227 .heading-1}

حتی با pagination، کاربران ممکن است ۱۰، ۲۰ یا ۵۰ صفحه از داده بارگذاری کنند. اگر هر صفحه ۱۰ آیتم داشته باشد، ۵۰۰ عنصر DOM است که هر کدام listenerهای رویداد خود، محاسبات layout و مصرف حافظه دارند. در دستگاه‌های کُندتر، این می‌تواند صفحه را کُند نشان دهد. این را می‌توان با بررسی DOM در مرورگر نشان داد.

شکل ۸.۶ — بررسی DOM

همان‌طور که می‌بینیم، عناصر DOM بیشتری رندر شده‌اند تا آنچه کاربر در یک لحظه می‌تواند ببیند. این می‌تواند یک bottleneck performance باشد، به‌ویژه در دستگاه‌های کُندتر. اینجاست که list virtualization وارد می‌شود.

List virtualization تکنیکی است که فقط آیتم‌های فعلی قابل مشاهده در viewport را رندر می‌کند. به‌جای ساخت عناصر DOM برای همهٔ ۵۰۰ آیتم در یک لیست، شاید فقط ۱۰ تایی را که قابل مشاهده هستند رندر کنیم، به‌علاوهٔ چند مورد اضافی برای اسکرول روان. هنگام اسکرول کاربر، آیتم‌ها هنگام ورود و خروج از viewport mount و unmount می‌شوند.

شکل ۸.۷ — List virtualization

از کتابخانهٔ [`@tanstack/react-virtual`](https://tanstack.com/virtual) برای پیاده‌سازی این استفاده خواهیم کرد. این کتابخانه تمام ریاضیات پیچیدهٔ اسکرول و موقعیت‌یابی عناصر را برای ما مدیریت می‌کند. این نسخهٔ virtualized شدهٔ لیست ideas ماست:

```typescript
// src/features/ideas/components/virtualized-ideas-list.tsx

import { useWindowVirtualizer } from '@tanstack/react-virtual';

export function VirtualizedIdeasList({
  ideas,
  isLoading,
  emptyMessage,
  error,
}: VirtualizedIdeasListProps) {
  const virtualizer = useWindowVirtualizer({
    count: ideas?.length || 0,
    estimateSize: () => 150,
    overscan: 5,
  });

  // ...

  return (
    <Card>
      <CardContent className="p-0">
        <div
          style={{
            height: `${virtualizer.getTotalSize()}px`,
            width: '100%',
            position: 'relative',
          }}
        >
          {virtualizer.getVirtualItems().map((virtualItem) => {
            const idea = ideas[virtualItem.index];
            return (
              <div
                key={virtualItem.key}
                data-index={virtualItem.index}
                ref={virtualizer.measureElement}
                style={{
                  position: 'absolute',
                  top: 0,
                  left: 0,
                  width: '100%',
                  transform: `translateY(${virtualItem.start}px)`,
                }}
              >
                <div className="px-6 border-b border-border last:border-b-0">
                  <IdeaListItem idea={idea} />
                </div>
              </div>
            );
          })}
        </div>
      </CardContent>
    </Card>
  );
}
```

Hook `useWindowVirtualizer` سه گزینهٔ کلیدی می‌گیرد:

- `count` — تعداد کل آیتم‌ها در لیست. virtualizer باید این را بداند تا ارتفاع کل اسکرول را محاسبه کند.
- `estimateSize` — تابعی که ارتفاع تقریبی هر آیتم را برمی‌گرداند. لازم نیست دقیق باشد، virtualizer اندازهٔ واقعی را هنگام رندر آیتم‌ها اندازه‌گیری می‌کند. یک تخمین خوب فقط به layout اولیه کمک می‌کند و از layout shift و اسکرول ناهموار در رندرهای اولیه جلوگیری می‌کند.
- `overscan` — چند آیتم خارج از ناحیهٔ قابل مشاهده رندر شوند. مقدار بالاتر اسکرول را روان‌تر می‌کند (آیتم‌ها قبل از قابل مشاهده شدن آماده هستند) اما آیتم‌های بیشتری رندر می‌کند. ۵ نقطهٔ شروع خوبی است.

virtualizer چندین متد به ما می‌دهد که جادو را ایجاد می‌کنند:

- `getTotalSize()` — ارتفاع کل مورد نیاز برای دربرگرفتن همهٔ آیتم‌ها را برمی‌گرداند. از این برای ارتفاع container استفاده می‌کنیم تا scrollbar اندازهٔ درستی داشته باشد.
- `getVirtualItems()` — فقط آیتم‌هایی را برمی‌گرداند که الان باید رندر شوند. این کلید virtualization است، به‌جای map کردن روی همهٔ ۵۰۰ idea، فقط روی ۱۵ تا ۲۰ آیتمی که قابل مشاهده هستند map می‌کنیم.
- `measureElement` — اندازهٔ واقعی هر آیتم را با ref callback اندازه‌گیری می‌کند. این مهم است چون ارتفاع آیتم‌ها متفاوت باشد.

از آنجا که virtualization یک بهینه‌سازی client-side است که به موقعیت اسکرول بستگی دارد، باید در حین server-side rendering از لیست معمولی استفاده کنیم چون سرور نمی‌داند چه چیزی در viewport قابل مشاهده است یا موقعیت اسکرول چیست. می‌توانیم به‌صورت شرطی لیست virtualized را در client بعد از hydration استفاده کنیم:

```typescript
// src/app/routes/ideas/ideas.tsx

function Ideas() {
  // ...

  const isClient = useIsClient();

  const ListComponent = isClient ? VirtualizedIdeasList : IdeasList;

  return (
    <div className="container mx-auto px-4 py-8">
      {/* ... */}

      <ListComponent
        ideas={allIdeas}
        isLoading={ideasInfiniteQuery.isLoading && allIdeas.length === 0}
        emptyMessage="No ideas available yet"
        error={ideasInfiniteQuery.error}
      />

      {/* ... */}
    </div>
  );
}
```

می‌توانیم با بررسی DOM تأیید کنیم که virtualization کار می‌کند. حتی با بارگذاری صدها ide، فقط تعداد کمی عنصر DOM وجود دارد.

شکل ۸.۸ — DOM لیست virtualized

همان‌طور که می‌بینیم، فقط آیتم‌های قابل مشاهده در viewport رندر شده‌اند. بقیه هنگام اسکرول کاربر ایجاد می‌شوند و DOM را لاغر نگه می‌دارند، صرف‌نظر از اینکه چند آیتم در لیست باشد.

## به‌روزرسانی‌های خوش‌بینانه (Optimistic updates) {#h1_228 .heading-1}

وقتی کاربر یک فرم ارسال می‌کند، معمولاً باید منتظر پاسخ سرور بماند تا تغییراتش را ببیند. این رفت و برگشت می‌تواند بسته به شرایط network و زمان پاسخ سرور وقت‌گیر باشد و اپلیکیشن را کُند نشان دهد. به نوشتن یک review فکر کنید: روی submit کلیک می‌کنیم، spinner بارگذاری می‌بینیم، منتظر سرور می‌مانیم، و بالاخره review خود را می‌بینیم. کار می‌کند، اما گاهی احساس کُندی دارد.

Optimistic updates این مشکل را با نمایش فوری نتیجهٔ مورد انتظار قبل از تأیید سرور حل می‌کنند. وقتی کاربر review ارسال می‌کند، آن را فوراً نشان می‌دهیم گویی قبلاً موفق شده. در بیشتر موارد، سرور عمل را چند لحظه بعد تأیید می‌کند و کاربر هرگز متوجه تفاوت نمی‌شود. اپلیکیشن فوری به نظر می‌رسد.

این رویکرد سه بخش دارد:

- به‌روزرسانی فوری — وقتی کاربر ارسال می‌کند، UI را به‌گونه‌ای به‌روزرسانی کنید که گویی درخواست موفق بوده
- مدیریت خطا — اگر درخواست ناموفق بود، به state قبلی برگردید و خطا نشان دهید
- همگام‌سازی با سرور — وقتی درخواست موفق بود، دادهٔ خوش‌بینانه را با دادهٔ واقعی سرور جایگزین کنید

بیایید optimistic updates را برای ایجاد reviews پیاده‌سازی کنیم.

ابتدا، به‌روزرسانی خوش‌بینانه را در `onMutate` مدیریت می‌کنیم. این callback قبل از ارسال درخواست به سرور اجرا می‌شود:

```typescript
// src/features/reviews/api/create-review.ts

export function useCreateReviewMutation({
  options,
  onSuccess,
  onError,
  ideaId,
}: {
  options?: Omit<
    UseMutationOptions<CreateReviewResponse, Error, CreateReviewData['body']>,
    'mutationFn'
  >;
  onSuccess: () => void;
  onError: () => void;
  ideaId: string;
}) {
  const queryClient = useQueryClient();
  const user = useUser();
  
  return useMutation({
    ...getCreateReviewMutationOptions(),
    ...options,
    onMutate: async (newReview) => {
      // Cancel any outgoing refetches to prevent them from overwriting our optimistic update
      await queryClient.cancelQueries({
        queryKey: reviewsQueryKeys.byIdea(ideaId),
      });
      await queryClient.cancelQueries({
        queryKey: reviewsQueryKeys.current(),
      });

      // Snapshot the previous values
      const previousReviewsByIdea =
        queryClient.getQueryData<GetReviewsByIdeaResponse>(
          reviewsQueryKeys.byIdea(ideaId),
        );
      const previousCurrentReviews =
        queryClient.getQueryData<GetReviewsByIdeaResponse>(
          reviewsQueryKeys.current(),
        );

      // Optimistically update the cache with the new review
      if (user) {
        const optimisticReview: Review = {
          id: `temp-${Date.now()}`,
          content: newReview.content,
          rating: newReview.rating,
          authorId: user.id,
          author: {
            username: user.username,
            email: user.email,
            id: user.id,
          },
          ideaId,
          createdAt: new Date().toISOString(),
          updatedAt: new Date().toISOString(),
        };

        queryClient.setQueryData<GetReviewsByIdeaResponse>(
          reviewsQueryKeys.byIdea(ideaId),
          (old) => ({
            data: old?.data
              ? [optimisticReview, ...old.data]
              : [optimisticReview],
          }),
        );

        queryClient.setQueryData<GetReviewsByIdeaResponse>(
          reviewsQueryKeys.current(),
          (old) => ({
            data: old?.data
              ? [optimisticReview, ...old.data]
              : [optimisticReview],
          }),
        );
      }

      // Return context with the previous values for rolling back if the request fails
      return { previousReviewsByIdea, previousCurrentReviews };
    },
    // ...
  });
}
```

بیایید بررسی کنیم چه اتفاقی می‌افتد. Callback `onMutate` قبل از شروع mutation اجرا می‌شود و به ما فرصت می‌دهد فوراً UI را به‌روزرسانی کنیم:

- لغو queryهای در حال اجرا — هر query که ممکن است reviews را دریافت کند لغو کنید. اگر این کار را نکنیم، یک refetch در حال انتظار که بعد از به‌روزرسانی خوش‌بینانهٔ ما رخ می‌دهد ممکن است آن را با دادهٔ قدیمی بازنویسی کند و باعث شود UI به‌طور مختصر برگردد.
- عکس‌برداری از مقادیر قبلی — دادهٔ کش فعلی را ذخیره کنید تا بتوانیم در صورت ناموفق بودن درخواست آن را بازیابی کنیم. این plan بازگشت ماست.
- ایجاد review خوش‌بینانه — شیء review بسازید که شبیه چیزی باشد که سرور برمی‌گرداند. از ID موقت استفاده کنید چون هنوز ID واقعی را نمی‌دانیم.
- به‌روزرسانی کش — review خوش‌بینانه را به کش اضافه کنید. React Query فوراً هر کامپوننتی که از این داده استفاده می‌کند را دوباره رندر می‌کند.
- برگرداندن context — مقادیر قبلی را برگردانید تا بتوانیم در `onError` در صورت نیاز به بازگشت به آن‌ها دسترسی داشته باشیم.

سپس خطاها و موفقیت را مدیریت می‌کنیم:

```typescript
export function useCreateReviewMutation({
  // ...
}) {
  const queryClient = useQueryClient();
  const user = useUser();
  
  return useMutation({
    // ...
    onError: (_err, _newReview, context) => {
      // Rollback to the previous values on error
      if (context?.previousReviewsByIdea) {
        queryClient.setQueryData(
          reviewsQueryKeys.byIdea(ideaId),
          context.previousReviewsByIdea,
        );
      }
      if (context?.previousCurrentReviews) {
        queryClient.setQueryData(
          reviewsQueryKeys.current(),
          context.previousCurrentReviews,
        );
      }
      onError();
    },
    onSuccess: () => {
      // Refetch to get the server data with the real ID
      queryClient.invalidateQueries({
        queryKey: reviewsQueryKeys.byIdea(ideaId),
      });
      queryClient.invalidateQueries({
        queryKey: reviewsQueryKeys.current(),
      });
      onSuccess();
    },
  });
}
```

اگر درخواست ناموفق بود، دادهٔ کش قبلی را از context که در `onMutate` برگرداندیم بازیابی می‌کنیم. review خوش‌بینانه ناپدید می‌شود و `onError` را فراخوانی می‌کنیم تا کامپوننت بتواند پیام خطا نشان دهد.

اگر درخواست موفق بود، queryها را invalidate می‌کنیم تا مجدداً از سرور دریافت شوند. این review خوش‌بینانهٔ ما (با ID موقت) را با review واقعی از سرور جایگزین می‌کند. کاربر متوجه این تعویض نمی‌شود چون refetch در پس‌زمینه انجام می‌شود و review در جای خود باقی می‌ماند، اما حالا ID صحیح و هرگونه تغییر سمت سرور را دارد.

تجربهٔ کاربری اکنون بسیار بهتر است. کاربران review خود را فوراً هنگام ارسال می‌بینند. اگر مشکلی پیش بیاید، review ناپدید می‌شود و پیام خطا می‌بینند. در بیشتر موارد، سرور عمل را تأیید می‌کند و کاربران حتی متوجه refetch پس‌زمینه‌ای که دادهٔ واقعی را جایگزین می‌کند نمی‌شوند.

## خلاصه {#h1_229 .heading-1}

در این فصل، چندین تکنیک برای سریع‌تر و واکنش‌گراتر کردن اپلیکیشن React ما بررسی کردیم. یاد گرفتیم چگونه از React DevTools برای شناسایی مشکلات performance استفاده کنیم و سپس آن مشکلات را با بهینه‌سازی‌های هدفمند حل کردیم.

با بهترین شیوه‌های مدیریت state شروع کردیم: قرار دادن state در نزدیک‌ترین محل و نگه‌داشتن کامپوننت‌ها متمرکز. وقتی state نزدیک محل استفاده‌اش باشد، کامپوننت‌های کمتری هنگام تغییر آن دوباره رندر می‌شوند.

سپس code splitting و lazy loading برای کاهش اندازهٔ bundle اولیه را بررسی کردیم. با بارگذاری on-demand کد، کاربران فقط آنچه برای صفحهٔ فعلی نیاز دارند را دانلود می‌کنند.

Streaming محتوای کُند در حین SSR به ما اجازه داد محتوا را به‌تدریج نشان دهیم به‌جای اینکه منتظر همهٔ داده‌ها بمانیم. محتوای سریع فوراً ظاهر می‌شود، در حالی که محتوای کُندتر در پس‌زمینه بارگذاری می‌شود.

Debouncing درخواست‌های API غیرضروری هنگام تایپ کردن کاربران در فیلدهای جستجو را کاهش داد. به‌جای ۹ درخواست برای تایپ «Education»، فقط ۱ درخواست پس از توقف کاربر ارسال می‌کنیم.

برای data setهای بزرگ، pagination را با list virtualization ترکیب کردیم. Pagination پاسخ‌های API را کوچک و سریع نگه می‌دارد. Virtualization DOM را لاغر نگه می‌دارد با رندر کردن فقط آیتم‌های قابل مشاهده، صرف‌نظر از اینکه کاربر چند آیتم بارگذاری کرده باشد.

در نهایت، optimistic updates mutationها را فوری نشان دادند با نمایش تغییرات قبل از تأیید سرور. UI فوراً به‌روزرسانی می‌شود و ما در پس‌زمینه با سرور همگام می‌شویم.

این تکنیک‌ها مکمل یکدیگر هستند. اپلیکیشن بهینه‌شده از ابزار درست برای هر موقعیت استفاده می‌کند: debouncing برای ورودی کاربر، pagination برای لیست‌های بزرگ، streaming برای دادهٔ کُند، و optimistic updates برای mutationها. هدف همیشه یکی است: اپلیکیشن را فوری و واکنش‌گرا برای کاربران احساس کنند.
