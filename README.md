# telegram-crypto-autobot-pro
crypto-auto-trade-bot/ │ ├─ bot.py          ← Telegram bot &amp; trade logic ├─ webhook.py      ← TradingView webhook endpoint ├─ requirements.txt ├─ Procfile └─ config.env      ← simpan API keys
python-telegram-bot==20.3
ccxt
Flask
gunicorn
python-dotenv
BOT_TOKEN=YOUR_TELEGRAM_BOT_TOKEN
API_KEY=YOUR_EXCHANGE_API_KEY
API_SECRET=YOUR_EXCHANGE_API_SECRET
TRADE_PAIR=BTC/USDT
import os
import ccxt
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes

BOT_TOKEN = os.getenv("BOT_TOKEN")
API_KEY = os.getenv("API_KEY")
API_SECRET = os.getenv("API_SECRET")
TRADE_PAIR = os.getenv("TRADE_PAIR", "BTC/USDT")

exchange = ccxt.binance({
    "apiKey": API_KEY,
    "secret": API_SECRET
})

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("🤖 Bot Auto Trade Aktif!")

async def buy(update: Update, context: ContextTypes.DEFAULT_TYPE):
    amount = 0.001
    order = exchange.create_market_buy_order(TRADE_PAIR, amount)
    await update.message.reply_text(f"BUY executed\n{order}")

async def sell(update: Update, context: ContextTypes.DEFAULT_TYPE):
    amount = 0.001
    order = exchange.create_market_sell_order(TRADE_PAIR, amount)
    await update.message.reply_text(f"SELL executed\n{order}")

app = ApplicationBuilder().token(BOT_TOKEN).build()
app.add_handler(CommandHandler("start", start))
app.add_handler(CommandHandler("buy", buy))
app.add_handler(CommandHandler("sell", sell))

if __name__ == "__main__":
    app.run_polling()
from flask import Flask, request
import os
import ccxt

app = Flask(__name__)

API_KEY = os.getenv("API_KEY")
API_SECRET = os.getenv("API_SECRET")
TRADE_PAIR = os.getenv("TRADE_PAIR", "BTC/USDT")

exchange = ccxt.binance({
    "apiKey": API_KEY,
    "secret": API_SECRET
})

@app.route('/webhook', methods=['POST'])
def webhook():
    data = request.json
    if 'passphrase' not in data or data['passphrase'] != 'secret123':
        return {"error": "unauthorized"}, 403

    side = data.get('side', 'buy')
    amount = float(data.get('amount', 0.001))

    if side == "buy":
        order = exchange.create_market_buy_order(TRADE_PAIR, amount)
    elif side == "sell":
        order = exchange.create_market_sell_order(TRADE_PAIR, amount)
    else:
        return {"error": "invalid side"}, 400

    return {"status": "success", "order": str(order)}, 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=int(os.environ.get("PORT", 5000)))
