import json
import telebot
from flask import Flask, request

TOKEN = "8504054803:AAH5beDVi_Cdn2Mo_ZlXTbI37q0c57ct2p0"
bot = telebot.TeleBot(TOKEN)
app = Flask(__name__)

TEMPLATES_FILE = "/opt/render/project/src/templates.json"

def load_templates():
    try:
        with open(TEMPLATES_FILE) as f:
            return json.load(f)
    except:
        return {}

def save_templates(data):
    with open(TEMPLATES_FILE, "w") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)

user_data = {}

VORONKA_DEFAULTS = {
    "coffee": "15", "water": "260", "temp": "95", "grind": "18",
    "time_total": "3:40", "step1_water": "40", "step2_water": "60",
    "step3_water": "70", "step4_water": "90",
    "step1_total": "40", "step2_total": "100",
    "step3_total": "170", "step4_total": "260"
}

ESPRESSO_DEFAULTS = {"coffee": "18,5", "yield": "40", "time": "28"}

@bot.message_handler(commands=["start"])
def start(message):
    markup = telebot.types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("☕ Воронка", "⚡ Эспрессо")
    bot.send_message(message.chat.id,
        "☕ Привет! Я бот для рецептов кофе.\n\nВыбери тип заваривания:",
        reply_markup=markup
    )

@bot.message_handler(func=lambda m: m.text in ["☕ Воронка", "⚡ Эспрессо"])
def choose_type(message):
    chat_id = message.chat.id
    if message.text == "☕ Воронка":
        user_data[chat_id] = {"type": "voronka", "step": "coffee", "data": VORONKA_DEFAULTS.copy()}
        bot.send_message(chat_id, "🍃 ВОРОНКА\n\nВведи количество кофе (грамм):\nПо умолчанию: 15",
                         reply_markup=telebot.types.ReplyKeyboardRemove())
    else:
        user_data[chat_id] = {"type": "espresso", "step": "coffee", "data": ESPRESSO_DEFAULTS.copy()}
        bot.send_message(chat_id, "⚡ ЭСПРЕССО\n\nВведи количество кофе (грамм):\nПо умолчанию: 18,5",
                         reply_markup=telebot.types.ReplyKeyboardRemove())

@bot.message_handler(func=lambda m: m.chat.id in user_data and "step" in user_data[m.chat.id])
def process_step(message):
    chat_id = message.chat.id
    user = user_data[chat_id]
    step = user["step"]
    text = message.text.strip()
    if text:
        user["data"][step] = text

    if user["type"] == "voronka":
        steps = [
            ("coffee", "Введи объём воды (мл):\nПо умолчанию: 260", "water"),
            ("water", "Введи температуру воды (°C):\nПо умолчанию: 95", "temp"),
            ("temp", "Введи помол на Comandante C40 (щелчков):\nПо умолчанию: 18", "grind"),
            ("grind", "Введи общее время заваривания:\nПо умолчанию: 3:40", "time_total"),
            ("time_total", "Этап 1 — влей воды (мл):\nПо умолчанию: 40", "step1_water"),
            ("step1_water", "Этап 2 — влей воды (мл):\nПо умолчанию: 60", "step2_water"),
            ("step2_water", "Этап 3 — влей воды (мл):\nПо умолчанию: 70", "step3_water"),
            ("step3_water", "Этап 4 — влей воды (мл):\nПо умолчанию: 90", "step4_water"),
            ("step4_water", None, None),
        ]
    else:
        steps = [
            ("coffee", "Введи выход напитка (грамм):\nПо умолчанию: 40", "yield"),
            ("yield", "Введи время заваривания (секунд):\nПо умолчанию: 28", "time"),
            ("time", None, None),
        ]

    for s_name, s_prompt, s_next in steps:
        if step == s_name:
            if s_next:
                user["step"] = s_next
                bot.send_message(chat_id, s_prompt)
            else:
                del user["step"]
                show_result(chat_id)
            return

def show_result(chat_id):
    user = user_data.get(chat_id)
    if not user:
        return
    d = user["data"]

    if user["type"] == "voronka":
        text = f"""Рекомендуем использовать кофе, обжаренный не менее недели назад.

Рецепт заваривания в воронке:
Молотого кофе: {d['coffee']} грамм
Объем воды: {d['water']} мл
Температура воды: {d['temp']}°
Помол на кофемолке Comandante C40 Nitro Blade: {d['grind']} щелчков.

Этапы заваривания:
0:00 - вливаем {d['step1_water']} мл воды
0:45 - вливаем {d['step2_water']} мл воды (всего {d['step2_total']} мл)
1:30 - вливаем {d['step3_water']} мл воды (всего {d['step3_total']} мл)
2:15 - вливаем {d['step4_water']} мл воды (всего {d['step4_total']} мл)

Общее время заваривания {d['time_total']}

Рецепт к данному лоту был подобран с использованием многоразового фильтра для заваривания кофе Precise Brew.

Подробнее о том, как заваривать кофе в воронке."""
    else:
        text = f"""Рекомендуем использовать кофе, обжаренный не менее недели назад.

Рецепт приготовления в эспрессо-машине:
Молотого кофе: {d['coffee']} грамм
Напитка на выходе: {d['yield']} грамм
Время заваривания: {d['time']} секунд

Это рецепт, который мы подобрали в нашем цехе для кофемашины Ottima 2Gr. Рецепт идеальной чашки на вашей кофемашине будет отличаться, но мы советуем вам использовать его как стартовую точку для настройки."""

    markup = telebot.types.InlineKeyboardMarkup()
    markup.add(telebot.types.InlineKeyboardButton("💾 Сохранить рецепт", callback_data="save"))
    markup.add(telebot.types.InlineKeyboardButton("🔄 Новый рецепт", callback_data="new"))
    bot.send_message(chat_id, text, reply_markup=markup)

@bot.callback_query_handler(func=lambda call: call.data == "save")
def save_recipe(call):
    chat_id = call.message.chat.id
    user = user_data.get(chat_id, {})
    templates = load_templates()
    recipe_type = "Воронка" if user.get("type") == "voronka" else "Эспрессо"
    num = len(templates) + 1
    name = f"{recipe_type} #{num}"
    templates[name] = {"type": user.get("type"), "data": user.get("data", {})}
    save_templates(templates)
    bot.send_message(chat_id, f"💾 Сохранено как «{name}»")

@bot.callback_query_handler(func=lambda call: call.data == "new")
def new_recipe(call):
    markup = telebot.types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("☕ Воронка", "⚡ Эспрессо")
    bot.send_message(call.message.chat.id, "Выбери тип заваривания:", reply_markup=markup)

@bot.message_handler(commands=["templates"])
def list_templates(message):
    templates = load_templates()
    if not templates:
        bot.send_message(message.chat.id, "Нет сохранённых рецептов")
        return
    markup = telebot.types.InlineKeyboardMarkup()
    for name in templates:
        markup.add(telebot.types.InlineKeyboardButton(name, callback_data=f"load_{name}"))
    bot.send_message(message.chat.id, "📂 Сохранённые рецепты:", reply_markup=markup)

@bot.callback_query_handler(func=lambda call: call.data.startswith("load_"))
def load_template(call):
    name = call.data[5:]
    templates = load_templates()
    if name in templates:
        user_data[call.message.chat.id] = templates[name]
        show_result(call.message.chat.id)

@app.route(f"/{TOKEN}", methods=["POST"])
def webhook():
    bot.process_new_updates([telebot.types.Update.de_json(request.get_json())])
    return "OK"

@app.route("/")
def index():
    return "Бот работает"
