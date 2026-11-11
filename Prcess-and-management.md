<div dir="rtl">

# آزمایشگاه ۰۳: کالبدشکافی پروسه‌ها، فایل‌سیستم `/proc` و وضعیت‌های زامبی و یتیم

## ۱. تفاوت برنامه و پروسه و اتاق شیشه‌ای `/proc`
* **برنامه (Program):** فایلِ باینری و بی‌جان روی هارد دیسک (مثل `/bin/zsh`).
* **پروسه (Process):** کپیِ زنده و در حال اجرای برنامه داخلِ **RAM** که دارای شناسه‌ی یکتا (`PID`) و شناسه‌ی پدر (`PPID`) است.
* **فایل‌سیستم مجازی `/proc`:** این پوشه روی هارد دیسک نیست و مستقیم داخلِ RAM توسط کرنل ساخته می‌شود. به ازای هر پروسه‌ی زنده، یک پوشه به نام `PID` آن (مثلاً `/proc/3342/status`) وجود دارد که ابزارهایی مثل `ps` و `top` اطلاعاتشان را از روی آن می‌خوانند.

<div dir="ltr">

```bash
# Get current shell PID and inspect its live status in RAM
echo $$ head -n 10 /proc/$$/status
```

</div>

---

## ۲. حالت‌های پنج‌گانه‌ی پروسه (The 5 Process States)
در ستون `STAT` خروجی `ps` یا خط `State` در `/proc/PID/status`، هر پروسه در یکی از این ۵ وضعیت قرار دارد:

* **`R` (Running / Runnable):** در حال پردازش زیر دست CPU یا در صفِ اجرا.
* **`S` (Interruptible Sleep):** خوابِ سبک؛ ۹۹٪ پروسه‌های سیستم (مثل `zsh` یا مرورگر) که منتظر دریافت ورودی هستند در این حالت قرار دارند.
* **`D` (Uninterruptible Sleep):** خوابِ سنگین / کُمای سخت‌افزاری. پروسه به دلیل زدن `System Call` و انتظار برای جوابِ سخت‌افزار (مثل دیسک خراب یا پوشه‌ی شبکه‌ایِ قطع‌شده‌ی NFS) داخل `Kernel Space` گیر کرده است. این پروسه حتی با `kill -9` هم کشته نمی‌شود و باید مشکل سخت‌افزار یا Mount حل شود.
* **`T` (Stopped / Traced):** پروسه‌ای که توسط کاربر منجمد (Pause) شده است (با زدن `Ctrl + Z` یا سیگنال `STOP`).
* **`Z` (Zombie / `<defunct>`):** پروسه‌ی فرزند کارش تمام شده و مرده است (هیچ RAM و CPU مصرف نمی‌کند)، اما چون پروسه‌ی پدر (`PPID`) هنوز گواهی فوت (`Exit Status`) را تایید نکرده، نامش در جدول کرنل باقی مانده است.

---

## ۳. پروسه‌ی یتیم (Orphan) در برابر زامبی (Zombie) و دستورات شکار

* **پروسه‌ی یتیم (Orphan):** پروسه‌ی پدر مرده اما فرزند هنوز زنده است و کار می‌کند. کرنل بلافاصله سرپرستی فرزند (`PPID`) را به `systemd` (`PID = 1` یا سرویس `systemd --user`) می‌سپارد.
* **نحوه‌ی کشتن زامبی:** دستور `kill -9` روی خودِ پروسه‌ی زامبی کار نمی‌کند چون از قبل مرده است. برای پاک کردن زامبی از جدول، باید **پروسه‌ی پدر (`PPID`)** را بکشیم.

<div dir="ltr">

```bash
# View all processes with CPU/MEM usage and STAT column (BSD style)
ps aux

# View all processes with PID and Parent PID (PPID) columns (UNIX style)
ps -ef

# Display process family tree with PIDs
pstree -p

# Hunt down Zombie processes
ps aux | grep -E "Z|defunct"
```

</div>

</div>
