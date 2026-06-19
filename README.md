import os
import time
import requests
import yfinance as yf
import pandas as pd

TOKEN = "8855739302:AAFMmx-t-BdFfzVz7znm3ZjKDrm0puHQgZU"
subscribers = set()

def get_pro_signal():
    try:
        gold = yf.Ticker("GC=F")
        df = gold.history(period="3mo")
        price = round(df['Close'].iloc[-1], 2)
        
        # حساب المؤشرات
        ema50 = df['Close'].rolling(window=50).mean().iloc[-1]
        delta = df['Close'].diff()
        gain = (delta.where(delta > 0, 0)).rolling(window=14).mean()
        loss = (-delta.where(delta < 0, 0)).rolling(window=14).mean()
        rs = gain / loss
        rsi = 100 - (100 / (1 + rs.iloc[-1]))
        
        # منطق التوصية الاحترافي
        trend = "صاعد" if ema50 < price else "هابط"
        signal = "شراء قوي" if rsi < 30 else ("بيع قوي" if rsi > 70 else ("انتظار/حيادي"))
        
        tp1, tp2 = (price*1.005, price*1.01) if trend == "صاعد" else (price*0.995, price*0.99)
        sl = price*0.992 if trend == "صاعد" else price*1.008
        
        msg = (f"📈 **تقرير الذهب اللحظي**\n"
               f"السعر الحالي: {price}\n"
               f"الاتجاه العام: {trend}\n"
               f"قوة المؤشر (RSI): {round(rsi, 2)}\n"
               f"التوصية: **{signal}**\n\n"
               f"🎯 هدف 1: {round(tp1, 2)}\n🎯 هدف 2: {round(tp2, 2)}\n🛑 وقف الخسارة: {round(sl, 2)}")
        return msg
    except:
        return None

# دالة التشغيل الرئيسية
def main():
    print("البوت يعمل بكامل طاقته...")
    while True:
        try:
            # تحديث المشتركين وإرسال التقارير
            msg = get_pro_signal()
            if msg:
                for cid in list(subscribers):
                    requests.post(f"https://api.telegram.org/bot{TOKEN}/sendMessage", data={"chat_id": cid, "text": msg, "parse_mode": "Markdown"})
        except:
            pass
        time.sleep(600) # يرسل تحديثاً كاملاً كل 10 دقائق

if name == "__main__":
    main()