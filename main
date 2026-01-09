import vk_api
from vk_api.bot_longpoll import VkBotLongPoll, VkBotEventType
import requests
import random
import os
import yt_dlp
import time
import subprocess
import re
import glob

# ========== НАСТРОЙКИ ==========
VK_TOKEN = ''
GROUP_ID = ''
YOUTUBE_API_KEY = ''
MY_USER_ID = 
# ===============================

print("🚀 YouTube Downloader Bot v5.0")
print(f"👤 Работает только для: {MY_USER_ID}")


# ========== НОВЫЕ ФУНКЦИИ ОЧИСТКИ ==========
def cleanup_old_files(max_age_hours=24):
    """Удаляет старые временные файлы при запуске бота"""
    print("🧹 Проверяю старые файлы...")
    current_time = time.time()
    deleted_count = 0

    for filepath in glob.glob("video_*.mp4"):
        try:
            file_age = current_time - os.path.getmtime(filepath)
            if file_age > max_age_hours * 3600:
                os.remove(filepath)
                deleted_count += 1
        except:
            pass

    if deleted_count > 0:
        print(f"✅ Удалено старых файлов: {deleted_count}")


def safe_cleanup(file_path, description=""):
    """Безопасное удаление файла с защитой от ошибок"""
    if not file_path or not os.path.exists(file_path):
        return

    try:
        # Даем файлу время на освобождение (если еще используется)
        for _ in range(3):  # 3 попытки
            try:
                os.remove(file_path)
                print(f"✅ Удален {description}: {os.path.basename(file_path)}")
                return
            except PermissionError:
                time.sleep(1)  # Ждем секунду и пробуем снова
                continue

        # Если не удалось удалить после 3 попыток
        print(f"⚠️ Не удалось удалить {file_path} (файл занят)")

        # Записываем в лог для ручной очистки
        with open("locked_files.txt", "a") as f:
            f.write(f"{time.ctime()}: {file_path}\n")

    except Exception as e:
        print(f"⚠️ Ошибка при удалении {file_path}: {e}")


# Запускаем очистку старых файлов при старте
cleanup_old_files()

# ========== ИНИЦИАЛИЗАЦИЯ ==========
vk_session = vk_api.VkApi(token=VK_TOKEN)
longpoll = VkBotLongPoll(vk_session, GROUP_ID)
vk = vk_session.get_api()
user_states = {}


# ========== ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ ==========
def iso8601_to_seconds(duration_iso):
    """Конвертирует PT1H2M3S в секунды"""
    pattern = re.compile(r'P(?:(\d+)D)?T(?:(\d+)H)?(?:(\d+)M)?(?:(\d+)S)?')
    match = pattern.match(duration_iso)
    if not match:
        return 0
    days = int(match.group(1) or 0)
    hours = int(match.group(2) or 0)
    minutes = int(match.group(3) or 0)
    seconds = int(match.group(4) or 0)
    return days * 86400 + hours * 3600 + minutes * 60 + seconds


def format_duration(seconds):
    """Форматирует секунды в ММ:СС или ЧЧ:ММ:СС"""
    if seconds == 0:
        return "?"
    hours = seconds // 3600
    minutes = (seconds % 3600) // 60
    seconds = seconds % 60
    if hours > 0:
        return f"{hours}:{minutes:02d}:{seconds:02d}"
    return f"{minutes}:{seconds:02d}"


# ========== YOUTUBE ПОИСК (РАБОЧИЙ) ==========
def get_video_details(video_id):
    """Получает информацию о видео"""
    url = "https://www.googleapis.com/youtube/v3/videos"
    params = {
        "part": "statistics,contentDetails,snippet",
        "id": video_id,
        "key": YOUTUBE_API_KEY
    }

    try:
        response = requests.get(url, params=params, timeout=10)
        if response.status_code == 200 and response.json().get('items'):
            data = response.json()['items'][0]

            # Просмотры
            views = int(data['statistics'].get('viewCount', 0))
            views_formatted = f"{views:,}".replace(",", ".")

            # Длительность
            duration_iso = data['contentDetails']['duration']
            duration_seconds = iso8601_to_seconds(duration_iso)
            duration_formatted = format_duration(duration_seconds)

            return {
                'views': views_formatted,
                'duration': duration_formatted,
                'duration_seconds': duration_seconds,
                'title': data['snippet']['title'],
                'channel': data['snippet']['channelTitle']
            }
    except:
        pass

    return {'views': 'N/A', 'duration': 'N/A', 'duration_seconds': 0, 'title': '', 'channel': ''}


def youtube_search(query, max_results=5):
    """Поиск видео (исправленная версия)"""
    print(f"🔍 Поиск: '{query}'")

    # Шаг 1: Поиск через YouTube API
    url = "https://www.googleapis.com/youtube/v3/search"
    params = {
        "part": "snippet",
        "q": query,
        "key": YOUTUBE_API_KEY,
        "maxResults": 10,  # Ищем больше, потом отфильтруем
        "type": "video",
        "relevanceLanguage": "ru",
        "order": "relevance"
    }

    try:
        response = requests.get(url, params=params, timeout=10)
        if response.status_code != 200:
            print(f"❌ API ошибка: {response.status_code}")
            return None

        data = response.json()
        if 'items' not in data or not data['items']:
            print("❌ Нет результатов")
            return None

        videos = []
        for item in data['items']:
            try:
                video_id = item['id']['videoId']

                # Шаг 2: Получаем детали
                details = get_video_details(video_id)
                if not details:
                    continue

                # Фильтруем шортсы (меньше 60 секунд)
                if details['duration_seconds'] < 60:
                    continue

                # Фильтруем слишком длинные (>4 часов)
                if details['duration_seconds'] > 14400:
                    continue

                videos.append({
                    'title': details['title'] or item['snippet']['title'],
                    'video_id': video_id,
                    'url': f"https://youtu.be/{video_id}",
                    'channel': details['channel'] or item['snippet']['channelTitle'],
                    'views': details['views'],
                    'duration': details['duration'],
                    'duration_seconds': details['duration_seconds'],
                    'thumbnail': item['snippet']['thumbnails']['high']['url']
                })

                if len(videos) >= max_results:
                    break

            except Exception as e:
                print(f"⚠️ Ошибка обработки видео: {e}")
                continue

        print(f"✅ Найдено видео: {len(videos)}")
        return videos if videos else None

    except Exception as e:
        print(f"❌ Ошибка поиска: {e}")
        return None


# ========== СКАЧИВАНИЕ ВИДЕО (ИСПРАВЛЕННОЕ) ==========
def download_youtube_video(video_url, user_id):
    """Скачивает видео (работает всегда)"""
    print(f"\n📥 Скачиваю: {video_url}")

    timestamp = int(time.time())
    filename = f"video_{user_id}_{timestamp}.mp4"

    # УНИВЕРСАЛЬНЫЙ формат, который ВСЕГДА работает
    # Порядок приоритетов: 480p → 360p → любой формат → худший
    universal_format = 'best[height<=480]/best[height<=360]/best/worst'

    ydl_opts = {
        'format': universal_format,
        'outtmpl': filename,
        'quiet': False,  # Пусть показывает процесс
        'no_warnings': False,
        'ignoreerrors': True,  # Игнорируем ошибки форматов
        'merge_output_format': 'mp4',
        'extractor_args': {
            'youtube': {
                'player_client': ['android', 'web'],
                'skip': ['hls', 'dash'],
            }
        },
    }

    try:
        print("🎬 Скачиваю...")
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            info = ydl.extract_info(video_url, download=True)
            title = info.get('title', 'YouTube видео')

        if os.path.exists(filename):
            size_mb = os.path.getsize(filename) / (1024 * 1024)
            print(f"✅ Скачано: {size_mb:.1f} MB")
            return filename, title
        else:
            print("❌ Файл не создан")
            return None, None

    except Exception as e:
        print(f"❌ Ошибка скачивания: {e}")
        return None, None


# ========== ЗАГРУЗКА В ВК ==========
def upload_to_vk(video_path, title=""):
    """Загружает видео как документ в ВК"""
    print(f"📤 Загружаю в ВК...")

    try:
        # Получаем URL для загрузки
        upload_data = vk.docs.getMessagesUploadServer(type='doc', peer_id=MY_USER_ID)
        upload_url = upload_data['upload_url']

        # Проверяем размер файла
        file_size = os.path.getsize(video_path) / (1024 * 1024)
        if file_size > 200:
            print(f"⚠️ Файл слишком большой: {file_size:.1f}MB")
            return None

        print(f"📦 Размер файла: {file_size:.1f}MB")

        # Загружаем файл
        with open(video_path, 'rb') as f:
            files = {'file': f}
            response = requests.post(upload_url, files=files, timeout=300)

        if response.status_code == 200:
            upload_result = response.json()

            # Сохраняем документ
            save_result = vk.docs.save(
                file=upload_result['file'],
                title=title[:90] if title else 'YouTube Video',
                tags='youtube'
            )

            # Извлекаем ID документа
            if 'doc' in save_result:
                doc = save_result['doc']
            elif 'video' in save_result:
                doc = save_result['video']
            else:
                doc = list(save_result.values())[0]

            doc_id = f"doc{doc['owner_id']}_{doc['id']}"
            print(f"✅ Загружено: {doc_id}")
            return doc_id
        else:
            print(f"❌ Ошибка загрузки: {response.status_code}")
            return None

    except Exception as e:
        print(f"❌ Ошибка: {e}")
        return None


# ========== ОТПРАВКА СООБЩЕНИЙ ==========
def send_message(user_id, text):
    """Отправляет текстовое сообщение"""
    try:
        vk.messages.send(
            user_id=user_id,
            message=text,
            random_id=random.randint(0, 1000000)
        )
        return True
    except Exception as e:
        print(f"❌ Ошибка отправки: {e}")
        return False


def send_video(user_id, video_id, title=""):
    """Отправляет видео"""
    try:
        vk.messages.send(
            user_id=user_id,
            message=title[:100] if title else "🎬 Видео готово!",
            attachment=video_id,
            random_id=random.randint(0, 1000000)
        )
        return True
    except:
        # Если не вышло прикрепить, отправляем ID
        send_message(user_id, f"Видео ID: {video_id}")
        return False


# ========== ОСНОВНОЙ ЦИКЛ ==========
print("\n" + "=" * 50)
print("🤖 Бот запущен и готов к работе!")
print("Команды:")
print("• поиск [запрос] - найти видео")
print("• ссылка YouTube - скачать сразу")
print("• помощь - инструкции")
print("=" * 50)

for event in longpoll.listen():
    if event.type == VkBotEventType.MESSAGE_NEW:
        msg = event.object.message
        user_id = msg['from_id']
        text = msg.get('text', '').strip()

        # Только для тебя!
        if user_id != MY_USER_ID:
            continue

        print(f"\n👤 {user_id}: {text[:50]}...")
        state = user_states.get(user_id, {})

        # ===== КОМАНДА ПОИСКА =====
        if text.lower().startswith('поиск '):
            query = text[6:].strip()

            if not query:
                send_message(user_id, "❌ Напишите что искать")
                continue

            send_message(user_id, f"🔍 Ищу '{query}'...")

            # Ищем видео
            videos = youtube_search(query)

            if videos:
                # Сохраняем результаты
                user_states[user_id] = {
                    "state": "choosing",
                    "data": videos,
                    "query": query
                }

                # Формируем список
                response = f"📺 Найдено '{query}':\n\n"
                for i, vid in enumerate(videos, 1):
                    response += f"{i}. {vid['title'][:50]}...\n"
                    response += f"   👁 {vid['views']} | ⏱ {vid['duration']}\n\n"

                response += f"📝 Ответьте цифрой (1-{len(videos)}):"

                send_message(user_id, response)
            else:
                send_message(user_id, "❌ Ничего не найдено. Попробуйте другой запрос.")

        # ===== ВЫБОР ВИДЕО ИЗ РЕЗУЛЬТАТОВ =====
        elif state.get('state') == 'choosing' and text.isdigit():
            videos = state['data']
            choice = int(text)

            if 1 <= choice <= len(videos):
                selected = videos[choice - 1]

                # Очищаем состояние
                user_states[user_id] = {}

                send_message(user_id, f"⏳ Скачиваю: {selected['title'][:50]}...\nПодождите 1-3 минуты...")

                # Скачиваем видео
                video_file = None
                video_id = None

                try:
                    video_file, video_title = download_youtube_video(selected['url'], user_id)

                    if video_file and os.path.exists(video_file):
                        send_message(user_id, "📤 Загружаю в ВК...")

                        # Загружаем в ВК
                        video_id = upload_to_vk(video_file, video_title)

                        if video_id:
                            # Отправляем видео
                            if send_video(user_id, video_id, video_title):
                                print("✅ Видео отправлено успешно!")
                            else:
                                send_message(user_id, f"✅ Видео загружено! ID: {video_id}")
                        else:
                            send_message(user_id, "❌ Не удалось загрузить в ВК")

                    else:
                        send_message(user_id, "❌ Не удалось скачать видео")

                finally:
                    # ГАРАНТИРОВАННАЯ очистка в ЛЮБОМ случае
                    if video_file:
                        safe_cleanup(video_file, "временный файл после поиска")
            else:
                send_message(user_id, f"❌ Выберите номер от 1 до {len(videos)}")

        # ===== ПРЯМАЯ ССЫЛКА YOUTUBE =====
        elif ('youtube.com/watch' in text or 'youtu.be/' in text) and ' ' not in text:
            # Очищаем ссылку от параметров
            if '?si=' in text:
                clean_url = text.split('?si=')[0]
            else:
                clean_url = text

            send_message(user_id, f"⏳ Скачиваю по ссылке...\nПодождите 1-3 минуты...")

            # Скачиваем
            video_file = None
            video_id = None

            try:
                video_file, video_title = download_youtube_video(clean_url, user_id)

                if video_file and os.path.exists(video_file):
                    send_message(user_id, "📤 Загружаю в ВК...")

                    # Загружаем
                    video_id = upload_to_vk(video_file, video_title)

                    if video_id:
                        if send_video(user_id, video_id, video_title):
                            print("✅ Отправлено!")
                        else:
                            send_message(user_id, f"✅ Готово! ID: {video_id}")
                    else:
                        send_message(user_id, "❌ Не удалось загрузить в ВК")

                else:
                    send_message(user_id, "❌ Не удалось скачать видео")

            finally:
                # ГАРАНТИРОВАННАЯ очистка в ЛЮБОМ случае
                if video_file:
                    safe_cleanup(video_file, "временный файл по ссылке")

        # ===== ОТМЕНА =====
        elif text.lower() in ['отмена', 'cancel', 'стоп']:
            if user_id in user_states:
                user_states[user_id] = {}
                send_message(user_id, "❌ Поиск отменен")

        # ===== ПОМОЩЬ =====
        elif text.lower() in ['помощь', 'help', 'start', '/start']:
            help_text = """🎬 YouTube Downloader Bot

📌 КАК ИСПОЛЬЗОВАТЬ:
1. Поиск видео: "поиск [запрос]"
   Пример: поиск смешные кошки

2. Выбор: ответьте цифрой (1, 2, 3...)

3. Прямая ссылка: отправьте ссылку на YouTube видео

4. Отмена: "отмена"

⚙️ ОСОБЕННОСТИ:
• Автоматическое определение качества
• Фильтрация коротких видео
• Быстрая загрузка в ВК

⏱ Время обработки: 1-3 минуты
👤 Работает только для вас!"""

            send_message(user_id, help_text)

        # ===== НЕПОНЯТНАЯ КОМАНДА =====
        else:
            if state.get('state') == 'choosing':
                send_message(user_id, f"❌ Ответьте цифрой 1-{len(state['data'])} или напишите 'отмена'")
            else:
                send_message(user_id,
                             "📌 Напишите 'поиск [запрос]' или отправьте ссылку на YouTube видео\nИли 'помощь' для инструкций")
