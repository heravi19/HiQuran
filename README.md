import telebot
import requests
from flask import Flask, request
import random

TOKEN = "YOUR_BOT_TOKEN"

bot = telebot.TeleBot(TOKEN)
app = Flask(name)

QURAN_API = "https://api.quran.com/api/v4"

# ---------- Keyboard Menu ----------
def main_menu():
    markup = telebot.types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("📖 قرآن کریم")
    markup.add("🔎 جستجوی آیه")
    markup.add(" آیه تصادفی")
    markup.add("🎧 تلاوت قرآن")
    return markup

# ---------- Start Command ----------
@bot.message_handler(commands=['start'])
def start(message):
    bot.send_message(
        message.chat.id,
        "به ربات قرآن کریم خوش آمدید",
        reply_markup=main_menu()
    )

# ---------- Random Verse ----------
@bot.message_handler(func=lambda m: m.text == " آیه تصادفی")
def random_verse(message):
    try:
        chapter = random.randint(1, 114)

        response = requests.get(
            f"{QURAN_API}/quran/verses/uthmani?chapter_number={chapter}"
        ).json()

        verse = random.choice(response["verses"])["text_uthmani"]

        bot.send_message(message.chat.id, verse)

    except Exception:
        bot.send_message(message.chat.id, "در دریافت آیه خطایی رخ داد.")

# ---------- Search Verse ----------
@bot.message_handler(func=lambda m: ":" in m.text)
def search_verse(message):
    try:
        chapter, verse_number = map(int, message.text.split(":"))

        response = requests.get(
            f"{QURAN_API}/quran/verses/uthmani?chapter_number={chapter}"
        ).json()

        verse = response["verses"][verse_number - 1]["text_uthmani"]

        bot.send_message(message.chat.id, verse)

    except Exception:
        bot.send_message(message.chat.id, "آیه مورد نظر یافت نشد.")

# ---------- Audio Recitation ----------
@bot.message_handler(func=lambda m: m.text == "🎧 تلاوت قرآن")
def audio_recitation(message):
    audio_url = "https://download.quranicaudio.com/quran/mishaari_raashid_al_3afaasy/001.mp3"
    bot.send_audio(message.chat.id, audio_url)

# ---------- Webhook ----------
@app.route(f"/{TOKEN}", methods=["POST"])
def webhook():
    update = telebot.types.Update.de_json(
        request.get_data().decode("utf-8")
    )
    bot.process_new_updates([update])
    return "OK"

@app.route("/")
def home():
    return "Quran Bot is Running"

if name == "main":
    bot.remove_webhook()
    bot.set_webhook(url="7980132121:AAEje8ATTPzmkJkT5Frv64kmYsy6F4pP2Dc)

    app.run(host="0.0.0.0", port=5000)# HiQuran
