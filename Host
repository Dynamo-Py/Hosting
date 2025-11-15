import os, subprocess, telebot, threading, time, signal, sys

BOT_TOKEN = "8033436420:AAGmDzGAvHZSqHz0jyHEbY37uLq9iPCcA4w"
ADMIN_ID = 1725301348

bot = telebot.TeleBot(BOT_TOKEN)
BOTS_DIR = "hosted_bots"
LOGS_DIR = "bot_logs"
os.makedirs(BOTS_DIR, exist_ok=True)
os.makedirs(LOGS_DIR, exist_ok=True)
running_bots = {}

def start_bot(filepath):
    name = os.path.basename(filepath)
    out_log = open(os.path.join(LOGS_DIR, f"{name}.log"), "ab")
    p = subprocess.Popen(["python3", filepath], stdout=out_log, stderr=subprocess.STDOUT)
    running_bots[name] = (p, out_log)

def stop_all_bots():
    for name, (proc, log) in list(running_bots.items()):
        try:
            proc.terminate()
            proc.wait(timeout=3)
        except:
            proc.kill()
        log.close()
        del running_bots[name]

def restore_bots():
    for f in os.listdir(BOTS_DIR):
        if f.endswith(".py"):
            start_bot(os.path.join(BOTS_DIR, f))

def monitor_bots():
    while True:
        for name, (proc, log) in list(running_bots.items()):
            if proc.poll() is not None:
                log.close()
                start_bot(os.path.join(BOTS_DIR, name))
        time.sleep(5)

@bot.message_handler(commands=['start'])
def start_cmd(m):
    if m.from_user.id != ADMIN_ID:
        bot.reply_to(m, "Access Denied.")
        return
    bot.reply_to(m, "Send your .py Telegram bot files to host them here.")

@bot.message_handler(content_types=['document'])
def upload_bot(m):
    if m.from_user.id != ADMIN_ID:
        bot.reply_to(m, "Access Denied.")
        return
    try:
        info = bot.get_file(m.document.file_id)
        data = bot.download_file(info.file_path)
        name = m.document.file_name
        if not name.endswith(".py"):
            bot.reply_to(m, "Only .py files allowed.")
            return
        path = os.path.join(BOTS_DIR, name)
        with open(path, "wb") as f:
            f.write(data)
        bot.reply_to(m, f"{name} uploaded. Starting...")
        threading.Thread(target=start_bot, args=(path,), daemon=True).start()
    except Exception as e:
        bot.reply_to(m, f"Error: {e}")

@bot.message_handler(commands=['list'])
def list_bots(m):
    if m.from_user.id != ADMIN_ID:
        bot.reply_to(m, "Access Denied.")
        return
    if not running_bots:
        bot.reply_to(m, "No bots running.")
        return
    txt = "Running bots:\n" + "\n".join(running_bots.keys())
    bot.reply_to(m, txt)

@bot.message_handler(commands=['stop'])
def stop_cmd(m):
    if m.from_user.id != ADMIN_ID:
        bot.reply_to(m, "Access Denied.")
        return
    stop_all_bots()
    bot.reply_to(m, "All bots stopped.")

threading.Thread(target=monitor_bots, daemon=True).start()
restore_bots()

def handle_exit(sig, frame):
    stop_all_bots()
    sys.exit(0)

signal.signal(signal.SIGINT, handle_exit)
signal.signal(signal.SIGTERM, handle_exit)

print("Host bot started.")
bot.infinity_polling()
