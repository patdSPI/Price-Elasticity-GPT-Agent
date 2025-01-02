import sqlite3
import openai
import streamlit as st
from pathlib import Path

OPENAI_API_KEY = st.secrets['api_key']
DATABASE_PATH = "restaurant_sales.sqlite3"
MODEL = 'gpt-4o'

def get_db_path(filename):
    db_path = Path(__file__).parents[0] / filename
    return db_path.__str__()

def get_conn(filename):
    db_path = get_db_path(filename)
    conn = sqlite3.connect(db_path)
    return conn

def handle_user_question(conn, api_key, user_query, model):
    # Use GPT for general questions
    if not any(keyword in user_query.lower() for keyword in ["store", "category", "sales", "units", "elasticity"]):
        openai.api_key = api_key
        response = openai.ChatCompletion.create(
            model=model,
            messages=[
                {"role": "system", "content": "You are a helpful assistant."},
                {"role": "user", "content": user_query}
            ]
        )
        return response['choices'][0]['message']['content']
    
    # Handle database queries
    try:
        cursor = conn.cursor()
        if "highest netsales" in user_query.lower():
            cursor.execute("SELECT Store, MAX(NetSales) FROM sales_data")  # Replace 'sales_data' with your actual table name
            result = cursor.fetchone()
            return f"The store with the highest NetSales is {result[0]} with sales of {result[1]}."
        elif "largest netunits" in user_query.lower():
            cursor.execute("SELECT ItemDescription, MAX(NetUnits) FROM sales_data")  # Replace 'sales_data' with your actual table name
            result = cursor.fetchone()
            return f"The item with the largest NetUnits sold is {result[0]} with {result[1]} units."
        else:
            return "I'm not sure how to answer that. Can you rephrase or provide more details?"
    except Exception as e:
        return f"An error occurred while querying the database: {e}"

def main():
    st.title("Restaurant Pricing Analyst")
    if "messages" not in st.session_state:
        st.session_state["messages"] = []

        welcome_message = (
            "Hello, and welcome to the **Restaurant Pricing Analyst**!\n\n"
            "I'm here to help you explore restaurant sales data and elasticity. "
            "Below are some **sample questions** you might ask:\n"
            "- *Which store has the highest NetSales?*\n"
            "- *What is the sales-weighted elasticity for each Category?*\n"
            "- *Which item descriptions have the largest NetUnits sold?*\n\n"
            "Feel free to experiment by typing your question in the chat box below!"
        )

        st.session_state["messages"].append({"role": "assistant", "content": welcome_message})

    conn = get_conn(DATABASE_PATH)
    user_query = st.chat_input("Ask a question about the restaurant data...")

    if user_query:
        st.session_state["messages"].append({"role": "user", "content": user_query})
        
        try:
            answer = handle_user_question(conn, OPENAI_API_KEY, user_query, MODEL)
        except Exception as e:
            answer = f"An error occurred: {e}"
        
        st.session_state["messages"].append({"role": "assistant", "content": answer})

        for msg in st.session_state["messages"]:
            if msg["role"] == "user":
                with st.chat_message("user"):
                    st.write(msg["content"])
            elif msg["role"] == "assistant":
                with st.chat_message("assistant"):
                    st.write(msg["content"])

if __name__ == "__main__":
    main()
