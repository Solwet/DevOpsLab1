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
