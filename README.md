# Импортируем нужные модули
import sounddevice as sd              # Модуль для работы с устройствами звука
import scipy.io.wavfile as wav        # Библиотека для сохранения файла .wav
import speech_recognition as sr      # Библиотека для распознавания речи
from deep_translator import GoogleTranslator  # Библиотека для перевода текста
import random                        # Генерируем случайные слова

# Создаем словарь слов по уровням сложности
words_by_level = {
    "easy": ["cat", "dog", "apple", "milk", "sun"],       # Легкий уровень
    "medium": ["banana", "school", "friend", "window", "yellow"],   # Средний уровень
    "hard": ["technology", "university", "information", "pronunciation", "imagination"]  # Сложный уровень
}

# Создаем словарь поддерживаемых языков для перевода
supported_languages = {
    'ru': 'Русский'           # Код языка : Название языка
}

# Функция распознавания и перевода произнесённых слов
def recognize_and_translate(word, target_language):
    """
    Эта функция записывает звуковую дорожку,
    распознаёт произнесённые слова и переводит их на русский язык.
    """
    duration = 5                      # Время записи (секунды)
    sample_rate = 44444               # Частота дискретизации
    recording = sd.rec(int(duration * sample_rate), samplerate=sample_rate, channels=1, dtype="int16")
    sd.wait()                          # Ждём окончания записи
    wav.write("output.wav", sample_rate, recording)  # Сохраняем запись в файл

    recognizer = sr.Recognizer()     # Создаем объект распознавателя
    with sr.AudioFile("output.wav") as source:
        audio = recognizer.record(source)                  # Читаем аудиофайл
        try:
            # Распознаём речь на английском языке
            text = recognizer.recognize_google(audio, language="en-US").lower()
            # Переводим полученный текст на русский язык
            translated = GoogleTranslator(source='auto', target=target_language).translate(text)
            return text, translated
        except sr.UnknownValueError:
            print("Не удалось распознать речь.")
            return "", ""                     # Возвращаем пустой ответ, если ничего не распознали

# Начало игры
print("Добро пожаловать в игру \"Say the Word Correctly\" 🎤🎧")

# Предоставляем пользователю выбор языка для перевода
print("\n📌 Choose a translation language:")
for code, name in supported_languages.items():
    print(f"- {code}: {name}")

target_language = input("Enter language code (e.g., ru): ").strip().lower()

# Проверяем, поддерживает ли наша игра указанный язык
if target_language not in supported_languages:
    print(f"❌ Language '{target_language}' is not supported. Using Russian (ru).")
    target_language = 'ru'
else:
    print(f"✅ Selected language: {supported_languages[target_language]}")

# Предоставляем пользователю выбор уровня сложности
print("\n🖊️ Select difficulty level:")
print("- easy\n- medium\n- hard")
level = input("Specify level (e.g., easy): ").strip().lower()

# Проверяем, доступен ли указанный уровень сложности
if level not in words_by_level:
    print(f"❌ Level '{level}' not found. Using easy.")
    level = 'easy'
else:
    print(f"✅ Set level: {level}")

# Определяем начальные показатели игры
attempts_left = 3                   # Максимальное количество попыток
correct_answers = 0                # Количество правильных ответов
total_words = len(words_by_level[level])  # Общее количество слов на уровне

# Основной цикл игры
while attempts_left > 0:
    word = random.choice(words_by_level[level])  # Случайно выбираем слово
    print(f"\n🗣️ Say the word: {word}")
    recorded_text, translated_text = recognize_and_translate(word, target_language)

    # Показываем полученные результаты
    print(f"Expected answer: {word}")
    print(f"Received answer: {recorded_text}")
    print(f"Russian Translation: {translated_text}")

    # Проверяем, совпадает ли ответ с оригиналом
    if recorded_text == word:
        correct_answers += 1
        print("✅ Great job! Answer is correct.")
    else:
        attempts_left -= 1
        print(f"❌ Incorrect. Attempts left: {attempts_left}")

    # Завершаем игру, если закончились попытки
    if attempts_left <= 0:
        print("\n⛳️ GAME OVER")
        break

# Высчитываем итоговый балл игрока
final_score = round((correct_answers / total_words) * 100)
print(f"\n👑 Final score: {final_score}%")

# Награждение игроков
if final_score >= 80:
    print("🥇 Gold medal!\nCongratulations, you’ve earned a fluffy friend 🐾\nYour new pet is waiting for you at home!")
elif final_score >= 60:
    print("🥈 Silver medal!\nYou did well, here’s your bonus kitten 🐱\nNow it will cheer you up every day!")
elif final_score >= 40:
    print("🥉 Bronze medal!\nKeep practicing, and soon you'll have a new friend!")
else:
    print("😅 Need more practice!")
