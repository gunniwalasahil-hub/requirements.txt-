# requirements.txt-

streamlit
yfinance
pandas
plotly
import streamlit as st
import yfinance as yf
import pandas as pd
import plotly.graph_objects as go

# ---------------- PAGE CONFIG ----------------
st.set_page_config(
    page_title="Stock Market Dashboard",
    page_icon="📈",
    layout="wide"
)

# ---------------- CUSTOM CSS ----------------
st.markdown("""
<style>
.stApp {
    background-color: #0e1117;
    color: white;
}
h1, h2, h3 {
    color: #00d4aa;
}
[data-testid="stMetric"] {
    background-color: #1c2333;
    padding: 15px;
    border-radius: 10px;
}
</style>
""", unsafe_allow_html=True)

# ---------------- HEADER ----------------
st.title("📈 Stock Market Dashboard")
st.caption("Live Market Data | Stock Analysis | Price Charts")

# ---------------- SIDEBAR ----------------
st.sidebar.title("⚙️ Dashboard Settings")

stock_symbol = st.sidebar.text_input(
    "Enter Stock Symbol",
    value="RELIANCE.NS"
).upper().strip()

period = st.sidebar.selectbox(
    "Select Time Period",
    ["1mo", "3mo", "6mo", "1y", "2y", "5y"],
    index=2
)

st.sidebar.markdown("---")
st.sidebar.markdown("### Popular Indian Stocks")

popular_stocks = {
    "Reliance": "RELIANCE.NS",
    "TCS": "TCS.NS",
    "Infosys": "INFY.NS",
    "HDFC Bank": "HDFCBANK.NS",
    "Tata Motors": "TATAMOTORS.NS",
    "Nifty 50": "^NSEI"
}

selected_stock = st.sidebar.selectbox(
    "Choose a stock",
    list(popular_stocks.keys())
)

if st.sidebar.button("Load Selected Stock"):
    stock_symbol = popular_stocks[selected_stock]

# ---------------- MARKET OVERVIEW ----------------
st.subheader("🌍 Market Overview")

market_symbols = {
    "NIFTY 50": "^NSEI",
    "SENSEX": "^BSESN",
    "NASDAQ": "^IXIC",
    "S&P 500": "^GSPC"
}

market_cols = st.columns(4)

for col, (name, symbol) in zip(market_cols, market_symbols.items()):
    try:
        data = yf.Ticker(symbol).history(period="5d")

        if len(data) >= 2:
            latest = data["Close"].iloc[-1]
            previous = data["Close"].iloc[-2]
            change = latest - previous
            percent = (change / previous) * 100

            col.metric(
                name,
                f"{latest:,.2f}",
                f"{percent:+.2f}%"
            )
        else:
            col.metric(name, "No Data")

    except Exception:
        col.metric(name, "Unavailable")

st.markdown("---")

# ---------------- STOCK DATA ----------------
st.subheader(f"📊 Stock Analysis: {stock_symbol}")

try:
    stock = yf.Ticker(stock_symbol)
    df = stock.history(period=period)

    if df.empty:
        st.error("Stock data not found. Please enter a valid symbol.")
        st.stop()

    latest = df["Close"].iloc[-1]
    previous = df["Close"].iloc[-2] if len(df) >= 2 else latest

    change = latest - previous
    percent_change = (change / previous) * 100 if previous else 0

    day_high = df["High"].iloc[-1]
    day_low = df["Low"].iloc[-1]
    volume = df["Volume"].iloc[-1]

    # ---------------- METRICS ----------------
    col1, col2, col3, col4, col5 = st.columns(5)

    col1.metric("Current Price", f"₹{latest:,.2f}")
    col2.metric("Change", f"₹{change:,.2f}", f"{percent_change:+.2f}%")
    col3.metric("Day High", f"₹{day_high:,.2f}")
    col4.metric("Day Low", f"₹{day_low:,.2f}")
    col5.metric("Volume", f"{volume:,.0f}")

    # ---------------- CHART ----------------
    st.subheader("📉 Stock Price Chart")

    fig = go.Figure()

    fig.add_trace(
        go.Candlestick(
            x=df.index,
            open=df["Open"],
            high=df["High"],
            low=df["Low"],
            close=df["Close"],
            name="Price"
        )
    )

    fig.update_layout(
        title=f"{stock_symbol} Stock Price",
        xaxis_title="Date",
        yaxis_title="Price",
        template="plotly_dark",
        height=550,
        xaxis_rangeslider_visible=False
    )

    st.plotly_chart(fig, use_container_width=True)

    # ---------------- DATA TABLE ----------------
    st.subheader("📋 Historical Stock Data")

    display_df = df.sort_index(ascending=False).copy()
    display_df.index = display_df.index.strftime("%Y-%m-%d")

    st.dataframe(
        display_df,
        use_container_width=True
    )

    # ---------------- DOWNLOAD ----------------
    csv = df.to_csv().encode("utf-8")

    st.download_button(
        label="📥 Download Stock Data CSV",
        data=csv,
        file_name=f"{stock_symbol}_stock_data.csv",
        mime="text/csv"
    )

except Exception as e:
    st.error(f"Error loading stock data: {e}")

# ---------------- FOOTER ----------------
st.markdown("---")
st.markdown(
    "💻 **Stock Market Dashboard** | Internship Project | "
    "Developed using Python & Streamlit"
)

st.caption(
    "Note: Market data may be delayed. This dashboard is for "
    "educational purposes only, not investment advice."
)
