## ۱. نصب Node.js و MySQL
نصب Node.js و npm
```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install nodejs npm -y
```
## بررسی نصب
```
node -v
```
```
npm -v
```
## ۲. ساخت پروژه Node.js
یک پوشه برای پروژه بسازید و وارد آن شوید
```
mkdir telegram-bot && cd telegram-bot
```
```
npm init -y
```
## نصب کتابخانه‌های مورد نیاز
```
npm install mysql2 node-telegram-bot-api dotenv
```
## ۳. کدنویسی ربات تلگرام
ایجاد فایل .env
```
nano .env
```

اطلاعات زیر را اضافه کنید
```
BOT_TOKEN=توکن_ربات_شما
```
ایجاد فایل index.js
```
nano index.js
```
کدنویسی ربات
کد زیر را در فایل index.js وارد کنید
```
require('dotenv').config();
const TelegramBot = require('node-telegram-bot-api');
const mysql = require('mysql2');

// تنظیمات دیتابیس
const db = mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: 'رمزعبور_دیتابیس',
    database: 'movie_bot',
});

// اتصال به دیتابیس
db.connect(err => {
    if (err) throw err;
    console.log('Connected to MySQL!');
});

// تنظیمات ربات تلگرام
const bot = new TelegramBot(process.env.BOT_TOKEN, { polling: true });

bot.on('message', async msg => {
    const chatId = msg.chat.id;
    const text = msg.text;

    if (text.startsWith('https://www.imdb.com/')) {
        db.query(
            'SELECT file_path FROM movies WHERE imdb_link = ?',
            [text],
            (err, results) => {
                if (err) {
                    bot.sendMessage(chatId, 'خطایی رخ داده است.');
                    console.error(err);
                    return;
                }

                if (results.length > 0) {
                    const filePath = results[0].file_path;
                    bot.sendDocument(chatId, filePath);
                } else {
                    bot.sendMessage(chatId, 'متأسفم، این فیلم موجود نیست.');
                }
            }
        );
    } else {
        bot.sendMessage(chatId, 'لینک IMDb را ارسال کنید.');
    }
});
```
۵. اجرای پروژه
اجرای ربات
```
node index.js
```
۶. تست ربات
در تلگرام به ربات خود پیام دهید.
لینک IMDb فیلم مورد نظر را ارسال کنید.
اگر فایل مرتبط در دیتابیس موجود باشد، ربات آن را ارسال می‌کند؛ در غیر این صورت، پیامی مبنی بر عدم وجود فایل نمایش داده می‌شود.
۷. بهبود و امنیت
تنظیم دسترسی به MySQL: مطمئن شوید که MySQL فقط از localhost قابل دسترسی است.
مدیریت خطاها: پیام‌های خطا را به گونه‌ای مدیریت کنید که اطلاعات حساس را فاش نکنند.
بهینه‌سازی جستجو: برای جستجوهای سریع‌تر، ستون imdb_link را ایندکس کنید:
sql
Copy code
CREATE UNIQUE INDEX idx_imdb_link ON movies (imdb_link);
