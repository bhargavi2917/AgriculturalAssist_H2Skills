# AgriculturalAssist_H2Skills
# AgriAssist - Streamlit Frontend with Google AI Backend

import streamlit as st
import requests
from PIL import Image
import base64

# Gemini API setup (Replace with your actual API key and URL)
GEMINI_API_KEY = 'AIzaSyDMUg3po5O4BDuoWi5V5cvXtP3grC7fwCU'
VERTEX_PEST_MODEL_URL = 'YOUR_VERTEX_AI_ENDPOINT_FOR_PEST_DETECTION'
VERTEX_PRICE_MODEL_URL = 'YOUR_VERTEX_AI_ENDPOINT_FOR_PRICE_PREDICTION'

st.set_page_config(page_title="AgriAssist - Smart Farming Companion")
st.title("🌱 AgriAssist: AI-Powered Farming Assistant")

# Sidebar menu
menu = st.sidebar.radio("Choose Feature", [
    "AI Farming Chat Assistant",
    "Pest Detection",
    "Market Price Forecast"
])

# 1. AI Chat Assistant
if menu == "AI Farming Chat Assistant":
    st.header("💬 Ask the AI Anything about Farming")
    user_query = st.text_area("Type your question:")

    if st.button("Ask Gemini") and user_query:
        headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {GEMINI_API_KEY}"
        }
        body = {
            "contents": [{"parts": [{"text": user_query}]}]
        }
        response = requests.post(
            "https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent",
            headers=headers,
            json=body
        )
        if response.status_code == 200:
            reply = response.json()['candidates'][0]['content']['parts'][0]['text']
            st.success("Gemini Response:")
            st.write(reply)
        else:
            st.error("Error communicating with Gemini API")

# 2. Pest Detection
elif menu == "Pest Detection":
    st.header("🐛 Upload Image for Pest/Disease Detection")
    uploaded_file = st.file_uploader("Choose a crop image", type=["jpg", "png", "jpeg"])

    if uploaded_file is not None:
        st.image(uploaded_file, caption="Uploaded Image", use_column_width=True)
        image_bytes = uploaded_file.read()
        encoded_image = base64.b64encode(image_bytes).decode("utf-8")

        if st.button("Analyze Image"):
            payload = {"instances": [{"b64": encoded_image}]}
            response = requests.post(VERTEX_PEST_MODEL_URL, json=payload)
            if response.status_code == 200:
                result = response.json()['predictions'][0]
                st.success("Detected Pest/Disease:")
                st.write(result)
            else:
                st.error("Error calling Vertex AI pest model")

# 3. Market Price Forecast
elif menu == "Market Price Forecast":
    st.header("📈 Predict Market Price for Crop")
    crop_name = st.selectbox("Select Crop", ["Wheat", "Rice", "Tomato", "Cotton"])
    region = st.text_input("Enter Region")

    if st.button("Forecast Price"):
        data = {
            "instances": [{"crop": crop_name, "region": region}]
        }
        response = requests.post(VERTEX_PRICE_MODEL_URL, json=data)
        if response.status_code == 200:
            price = response.json()['predictions'][0]['price']
            st.success(f"Predicted price for {crop_name} in {region}: ₹{price:.2f}/quintal")
        else:
            st.error("Error fetching price prediction from Vertex AI")
