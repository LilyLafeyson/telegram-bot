
import os
import logging
from telegram import Update
from telegram.ext import Updater, MessageHandler, Filters, 
CallbackContext, CommandHandler
from telegram.error import BadRequest

# Включаем логирование
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

# ID групп (замени на реальные)
SOURCE_GROUP_IDS = [-1001898562129, -1001836819586]  # Группы А и С
MODERATOR_GROUP_ID = -1002704329930  # Группа В

# Словарь пересланных сообщений {mod_message_id: user_id}
forwarded_messages = {}

def handle_tagged_message(update: Update, context: CallbackContext):
    message = update.message
    if not message or not message.text:
        return

    if '#помощь' in message.text.lower():
        user = message.from_user
        user_id = user.id
        username = f"@{user.username}" if user.username else 
user.first_name

        try:
            sent = context.bot.send_message(
                chat_id=MODERATOR_GROUP_ID,
                text=f"Сообщение от {username} (ID: 
{user_id}):\n{message.text}"
            )
            forwarded_messages[sent.message_id] = user_id

            # Удаление оригинального сообщения в группе
            context.bot.delete_message(chat_id=message.chat_id, 
message_id=message.message_id)

        except BadRequest as e:
            logger.warning(f"Ошибка при пересылке или удалении: {e}")

def handle_reply(update: Update, context: CallbackContext):
    message = update.message
    if not message.reply_to_message:
        return

    replied_message_id = message.reply_to_message.message_id
    moderator = message.from_user

    if replied_message_id in forwarded_messages:
        user_id = forwarded_messages[replied_message_id]
        try:
            # Отправка ответа пользователю
            context.bot.send_message(chat_id=user_id, text=message.text)
            context.bot.send_message(chat_id=MODERATOR_GROUP_ID, text="✅ 
Ответ отправлен пользователю.")

            # Обновляем текст исходного сообщения с отметкой
            old_text = message.reply_to_message.text
            if "✅ Ответ дан" not in old_text:
                context.bot.edit_message_text(
                    chat_id=MODERATOR_GROUP_ID,
                    message_id=replied_message_id,
                    text=old_text + "\n\n✅ Ответ дан"
                )
        except BadRequest:
            context.bot.send_message(chat_id=MODERATOR_GROUP_ID, text="❌ 
Не удалось отправить сообщение. Пользователь не написал боту первым.")

def ping_command(update: Update, context: CallbackContext):
    user_id = update.effective_user.id
    chat_member = context.bot.get_chat_member(MODERATOR_GROUP_ID, user_id)
    if chat_member.status in ["administrator", "creator"]:
        update.message.reply_text("Бот работает ✅")
    else:
        update.message.reply_text("Команда доступна только модераторам.")

def main():
    # Берём токен из переменной окружения
    TOKEN = os.environ.get("BOT_TOKEN")
    if not TOKEN:
        raise ValueError("❌ Ошибка: переменная окружения BOT_TOKEN не 
найдена. Установи её в Render!")

    PORT = int(os.environ.get("PORT", 5000))

    updater = Updater(TOKEN, use_context=True)
    dp = updater.dispatcher

    # Добавляем обработчики
    dp.add_handler(MessageHandler(Filters.text & 
Filters.chat(chat_id=SOURCE_GROUP_IDS), handle_tagged_message))
    dp.add_handler(MessageHandler(Filters.reply & 
Filters.chat(chat_id=MODERATOR_GROUP_ID), handle_reply))
    dp.add_handler(CommandHandler("ping", ping_command))

    # Настройка webhook для Render
    updater.start_webhook(
        listen="0.0.0.0",
        port=PORT,
        url_path=TOKEN
    )
    
updater.bot.set_webhook(f"https://telegram-bot-5fxs.onrender.com/{TOKEN}")

    logger.info("Бот запущен на webhook")
    updater.idle()

if __name__ == '__main__':
    main()



