# Если не определяются мониторы

    sudo apt-get update && sudo apt-get upgrade
    sudo apt dist-upgrade

Ребутнуться

# Создать sh скрипт

    nano script_name.sh

Пишем команды, лучше обойтись без тех, которые требуют ввода y или yes. Обычно скрипты начинаются со строчки

    #!/bin/bash

Чекнуть права и выдать их се:

     ls - l script_name.sh
    sudo chmod 774 script_name.sh

Готово, чтобы запустить:

    ./script_name.sh