# import json
import random
import logging
from telegram import Update
from telegram.ext import Updater, CommandHandler, CallbackContext

# Увімкнемо логування
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO
)
logger = logging.getLogger(__name__)
  professions = ["Хірург", "Інженер", "Вчитель", "Механік", "Фермер", "Програміст", "Психолог", "Пожежник", "Музикант", "Шеф-кухар"]
health_conditions = ["Здоровий", "Астма", "Діабет", "Сліпота", "Інвалідність", "Серцева недостатність", "Онко хвороба", "Хронічний біль"]
hobbies = ["Полювання", "Кулінарія", "Читання", "Малювання", "Туризм", "Гейминг", "Футбол", "Шахи"]
fears = ["Павуки", "Темрява", "Висота", "Замкнуті простори", "Клоуни", "Кров"]
baggage = ["Аптечка", "Ніж", "Ліхтарик", "Газова плитка", "Консерви", "Вода", "Сокира", "Радіоприймач"]
character_traits = ["Сильна воля", "Агресивний", "Лідер", "Дуже розумний", "Спокійний", "Панікер"]
residence = ["Київ", "Село в Карпатах", "Маріуполь", "Одеса", "Полтава", "Чернівці", "Луцьк"]
skills = ["Виживання в лісі", "Надання першої допомоги", "Володіння зброєю", "Будівництво укриттів", "Готування їжі"]
bad_habits = ["Палить", "Алкоголік", "Психічно нестабільний", "Залежний від телефону", "Ненадійний"]. 
players_count = 12

cards = []
used_combinations = set()

def generate_unique_card():
    attempts = 0
    while attempts < 100:
        card = {
            "Професія": random.choice(professions),
            "Вік": random.randint(18, 70),
            "Стан здоров’я": random.choice(health_conditions),
            "Хобі": random.choice(hobbies),
            "Фобія": random.choice(fears),
            "Багаж": random.choice(baggage),
            "Характеристика": random.choice(character_traits),
            "Місце проживання": random.choice(residence),
            "Додаткова навичка": random.choice(skills),
            "Шкідлива звичка": random.choice(bad_habits)
        }
        key = tuple(card.values())
        if key not in used_combinations:
            used_combinations.add(key)
            return card
        attempts += 1
    return None

for _ in range(players_count):
    card = generate_unique_card()
    if card:
        cards.append(card)
output_file = output_folder / "bunker_cards_v2.json"
with open(output_file, "w", encoding="utf-8") as f:
    json.dump(cards, f, ensure_ascii=False, indent=2)

output_file.name
with open('bunker_cards_v2.json', encoding='utf-8') as f:
    cards = json.load(f)

players = set()
player_cards = {}
def join(update: Update, context: CallbackContext):
    user = update.effective_user
    if user.id in players:
        update.message.reply_text(f"{user.first_name}, ти вже в грі!")
    else:
        players.add(user.id)
        update.message.reply_text(f"{user.first_name} приєднався до гри! Загалом гравців: {len(players)}")

def startgame(update: Update, context: CallbackContext):
    global player_cards
    if len(players) < 3:
        update.message.reply_text("Для початку гри потрібно мінімум 3 гравці!")
        return
    player_cards = {}
    selected_cards = random.sample(cards, len(players))
    # Розсилаємо кожному гравцю картку в приват
    for user_id, card in zip(players, selected_cards):
        player_cards[user_id] = card
        context.bot.send_message(chat_id=user_id, text=card_to_text(card))
    update.message.reply_text("Гра почалась! Картки розіслані в особисті повідомлення.")

def card_to_text(card):
    lines = [f"{key}: {value}" for key, value in card.items()]
    return "\n".join(lines)

def main():
    TOKEN =7635553359:AAHKlFZ7h0F6CDuj676jpBhS6v_hLgJseOc

    updater = Updater(TOKEN)
    dp = updater.dispatcher

    dp.add_handler(CommandHandler("join", join))
    dp.add_handler(CommandHandler("startgame", startgame))

    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
