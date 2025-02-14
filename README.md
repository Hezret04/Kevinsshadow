# Kevinsshadow
Shadow 
#!/bin/bash

# Установка учетных данных администратора
ADMIN_USER="Kevin"
ADMIN_PASSWORD="12345678"

# Функция для проверки пароля
check_password() {
    echo "Введите пароль администратора:"
    read -s input_password
    if [[ "$input_password" != "$ADMIN_PASSWORD" ]]; then
        echo "Неверный пароль. Доступ запрещен."
        exit 1
    fi
}

# Функция для создания ключа
create_key() {
    echo "Введите порт для нового ключа:"
    read port
    echo "Введите пароль для нового ключа:"
    read password
    echo "Добавление ключа..."
    jq --arg port "$port" --arg password "$password" '.port_password[$port] = $password' /etc/shadowsocks.json > tmp.$$.json && mv tmp.$$.json /etc/shadowsocks.json
    echo "Ключ добавлен."
}

# Функция для удаления ключа
delete_key() {
    echo "Введите порт ключа для удаления:"
    read port
    echo "Удаление ключа..."
    jq "del(.port_password[$port])" /etc/shadowsocks.json > tmp.$$.json && mv tmp.$$.json /etc/shadowsocks.json
    echo "Ключ удален."
}

# Функция для редактирования ключа
edit_key() {
    echo "Введите порт ключа для редактирования:"
    read port
    echo "Введите новый пароль для ключа:"
    read password
    echo "Редактирование ключа..."
    jq --arg port "$port" --arg password "$password" '.port_password[$port] = $password' /etc/shadowsocks.json > tmp.$$.json && mv tmp.$$.json /etc/shadowsocks.json
    echo "Ключ изменен."
}

# Функция для вывода базы клиентов
list_keys() {
    echo "Список ключей:"
    jq '.port_password' /etc/shadowsocks.json
}

# Функция для изменения порта ключа
change_port() {
    echo "Введите текущий порт ключа:"
    read current_port
    echo "Введите новый порт:"
    read new_port
    echo "Изменение порта ключа..."
    password=$(jq -r ".port_password[$current_port]" /etc/shadowsocks.json)
    jq "del(.port_password[$current_port]) | .port_password[$new_port] = \"$password\"" /etc/shadowsocks.json > tmp.$$.json && mv tmp.$$.json /etc/shadowsocks.json
    echo "Порт ключа изменен."
}

# Основное меню
check_password

while true; do
    echo "Выберите действие:"
    echo "1. Создать ключ"
    echo "2. Удалить ключ"
    echo "3. Редактировать ключ"
    echo "4. Вывести базу клиентов"
    echo "5. Изменить порт ключа"
    echo "6. Выйти"

    read choice
    case $choice in
        1) create_key ;;
        2) delete_key ;;
        3) edit_key ;;
        4) list_keys ;;
        5) change_port ;;
        6) exit 0 ;;
        *) echo "Неверный выбор. Попробуйте снова." ;;
    esac
done
