import pyttsx3
import speech_recognition as sr
import datetime
import json
import random

# Simulated devices
devices = {
    "lights": False,
    "fan": False,
    "ac": False,
    "tv": False,
    "heater": False
}

# Preferences (AI component based on time)
preferences = {
    "morning": ["lights", "heater"],
    "afternoon": ["ac", "fan"],
    "evening": ["lights", "tv"],
    "night": ["lights"]
}

# Initialize voice engine
engine = pyttsx3.init()
engine.setProperty('rate', 150)

def speak(text):
    engine.say(text)
    engine.runAndWait()

def listen():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        audio = r.listen(source, phrase_time_limit=5)
    try:
        command = r.recognize_google(audio)
        print("User said:", command)
        return command.lower()
    except sr.UnknownValueError:
        speak("Sorry, I did not catch that.")
        return ""
    except sr.RequestError:
        speak("Sorry, speech service is not available.")
        return ""

def get_time_period():
    hour = datetime.datetime.now().hour
    if 5 <= hour < 12:
        return "morning"
    elif 12 <= hour < 17:
        return "afternoon"
    elif 17 <= hour < 21:
        return "evening"
    else:
        return "night"

def perform_action(command):
    for device in devices.keys():
        if f"turn on {device}" in command or f"switch on {device}" in command:
            devices[device] = True
            speak(f"{device.capitalize()} turned on.")
            return
        elif f"turn off {device}" in command or f"switch off {device}" in command:
            devices[device] = False
            speak(f"{device.capitalize()} turned off.")
            return
    if "status" in command:
        status_report = ". ".join([f"{k} is {'on' if v else 'off'}" for k, v in devices.items()])
        speak(f"Current device status: {status_report}")
        return
    if "automate" in command or "suggest" in command:
        automate_devices()
        return
    speak("Command not recognized.")

def automate_devices():
    time_period = get_time_period()
    suggested_devices = preferences.get(time_period, [])
    for device in suggested_devices:
        devices[device] = True
    speak(f"Based on the {time_period}, I have turned on: {', '.join(suggested_devices)}.")

def main():
    speak("Welcome to Smart Home Automation System.")
    while True:
        speak("How can I assist you?")
        command = listen()
        if "exit" in command or "stop" in command or "bye" in command:
            speak("Goodbye! Turning off the system.")
            break
        perform_action(command)

if __name__ == "__main__":
    main()
