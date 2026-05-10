import os
from flask import Flask, render_template, jsonify
import requests
import pandas as pd
import pandas_ta as ta

app = Flask(__name__)

def get_million_dollar_signal(symbol):
    try:
        # 1. جلب البيانات (15 دقيقة لتجنب التذبذب الكاذب)
        url = f"https://api.binance.com/api/v3/klines?symbol={symbol}&interval=15m&limit=100"
        data = requests.get(url, timeout=10).json()
        
        df = pd.DataFrame(data, columns=['time', 'open', 'high', 'low', 'close', 'vol', 'close_time', 'q_vol', 'trades', 'tb_base', 'tb_quote', 'ignore'])
        df['close'] = df['close'].astype(float)
        df['vol'] = df['vol'].astype(float)

        # 2. التحليل الفني المتقدم
        df['rsi'] = ta.rsi(df['close'], length=14)
        bbands = ta.bbands(df['close'], length=20, std=2)
        df = pd.concat([df, bbands], axis=1)
        
        last_close = df['close'].iloc[-1]
        last_rsi = df['rsi'].iloc[-1]
        lower_band = df['BBL_20_2.0'].iloc[-1]
        upper_band = df['BBU_20_2.0'].iloc[-1]
        
        # 3. رصد سيولة الحيتان
        avg_vol = df['vol'].mean()
        current_vol = df['vol'].iloc[-1]
        is_whale = current_vol > (avg_vol * 2.5) # سيولة أكبر بـ 2.5 مرة من المعتاد

        # 4. بناء استراتيجية الدخول والخروج (الربحية)
        status = "SCANNING MARKET ⏳"
        color = "#444444"
        entry_type = "WAIT"
        tp = last_close
        sl = last_close

        # استراتيجية الدخول (Buy/Long) - التقاط القاع مع دخول حوت
        if last_rsi < 35 and last_close <= lower_band and is_whale:
            status = "WHALE ENTRY (BUY) 🟢"
            color = "#00ffd5"
            entry_type = "LONG"
            tp = last_close * 1.02 # هدف ربح 2%
            sl = last_close * 0.99 # وقف الخسارة الصارم 1%
        
        # استراتيجية الخروج/البيع (Sell/Short) - التشبع والاصطدام بالسقف
        elif last_rsi > 65 and last_close >= upper_band:
            status = "TAKE PROFIT / SELL 🔴"
            color = "#ff0055"
            entry_type = "SHORT"
            tp = last_close * 0.98
            sl = last_close * 1.01

        return {
            "price": f"{last_close:,.2f}",
            "status": status,
            "color": color,
            "entry": entry_type,
            "tp": f"{tp:,.2f}",
            "sl": f"{sl:,.2f}",
            "rsi": f"{last_rsi:.1f}"
        }
    except Exception as e:
        print(f"Error: {e}")
        return None

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/api/data')
def api_data():
    return jsonify({
        "gold": get_million_dollar_signal("PAXGUSDT"),
        "btc": get_million_dollar_signal("BTCUSDT")
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=int(os.environ.get('PORT', 5000)))
