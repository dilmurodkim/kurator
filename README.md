# main.py
import logging
import os
from aiogram import Bot, Dispatcher, types
from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton, ReplyKeyboardMarkup, KeyboardButton
from aiogram.utils.executor import start_webhook
from dotenv import load_dotenv

from data.hangeul import hangeul_letters_data
from data.grammar import grammar_1A, grammar_1B

load_dotenv()

API_TOKEN = os.getenv("BOT_TOKEN")
ADMIN_ID = int(os.getenv("ADMIN_ID"))

WEBHOOK_HOST = f"https://{os.getenv('RENDER_EXTERNAL_HOSTNAME')}"
WEBHOOK_PATH = "/webhook"
WEBHOOK_URL = f"{WEBHOOK_HOST}{WEBHOOK_PATH}"

WEBAPP_HOST = "0.0.0.0"
WEBAPP_PORT = int(os.getenv("PORT", 8000))

logging.basicConfig(level=logging.INFO)

bot = Bot(token=API_TOKEN)
dp = Dispatcher(bot)

main_menu = ReplyKeyboardMarkup(resize_keyboard=True)
main_menu.add(
    KeyboardButton("\U0001F4DA TOPIK 1"),
    KeyboardButton("\U0001F4D6 \uc11c\uc6b8\ub300 \ud55c\uad6d\uc5b4 1A/1B"),
    KeyboardButton("\u2600\ufe0f Harflar"),
    KeyboardButton("\U0001F48E Premium darslar")
)

@dp.message_handler(commands=["start"])
async def start_handler(message: types.Message):
    await message.answer(
        "Assalomu alaykum!\nKareys tili o'rgatadigan botga xush kelibsiz.\nQuyidagi menylardan birini tanlang:",
        reply_markup=main_menu
    )

@dp.message_handler(lambda message: message.text == "\u2600\ufe0f Harflar")
async def show_letter_menu(message: types.Message):
    markup = InlineKeyboardMarkup(row_width=4)
    for harf in hangeul_letters_data.keys():
        markup.insert(InlineKeyboardButton(harf, callback_data=f"harf_{harf}"))
    markup.add(InlineKeyboardButton("\u2B05\ufe0f Orqaga", callback_data="back_to_main"))
    await message.answer("Quyidagi harflardan birini tanlang:", reply_markup=markup)

@dp.callback_query_handler(lambda c: c.data.startswith("harf_"))
async def show_letter_info(callback: types.CallbackQuery):
    harf = callback.data.replace("harf_", "")
    matn = hangeul_letters_data.get(harf, "Ma’lumot topilmadi")
    markup = InlineKeyboardMarkup().add(InlineKeyboardButton("\u2B05\ufe0f Orqaga", callback_data="back_to_letters"))
    await callback.message.edit_text(f"\u2600\ufe0f {harf}\n{matn}", reply_markup=markup)
    await callback.answer()

@dp.callback_query_handler(lambda c: c.data == "back_to_letters")
async def back_to_letters(callback: types.CallbackQuery):
    markup = InlineKeyboardMarkup(row_width=4)
    for harf in hangeul_letters_data.keys():
        markup.insert(InlineKeyboardButton(harf, callback_data=f"harf_{harf}"))
    markup.add(InlineKeyboardButton("\u2B05\ufe0f Orqaga", callback_data="back_to_main"))
    await callback.message.edit_text("Quyidagi harflardan birini tanlang:", reply_markup=markup)
    await callback.answer()

@dp.message_handler(lambda message: message.text == "\U0001F4D6 \uc11c\uc6b8\ub300 \ud55c\uad6d\uc5b4 1A/1B")
async def show_books(message: types.Message):
    markup = InlineKeyboardMarkup(row_width=2)
    markup.add(
        InlineKeyboardButton("1A \U0001F4DA", callback_data="book_1A"),
        InlineKeyboardButton("1B \U0001F4D6", callback_data="book_1B"),
        InlineKeyboardButton("\u2B05\ufe0f Orqaga", callback_data="back_to_main")
    )
    await message.answer("Sizga qaysi kitob kerak:", reply_markup=markup)

@dp.callback_query_handler(lambda c: c.data == "back_to_main")
async def back_to_main(callback: types.CallbackQuery):
    await bot.send_message(callback.from_user.id, "Assalomu alaykum! Tanlang:", reply_markup=main_menu)
    await callback.message.delete()
    await callback.answer()

@dp.callback_query_handler(lambda c: c.data == "book_1A")
async def show_1a_menu(callback: types.CallbackQuery):
    markup = InlineKeyboardMarkup(row_width=1)
    for key in grammar_1A:
        markup.add(InlineKeyboardButton(key, callback_data=key))
    markup.add(InlineKeyboardButton("\u2B05\ufe0f Orqaga", callback_data="show_books_menu"))
    await callback.message.edit_text("1A grammatikalaridan birini tanlang:", reply_markup=markup)
    await callback.answer()

@dp.callback_query_handler(lambda c: c.data == "book_1B")
async def show_1b_menu(callback: types.CallbackQuery):
    markup = InlineKeyboardMarkup(row_width=1)
    for key in grammar_1B:
        markup.add(InlineKeyboardButton(key, callback_data=key))
    markup.add(InlineKeyboardButton("\u2B05\ufe0f Orqaga", callback_data="show_books_menu"))
    await callback.message.edit_text("1B grammatikalaridan birini tanlang:", reply_markup=markup)
    await callback.answer()

@dp.callback_query_handler(lambda c: c.data == "show_books_menu")
async def show_books_menu(callback: types.CallbackQuery):
    markup = InlineKeyboardMarkup(row_width=2)
    markup.add(
        InlineKeyboardButton("1A \U0001F4DA", callback_data="book_1A"),
        InlineKeyboardButton("1B \U0001F4D6", callback_data="book_1B"),
        InlineKeyboardButton("\u2B05\ufe0f Orqaga", callback_data="back_to_main")
    )
    await callback.message.edit_text("Sizga qaysi kitob kerak:", reply_markup=markup)
    await callback.answer()

@dp.callback_query_handler(lambda c: c.data in grammar_1A)
async def show_1a_grammar(callback: types.CallbackQuery):
    key = callback.data
    text = grammar_1A.get(key, "Ma'lumot topilmadi")
    markup = InlineKeyboardMarkup().add(InlineKeyboardButton("\u2B05\ufe0f Orqaga", callback_data="book_1A"))
    await callback.message.edit_text(f"\U0001F4D8 {text}", reply_markup=markup)
    await callback.answer()

@dp.callback_query_handler(lambda c: c.data in grammar_1B)
async def show_1b_grammar(callback: types.CallbackQuery):
    key = callback.data
    text = grammar_1B.get(key, "Ma'lumot topilmadi")
    markup = InlineKeyboardMarkup().add(InlineKeyboardButton("\u2B05\ufe0f Orqaga", callback_data="book_1B"))
    await callback.message.edit_text(f"\U0001F4D7 {text}", reply_markup=markup)
    await callback.answer()

# === WEBHOOK START ===
async def on_startup(dp):
    await bot.set_webhook(WEBHOOK_URL)
    print("\u2705 Webhook o‘rnatildi:", WEBHOOK_URL)

async def on_shutdown(dp):
    await bot.delete_webhook()
    print("\u274C Webhook o‘chirildi")

if __name__ == '__main__':
    start_webhook(
        dispatcher=dp,
        webhook_path=WEBHOOK_PATH,
        on_startup=on_startup,
        on_shutdown=on_shutdown,
        skip_updates=True,
        host=WEBAPP_HOST,
        port=WEBAPP_PORT,
    )
