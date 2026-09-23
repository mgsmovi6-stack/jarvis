from flask import Flask, request, jsonify, send_from_directory
from openai import OpenAI
import os

app = Flask(__name__)

# API key Render Environment Variable से आएगी
api_key = os.getenv("OPENAI_API_KEY")

if not api_key:
    raise RuntimeError("OPENAI_API_KEY set nahi hai.")

client = OpenAI(api_key=api_key)

SYSTEM_PROMPT = """
Tum Jarvis ho, ek helpful personal AI assistant.

User Hindi, Hinglish, English aur doosri languages mein
baat kar sakta hai.

User ki language automatically samjho.
Default response simple Hindi/Hinglish mein do.

Tum:
- General knowledge ke questions answer karo
- Maths solve karo
- Science samjhao
- Python, HTML, CSS, JavaScript sikhao
- Coding mein help karo
- Technology explain karo
- Games aur websites banane mein help karo
- Step-by-step explanation do
- User detail maange to detailed answer do

Fixed commands tak limited mat raho.
Natural language ko samajhkar answer do.

Agar current information chahiye aur tumhare paas
current data available nahi hai, to clearly batao.
"""

@app.route("/")
def home():
    return send_from_directory(".", "index.html")


@app.route("/ask", methods=["POST"])
def ask():

    data = request.get_json(silent=True) or {}
    question = str(data.get("question", "")).strip()

    if not question:
        return jsonify({
            "answer": "Bhai, kuch poochho."
        })

    try:
        response = client.responses.create(
            model="gpt-5.6-luna",
            instructions=SYSTEM_PROMPT,
            input=question
        )

        return jsonify({
            "answer": response.output_text
        })

    except Exception as e:
        print("OpenAI error:", repr(e))

        return jsonify({
            "answer": "Bhai, AI se connection mein problem aa gayi."
        }), 500


@app.get("/health")
def health():
    return jsonify({"status": "Jarvis online"})


if __name__ == "__main__":
    port = int(os.environ.get("PORT", 5000))

    app.run(
        host="0.0.0.0",
        port=port,
        debug=False
    )
