# Remote Access

- [Remote Access](#remote-access)
  - [Настройка удаленного подключения через ssh без паролей](#настройка-удаленного-подключения-через-ssh-без-паролей)

## Настройка удаленного подключения через ssh без паролей

1. Генерация ssh ключа:

    ```bash
      ssh-keygen -t ed25519 -C "your_email@example.com"
        # -t ed25519 — тип ключа (Более безопасный и быстрый чем RSA)
        # -C — комментарий (часто используется email для идентификации)
    ```

2. Копирование ключа на сервер:

    ```bash
      ssh-copy-id -i ~/.ssh/id_rsa.pub user@your.server.com
    ```

3. Подключение к серверу без пароля:

    ```bash
      ssh user@your.server.com
    ```
