# Realistic-TTS-voice
Flask==2.3.2
torch==2.0.1
transformers==4.30.2
whisper==1.0.0
coqui-tts==0.11.1
from flask import Flask, request, send_file, jsonify
import torch
from transformers import pipeline
import whisper
from coqui_tts import TTS
import os

app = Flask(__name__)

# Load OpenAI Whisper (for speech-to-text)
whisper_model = whisper.load_model("base")

# Load Hugging Face Transformers (for TTS)
transformers_tts = pipeline("text-to-speech", model="facebook/fastspeech2-hifigan")

# Load Coqui TTS (for voice cloning)
coqui_tts = TTS(model_name="tts_models/multilingual/multi-dataset/your_tts", progress_bar=False)

@app.route('/')
def home():
    return '''
    <h1>Advanced Text-to-Speech System</h1>
    <form action="/convert" method="post">
        <textarea name="text" rows="5" cols="40" placeholder="Enter text here..."></textarea><br><br>
        <label for="language">Language:</label>
        <select name="language" id="language">
            <option value="hi">Hindi</option>
            <option value="en">English</option>
        </select><br><br>
        <button type="submit">Convert to Speech</button>
    </form>
    '''

@app.route('/convert', methods=['POST'])
def convert():
    text = request.form['text']
    language = request.form['language']

    # Use Hugging Face Transformers for TTS
    audio_output = transformers_tts(text, language=language)
    audio_file = "output_transformers.mp3"
    with open(audio_file, "wb") as f:
        f.write(audio_output["audio"])

    # Use Coqui TTS for voice cloning (optional)
    coqui_tts.tts_to_file(text=text, file_path="output_coqui.wav")

    # Use OpenAI Whisper for speech-to-text (optional)
    transcription = whisper_model.transcribe(audio_file)
    print("Transcription:", transcription['text'])

    return send_file(audio_file, as_attachment=True)

if __name__ == '__main__':
    app.run(debug=True)
<!DOCTYPE html>
<html>
<head>
    <title>Advanced TTS</title>
</head>
<body>
    <h1>Advanced Text-to-Speech</h1>
    <textarea id="text" rows="5" cols="40" placeholder="Enter text here..."></textarea><br><br>
    <label for="language">Language:</label>
    <select id="language">
        <option value="hi">Hindi</option>
        <option value="en">English</option>
    </select><br><br>
    <button onclick="convertText()">Convert to Speech</button>
    <audio id="audio" controls></audio>

    <script>
        async function convertText() {
            const text = document.getElementById('text').value;
            const language = document.getElementById('language').value;
            const response = await fetch('/convert', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded',
                },
                body: `text=${encodeURIComponent(text)}&language=${language}`,
            });
            const audioBlob = await response.blob();
            const audioUrl = URL.createObjectURL(audioBlob);
            document.getElementById('audio').src = audioUrl;
        }
    </script>
</body>
</html>
