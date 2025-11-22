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

   5.2 Написание простого Python-приложения

Создайте файл `app.py`:
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
```bash
cat > Dockerfile << 'EOF'
FROM python:3.11-slim

WORKDIR /app

COPY app.py .

EXPOSE 1234

CMD ["python", "app.py"]
EOF
```
   5.4 Убедись что файлы в системе:
```bash
ls -l
```
   <img width="637" height="96" alt="image" src="https://github.com/user-attachments/assets/820624a7-3e59-4210-a454-d68696f5df6c" />
6. **Cобери образ**
```bash
docker build -t myapp .
```
7. **Запусти контейнер**
```bash
docker run -d -p 1234:1234 --name myapp_container myapp
```
   7.1 Проверь в браузере Windows:
       http://localhost:1234/
       <img width="641" height="88" alt="image" src="https://github.com/user-attachments/assets/39831f07-699f-42c1-aeaf-1db431e79a25" />
8. **Создаем docker-comprose.yml**
```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  app:
    build: .
    ports:
      - "1234:1234"
    depends_on:
      - db
    environment:
      - DB_HOST=db
      - DB_USER=myuser
      - DB_PASSWORD=mypass
      - DB_NAME=mydb

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypass
      POSTGRES_DB: mydb
    volumes:
      - pg/var/lib/postgresql/data

volumes:
  pg
EOF
```
9. **Обновим код app.py для проверки подключения к PostgreSQL**
```bash
import os
import psycopg2
from http.server import HTTPServer, BaseHTTPRequestHandler

def check_db():
    try:
        conn = psycopg2.connect(
            host=os.getenv('DB_HOST', 'db'),
            user=os.getenv('DB_USER', 'myuser'),
            password=os.getenv('DB_PASSWORD', 'mypass'),
            dbname=os.getenv('DB_NAME', 'mydb')
        )
        conn.close()
        return "✅ БД доступна"
    except Exception as e:
        return f"❌ Ошибка БД: {e}"

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header("Content-type", "text/plain; charset=utf-8")
        self.end_headers()
        msg = "Привет мир!\n" + check_db()
        self.wfile.write(msg.encode('utf-8'))

if __name__ == "__main__":
    HTTPServer(('', 1234), Handler).serve_forever()
```
10. **Обновим DockerFile**
```bash
FROM python:3.11-slim

RUN apt-get update && apt-get install -y gcc libpq-dev && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY app.py .

RUN pip install psycopg2-binary

CMD ["python3", "app.py"]
```
11. **Собираем**
```bash
docker compose up --build -d
```
12.**Если все работает, то получаем по адресу** *http://localhost:1234/*
Привет мир!
✅ Успешное подключение к PostgreSQL!
<img width="714" height="210" alt="image" src="https://github.com/user-attachments/assets/50347712-6f05-4d99-88bd-1386b4c3001e" />

   
   
