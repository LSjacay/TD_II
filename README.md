# TD_II
# parte del reconocimiento facial, no reconocimiento, etc.
import cv2
import face_recognition
import numpy as np
import RPi.GPIO as GPIO
import time
from telegram import Bot, Update
from telegram.ext import CommandHandler, CallbackContext, Updater

# Configuración del LED
LED_PIN = 18  # Cambia esto al GPIO que estés usando
GPIO.setmode(GPIO.BCM)
GPIO.setup(LED_PIN, GPIO.OUT)

# Configuración del bot de Telegram
TELEGRAM_TOKEN = 'YOUR_TELEGRAM_BOT_TOKEN'
bot = Bot(token=TELEGRAM_TOKEN)

# Listas para rostros conocidos
known_face_encodings = []
known_face_names = []

# Función para cargar rostros conocidos
def load_known_faces():
    # Ejemplo: agregar un rostro conocido
    image = face_recognition.load_image_file("tu_imagen.jpg")
    encoding = face_recognition.face_encodings(image)[0]
    known_face_encodings.append(encoding)
    known_face_names.append("Nombre de la persona")

load_known_faces()

# Función para enviar mensaje por Telegram
def send_telegram_message(chat_id, message):
    bot.send_message(chat_id=chat_id, text=message)

# Función para reconocer rostros
def recognize_faces(chat_id):
    video_capture = cv2.VideoCapture(0)

    while True:
        ret, frame = video_capture.read()
        rgb_frame = frame[:, :, ::-1]
        
        face_locations = face_recognition.face_locations(rgb_frame)
        face_encodings = face_recognition.face_encodings(rgb_frame, face_locations)

        for face_encoding in face_encodings:
            matches = face_recognition.compare_faces(known_face_encodings, face_encoding)
            name = "Desconocido"

            if True in matches:
                first_match_index = matches.index(True)
                name = known_face_names[first_match_index]

            # Lógica para permitir o denegar
            if name == "Desconocido":
                send_telegram_message(chat_id, "Rostro no reconocido. Opciones: Abrir la puerta o No abrir la puerta.")
                # Aquí podrías esperar la respuesta del usuario
                user_choice = "No abrir la puerta"  # Simulación de respuesta
                if user_choice == "Abrir la puerta":
                    GPIO.output(LED_PIN, True)  # Enciende el LED
                    time.sleep(5)  # Mantén el LED encendido por 5 segundos
                    GPIO.output(LED_PIN, False)  # Apaga el LED
                else:
                    send_telegram_message(chat_id, "Puerta no abierta.")
            else:
                send_telegram_message(chat_id, "Rostro reconocido. Pasando...")

        # Muestra el video
        cv2.imshow('Video', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    video_capture.release()
    cv2.destroyAllWindows()
    GPIO.cleanup()

def start(update: Update, context: CallbackContext):
    update.message.reply_text("Iniciando reconocimiento facial...")
    recognize_faces(update.message.chat_id)

def main():
    updater = Updater(TELEGRAM_TOKEN)
    dp = updater.dispatcher

    dp.add_handler(CommandHandler("start", start))

    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
