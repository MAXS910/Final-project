# Final-project
from flask import Flask, request, jsonify
import math
import re

app = Flask(__name__)

def calculate_entropy(password):
    """
    Calculate entropy of the password based on length and character variety.
    """
    length = len(password)
    char_variety = 0

    if any(c.islower() for c in password):
        char_variety += 26  # Lowercase letters
    if any(c.isupper() for c in password):
        char_variety += 26  # Uppercase letters
    if any(c.isdigit() for c in password):
        char_variety += 10  # Numbers
    if any(c in "!@#$%^&*()_+-=[]{}|;:,.<>?/`~" for c in password):
        char_variety += 32  # Special characters

    if char_variety == 0:
        return 0

    entropy = length * math.log2(char_variety)
    return round(entropy, 2)

def generate_feedback(password):
    """
    Generate feedback on password strength based on various criteria.
    """
    feedback = []
    if len(password) < 8:
        feedback.append("Increase the length to at least 8 characters.")
    if not any(c.isupper() for c in password):
        feedback.append("Include at least one uppercase letter.")
    if not any(c.isdigit() for c in password):
        feedback.append("Include at least one number.")
    if not any(c in "!@#$%^&*()_+-=[]{}|;:,.<>?/`~" for c in password):
        feedback.append("Include at least one special character.")
    if re.search(r'(.)\1\1', password):
        feedback.append("Avoid repeated characters.")
    if re.search(r'(123|abc|password|qwerty)', password, re.IGNORECASE):
        feedback.append("Avoid common patterns or weak words.")
    return feedback

@app.route('/check', methods=['POST'])
def check_password():
    """
    API endpoint to check password strength and provide feedback.
    """
    data = request.json
    password = data.get('password', '')

    if not password:
        return jsonify({'error': 'Password cannot be empty!'}), 400

    entropy = calculate_entropy(password)
    feedback = generate_feedback(password)
    strength = 'Weak'
    if entropy >= 60:
        strength = 'Strong'
    elif entropy >= 40:
        strength = 'Medium'

    return jsonify({
        'entropy': entropy,
        'feedback': feedback,
        'strength': strength
    })

if __name__ == '__main__':
    app.run(debug=True)
