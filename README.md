# idiomhelper
from telegram import Update, ReplyKeyboardMarkup, ReplyKeyboardRemove, InlineKeyboardMarkup, InlineKeyboardButton
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ConversationHandler, CallbackContext, CallbackQueryHandler
from datetime import datetime
from typing import Union
from telegram import CallbackQuery
import sys
print(sys.version)
import logging
import aiohttp
import random

TOKEN = "8081499861:AAEXa3gv8-1ReqmavC-UD-XMSFpLwo1W25Y"

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

NAME, AGE, CITY = range(3)

# База данных идиом
idioms = [
    {
        "idiom": "Aid and abet ",
        "meaning": "Содействовать",
        "example": "They plan to aid and abet his escape from prison.",
        "gif": "CgACAgIAAxkBAAICeWgXKcFaKXluy_4xpJ3TPWR7d5ioAAKXdQACvlq4SLYisrVhotIbNgQ"

    },
    {
        "idiom": "Back up",
        "meaning":"Поддерживать ",
        "example":"They will, however, still back up on the queen.",
        "gif": "CgACAgIAAxkBAAIBrGgXG1YinfYksgHrvTeq7hI0HhQZAAL-dAACvlq4SEOT79wT1HF8NgQ"

    },
    {
        "idiom": "Bail out",
        "meaning": "Спасать, выручать",
        "example": "Governments had to bail out their banks.",
        "gif": "CgACAgIAAxkBAAICm2gXNA7k51AnHXUfydBQKgmvQnJvAAJ1awACvlrASHytTGKX-j4BNgQ"

    },
    {
        "idiom": "Be there for",
        "meaning": "Быть готовым помочь",
        "example":"She's a very good friend, she's always there for me when I need her.",
        "gif": "CgACAgIAAxkBAAICnWgXNBM1rkcbn9ru2tIzBnmW601RAAJ2awACvlrASE4pwPK6DxHWNgQ"

    },
    {
        "idiom": "Come to the aid of",
        "meaning": "Прийти на помощь ",
        "example": "But, good friends come to the aid of each other.",
        "gif": "CgACAgIAAxkBAAICe2gXKeWyHpd4LRCA7bmcd3_NeHsfAAKZdQACvlq4SN2ROb1TkzlNNgQ"

    },
    {
        "idiom": "Come to the rescue",
        "meaning": "Приходить на помощь ",
        "example": "Residents are always ready to come to the rescue and welcome tourists.",
        "gif": "CgACAgIAAxkBAAICo2gXNCKgrKZL5zAZhwl18C7RguHgAAJ8awACvlrASPHjx79_1VFkNgQ"

    },
    {
        "idiom": "Extend a helping hand ",
        "meaning": "Протянуть руку помощи",
        "example": "Extend a helping hand to your brothers and sisters.",
        "gif": "CgACAgIAAxkBAAICpWgXNCbJcaS8T0P0TK3nnj-lPdTNAAJ9awACvlrASIYnICc2JaV2NgQ"

    },
    {
        "idiom": "Go to bat for",
        "meaning": "Заступиться",
        "example": "I will go to bat for you.",
        "gif": "CgACAgIAAxkBAAICc2gXKahywJuvVeIUmtCwsxZ8dxUMAAKTdQACvlq4SBf24Or_S3aiNgQ"

    },
    {
        "idiom": "Give one's full backing to somebody",
        "meaning": "Поддержать",
        "example": "The Security Council must give its full backing to this effort."

    },
    {
        "idiom": "Pitch in",
        "meaning": "Помогать, внести свою лепту",
        "example": "Everyone in the family has to pitch in.",
        "gif": "CgACAgIAAxkBAAIDD2gXZXWOLw4ZDLl2Zgj3sYitcJgXAALXbQACvlrASAiY2G8-RsBqNgQ"

    },
    {
        "idiom": "Rally around",
        "meaning": "Сплотиться",
        "example": "We have to rally around his family.",
        "gif": "CgACAgIAAxkBAAIDEWgXZXr9CJKWThuwRy_wd1F3gxRJAAIXbwACVrm4SLH5rXm01uXYNgQ"

    },
    {
        "idiom": "Scratch one’s back",
        "meaning": "Сделать взаимную услугу, «рука руку моет»",
        "example": "If you scratch my back during the weekend, I'll help you next week."

    },
    {
        "idiom": "Stick up for",
        "meaning": "Заступиться за",
        "example": "You're supposed to stick up for me.",
        "gif": "CgACAgIAAxkBAAIBrmgXG5j7CjpNgIfIe6scV6d5jMLBAAL_dAACvlq4SIPUxdQ62EmQNgQ"

    },
    {
        "idiom": "Throw a lifeline",
        "meaning": "Протянуть руку помощи",
        "example": "Let's throw him a lifeline."

    },
    {
        "idiom": "A shoulder to lean on",
        "meaning": "Плечо, чтобы опереться",
        "example": "My best friend was always there for me when I needed a shoulder to lean on.",
        "gif": "CgACAgIAAxkBAAICk2gXM8KgJS6agi4YgoDluinmEd1wAAJrawACvlrASD-PIYiz3ZoyNgQ"

    },
    {
        "idiom": "Save one’s bacon",
        "meaning": "Спасти",
        "example": "I was grateful to him as he saved my bacon.",
        "gif": "CgACAgIAAxkBAAIDE2gXZX5xgyKpVeaItKw6YYv511y-AALYbQACvlrASE0jpgskFMAfNgQ"

    },
    {
        "idiom": "A leg up",
        "meaning": "Подмога",
        "example": "Her friend gave me a leg up.",
        "gif": "CgACAgIAAxkBAAIBsGgXG86J1V2HdNvRciIQF1uECYz9AAIDdQACvlq4SOV1X4VKIIXFNgQ"

    },
    {
        "idiom": "Bear someone or something up",
        "meaning": "Подбодрить, приободрить ",
        "example": "She's bearing up remarkably well.",
        "gif": "CgACAgIAAxkBAAIBsmgXHAGXNGexxN86j769rUxgZLC-AAIEdQACvlq4SFOrzIfQdV09NgQ"

    },
    {
        "idiom": "Beat the drum for someone",
        "meaning": "Поддержать, болеть за кого-то",
        "example": "Commander is always beating the drum for his division."

    },
    {
        "idiom": "Bolster someone up",
        "meaning": "Укрепить",
        "example": "Our goal is to bolster up the community.",
        "gif": "CgACAgIAAxkBAAIBtmgXHDG8-t1gH6Czc7LrpMVbDJKfAAIKdQACvlq4SAiEKHvvm0ZBNgQ"

    },
    {
        "idiom": "Buoy someone up",
        "meaning": "поднять дух",
        "example": "Her positive attitude helped to buoy up the entire group.",
        "gif": "CgACAgIAAxkBAAIBuGgXHFT0cyLw6FUJybGlNaIxs8jkAAIOdQACvlq4SFNyoPcUcirFNgQ"

    },
    {
        "idiom": "Bring through something",
        "meaning": "помочь справиться с трудностью",
        "example": "It helped him bring through the effects of the trauma.",
        "gif": "CgACAgIAAxkBAAICn2gXNBiXeGUXMZ_b1KdC0iIp3xz-AAJ3awACvlrASNhPU5UNGmpVNgQ"

    },
    {
        "idiom": "Stand by someone’s side",
        "meaning": "Поддерживать, быть на чьей-либо стороне",
        "example": "You must always to stand by my side, never to leave it.",
        "gif": "CgACAgIAAxkBAAICgWgXKi3YgqGGzf4T3wtszt7BpInsAAKjdQACvlq4SK511RlmW6rXNgQ"

    },
    {
        "idiom": "Prop up",
        "meaning": "Поддерживать",
        "example": "They decided to prop up the campaign with celebrity endorsements.",
        "gif": "CgACAgIAAxkBAAIBtGgXHBziI5DDfy3ccdjq5niAMIO5AAIJdQACvlq4SNzanSHge-OsNgQ"

    },
    {
        "idiom": "Take under wing",
        "meaning": "Взять под крыло ",
        "example": "I'm currently looking for students to take under my wing.",
        "gif": "CgACAgIAAxkBAAIBZWgW7z6uUTAZKlq1mdqv8UE0TntuAAL0cwACvlq4SMaph_RgoXPaNgQ"
    },
    {
        "idiom": "Through thick and thin",
        "meaning": "Сквозь огонь и воду",
        "example":"She has stuck with me through thick and thin."

    },
    {
        "idiom": "Not have a leg to stand on",
        "meaning": "Без серьезной опоры ",
        "example": "You don't have a leg to stand on.",
        "gif": "CgACAgIAAxkBAAICg2gXKk5oa9gZIKMxKs8rHtMVe8AnAAKmdQACvlq4SDYs4Wb4fMiUNgQ"

    },
    {
        "idiom": "Someone’s hands are tied",
        "meaning": "Бессилен ",
        "example": "The manager explained that her hands are tied in this situation.",
        "gif": "CgACAgIAAxkBAAIDFWgXZYPOPLekgpZ8wYq50KUe4-5JAALZbQACvlrASJRZKwnqbSzfNgQ"

    },
    {
        "idiom": "Stand there with one’s bare face hanging out",
        "meaning": "Стоять без прикрытия ",
        "example": "When he arrived without an invitation, he stood there with his bare face hanging out."

    },
    {
        "idiom": "Go it alone",
        "meaning": "Действовать в одиночку, без помощи",
        "example": "She decided to go it alone and start her own business.",
        "gif": "CgACAgIAAxkBAAICfWgXKgL5YiRJ70W7KkwMO23Za-MgAAKddQACvlq4SJXLKAgo8K8XNgQ"

    },
    {
        "idiom": "No skin off one's back",
        "meaning": "До лампочки ",
        "example": "It’s no skin off my back anyway.",
        "gif": "CgACAgIAAxkBAAIDC2gXZWx3sSOFka36seIhqRi1hyyGAALUbQACvlrASBwYHTtBHOhVNgQ"

    },
    {
        "idiom": "Twiddle one's thumbs",
        "meaning": "Сидеть сложа руки",
        "example": "We can't just twiddle our thumbs and be victims anymore."

    },
    {
        "idiom": "Not lift a finger (hand)",
        "meaning": "Пальцем не пошевельнуть",
        "example": "I will not lift a finger in that direction.",
        "gif": "CgACAgIAAxkBAAIDDWgXZXEZrz404KzO-VSDa05reZ_xAALVbQACvlrASL1HcmqTGn17NgQ"

    },
    {
        "idiom": "Hang someone out to dry",
        "meaning": "Оставить кого-то в беде ",
        "example": "He hung me out to dry during the meeting when he didn't defend my project.",
        "gif": "CgACAgIAAxkBAAIDCWgXZWZ0BrXQ2dcl-glMJj3p-IUvAALRbQACvlrASHcdZCpdh4zINgQ"

    },
    {
        "idiom": "Leave somebody high and dry",
        "meaning": "Оставить кого-то без помощи",
        "example": "I was left high and dry when my partner pulled out of the deal.",
        "gif": "CgACAgIAAxkBAAIDF2gXZYm9pEB18O_4M8PCNZ8PU5j2AALabQACvlrASCxc7U94UEzlNgQ"

    },
    {
        "idiom": "Leave to sink or swim",
        "meaning": "Бросить на произвол судьбы ",
        "example": "The new employee was left to sink or swim with little training."

    },
    {
        "idiom": "To do an ill turn",
        "meaning": "Навредить",
        "example": "He did her an ill turn by spreading rumors."

    },
    {
        "idiom": "Throw a monkey wrench into",
        "meaning": "Вставить палки в колеса",
        "example": "The unexpected rain really threw a monkey wrench into our plans for the picnic."

    },
    {
        "idiom": "To get underfoot",
        "meaning": "Мешать",
        "example":  "The kids were getting underfoot while I was trying to cook.",
        "gif": "CgACAgIAAxkBAAICf2gXKhY00OenRIkQymB09D4oPrCzAAKhdQACvlq4SKSy8Rn8OZrSNgQ"

    },
    {
        "idiom": "Stand in the way",
        "meaning": "Стоять на пути",
        "example": "Don't stand in the way of progress.",
        "gif": "CgACAgIAAxkBAAIChWgXKmcr53DKWY8nEkfndymxcIBzAAKqdQACvlq4SHryaxVaa1qFNgQ"

    },
    {
        "idiom": "Bollix up the whole schmear",
        "meaning": "Испортить всё дело",
        "example": "If we don't follow the plan, we might bollix up the whole schmear."

    },
    {
        "idiom": "Tread on someone's corns",
        "meaning": "Наступить на больное место",
        "example": "Be careful not to tread on his corns by bringing up that topic."

    }
]

GROUP_CHAT_ID ='-4742919192'

# Отправка сообщений в группу
async def send_to_group(message):
    url = f'https://api.telegram.org/bot{TOKEN}/sendMessage'
    payload = {
        'chat_id': GROUP_CHAT_ID,
        'text': message
    }
    async with aiohttp.ClientSession() as session:
        async with session.post(url, json=payload) as response:
            return await response.json()

# Исправленная функция feedback_command
async def feedback_command(update: Update, context: CallbackContext):
    feedback_text = ' '.join(context.args)
    if feedback_text:
        try:
            await handle_feedback(feedback_text) # Await the coroutine
            await update.message.reply_text("Спасибо за ваше пожелание!")
        except Exception as e:
            logging.error(f"Error handling feedback: {e}")
            await update.message.reply_text("Произошла ошибка при обработке вашего пожелания.")
    else:
            await update.message.reply_text("Пожалуйста, напишите ваше пожелание используя команду /feedback.")

# Пример обработки ошибки
try:
    # Ваш код бота
    pass
except Exception as e:
    error_message = f"Ошибка: {str(e)}"
    logging.error(error_message)
    send_to_group(error_message)

# Пример обработки пожеланий
async def handle_feedback(feedback):
    feedback_message = f"Пожелание: {feedback}"
    logging.info(feedback_message)
    await send_to_group(feedback_message)  # Если send_to_group асинхронная

# Функция для выбора идиомы дня
def get_idiom_of_the_day():
    # Используем текущую дату для выбора идиомы
    today = datetime.now().date()
    # Преобразуем дату в число, чтобы использовать как индекс
    day_of_year = today.timetuple().tm_yday
    # Выбираем идиому по индексу (остаток от деления на количество идиом)
    index = day_of_year % len(idioms)
    return idioms[index]

# Обработчик команды /today
async def today_idiom(update: Update, context: CallbackContext) -> None:
    idiom = get_idiom_of_the_day()
    response = f"Идиома дня📆:\n\n{idiom['idiom']}\nЗначение: {idiom['meaning']}\nПример использования: {idiom['example']}"

    # Отправляем только одно сообщение
    if 'gif' in idiom:
        await update.message.reply_animation(idiom['gif'], caption=response)
    else:
        await update.message.reply_text(response)

# Состояния для викторины
QUIZ, ANSWER = range(2)

# Функция чтобы начать викторину
async def start_quiz(update: Update, context: CallbackContext) -> int:
    # Выбираем случайную идиому
    context.user_data['current_idiom'] = random.choice(idioms)
    idiom = context.user_data['current_idiom']

    # Формируем вопрос с вариантами ответа
    options = [idiom['meaning']]
    while len(options) < 4:
        random_option = random.choice([i['meaning'] for i in idioms if i['idiom'] != idiom['idiom']])
        if random_option not in options:
            options.append(random_option)

    random.shuffle(options)

    # Сохраняем правильный ответ и варианты
    context.user_data['correct_answer'] = idiom['meaning']
    context.user_data['options'] = options

    # Отправляем вопрос пользователю
    await update.message.reply_text(
        f"Что означает идиома '{idiom['idiom']}'?\n\n" +
        "\n".join([f"{i+1}. {option}" for i, option in enumerate(options)])
    )

    return QUIZ

# Функция чтобы обработать ответ пользователя
async def handle_quiz_answer(update: Update, context: CallbackContext) -> int:
    user_answer = update.message.text.strip()  # Получаем ответ пользователя
    options = context.user_data['options']  # Получаем варианты ответа
    correct_answer = context.user_data['correct_answer']  # Получаем правильный ответ

    try:
        # Пытаемся преобразовать ответ в число
        user_choice = int(user_answer) - 1  # Преобразуем в индекс (начинаем с 0)
        if 0 <= user_choice < len(options):
            # Проверяем, совпадает ли выбранный вариант с правильным ответом
            if options[user_choice] == correct_answer:
                await update.message.reply_text("Правильно!")
            else:
                await update.message.reply_text(f"Неправильно. Правильный ответ: {correct_answer}")
        else:
            await update.message.reply_text("Выбери номер из предложенных вариантов.")
            return QUIZ
    except ValueError:
        await update.message.reply_text("Введи номер ответа цифрой.")
        return QUIZ

    # Предлагаем продолжить или завершить викторину
    keyboard = [['Продолжить викторину', 'Завершить викторину']]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True)
    await update.message.reply_text("Хочешь продолжить викторину?", reply_markup=reply_markup)

    return ANSWER

# Функция чтобы обработать выбор пользователя (продолжить или завершить)
async def handle_quiz_choice(update: Update, context: CallbackContext) -> int:
    user_choice = update.message.text

    if user_choice == 'Продолжить викторину':
        return await start_quiz(update, context)
    else:
        await update.message.reply_text("Викторина завершена. Спасибо за участие!", reply_markup=ReplyKeyboardRemove())
        await return_to_main_menu(update, context)
        return ConversationHandler.END

# Функция чтобы отменить викторину
async def cancel_quiz(update: Update, context: CallbackContext) -> int:
    await update.message.reply_text("Викторина прервана. Если хочешь начать заново, напиши /quiz.")
    return ConversationHandler.END

# Функция чтобы вернуться в главное меню
async def return_to_main_menu(update: Update, context: CallbackContext) -> None:
    keyboard = [['/start', '/help'], ['/about', '/random'], ['/search'], ['/keywords'], ['/list'], ['/quiz'], ['/menu'], ['/profile'], ['/today'], ['/feedback']]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True)
    await update.message.reply_text("Ты вернулся в главное меню.", reply_markup=reply_markup)

# Функция чтобы начать диалог
async def start(update: Update, context: CallbackContext) -> int:
    # Создаем клавиатуру с кнопками
    keyboard = [['/start', '/help'], ['/about', '/random'], ['/search'], ['/keywords'], ['/list'], ['/quiz'], ['/profile'], ['/today'], ['/feedback']]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True)
    await update.message.reply_text("Привет!😊 Как тебя зовут?", reply_markup=reply_markup)
    return NAME

# Функция чтобы обработать имя
async def get_name(update: Update, context: CallbackContext) -> int:
    user_name = update.message.text
    context.user_data['name'] = user_name
    await update.message.reply_text(f"Приятно познакомиться, {user_name}! Сколько тебе лет?")
    return AGE

# Функция чтобы обработать возраст
async def get_age(update: Update, context: CallbackContext) -> int:
    user_age = update.message.text
    context.user_data['age'] = user_age
    await update.message.reply_text("Из какого ты города?")
    return CITY

# Функция чтобы обработать город
async def get_city(update: Update, context: CallbackContext) -> int:
    user_city = update.message.text
    context.user_data['city'] = user_city

    # Получаем данные из context.user_data
    name = context.user_data['name']
    age = context.user_data['age']
    city = context.user_data['city']

    # Отправка финального сообщения
    await update.message.reply_text(f"Отлично, {name}! Тебе {age} лет и ты из города {city}. Приятно познакомиться! Я твой Telegram-бот🤖, созданный для изучения идиом. Используй /help для получения инструкций.")
    return ConversationHandler.END

# Функция чтобы отменить запрос
async def cancel(update: Update, context: CallbackContext) -> int:
    await update.message.reply_text("Диалог прерван. Если хочешь начать заново, напиши /start.")
    return ConversationHandler.END

# Функция для отображения профиля пользователя
async def show_profile(update: Update, context: CallbackContext) -> None:
    # Получаем данные из context.user_data
    name = context.user_data.get('name', 'не указано')
    age = context.user_data.get('age', 'не указано')
    city = context.user_data.get('city', 'не указано')

    # Формируем сообщение с данными пользователя
    profile_text = f"Твой профиль👤:\nИмя: {name}\nВозраст: {age}\nГород: {city}"
    await update.message.reply_text(profile_text)

# Функция чтобы вывести список команд
async def help_command(update: Update, context: CallbackContext) -> None:
    help_text = """
    Доступные команды:
    /start – начать работу с ботом.🤖
    /profile – показать профиль пользователя.👤
    /help – получить инструкцию по использованию.❔
    /about – узнать о проекте.
    /random – получить случайную идиому с объяснением и примером использования.🎲
    /search – найти идиому по ключевому слову (чтобы узнать список ключевых слов, напиши /keywords).🔎
    /keywords – показать список ключевых слов.
    /list – показать список идиом в алфавитном порядке.
    /quiz – запустить викторину для проверки идиом.
    /today – получить идиому дня.📆
    /menu – вернуться в главное меню.
    /feedback – отправить сообщение об ошибке или написать пожелания.
    """
    await update.message.reply_text(help_text)

# Функция для вывода информации о проекте
async def about_command(update: Update, context: CallbackContext) -> None:
    about_text = """
    Этот бот создан для помощи в изучении английских идиом семантического поля "Помощь/Отсутствие помощи"🇬🇧. Здесь ты можешь находить, изучать и практиковать идиомы в интерактивной форме.
Ссылки на разработчиков: @feldgd @tichoaferist @Liza_2201 @spaccedsgn @lunafwwer @Paratatiru_paratata
    """
    await update.message.reply_text(about_text)

# Функция чтобы выдать случайную идиому
async def send_idiom_with_gif(update: Update, context: CallbackContext):
    idiom = random.choice(idioms)
    response = f"{idiom['idiom']}\nЗначение: {idiom['meaning']}\nПример: {idiom['example']}"

    keyboard = [
        ['/start', '/help'],
        ['/about', '/random'],
        ['/today', '/quiz']
    ]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True)

    if idiom.get('gif'):
        await update.message.reply_animation(
            idiom['gif'],
            caption=response,
            reply_markup=reply_markup
        )
    else:
        await update.message.reply_text(
            response,
            reply_markup=reply_markup
        )
# Функция возврата в меню
async def back_to_menu(update: Update, context: CallbackContext):
    query = update.callback_query
    await query.answer()

    keyboard = [
        ['/start', '/help'],
        ['/about', '/random'],
        ['/today', '/quiz']
    ]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True)

    await query.edit_message_text(
        text="Вы вернулись в главное меню",
        reply_markup=reply_markup
    )
# Функция для списка идиом
async def list_idioms(update_or_query: Union[Update, CallbackQuery], context: CallbackContext):
    if isinstance(update_or_query, CallbackQuery):
        query = update_or_query
        message = query.message
    else:
        update = update_or_query
        message = update.message

    letters = sorted({idiom['idiom'].strip()[0].upper() for idiom in idioms})
    keyboard = []
    for i in range(0, len(letters), 5):
        row = [
            InlineKeyboardButton(letter, callback_data=f"list_{letter}")
            for letter in letters[i:i+5]
        ]
        keyboard.append(row)
    keyboard.append([InlineKeyboardButton("🔙 Главное меню", callback_data="menu_back")])

    if isinstance(update_or_query, CallbackQuery):
        await query.edit_message_text(
            "Выберите букву:",
            reply_markup=InlineKeyboardMarkup(keyboard)
        )
    else:
        await message.reply_text(
            "Выберите букву:",
            reply_markup=InlineKeyboardMarkup(keyboard)
        )
# Функция для показа буквы в выборе идиом
async def show_letter_idioms(update: Update, context: CallbackContext):
    query = update.callback_query
    await query.answer()

    # Получаем букву из callback_data (формат "list_A")
    letter = query.data.split('_')[1]

    # Фильтруем идиомы по выбранной букве (игнорируя пробелы в начале)
    letter_idioms = [
        idiom for idiom in idioms
        if idiom['idiom'].strip()[0].upper() == letter
    ]

    if not letter_idioms:
        await query.edit_message_text(f"Идиом на букву {letter} не найдено")
        return

    # Формируем сообщение с идиомами
    response = f"📖 Идиомы на букву {letter}:\n\n"
    response += "\n".join(
        f"{i+1}. {idiom['idiom'].strip()}"
        for i, idiom in enumerate(letter_idioms)
    )

    # Кнопки навигации
    keyboard = [
        [InlineKeyboardButton("🔙 К списку букв", callback_data="back_to_letters")]
    ]

    await query.edit_message_text(
        text=response,
        reply_markup=InlineKeyboardMarkup(keyboard))

# Функция для подробной информации для идиомы
async def show_idiom_details(update: Update, context: CallbackContext):
    query = update.callback_query
    idiom_name = query.data.split('_')[1]

    # Находим идиому
    idiom = next(i for i in idioms if i['idiom'] == idiom_name)

    response = (
        f"🔹 {idiom['idiom']}\n"
        f"📖 Значение: {idiom['meaning']}\n"
        f"📝 Пример: {idiom['example']}"
    )

    # Кнопки навигации
    keyboard = [
        [InlineKeyboardButton("← Назад к списку", callback_data=f"letter_{idiom['idiom'][0]}")]
    ]

    await query.edit_message_text(
        text=response,
        reply_markup=InlineKeyboardMarkup(keyboard))

# Функция для отображения ключевых слов
async def show_keywords(update: Update, context: CallbackContext) -> None:
    keywords = ["aid", "go", "get", "up", "stand", "way", "come"]
    # Создаем кнопки с ключевыми словами
    keyboard = [
        [InlineKeyboardButton(keyword, callback_data=f"kw_{keyword}")]
        for keyword in keywords
    ]
    # Добавляем кнопку возврата
    keyboard.append([InlineKeyboardButton("🔙 Главное меню", callback_data="menu_back")])

    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text("Выберите ключевое слово:", reply_markup=reply_markup)

# Функция выбора буквы
async def handle_letter_selection(update: Update, context: CallbackContext):
    query = update.callback_query
    await query.answer()

    # Обрабатываем только callback_data, начинающиеся с list_
    if not query.data.startswith('list_'):
        return

    letter = query.data.split('_')[1]

    # Фильтруем идиомы по выбранной букве (с обработкой пробелов)
    letter_idioms = [
        idiom for idiom in idioms
        if idiom['idiom'].strip()[0].upper() == letter
    ]

    if not letter_idioms:
        await query.edit_message_text(f"Идиом на букву {letter} не найдено")
        return

    # Формируем список идиом
    response = f"📚 Идиомы на букву {letter}:\n\n"
    response += "\n".join(
        f"{i+1}. {idiom['idiom'].strip()}"
        for i, idiom in enumerate(letter_idioms)
    )

    # Кнопки навигации
    keyboard = [
        [InlineKeyboardButton("🔙 К списку букв", callback_data="back_to_letters")]
    ]

    await query.edit_message_text(
        text=response,
        reply_markup=InlineKeyboardMarkup(keyboard)
    )

# Функция возврата в меню к буквам
async def back_to_letters(update: Update, context: CallbackContext):
    query = update.callback_query
    await query.answer()
    # Используем существующую функцию list_idioms
    if query.message:
        await list_idioms(query, context)
    else:
        await list_idioms(update, context)

# Функция выбора ключевого слова
async def handle_keyword_selection(update: Update, context: CallbackContext):
    query = update.callback_query
    await query.answer()

    # Обрабатываем только callback_data, начинающиеся с kw_
    if not query.data.startswith('kw_'):
        return

    keyword = query.data[3:].lower()  # Убираем префикс kw_
    found_idioms = [
        idiom for idiom in idioms
        if keyword in idiom['idiom'].lower() or
           keyword in idiom['meaning'].lower()
    ]

    if found_idioms:
        response = "🔍 Найденные идиомы:\n\n"
        for idiom in found_idioms:
            response += f"• {idiom['idiom']}\n"
        response += "\nВыберите идиому для подробностей"

        # Создаем кнопки для найденных идиом
        keyboard = [
            [InlineKeyboardButton(idiom['idiom'], callback_data=f"detail_{idiom['idiom']}")]
            for idiom in found_idioms
        ]
        keyboard.append([InlineKeyboardButton("Назад к ключевым словам", callback_data="back_to_keywords")])

        await query.edit_message_text(
            text=response,
            reply_markup=InlineKeyboardMarkup(keyboard))
    else:
        await query.edit_message_text(
            text=f"Идиом по запросу '{keyword}' не найдено",
            reply_markup=InlineKeyboardMarkup([[InlineKeyboardButton("Назад к ключевым словам", callback_data="back_to_keywords")]])
        )

# Функция возврата к ключевым словам
async def back_to_keywords(update: Update, context: CallbackContext):
    query = update.callback_query
    await query.answer()
    await show_keywords(query, context)

# Функция получения ID GIF-иллюстрации
async def get_file_id(update: Update, context: CallbackContext) -> None:
    if update.message.animation:
        file_id = update.message.animation.file_id
        await update.message.reply_text(f"File ID: {file_id}")
    else:
        await update.message.reply_text("Отправьте GIF, чтобы получить file_id.")

# Функция для поиска идиом по ключевому слову
async def search_idiom(update: Update, context: CallbackContext) -> None:
    keyword = ''.join(context.args).lower()

    if not keyword:
        await update.message.reply_text("Введи ключевое слово для поиска. Например: /search help")
        return

    found_idioms = [
        idiom for idiom in idioms
        if keyword in idiom['idiom'].lower() or keyword in idiom['meaning'].lower()
        ]

    if found_idioms:
        response = "Найденные идиомы:\n\n"
        for idiom in found_idioms:
            response +=f"Идиома: {idiom['idiom']}\nЗначение: {idiom['meaning']}\nПример использования: {idiom['example']}\n\n"
    else:
        response = "Идиомы по твоему запросу не найдены."

    await update.message.reply_text(response)

def main() -> None:
    # токен API
    application = Application.builder().token("8081499861:AAEXa3gv8-1ReqmavC-UD-XMSFpLwo1W25Y").build()
    application.add_handler(MessageHandler(filters.ANIMATION, get_file_id))  # <- Добавьте эту строку

    # Создание ConversationHandler для викторины
    quiz_handler = ConversationHandler(
        entry_points=[CommandHandler('quiz', start_quiz)],
        states={
            QUIZ: [MessageHandler(filters.TEXT & ~filters.COMMAND, handle_quiz_answer)],
            ANSWER: [MessageHandler(filters.TEXT & ~filters.COMMAND, handle_quiz_choice)],
        },
        fallbacks=[CommandHandler('cancel', cancel_quiz)],
    )

    application.add_handler(quiz_handler)

    # Создание ConversationHandler для стартового диалога
    conv_handler = ConversationHandler(
        entry_points=[CommandHandler('start', start)],
        states={
            NAME: [MessageHandler(filters.TEXT & ~filters.COMMAND, get_name)],
            AGE: [MessageHandler(filters.TEXT & ~filters.COMMAND, get_age)],
            CITY: [MessageHandler(filters.TEXT & ~filters.COMMAND, get_city)],
        },
        fallbacks=[CommandHandler('cancel', cancel)],
    )

    application.add_handler(conv_handler)

    # Добавляем остальные обработчики команд
    application.add_handler(CommandHandler('help', help_command))
    application.add_handler(CommandHandler('profile', show_profile))
    application.add_handler(CommandHandler('about', about_command))
    application.add_handler(CommandHandler('random', send_idiom_with_gif))
    application.add_handler(CommandHandler('search', search_idiom))
    application.add_handler(CommandHandler('keywords', show_keywords))
    application.add_handler(CommandHandler('list', list_idioms))
    application.add_handler(CallbackQueryHandler(handle_keyword_selection, pattern=r"^kw_"))
    application.add_handler(CallbackQueryHandler(back_to_keywords, pattern="^back_to_keywords$"))
    application.add_handler(CallbackQueryHandler(handle_letter_selection, pattern=r"^list_"))
    application.add_handler(CallbackQueryHandler(back_to_letters, pattern="^back_to_letters$"))
    application.add_handler(CallbackQueryHandler(back_to_menu, pattern="^menu_back$"))
    application.add_handler(CommandHandler('today', today_idiom))
    application.add_handler(CommandHandler('menu', return_to_main_menu))
    application.add_handler(CommandHandler('feedback', feedback_command))
    application.add_handler(MessageHandler(filters.ANIMATION & ~filters.COMMAND, get_file_id))

# Обработчик для ключевых слов должен быть отдельным
    application.add_handler(CallbackQueryHandler(handle_keyword_selection, pattern=r"^(?!letter_|back_to_).*"))

    # Запуск бота
    application.run_polling()

if __name__ == '__main__':
    main()
