from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, MessageHandler, filters, ContextTypes, ConversationHandler

# কনফিগারেশন
BOT_TOKEN = "8644325893:AAFsXMbHHUzKDvdUEq77jNB7-ucNPFnXNFg"
ADMIN_ID = "7165257016"

# ডেটা
PACKAGES = {
    "p1": {"name": "1 Mo(Non-Download)", "price": "140 TK"}, "p2": {"name": "6 Mo(Non-Download)", "price": "700 TK"},
    "p3": {"name": "1 Yr(Non-Download)", "price": "1500 TK"}, "p4": {"name": "Lifetime(Non-Download)", "price": "2800 TK"},
    "p5": {"name": "1 Mo(Download)", "price": "399 TK"}, "p6": {"name": "6 Mo(Download)", "price": "1000 TK"},
    "p7": {"name": "1 Yr(Download)", "price": "2000 TK"}, "p8": {"name": "Lifetime(Download)", "price": "5600 TK"}
}

PAYMENTS = {
    "bkash": "01994747405 (Bkash)", "nagad": "01935085322 (Nagad)",
    "rocket": "01935085322 (Rocket)", "upay": "01935085322 (Upay)"
}

# স্টেটস
SELECTING_PACKAGE, SELECTING_PAYMENT, ASKING_NAME, ASKING_TXID = range(4)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [[InlineKeyboardButton(f"{v['name']} - {v['price']}", callback_data=k)] for k, v in PACKAGES.items()]
    await update.message.reply_text("প্যাকেজ নির্বাচন করুন:", reply_markup=InlineKeyboardMarkup(keyboard))
    return SELECTING_PACKAGE

async def package_callback(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    context.user_data['plan'] = PACKAGES[query.data]['name']
    context.user_data['price'] = PACKAGES[query.data]['price']
    
    keyboard = [[InlineKeyboardButton(k.upper(), callback_data=k)] for k in PAYMENTS.keys()]
    await query.edit_message_text(f"আপনি {context.user_data['plan']} সিলেক্ট করেছেন। এখন পেমেন্ট মেথড বেছে নিন:", reply_markup=InlineKeyboardMarkup(keyboard))
    return SELECTING_PAYMENT

async def payment_callback(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    method = query.data
    context.user_data['method'] = method
    await query.edit_message_text(f"সেন্ড মানি করুন এই নম্বরে: {PAYMENTS[method]}\n\nএখন আপনার নাম লিখুন:")
    return ASKING_NAME

async def get_name(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data['name'] = update.message.text
    await update.message.reply_text("আপনার ট্রানজেকশন আইডি (TXID) দিন/যে নম্বর থেকে টাকা পাঠিয়েন সেই নম্বর/আপনার টেলিগ্রাম আইডি নাম দিন:")
    return ASKING_TXID

async def get_txid(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data['txid'] = update.message.text
    # অ্যাডমিনকে রিপোর্ট
    msg = f"নতুন অর্ডার!\nপ্যাকেজ: {context.user_data['plan']}\nমূল্য: {context.user_data['price']}\nপদ্ধতি: {context.user_data['method']}\nনাম: {context.user_data['name']}\nTXID: {context.user_data['txid']}"
    await context.bot.send_message(chat_id=ADMIN_ID, text=msg)
    
    # ফাইনাল লিঙ্ক বাটন
    keyboard = [[InlineKeyboardButton("জয়েন করুন", url="https://t.me/+tfpkaQddzms3NmJk")]]
    await update.message.reply_text("ধন্যবাদ! তথ্য জমা হয়েছে।", reply_markup=InlineKeyboardMarkup(keyboard))
    return ConversationHandler.END

# মেইন ফাংশন
if __name__ == '__main__':
    app = ApplicationBuilder().token(BOT_TOKEN).build()
    conv = ConversationHandler(
        entry_points=[CommandHandler('start', start)],
        states={
            SELECTING_PACKAGE: [CallbackQueryHandler(package_callback)],
            SELECTING_PAYMENT: [CallbackQueryHandler(payment_callback)],
            ASKING_NAME: [MessageHandler(filters.TEXT, get_name)],
            ASKING_TXID: [MessageHandler(filters.TEXT, get_txid)],
        },
        fallbacks=[]
    )
    app.add_handler(conv)
    app.run_polling()