# رادار قناص السيولة (SPX Liquidity Sniper) 🎯

تطبيق متخصص في رصد سيولة عقود الخيارات (Options) لمؤشر SPX وتنبيه المستخدم عند وصول السعر لمستويات الدعم والمقاومة الذهبية.

## 🚀 المميزات
* رصد حي لسعر SPX.
* تنبيهات صوتية عند ملامسة الأهداف.
* تحليل آلي لأكثر العقود سيولة (Liquidity King).

## 🛠 المتطلبات
* Python 3.x
* yfinance
* Streamlit (في حال تشغيله كواجهة ويب)

import streamlit as st
import yfinance as yf
import pandas as pd
import time
from datetime import datetime

# إعدادات الصفحة
st.set_page_config(page_title="SPX Sniper", layout="wide")
st.title("🎯 رادار قناص السيولة - SPX")

# المستويات الذهبية
RESISTANCE = 6861.94
SUPPORT = 6846.52

def get_liquidity_king(sig_type):
    try:
        spx = yf.Ticker("^SPX")
        expiry = spx.options[0]
        opt_chain = spx.option_chain(expiry)
        df = opt_chain.calls if sig_type == 'CALL' else opt_chain.puts
        pro_contracts = df[df['ask'] < 2.00].copy()
        if pro_contracts.empty: return None
        pro_contracts['liquidity_score'] = pro_contracts['volume'] + pro_contracts['openInterest']
        return pro_contracts.sort_values(by='liquidity_score', ascending=False).iloc[0]
    except: return None

# واجهة التطبيق
col1, col2 = st.columns(2)
placeholder = st.empty()

while True:
    try:
        spx = yf.Ticker("^SPX")
        price = spx.fast_info['last_price']
        
        with placeholder.container():
            st.metric("سعر SPX الحالي", f"${price:.2f}")
            
            if price >= RESISTANCE:
                st.error("🔥 إشارة CALL - اختراق مقاومة!")
                opt = get_liquidity_king('CALL')
                if opt is not None:
                    st.write(f"📦 العقد الأقوى: {opt['contractSymbol']}")
                    st.write(f"💵 سعر الدخول: ${opt['ask']}")
            elif price <= SUPPORT:
                st.success("✅ إشارة PUT - كسر دعم!")
                opt = get_liquidity_king('PUT')
                if opt is not None:
                    st.write(f"📦 العقد الأقوى: {opt['contractSymbol']}")
                    st.write(f"💵 سعر الدخول: ${opt['ask']}")
            else:
                st.info("🟡 الرادار يراقب.. السوق هادي")
        
        time.sleep(10)
    except:
        st.warning("جاري محاولة الاتصال بالسوق...")
        time.sleep(5)
