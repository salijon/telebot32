import asyncio
import logging
import sys
from os import getenv

from aiogram import Bot, Dispatcher
from aiogram.client.default import DefaultBotProperties
from aiogram.enums import ParseMode
from aiogram.filters import CommandStart, Command
from aiogram.types import (
    Message, ReplyKeyboardMarkup, KeyboardButton
)
from aiogram.fsm.state import State, StatesGroup
from aiogram.fsm.context import FSMContext

TOKEN = "8427960259:AAF2TeI8YsoQoj8ysBM4SjWdUDUfhOkudCA"
CHANNEL_ID = -1002541363537 # 👉 o'zingizning kanal ID ni yozing

dp = Dispatcher()


# ================= STATES =================
class Form(StatesGroup):
    ism = State()
    familiya = State()
    texnologiya = State()
    narx = State()  
    maqsad = State()
    tel = State()   
    username = State()
    confirm = State()


# ================= KEYBOARDS =================
main_keyboard = ReplyKeyboardMarkup(
    keyboard=[
        [KeyboardButton(text="Sherik kerak"), KeyboardButton(text="Ish joyi kerak")],
        [KeyboardButton(text="Hodim kerak"), KeyboardButton(text="Ustoz kerak")],
        [KeyboardButton(text="Shogird kerak")]
    ],
    resize_keyboard=True
)

confirm_keyboard = ReplyKeyboardMarkup(
    keyboard=[
        [KeyboardButton(text="✅ Ha"), KeyboardButton(text="❌ Yo‘q")]
    ],
    resize_keyboard=True
)


# ================= COMMANDS =================
@dp.message(CommandStart())
async def start_handler(message: Message):
    await message.answer(
        "Assalom alaykum 💻\n"
        "UstozShogird rasmiy botiga xush kelibsiz!\n\n"
        "/help — yordam\n"
      ,
        reply_markup=main_keyboard
    )


@dp.message(Command("help"))
async def help_handler(message: Message):
    await message.answer(
        "📌 Bot orqali siz e’lon qoldira olasiz.\n"
        "Tugmalardan birini bosing va ma’lumotlarni kiriting."
    )





# ================= START FORM =================
@dp.message(lambda m: m.text in [
    "Sherik kerak", "Ish joyi kerak", "Hodim kerak", "Ustoz kerak", "Shogird kerak"
])
async def start_form(message: Message, state: FSMContext):
    await state.update_data(turi=message.text)
    await message.answer("Ismingizni kiriting:")
    await state.set_state(Form.ism)


@dp.message(Form.ism)
async def get_ism(message: Message, state: FSMContext):
    await state.update_data(ism=message.text)
    await message.answer("Familiyangizni kiriting:")
    await state.set_state(Form.familiya)


@dp.message(Form.familiya)
async def get_familiya(message: Message, state: FSMContext):
    await state.update_data(familiya=message.text)
    await message.answer("Texnologiya / Yo‘nalish:")
    await state.set_state(Form.texnologiya)


@dp.message(Form.texnologiya)
async def get_texnologiya(message: Message, state: FSMContext):
    await state.update_data(texnologiya=message.text)
    await message.answer("Narx:")
    await state.set_state(Form.narx)


@dp.message(Form.narx)
async def get_narx(message: Message, state: FSMContext):
    await state.update_data(narx=message.text)
    await message.answer("Maqsadingiz:")
    await state.set_state(Form.maqsad)


@dp.message(Form.maqsad)
async def get_maqsad(message: Message, state: FSMContext):
    await state.update_data(maqsad=message.text)
    await message.answer("Telefon raqamingiz:")
    await state.set_state(Form.tel)


@dp.message(Form.tel)
async def get_tel(message: Message, state: FSMContext):
    await state.update_data(tel=message.text)
    await message.answer("Telegram username (@siz):")
    await state.set_state(Form.username)


@dp.message(Form.username)
async def get_username(message: Message, state: FSMContext):
    await state.update_data(username=message.text)
    data = await state.get_data()

    text = (
        f"📌 Tekshirib chiqing:\n\n"
        f"👤 Ism: {data['ism']}\n"
        f"👤 Familiya: {data['familiya']}\n"
        f"💼 Turi: {data['turi']}\n"
        f"🛠 Texnologiya: {data['texnologiya']}\n"
        f"💰 Narx: {data['narx']}\n"
        f"🎯 Maqsad: {data['maqsad']}\n"
        f"📞 Tel: {data['tel']}\n"
        f"🔗 Username: {data['username']}"
    )

    await message.answer(text, reply_markup=confirm_keyboard)
    await state.set_state(Form.confirm)


# ================= CONFIRM =================
@dp.message(Form.confirm)
async def confirm_handler(message: Message, state: FSMContext, bot: Bot):
    if message.text == "✅ Ha":
        data = await state.get_data()

        post = (
            f"📢 Yangi e’lon!\n\n"
            f"👤 {data['ism']} {data['familiya']}\n"
            f"💼 {data['turi']}\n"
            f"🛠 {data['texnologiya']}\n"
            f"💰 {data['narx']}\n"
            f"🎯 {data['maqsad']}\n"
            f"📞 {data['tel']}\n"
            f"🔗 {data['username']}"
        )

        await bot.send_message(CHANNEL_ID, post)
        await message.answer("✅ Ma’lumot kanalga yuborildi!", reply_markup=main_keyboard)
        await state.clear()

    elif message.text == "❌ Yo‘q":
        await state.clear()
        await message.answer("Qaytadan boshlaymiz.\nIsmingizni kiriting:")
        await state.set_state(Form.ism)


# ================= MAIN =================
async def main():
    bot = Bot(
        token=TOKEN,
        default=DefaultBotProperties(parse_mode=ParseMode.HTML)
    )
    await dp.start_polling(bot)


if __name__ == "__main__":
    asyncio.run(main()) 
