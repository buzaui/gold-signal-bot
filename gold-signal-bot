import requests
from telegram import Bot
from datetime import datetime
import time

# === CONFIG ===
BOT_TOKEN = '8178173229:AAE2Tk2Tfc5giZ6pwDEt6QAI7_euklWkle4'  # NEW TOKEN
CHANNEL_ID = '@Manbuza12Id'
API_KEY = 'c02a230fa2864bab8cc61b738ec333cb'
SYMBOL = 'XAU/USD'
INTERVAL = '1h'
MA_SHORT = 10
MA_LONG = 60
ATR_PERIOD = 14

bot = Bot(token=BOT_TOKEN)

# === Fetch gold candles ===
def fetch_candles():
    url = f'https://api.twelvedata.com/time_series?symbol={SYMBOL}&interval={INTERVAL}&outputsize=100&apikey={API_KEY}'
    r = requests.get(url).json()
    candles = r['values'][::-1]
    return candles

# === Calculate SMA ===
def calc_sma(candles, period):
    closes = [float(c['close']) for c in candles[-period:]]
    return sum(closes) / period

# === Calculate ATR ===
def calc_atr(candles, period=ATR_PERIOD):
    trs = []
    for i in range(1, period + 1):
        high = float(candles[-i]['high'])
        low = float(candles[-i]['low'])
        close_prev = float(candles[-i - 1]['close'])
        tr = max(high - low, abs(high - close_prev), abs(low - close_prev))
        trs.append(tr)
    return sum(trs) / period

# === Detect 10/60 crossover ===
def check_crossover(candles):
    sma_short_prev = calc_sma(candles[:-1], MA_SHORT)
    sma_long_prev = calc_sma(candles[:-1], MA_LONG)
    sma_short_now = calc_sma(candles, MA_SHORT)
    sma_long_now = calc_sma(candles, MA_LONG)

    if sma_short_prev < sma_long_prev and sma_short_now > sma_long_now:
        return 'BUY'
    elif sma_short_prev > sma_long_prev and sma_short_now < sma_long_now:
        return 'SELL'
    return None

# === Get DXY context ===
def get_dxy_context():
    url = f'https://api.twelvedata.com/time_series?symbol=DXY/USD&interval=1h&outputsize=20&apikey={API_KEY}'
    r = requests.get(url).json()
    candles = r['values'][::-1]
    current = float(candles[-1]['close'])
    sma10 = sum([float(c['close']) for c in candles[-10:]]) / 10

    if current > sma10:
        return f"DXY is rising ({current:.2f} > MA10 {sma10:.2f}) → bearish gold pressure"
    else:
        return f"DXY is falling ({current:.2f} < MA10 {sma10:.2f}) → bullish gold support"

# === Send signal to Telegram ===
def send_signal(direction, entry, sl, tp, atr):
    now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    dxy_note = get_dxy_context()
    comment = f"Gold triggered a {direction} signal from a 10/60 MA crossover. ATR={round(atr, 2)}. {dxy_note}"

    msg = f"""
**AI Gold Signal – {now}**

Pair: XAU/USD  
Direction: {direction}  
Entry: {entry}  
Stop Loss: {sl}  
Take Profit: {tp}  
Risk: 1%

AI Note: {comment}
"""
    bot.send_message(chat_id=CHANNEL_ID, text=msg, parse_mode='Markdown')

# === MAIN LOOP ===
while True:
    try:
        candles = fetch_candles()
        action = check_crossover(candles)
        if action:
            entry_price = float(candles[-1]['close'])
            atr = calc_atr(candles)

            if action == 'BUY':
                sl = round(entry_price - (2 * atr), 2)
                tp = round(entry_price + (3 * atr), 2)
            else:
                sl = round(entry_price + (2 * atr), 2)
                tp = round(entry_price - (3 * atr), 2)

            send_signal(action, entry_price, sl, tp, atr)
        else:
            print("No crossover. No trade signal.")

        time.sleep(3600)  # Check once per hour
    except Exception as e:
        print("Error:", e)
        time.sleep(300)
