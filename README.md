import requests
from flask import Flask, request, render_template
import re

app = Flask(__name__)

#AI API for fake news detection 
API_URL = "https://organizer-api.com/analyze_news"  # Use the real API endpoint
API_KEY = "your_api_key_here"  # Replace with actual API key

#Function to call AI API for fake news analysis
def analyze_news_with_api(headline):
    headers = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}
    data = {"text": headline}

    response = requests.post(API_URL, json=data, headers=headers)

    if response.status_code == 200:
        return response.json()  
    else:
        return {"error": "API request failed"}

#Function to highlight suspicious words
def highlight_suspicious_words(text):
    suspicious_keywords = ["guaranteed", "free money", "urgent", "click here", "congratulations"]
    
    for word in suspicious_keywords:
        text = re.sub(rf"\b{word}\b", f"<mark>{word}</mark>", text, flags=re.IGNORECASE)

    return text

#Web Interface using Flask
@app.route("/", methods=["GET", "POST"])
def index():
    result = None
    highlighted_text = ""

    if request.method == "POST":
        headline = request.form["news_headline"]
        analysis = analyze_news_with_api(headline)

        if "error" in analysis:
            result = "Error: Unable to analyze news. Try again later."
        else:
            probability = analysis.get("fake_news_probability", 0)
            result = f"🛑 Fake News Probability: {probability * 100:.2f}%"
            highlighted_text = highlight_suspicious_words(headline)

    return render_template("index.html", result=result, highlighted_text=highlighted_text)

#Run the Flask App
if __name__ == "__main__":
    app.run(debug=True)

#html 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fake News Verifier</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 50px; }
        textarea { width: 100%; height: 50px; }
        button { background: #007BFF; color: white; padding: 10px; border: none; cursor: pointer; }
        mark { background-color: yellow; }
    </style>
</head>
<body>
    <h2>Fake News Verifier with AI</h2>
    <form method="POST">
        <textarea name="news_headline" placeholder="Enter a news headline..." required></textarea>
        <button type="submit">Analyze</button>
    </form>

    {% if result %}
        <h3>Result:</h3>
        <p>{{ result }}</p>
    {% endif %}

    {% if highlighted_text %}
        <h3>Suspicious Words Highlighted:</h3>
        <p>{{ highlighted_text | safe }}</p>
    {% endif %}
</body>
</html>
