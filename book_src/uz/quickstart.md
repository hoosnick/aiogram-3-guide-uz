---
title: Aiogram bilan tanishuv
description: Aiogram bilan tanishuv
---

# Aiogram bilan tanishuv

!!! info ""
    Amaldagi aiogram versiyasi: 3.1.1

!!! warning "Ba'zi tafsilotlar ataylab soddalashtirilgan!"

    Ushbu kitob muallifi nazariyadan tashqari amaliyot ham bo'lishi kerakligini ta'kidlaydi. Quyidagi kodlarni iloji boricha soddalashtirish uchun biz faqat local (mahalliy) dasturlash va o'qitish uchun mos keladigan yondashuvlardan foydalanishimizga to'g'ri keldi.

    Masalan, deyarli barcha boblarda bot tokeni to'g'ridan-to'g'ri kodlarda ko'rsatilgan. Bu noto'g'ri yondashuv, chunki agar siz kodni Public Repo (masalan, GitHub)ga yuborishdan oldin uni olib tashlashni unutib qo'ysangiz, bu nimalarga olib kelishini o'zingiz bilasiz.

    Yoki ba'zan ma'lumotlar faqat operativ xotirada joylashgan tuzilmalar (lug'atlar, listlar...)da saqlangan. Aslida, bu ham xato yondashuv, chunki botni to'xtatish bilan ma'lumotlarning qaytarib bo'lmaydigan qilib yuqolishiga olib keladi.

    Telegramdan updatelarni olish mexanizmi sifatida polling tanlangan, chunki u ko'pchilik muhitlarda ishlaydi va deyarli barcha dasturchilarga mos keladi.

    **Shuni yodda tutish kerakki, muallif o'z oldiga "Computer Science"ni barcha xilma-xilligi bilan emas, balki aiogram yordamida Telegram Bot API bilan qanday ishlashni aniq tushuntirishni maqsad qilgan.**

## Terminologiya {: id="glossary" }

Boblarda ishlatilgan atama va terminlar:

- PM — private message (ЛС - личные сообщения), shaxsiy xabar, foydalanuvchi bilan yakkama-yakka suhbat (bot bilan foydalanuvchi,foydalanuvchi bilan foydalanuvchi).
- Chat — PM, guruhlar, kanallarning umumiy nomi.
- Update (yangilanish) — ushbu [ro'yxat](https://core.telegram.org/bots/api#update)dagi har qanday hodisa:
  xabar, xabarni tahrirlash, callback, inline so'rov, to'lov, guruhga bot qo'shish va h.k.
- Handler — dispetcher/routerdan yangilanishni oladigan va uni qayta ishlovchi asinxron funksiya.
- Dispetcher — Telegramdan yangilanishlarni qabul qiladigan va qabul qilingan yangilanish bilan ishlash uchun qaysi handler chaqirilishini hal qiluvchi obyekt.
- Router — dispetcherga o'xshash, lekin handlerlar to'plami uchun javobgar.
- Filtr — odatda True yoki False qaytaradigan va handlerning chaqirilishi yoki chaqirilmasligiga ta'sir qiluvchi ifoda.
- Middleware — updatelar bilan ishlashdagi qatlam.

## O'rnatish {: id="installation" }

Birinchi bo'lib bot uchun "папка" yaratamiz, va u erda virtual muhit (venv) ochib, aktivlashtiramiz va [aiogram](https://github.com/aiogram/aiogram) kutubxonasini o'rnatamiz.
Python 3.9 versiyasi o'rnatilganligini tekshirib ko'ramiz (agar sizda 3.9 yoki undan yuqori versiya o'rnatilganligini bilsangiz, bu qismni o'tkazib yuborishingiz mumkin):


```plain
[groosha@main lesson_01]$ python3.9
Python 3.9.9 (main, Jan 11 2022, 16:35:07)
[GCC 11.1.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
[groosha@main lesson_01]$
```

Endi `requirements.txt` faylini yaratamiz, unda biz foydalanayotgan aiogram versiyasini ko'rsatamiz. Konfiguratsiya fayllari uchun bizga python-dotenv kutubxonasi ham kerak bo'ladi.

!!! important "Aiogram versiyalari haqida"
    Bu bobda aiogram **3.x** versiyasi qo‘llaniladi, lekin boshlashdan oldin kutubxonaning [relizlar kanalini](https://t.me/aiogram_live) ko‘rib chiqishingizni va yangiroq versiyasini tekshirishingizni tavsiya qilaman. 3 raqami bilan boshlanadigan har qanday relizni ishlatishingiz mumkin, chunki aiogram 2.x eskirgan va ancha farq qiladi.

```plain
[groosha@main 01_quickstart]$ python3.9 -m venv venv
[groosha@main 01_quickstart]$ echo "aiogram<4.0" > requirements.txt
[groosha@main 01_quickstart]$ echo "python-dotenv==1.0.0" >> requirements.txt
[groosha@main 01_quickstart]$ source venv/bin/activate
(venv) [groosha@main 01_quickstart]$ pip install -r requirements.txt
# ...bu yerda installation haqida bir necha qatorlar...
Successfully installed ...bu yerda ham...
[groosha@main 01_quickstart]$
```

Terminaldagi `venv` prefiksiga e'tibor bering. Bu bizning `venv` nomli virtual muhitida ekanligimizdan dalolat beradi.
Keling, venv ichida "python" buyrug'ini chaqiramiz (venv ichidagi python versiyasi ham global versiya bilan bir xil ekanligini ko'ramiz):

```plain
(venv) [groosha@main 01_quickstart]$ python
Python 3.9.9 (main, Jan 11 2022, 16:35:07)
[GCC 11.1.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
(venv) [groosha@main 01_quickstart]$ deactivate
[groosha@main 01_quickstart]$
```

Oxirgi `deactivate` buyrug'i bilan biz virtual muhitimizda chiqdik.

!!! info ""
    Agar siz botlarni yozish uchun PyCharmdan foydalansangiz, telegram obyektlarida kodga type-hintlarni qo‘llab-quvvatlash uchun [Pydantic](https://plugins.jetbrains.com/plugin/12861-pydantic) plaginini ham o‘rnatishni tavsiya qilaman.

## Birinchi bot {: id="hello-world" }

Keling, basic aiogram bot shabloniga ega `bot.py` faylini yarataylik:

```python title="bot.py"
import asyncio
import logging
from aiogram import Bot, Dispatcher, types
from aiogram.filters.command import Command

# Muhim xabarlarni loglarda o'tkazib yubormaslik uchun
logging.basicConfig(level=logging.INFO)
# Bot obyekti
bot = Bot(token="12345678:AaBbCcDdEeFfGgHh")
# Dispetcher
dp = Dispatcher()

# /start buyrug'i uchun handler
@dp.message(Command("start"))
async def cmd_start(message: types.Message):
    await message.answer("Hello!")

# Yangilanishlar uchun pollingni boshlash
async def main():
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
```

Shuni ta'kidlash kerakki, aiogram asinxron kutubxonadir, shuning uchun sizning funksiyalaringiz ham asinxron bo'lishi kerak, va API metodlarini chaqirishdan oldin siz **await** kalit so'zini qo'yishingiz kerak, chunki bu "chaqirish"lar [korutinalar](https://docs.python.org/3/library/asyncio-task.html#coroutines) qaytaradi.

!!! info "Pythonda asinxron dasturlash"
    Rasmiy "документация"larni e'tiborsiz qoldirmang! 
    [Python web-saytida](https://docs.python.org/3/library/asyncio-task.html) asyncio uchun qo'llanma mavjud.

Agar siz pyTelegramBotAPI kabi kutubxona bilan ishlagan bo'lsangiz, unda handler kontseptsiyasi sizga tanish, yagona farq shundaki, aiogramda handlerlar dispetcher tomonidan boshqariladi. Dispetcher handlerlar funksiyalarini ro'yxatdan o'tkazadi va ularni updatega qarab filtrlar orqali tartiblaydi mos kelmaydiganlarini ro'yxatdan olib tashlaydi. Qisqasi Telegramdan yangilanish (update) kelgach, qaysi handlerga borishi kerakligini tanlaydi.
Filtrga misol: «chat_id X va rasm caption uzunligi Y bo'lgan xabarlarni qayta ishlash».
Agar ikkita handler, bir xil mantiqiy filtrlarga ega bo'lsa, ro'yxatdan birinchi o'tgani chaqiriladi.

Funksiyani handler sifatida ro'yxatdan o'tkazish uchun, ikki xil usul mavjud:

1. Yuqoridagi misolda bo'lgani kabi, unga [decorator](https://devpractice.ru/python-lesson-19-decorators/) biriktirish.
2. To'g'ridan-to'g'ri dispetcherda ro'yxatdan o'tkazish.

Quyidagi kodni ko'ring:

```python
# /test1 buyrug'i uchun decorator biriktirilgan handler
@dp.message(Command("test1"))
async def cmd_test1(message: types.Message):
    await message.reply("Test 1")

# /test2 buyrug'i uchun handler
async def cmd_test2(message: types.Message):
    await message.reply("Test 2")
```

Keling, ular bilan botni ishga tushiramiz:
![/test2 buyrug'i ishlamaydi](images/quickstart/l01_1.jpg)

`/test2` buyrug'i uchun javobgar handler `cmd_test2` funksiyasi ishlamaydi, chunki dispetcher bu haqda bilmaydi. Keling, ushbu xatoni tuzatamiz va funksiyani ro'yxatdan o'tkazamiz:

```python
# /test2 buyrug'i uchun handler
async def cmd_test2(message: types.Message):
    await message.reply("Test 2")

# Boshqa joyda... Masalan, main() funksiyasida
dp.message.register(cmd_test2, Command("test2"))
```

Botni qayta ishga turshuramiz:
![Ikkala buyruq ham ishlaydi](images/quickstart/l01_2.jpg)

## Sintaktik shakar {: id="sugar" }

Kodni yanada toza yozilishi va o'qilishi uchun aiogram standart Telegram metodlarining imkoniyatlarini kengaytirgan.
Masalan, `bot.send_message(...)` o'rniga `message.answer(...)` yoki `message.reply(...)` yozishingiz mumkin. Oxirgi ikki holatda `chat_id`ni ko'rsatish shart emas, u `updade`dan o'zi oladi. `answer` va `reply` o'rtasidagi farq oddiy:  
Birinchi metod shunchaki chatga xabar yuboradi, ikkinchisi xabarga javob qaytaradi:

```python
@dp.message(Command("answer"))
async def cmd_answer(message: types.Message):
    await message.answer("Это простой ответ") # bu xabar yuboradi


@dp.message(Command("reply"))
async def cmd_reply(message: types.Message):
    await message.reply('Это ответ с "ответом"') # bu xabarga javob qaytaradi
```

![message.answer() va message.reply() o'rtasidagi farq](images/quickstart/l01_3.jpg)

Bundan tashqari, ko'pgina xabar turlari uchun `answer_{type}` yoki `reply_{type}` kabi yordamchi metodlar mavjud, ya'ni `.answer_photo(...)` yoki `.reply_video(...)` masalan:

```python
@dp.message(Command("dice"))
async def cmd_dice(message: types.Message):
    await message.answer_dice(emoji="🎲")
```

!!! info "'message: types.Message' nimani anglatadi ?"
    Python - [kuchli ammo dinamik turlanish](https://habr.com/ru/post/161205/)ga ega bo'lgan interpretor til, shuning uchun C++ yoki Javadagi kabi avvaldan ma'lum bir o'zgaruvchiga uni turini belgilab qo'yish mavjud emas. Biroq, 3.5-versiyadan boshlab, pythondagi variablelar turlariga ["hint(подсказка)"](https://docs.python.org/3/library/typing.html)lar berishni qo'llab-quvvatlashni boshladi, buning natijasida PyCharm kabi turli "checker" va IDElar variablelar turlarini tahlil qiladi va dasturchiga biror narsa noto'g'ri bo'lsa, xabar beradi.  
    Bizning holatda, `types.Message` hint(ko'rsatmasi) PyCharm’ga `message` o‘zgaruvchisi aiogram kutubxonasining `types` modulida aniqlanganidek `Message` turida ekanligini bildiradi (kod boshida importga qarang). Bu IDEga atributlar va funksiyalarni tezda taklif qilish imkonini beradi.

`/dice` buyrug'ini chaqirganda, bot o'sha chatga dice(🎲) yuboradi. Albatta, agar siz uni boshqa chatga yuborishingiz kerak bo‘lsa, eski uslubda `await bot.send_dice(...)`ga murojaat qilishingiz kerak bo‘ladi. Lekin `bot` obyekti (Bot classining ekzemplyari) ma'lum bir funksiyangiz bor modulda mavjud bo'lmasligi mumkin, ya'ni botingizni shabloniga asoslansak, `bot` obyekti ishga tushuruvchi fayldagina bo'ladi. Yaxshiyamki, bot obyekti barcha turdagi updatelarda mavjud: Message, CallbackQuery, InlineQuery va boshqalarda. Aytaylik, siz `/dice` buyrug'i yordamida 🎲ni suhbatlashiyotgan chatga emas, balki IDsi -100123456789 bo'lgan kanalga yubormoqchisiz. Oldingi funksiyani qayta yozamiz:

```python
# import qilish haqida unutmang
from aiogram.enums.dice_emoji import DiceEmoji

@dp.message(Command("dice"))
async def cmd_dice(message: types.Message, bot: Bot):
    await bot.send_dice(-100123456789, emoji=DiceEmoji.DICE)
```

## Qo'shimcha parametrlar {: id="pass-extras" }

Ba'zan, botni ishga tushirishda siz bir yoki bir nechta qo'shimcha qiymatlarni o'tkazishingiz kerak bo'lishi mumkin.
Bu o'zgaruvchi, konfiguratsiya obyekti, biron ro'yxat, vaqt yoki boshqa narsa bo'lishi mumkin.
Buni amalga oshirish uchun ushbu ma'lumotlarni dispetcherga - nomli argumentlar (kwargs - keyword arguments) sifatida uzatish kifoya.

Bu xususiyat botning ishlashi davomida o'zgarmasligi kerak bo'lgan obyektlarni uzatish uchun juda mos keladi (ya'ni, faqat o'qish uchun bo'lganlar). Agar qiymat vaqt o'tishi bilan o'zgarishi kerak bo'lsa, unda faqat [o'zgaruvchan obyektlar](https://mathspp.com/blog/pydonts/pass-by-value-reference-and-assignment) bilan ishlashini unutmang.

Handlerlarda qiymatlarni olish uchun ularni argument sifatida taqdim etish kifoya. Keling, bir misolni ko'rib chiqaylik:

```python
# Boshqa joyda...
# Masalan, dasturingiz boshida
from datetime import datetime

# bot = ...
dp = Dispatcher()
dp["started_at"] = datetime.now().strftime("%Y-%m-%d %H:%M")
await dp.start_polling(bot, mylist=[1, 2, 3])


@dp.message(Command("add_to_list"))
async def cmd_add_to_list(message: types.Message, mylist: list[int]):
    mylist.append(7)
    await message.answer("Добавлено число 7")


@dp.message(Command("show_list"))
async def cmd_show_list(message: types.Message, mylist: list[int]):
    await message.answer(f"Ваш список: {mylist}")


@dp.message(Command("info"))
async def cmd_info(message: types.Message, started_at: str):
    await message.answer(f"Бот запущен {started_at}")
```

Endi `start_at` o'zgaruvchisi va `mylist` ro'yxatini turli handlerlarda o'qish va yozish mumkin. Agar siz har bir update uchun unikal qiymatlarni yuborishingiz kerak bo'lsa (masalan, DB seansi obyektini), u holda [middlewarelar](filters-and-middlewares.md#middlewares) bobini tekshiring.

![Аргумент mylist может быть изменён между вызовами](images/quickstart/extra-args.png)

## Konfiguratsiya fayllari

Botning tokenini kodda saqlamaslik uchun uni alohida konfiguratsiya fayliga olib o'tishimiz mumkin. Prodakshnda o'zgaruvchilarini environmentni o'zida operatsion tizimga qarab belgilab qo'yishimiz [yetarli](https://t.me/advice17/26), ammo ushbu kitobda biz o'quvchilarni vaqtini tejash hamda osonroq bo'lishi uchun `.env` alohida faylidan foydalanamiz.

Shunday qilib, keling, `bot.py` yonida quyidagi kabiga ega `config_reader.py` alohida fayl yarataylik:

```python title="config_reader.py"
from pydantic_settings import BaseSettings, SettingsConfigDict
from pydantic import SecretStr


class Settings(BaseSettings):
    # bot tokeni kabi nozik ma'lumotlar uchun
    # str o'rniga SecretStr dan foydalanish tavsiya etiladi
    bot_token: SecretStr

    model_config = SettingsConfigDict(env_file='.env', env_file_encoding='utf-8')


# Faylni import qilishda config obyekti yaratiladi va tasdiqlanadi, 
# keyin uni turli joylardan import qilish mumkin
config = Settings()
```

Endi `bot.py`-ni biroz tahrirlaymiz:

```python title="bot.py"
# importlar
from config_reader import config

# Secret* turidagi o'zgaruvchilarda
# '********' o'rniga haqiqiy qiymatni olish uchun
# get_secret_value() metodini chaqirish kerak
bot = Bot(token=config.bot_token.get_secret_value())
```

Nihoyat, keling, `.env` faylini yaratamiz (boshida nuqta bilan), unda biz bot tokenini yozamiz:

```title=".env"
BOT_TOKEN = 0000000000:AaBbCcDdEeFfGgHhIiJjKkLlMmNn
```

Agar hamma narsa to'g'ri bajarilgan bo'lsa, ishga tushirilganda python-dotenv o'zgaruvchilarni `.env` faylidan yuklaydi, pydantic ularni tekshiradi va kerakli token bilan bot obyekti muvaffaqiyatli yaratiladi.

Shu yerda biz kutubxonaga kirishni yakunlaymiz va keyingi boblarda aiogram va Telegram Bot API ning boshqa “xususiyatlari”ni ko‘rib chiqamiz.
