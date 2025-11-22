# 🐳 Docker-стек: Python-приложение + PostgreSQL

Минимальный рабочий пример веб-приложения на Python с подключением к PostgreSQL, упакованный в Docker-контейнеры и развёрнутый через `docker compose` в WSL.

> **Ответ:** При обращении к `http://localhost:1234` возвращается:  
> ```
> Привет мир!
> ✅ Успешное подключение к PostgreSQL!
> ```



---

## Установка и настройка WSL

1. **Установка WSL2** (в PowerShell от администратора):
   ```powershell
   wsl --install
   ```
2. **Перезапуск пк**
3. **Обновление системы внутри WSL**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
4. **Установка Docker и docker-compose**
   4.1 Установка зависимостей
   ```bash
   sudo apt install -y ca-certificates curl gnupg lsb-release
   ```
   4.2 GPG-ключ
   ```bash
   sudo mkdir -p /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   ```
   4.3  Добавление репозитория
   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```
   4.4 Установка docker
   ```bash
   sudo apt update
   sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```
   4.5 Проверка
   ```bash
   docker --version
   docker compose version
   ```
   <img width="488" height="74" alt="image" src="https://github.com/user-attachments/assets/75e6e31d-6e70-4e86-a6a8-a6f9d0d83d4c" />
   <img width="372" height="43" alt="image" src="https://github.com/user-attachments/assets/2f5a6712-4324-4703-a920-1f45ca93e23d" />
   4.6 Добавляем себя в группу докер, чтобы не прописывать sudo
   ```bash
   sudo usermod -aG docker $USER
   ```
5 **Создаем приложение и Dockerfile**
   5.1 создаем папку проекта
   ```bash
   mkdir ~/myapp && cd ~/myapp
   ```
   <img width="311" height="83" alt="image" src="https://github.com/user-attachments/assets/2e2d25cb-d280-4330-bf75-86447fdf263f" />
   
   5.2 Пишем программу на python
      5.2.1 Создаем файл
      ```bash
      cat > app.py << 'EOF'
      from http.server import HTTPServer, BaseHTTPRequestHandler
      class Handler(BaseHTTPRequestHandler):
          def do_GET(self):
              self.send_response(200)
              self.send_header("Content-type", "text/plain; charset=utf-8")
              self.end_headers()
              self.wfile.write("Привет мир!".encode('utf-8'))
      if __name__ == "__main__":
          server = HTTPServer(('', 1234), Handler)
          print("Сервер запущен на порту 1234...")
          server.serve_forever()
      EOF
      ```
   5.3 Добавляем докер файл
   
