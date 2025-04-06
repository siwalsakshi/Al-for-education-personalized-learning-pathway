# Al-for-education-personalized-learning-pathway
from flask import Flask, request, jsonify
import random
import json

app = Flask(__name__)

# Mock data for lessons, quizzes, and rewards
LESSONS = {
    "Math": ["Understanding Addition", "Learning Subtraction", "Introduction to Multiplication"],
    "Science": ["The Water Cycle", "Basics of Photosynthesis", "Introduction to Gravity"]
}

QUIZZES = {
    "Math": [
        {"question": "What is 2 + 3?", "options": [4, 5, 6], "answer": 5},
        {"question": "What is 6 - 2?", "options": [3, 4, 5], "answer": 4}
    ],
    "Science": [
        {"question": "What happens to water when it boils?", "options": ["Ice", "Steam", "Liquid"], "answer": "Steam"},
        {"question": "Which process helps plants make food?", "options": ["Respiration", "Photosynthesis", "Digestion"], "answer": "Photosynthesis"}
    ]
}

USER_PROGRESS = {}

@app.route('/get_lessons', methods=['GET'])
def get_lessons():
    subject = request.args.get('subject')
    if subject in LESSONS:
        return jsonify({"lessons": LESSONS[subject]})
    return jsonify({"error": "Subject not found"}), 404

@app.route('/get_quiz', methods=['GET'])
def get_quiz():
    subject = request.args.get('subject')
    if subject in QUIZZES:
        quiz = random.choice(QUIZZES[subject])
        return jsonify({"quiz": quiz})
    return jsonify({"error": "Subject not found"}), 404

@app.route('/submit_answer', methods=['POST'])
def submit_answer():
    data = request.json
    user_id = data.get("user_id")
    subject = data.get("subject")
    question = data.get("question")
    answer = data.get("answer")
    
    if subject in QUIZZES:
        for quiz in QUIZZES[subject]:
            if quiz["question"] == question:
                correct_answer = quiz["answer"]
                if user_id not in USER_PROGRESS:
                    USER_PROGRESS[user_id] = {"score": 0, "badges": []}

                if answer == correct_answer:
                    USER_PROGRESS[user_id]["score"] += 10
                    if USER_PROGRESS[user_id]["score"] % 50 == 0:  # Gamified badge logic
                        USER_PROGRESS[user_id]["badges"].append("Achievement: Mastery Level")
                    return jsonify({"result": "Correct!", "progress": USER_PROGRESS[user_id]})
                return jsonify({"result": "Incorrect, try again!", "progress": USER_PROGRESS[user_id]})
    return jsonify({"error": "Invalid data or subject"}), 404

@app.route('/get_progress', methods=['GET'])
def get_progress():
    user_id = request.args.get('user_id')
    if user_id in USER_PROGRESS:
        return jsonify({"progress": USER_PROGRESS[user_id]})
    return jsonify({"error": "User not found"}), 404

if __name__ == '__main__':
    app.run(debug=True)
