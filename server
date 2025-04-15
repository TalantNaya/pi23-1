import socket  # Модуль для работы с сокетами
import threading  # Модуль для работы с потоками
import sys  # Модуль для доступа к некоторым функциям и переменным интерпретатора Python
import signal  # Модуль для обработки сигналов (например, Ctrl+C)
import logging  # Модуль для логирования
import os  # Модуль для работы с файловой системой

# Файл для хранения идентификации
IDENTIFICATION_FILE = 'identification.txt'

# Настройка логирования
logging.basicConfig(filename='server.log', level=logging.INFO,
                    format='%(asctime)s - %(levelname)s - %(message)s')

# Глобальные флаги
server_running = True  # Флаг состояния сервера (работает или завершен)
server_paused = False  # Флаг состояния прослушивания (активно или на паузе)

# Список для хранения активных клиентских потоков
client_threads = []

def client_handler(conn, addr):
    """
    Функция для обработки взаимодействия с клиентом в отдельном потоке.
    """
    logging.info(f"Подключен клиент {addr}")
    # Записываем информацию об идентификации клиента в файл
    with open(IDENTIFICATION_FILE, 'a') as f:
        f.write(f"Клиент {addr} подключился.\n")

    try:
        while True:
            # Бесконечный цикл для приема данных от клиента
            data = conn.recv(1024)
            # Получаем данные размером до 1024 байт
            if not data:
                # Если данных нет, значит клиент отключился
                break
            msg = data.decode()
            # Декодируем байтовые данные в строку
            logging.info(f"Сообщение от {addr}: {msg}")
            conn.send(data)
            # Отправляем данные обратно клиенту (эхо)
    except ConnectionResetError:
        # Обработка ситуации, когда клиент неожиданно отключился
        logging.warning(f"Соединение с клиентом {addr} было разорвано")
    finally:
        logging.info(f"Клиент {addr} отключился")
        conn.close()
        # Закрываем соединение с данным клиентом

def server_listener(sock):
    """
    Функция для прослушивания входящих подключений.
    Выполняется в отдельном потоке.
    """
    global server_running, server_paused

    while server_running:
        if server_paused:
            # Если сервер на паузе, ждем перед проверкой снова
            threading.Event().wait(1)
            continue

        try:
            conn, addr = sock.accept()
            # Принимаем новое входящее подключение
            client_thread = threading.Thread(target=client_handler, args=(conn, addr))
            # Создаем новый поток для обслуживания клиента
            client_thread.start()
            # Запускаем поток
            client_threads.append(client_thread)
            # Добавляем поток в список активных потоков
        except socket.timeout:
            # Если время ожидания соединения истекло, проверяем состояние сервера
            continue
        except OSError:
            # Если сокет был закрыт, выходим из цикла
            break

def main():
    global server_running, server_paused
    sock = socket.socket()
    # Создаем TCP-сокет
    # Устанавливаем опцию SO_REUSEADDR, чтобы переиспользовать адрес и порт
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    sock.bind(('', 9090))
    # Связываем сокет с адресом и портом.
    sock.listen()
    # Переводим сокет в режим прослушивания входящих подключений
    sock.settimeout(1)  # Устанавливаем таймаут для accept(), чтобы проверять флаги

    print("Сервер запущен и ожидает подключений...")
    logging.info("Сервер запущен и ожидает подключений...")

    # Запускаем поток для прослушивания входящих соединений
    listener_thread = threading.Thread(target=server_listener, args=(sock,))
    listener_thread.start()

    # Основной поток программы для принятия команд от пользователя
    try:
        while True:
            command = input("Введите команду (shutdown, pause, resume, show logs, clear logs, clear id): ").strip().lower()

            if command == 'shutdown':
                # Завершение работы сервера
                print("Завершение работы сервера...")
                logging.info("Сервер завершает работу по команде shutdown.")
                server_running = False
                server_paused = False  # На случай, если сервер был на паузе
                sock.close()  # Закрываем сокет, чтобы выйти из accept()
                break
            elif command == 'pause':
                if not server_paused:
                    server_paused = True
                    print("Сервер поставлен на паузу. Новые подключения не принимаются.")
                    logging.info("Сервер поставлен на паузу по команде pause.")
                else:
                    print("Сервер уже находится на паузе.")
            elif command == 'resume':
                if server_paused:
                    server_paused = False
                    print("Сервер возобновил прием подключений.")
                    logging.info("Сервер возобновил работу по команде resume.")
                else:
                    print("Сервер и так работает.")
            elif command == 'show logs':
                # Показываем содержимое файла логов
                if os.path.exists('server.log'):
                    with open('server.log', 'r') as log_file:
                        print("\n=== Содержимое логов ===")
                        print(log_file.read())
                        print("=== Конец логов ===\n")
                else:
                    print("Лог-файл отсутствует.")
            elif command == 'clear logs':
                # Очищаем файл логов
                if os.path.exists('server.log'):
                    open('server.log', 'w').close()
                    print("Логи очищены.")
                    logging.info("Логи были очищены по команде clear logs.")
                else:
                    print("Лог-файл отсутствует.")
            elif command == 'clear id':
                # Очищаем файл идентификации
                if os.path.exists(IDENTIFICATION_FILE):
                    open(IDENTIFICATION_FILE, 'w').close()
                    print("Файл идентификации очищен.")
                    logging.info("Файл идентификации был очищен по команде clear id.")
                else:
                    print("Файл идентификации отсутствует.")
            else:
                print("Неизвестная команда. Доступные команды: shutdown, pause, resume, show logs, clear logs, clear id.")

    except KeyboardInterrupt:
        # Обработка сигнала Ctrl+C
        print("\nЗавершение работы сервера...")
        logging.info("Сервер завершает работу по сигналу Ctrl+C.")
        server_running = False
        server_paused = False
        sock.close()

    # Ожидаем завершения потока прослушивания
    listener_thread.join()

    # Ожидаем завершения всех клиентских потоков
    for t in client_threads:
        t.join()

    print("Сервер остановлен.")
    logging.info("Сервер остановлен.")

if __name__ == "__main__":
    main()
