import asyncio
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes

# Replace with your actual bot token from BotFather
BOT_TOKEN = "Token"

# Store user data
user_data = {}

# Start command
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.chat_id
    print(f"/start received from {user_id}")
    await update.message.reply_text(
        "✨ Welcome to ShadowFortune! ✨\n"
        "We’re thrilled to have you here! 🎉\n"
        "Before we proceed, please provide your details:\n\n"
        "1. Your Full Name:\n"
        "2. Your Email Address:\n"
        "3. Your Phone Number:\n"
        "4. Your Location (City/Country):\n\n"
        "Your information is safe with us. 🔒"
    )
    user_data[user_id] = {}

# Store user details
async def store_details(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.chat_id
    text = update.message.text
    details = text.split("\n")
    
    if len(details) < 4:
        await update.message.reply_text("Please provide all details in the correct format.")
        return
    
    user_data[user_id] = {
        "name": details[0],
        "email": details[1],
        "phone": details[2],
        "location": details[3]
    }
    
    await update.message.reply_text(
        f"Thank you, {details[0]}! 🙌\n"
        "Now, let’s get you started with our exclusive products:\n\n"
        "1. Basic – 100 (Daily Limit: 500, Withdrawal Limit: 2,500)\n"
        "2. Standard – 300 (Daily Limit: 1,000, Withdrawal Limit: 5,000)\n"
        "3. Premium – 500 (Daily Limit: 2,000, Withdrawal Limit: 10,000)\n"
        "4. Custom – Contact us for pricing.\n\n"
        "Reply with the number of the product you’d like to purchase."
    )

# Product selection
async def select_product(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.chat_id
    text = update.message.text.strip()
    
    products = {
        "1": "Basic – 100",
        "2": "Standard – 300",
        "3": "Premium – 500",
        "4": "Custom – Contact us"
    }
    
    if text not in products:
        await update.message.reply_text("Please select a valid product by number (1-4).")
        return
    
    user_data[user_id]["product"] = products[text]
    
    await update.message.reply_text(
        f"Great choice, {user_data[user_id]['name']}! 🎉\n"
        f"You’ve selected: {products[text]}\n\n"
        "Now, please choose your preferred delivery method:\n"
        "1. Uber\n"
        "2. Bolt\n"
        "3. Bodaboda\n"
        "4. Wells Fargo\n\n"
        "Reply with the number of your preferred delivery method."
    )

# Delivery selection
async def select_delivery(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.chat_id
    text = update.message.text.strip()
    
    delivery_methods = {
        "1": "Uber",
        "2": "Bolt",
        "3": "Bodaboda",
        "4": "Wells Fargo"
    }
    
    if text not in delivery_methods:
        await update.message.reply_text("Please select a valid delivery method (1-4).")
        return
    
    user_data[user_id]["delivery"] = delivery_methods[text]
    
    await update.message.reply_text(
        f"Perfect! 🚀\nYour order will be delivered via {delivery_methods[text]}.\n\n"
        "To complete your order, please send $110 in BTC to the following wallet address:\n\n"
        "📌 **bc1qkhk4mq56vrlhyp3rrxtlzkr8578a9ka9nhexlk**\n\n"
        "This includes:\n"
        "✅ $100 for the product.\n"
        "✅ $10 for delivery.\n\n"
        "Once payment is confirmed, reply with 'PAID'."
    )

# Payment confirmation
async def confirm_payment(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.chat_id
    text = update.message.text.strip().lower()
    
    if text == "paid":
        admin_id = 6657763025  # Replace with your actual admin ID (as an integer)
        await context.bot.send_message(
            admin_id,
            (
                f"Payment received from {user_data[user_id]['name']}.\n"
                f"Order details:\n"
                f"📦 Product: {user_data[user_id]['product']}\n"
                f"🚚 Delivery: {user_data[user_id]['delivery']}\n"
                f"📍 Location: {user_data[user_id]['location']}\n"
                f"📧 Email: {user_data[user_id]['email']}\n"
                f"📞 Phone: {user_data[user_id]['phone']}"
            )
        )
        
        await update.message.reply_text(
            f"Thank you, {user_data[user_id]['name']}! 💳\n"
            "We have received your payment and are processing your order. 🚀\n\n"
            "You’ll receive:\n"
            "📧 A confirmation email shortly.\n"
            "📞 A phone call within the next 10 minutes to finalize pickup details.\n\n"
            "Thank you for choosing ShadowFortune! Need help? Type 'HELP'."
        )

# Help command
async def help_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Need assistance? Contact our support team at support@shadowfortune.com.")

# Main function to start the bot
async def main():
    app = Application.builder().token(BOT_TOKEN).build()

    # Add command handlers
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("help", help_command))
    
    # Add message handlers (consider using ConversationHandler for a better flow)
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, store_details))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, select_product))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, select_delivery))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, confirm_payment))

    print("Bot is running...")
    await app.run_polling()

if __name__ == "__main__":
    asyncio.run(main())
