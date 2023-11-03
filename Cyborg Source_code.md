import speech_recognition as sr
import pyttsx3
import re
import requests
import pyjokes
from nltk.corpus import wordnet
import keyboard
from googletrans import Translator
import pygame as py
from gtts import gTTS

# Initialize the recognizer with noise cancellation
recognizer = sr.Recognizer()
recognizer.dynamic_energy_threshold = True

# Initialize the text-to-speech engine
engine = pyttsx3.init()
engine.setProperty('rate', 150)  # Adjust the speech rate (words per minute)
engine.setProperty('volume', 1.0)  # Adjust the volume (0.0 to 1.0)
engine.setProperty('voice', 'com.apple.speech.synthesis.voice.daniel.premium')  # Change voice to a male voice

# Create a message queue
message_queue = []

# Function to add a message to the queue
def add_to_queue(text):
    message_queue.append(text)

# Function to speak messages from the queue
def speak_messages():
    while message_queue:
        message = message_queue.pop(0)
        engine.say(message)
        engine.runAndWait()

# Function for the robot to speak
def speak(text):
    add_to_queue(text)

# Function to listen and recognize speech
def listen():
    with sr.Microphone() as source:
        print("Robot: Hello, I am your festival anchor. What would you like to know or hear?")
        recognizer.adjust_for_ambient_noise(source)
        audio = recognizer.listen(source)
        try:
            user_query = recognizer.recognize_google(audio)
            return user_query
        except sr.UnknownValueError:
            return "Sorry, I didn't catch that."
        except sr.RequestError:
            return "I am having trouble with my speech recognition service. Please try again."

# Function to fetch WordNet definitions for the user's query
def get_definition(word):
    synsets = wordnet.synsets(word)
    if synsets:
        definition = synsets[0].definition()
        return definition
    else:
        return f"I couldn't find a definition for the word '{word}'."

def translate_to_malayalam(text):
    try:
        translator = Translator()
        translated = translator.translate(text, src='en', dest='ml')
        return translated.text
    except Exception as e:
        return f"Translation error: {str(e)}"

def play_music():
    py.init()
    py.mixer.init()
    py.mixer.music.load("output.mp3")
    py.mixer.music.play()
    while py.mixer.music.get_busy():
        pass
    py.mixer.quit()
def speak_gttts(text, language):
    if language == 'ml':
        text = translate_to_malayalam(text)
    tts = gTTS(text, lang='ml')  # Use 'ml' for Malayalam language
    tts.save("output.mp3")
    play_music()

def speak_pyttsx3(text, language):
    if language == 'ml':
        text = translate_to_malayalam(text)
    engine = pyttsx3.init()
    engine.say(text)
    engine.runAndWait()

# Function to get the current weather condition
def get_weather(city):
    api_key = '5cdb89e3e6991f4827d2f225f0acb41f'  # Replace with your OpenWeatherMap API key
    base_url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}"

    response = requests.get(base_url)
    data = response.json()

    if data["cod"] == 200:
        weather_info = data["weather"][0]["description"]
        temperature = data["main"]["temp"] - 273.15  # Convert temperature to Celsius
        return f"The weather in {city} is {weather_info}. The temperature is {temperature:.2f}°C."
    else:
        return f"Sorry, I couldn't fetch the weather information at the moment."

# Function to tell a random joke
def tell_joke():
    joke = pyjokes.get_joke()
    return "Here's a joke for you: " + joke

# Updated weather patterns with improved regular expressions
weather_patterns = [
    r'weather\s+in\s+(.+)',         # Pattern 1: "weather in [location]"
    r'what\s+is\s+the\s+weather\s+in\s+(.+)',  # Pattern 2: "what is the weather in [location]"
    r'weather\s+for\s+(.+)',         # Pattern 3: "weather for [location]"
    r'forecast\s+for\s+(.+)'         # Pattern 4: "forecast for [location]"
    r'hows\s+the\s+weather\s+in\s+(.+)'   # Pattern 5: "How for [location]"
]

# Function to handle weather queries
# Function to handle weather queries
def switch_to_gtts():
    global speaking_language
    global use_gtts
    speaking_language = 'ml'  # Set the language to Malayalam
    use_gtts = True
    speak_gttts("Hi, How can i help you","ml")

def switch_to_pyttsx3():
    global speaking_language
    global use_gtts
    speaking_language = 'en'  # Set the language to English
    use_gtts = False
    speak_pyttsx3("Hi, How can i help you?","en")

def handle_weather_query(query):
    location = None
    for pattern in weather_patterns:
        location_match = re.search(pattern, query, re.IGNORECASE)
        if location_match:
            location = location_match.group(1)
            break

    if location:
        weather_response = get_weather(location)
        if use_gtts:  # Check the global variable to determine which TTS mechanism to use
            speak_gttts(weather_response, speaking_language)
        else:
            speak_pyttsx3(weather_response, speaking_language)
    else:
        speak("I'm sorry, I couldn't determine the location from your query.")




def speak_on_key_press(e):
    if e.event_type == keyboard.KEY_DOWN:
        key_to_trigger = 'space'
        if e.name == key_to_trigger:
            speak_pyttsx3("Hello! I am here to assist you. Please ask your question or give a command.","en")

# Function for the festival anchor robot
def festival_anchor():
    global speaking_language
    global use_gtts
    speaking_language = 'en'  # Default language is English
    use_gtts = False

    while True:

        keyboard.hook(speak_on_key_press)
        user_query = listen()
        print("User: " + user_query)

        if "english" in user_query.lower():
            switch_to_pyttsx3()
        elif "malayalam" in user_query.lower():
            switch_to_gtts()

        if speaking_language == 'en':
            syn = wordnet.synsets(user_query) # want to fix this line to trigger when the user says definition of "Word"
            if syn:
                x = syn[0].definition()
                print(x)
                speak_pyttsx3(x,'en')
            elif "name" in user_query.lower():
                speak_pyttsx3("cyborg", speaking_language)
            elif "introduce" in user_query.lower() and "yourself" in user_query.lower():
                intro_text = "Hello, I'm your festival anchor robot. I'm here to assist you with information and entertainment. How can I help you today."
                speak_pyttsx3(intro_text, speaking_language)
            elif "built" in user_query.lower():
                speak_pyttsx3("Fayaz, JOE, Sidharth", speaking_language)
            elif "language" in user_query.lower():
                speak_pyttsx3("English", speaking_language)
            elif "weather" in user_query.lower():
                handle_weather_query(user_query)
            elif "tell" and "joke" in user_query.lower():
                speak_pyttsx3(tell_joke(), speaking_language)
            elif "shutdown" in user_query.lower():
                speak_pyttsx3("Goodbye!", speaking_language)
                break  # Exit the program

        elif speaking_language == 'ml':
            syn = wordnet.synsets(user_query)
            if syn:
                x = syn[0].definition()
                print(x)
                speak_gttts(x,'ml')
            elif "name" in user_query.lower():
                speak_gttts("cyborg", speaking_language)
            elif "introduce" in user_query.lower() and "yourself" in user_query.lower():
                intro_text = "Hello, I'm your festival anchor robot. I'm here to assist you with information and entertainment. How can I help you today?"
                speak_gttts(intro_text, speaking_language)
            elif "weather" in user_query.lower():
                handle_weather_query(user_query)
            elif "built" in user_query.lower():
                speak_gttts("Fayaz, JOE, Sidharth", speaking_language)
            elif "tell" and "joke" in user_query.lower():
                speak_gttts(tell_joke(), speaking_language)
            elif "language" in user_query.lower():
                speak_gttts("Malayalam", speaking_language)
            elif "shutdown" in user_query.lower():
                speak_gttts("Goodbye!", speaking_language)
                break  # Exit the program

if __name__ == "__main__":
    festival_anchor()
