import logging
from aiogram import Bot, Dispatcher, types
from aiogram.utils import executor
from aiogram.types import InlineKeyboardButton, InlineKeyboardMarkup
import os

API_TOKEN = '8152203224:AAGz8ngFOyNC8x5LYI4zDNJ4eKINx_OMZRU'
ADMIN_IDS = [7730254429]  # Firdavs ID
CHANNEL_ID = -1001234567890  # Obuna uchun kanal ID

# Fayllar bazasi
movies = {}  # {'avatar': 'file_id', '1234': 'file_id'}

# Log
logging.basicConfig(level=logging.INFO)
bot = Bot(token=API_TOKEN)
dp = Dispatcher(bot)

# Obuna tekshirish
async def check_subscribe(user_id):
    try:
        member = await bot.get_chat_member(CHANNEL_ID, user_id)
        return member.status in ["member", "creator", "administrator"]
    except:
        return False

# Start komandasi
@dp.message_handler(commands=['start'])
async def start_handler(msg: types.Message):
    if await check_subscribe(msg.from_user.id):
        await msg.answer("🎬 Salom! Kino nomini yoki kodini kiriting:")
    else:
        btn = InlineKeyboardMarkup().add(
            InlineKeyboardButton("🔔 Obuna bo'lish", url='https://t.me/YOUR_CHANNEL_USERNAME')
        )
        await msg.answer("❗ Botdan foydalanish uchun kanalga obuna bo‘ling.", reply_markup=btn)

# Kino qidirish
@dp.message_handler(content_types=types.ContentType.TEXT)
async def search_handler(msg: types.Message):
    if not await check_subscribe(msg.from_user.id):
        return await start_handler(msg)
    
    key = msg.text.lower()
    if key in movies:
        await msg.answer_video(movies[key], caption=f"🎬 Topildi: {key}")
    else:
        await msg.answer("❌ Bunday kino topilmadi.")

# Kino yuklash (faqat admin)
@dp.message_handler(content_types=types.ContentType.VIDEO)
async def upload_movie(msg: types.Message):
    if msg.from_user.id not in ADMIN_IDS:
        return
    if not msg.caption:
        return await msg.reply("❗ Kino nomi yoki kodi caption’da yozilmagan.")
    
    key = msg.caption.lower()
    movies[key] = msg.video.file_id
    await msg.reply(f"✅ Kino yuklandi: {key}")

# Run
if __name__ == '__main__':
    executor.start_polling(dp)
