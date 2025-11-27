# Docker-стек: Python-приложение + PostgreSQL


---

## Подготовка
переходим по ссылке https://www.docker.com/products/docker-desktop/
и скачиваем Docker Deskstop

Заходим в Microsoft Strore и устанавливаем Ubuntu

---


## Регистрация

После перезагрузки нужно зайти в докер декстоп и пройти регистрацию

---

## Установка и настройка WSL

1. **Установка WSL2** (в PowerShell от имени администратора):
   ```powershell
   wsl --install
   ```
   
    или

   ```powershell
   wsl --update
   ```

<img width="413" height="94" alt="image" src="https://github.com/user-attachments/assets/877087e8-61fa-4f14-9042-9fb7c1dbe335" />

   
   
   
---
2. **Перезапуск пк**
---
3. **Обновление системы внутри WSL**

Запускаем Ubuntu например через поиск. В нем пишем команду 

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
и ждем окончание обновления
---
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
---

5 **Создаем приложение и Dockerfile**
   5.1 создаем папку проекта
 ```bash
 mkdir ~/myapp && cd ~/myapp
 ```
   <img width="311" height="83" alt="image" src="https://github.com/user-attachments/assets/2e2d25cb-d280-4330-bf75-86447fdf263f" />

   5.2 Написание простого Python-приложения и добавление файла requirements.txt

Создайте файл `app.py`:
```bash
cat > app.py << 'EOF'
from flask import Flask
import psycopg2
import os

app = Flask(__name__)

def get_db_connection():
    conn = psycopg2.connect(
        host=os.getenv('POSTGRES_HOST', 'db'),
        database=os.getenv('POSTGRES_DB', 'testdb'),
        user=os.getenv('POSTGRES_USER', 'testuser'),
        password=os.getenv('POSTGRES_PASSWORD', 'testpass')
    )
    return conn

@app.route('/')
def hello():
    try:
        conn = get_db_connection()
        cur = conn.cursor()
        cur.execute('SELECT 1')
        cur.close()
        conn.close()
        return "Привет Докер и postgres!"
    except Exception as e:
        return f"Извините, но подключения к базе не случилось :( Ошибка: {str(e)}"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=1234)
EOF
```

---

```bash
nano requirements.txt
```

Записываем 

```bash
Flask
psycopg2-binary
```
Жмем ctrl + O, enter, ctrl + x



   5.3 Добавляем докер файл
```bash
cat > Dockerfile << 'EOF'
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 1234

CMD ["python", "app.py"]
EOF
```

   5.6 Добавляем docker-compose.yml

   ```bash
version: '3.8'

services:
  web:
    build: .
    ports:
      - "1234:1234"
    environment:
      - POSTGRES_HOST=db
      - POSTGRES_DB=testdb
      - POSTGRES_USER=testuser
      - POSTGRES_PASSWORD=testpass
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      - POSTGRES_DB=testdb
      - POSTGRES_USER=testuser
      - POSTGRES_PASSWORD=testpass
    # ports:
    #   - "5432:5432"   # ← закомментировано, чтобы не конфликтовать с локальным PostgreSQL
```

   5.5 Убедись что файлы в системе:
```bash
ls -l
```
   <img width="637" height="96" alt="image" src="https://github.com/user-attachments/assets/820624a7-3e59-4210-a454-d68696f5df6c" />
   
---

6. **Cобери образ**
```bash
docker build -t myapp .
```
---

7. **Запусти контейнер**
```bash
docker run -d -p 1234:1234 --name myapp_container myapp
```
   7.1 Проверь в браузере Windows:
       http://localhost:1234/
       <img width="641" height="88" alt="image" src="https://github.com/user-attachments/assets/39831f07-699f-42c1-aeaf-1db431e79a25" />

---


   
   
