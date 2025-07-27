from telegram import Update
from telegram.ext import ApplicationBuilder, MessageHandler, ContextTypes, filters
import os

BOT_TOKEN = os.getenv("BOT_TOKEN")
REQUIRED_CHANNELS = ["@LPKJUBEL"]
TARGET_CHANNEL = "@LPKJUBEL"
ALLOWED_HASHTAGS = ["#LpkBeli", "#Lpkjual"]

async def check_membership(user_id, context):
    for channel in REQUIRED_CHANNELS:
        try:
            member = await context.bot.get_chat_member(channel, user_id)
            if member.status not in ['member', 'administrator', 'creator']:
                return False
        except:
            return False
    return True

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    text = update.message.text

    if not await check_membership(user_id, context):
        await update.message.reply_text("❌ Kamu harus join channel @LPKJUBEL dulu untuk bisa kirim post.")
        return

    if not any(tag in text for tag in ALLOWED_HASHTAGS):
        await update.message.reply_text("❗Gunakan hashtag yang sesuai seperti #LpkBeli atau #Lpkjual.")
        return

    await context.bot.send_message(chat_id=TARGET_CHANNEL, text=text)
    await update.message.reply_text("✅ Pesanmu sudah berhasil dikirim ke channel!")

if __name__ == "__main__":
    app = ApplicationBuilder().token(BOT_TOKEN).build()
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    app.run_polling()
