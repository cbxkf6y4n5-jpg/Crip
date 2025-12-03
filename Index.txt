<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Тапалка "Крипер"</title>
    <!-- Подключение Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Использование шрифта Inter для современного вида */
        body { font-family: 'Inter', sans-serif; }
        
        /* Основная игровая область, которая заполняет почти весь экран */
        #game-area {
            position: relative;
            cursor: pointer;
            overflow: hidden;
            background-color: #1a1a1a; /* Темный фон для контраста */
            border-radius: 1.5rem; /* Скругленные углы */
            box-shadow: 0 10px 15px rgba(0, 0, 0, 0.5);
            touch-action: manipulation; /* Улучшение отклика на мобильных устройствах */
        }

        /* Стиль для самого Крипера */
        .creeper {
            position: absolute;
            width: 60px; /* Базовый размер */
            height: 60px;
            cursor: pointer;
            transition: transform 0.1s ease-out;
            will-change: transform;
            user-select: none;
            -webkit-tap-highlight-color: transparent;
        }

        /* Эффект при нажатии */
        .creeper:active {
            transform: scale(0.9);
        }

        /* Адаптивный размер Крипера для разных экранов */
        @media (min-width: 640px) {
            .creeper {
                width: 80px;
                height: 80px;
            }
        }
    </style>
    <script>
        // Конфигурация Tailwind для использования цвета Крипера
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        'creeper-green': '#00A100',
                        'creeper-dark': '#006400',
                    }
                }
            }
        }
    </script>
</head>
<body class="bg-gray-900 min-h-screen flex flex-col items-center justify-center p-4">

    <!-- Заголовок и счетчик -->
    <div class="w-full max-w-lg mb-6 flex justify-between items-center">
        <h1 class="text-3xl font-extrabold text-white">
            Тапалка "Крипер"
        </h1>
        <div class="p-3 bg-creeper-green text-gray-900 font-bold rounded-xl shadow-lg">
            Счет: <span id="score" class="text-2xl">0</span>
        </div>
    </div>

    <!-- Игровая область -->
    <div id="game-area" class="w-full max-w-lg h-[80vh] sm:h-[600px]">
        <!-- Крипер будет генерироваться здесь -->
    </div>

    <!-- Инструкции -->
    <p class="mt-4 text-gray-400 text-sm text-center max-w-md">
        Нажми на Крипера, чтобы он исчез и увеличить счет.
    </p>


    <script>
        // Глобальные переменные игры
        let score = 0;
        const gameArea = document.getElementById('game-area');
        const scoreDisplay = document.getElementById('score');
        const CREEPER_SIZE = 60; // Базовый размер в пикселях (будет увеличен CSS)

        /**
         * Адаптивно определяет текущий размер Крипера в пикселях.
         * @returns {number} Размер Крипера.
         */
        function getCreeperActualSize() {
            // Проверяем медиа-запрос, используемый в CSS для адаптивности
            if (window.matchMedia('(min-width: 640px)').matches) {
                return 80; // Размер для sm: и выше
            }
            return CREEPER_SIZE; // Базовый размер
        }


        /**
         * Генерирует SVG-изображение головы Крипера.
         * @returns {string} Строка с SVG.
         */
        function generateCreeperSVG() {
            // SVG-изображение головы Крипера в пиксельном стиле
            return `
                <svg viewBox="0 0 16 16" xmlns="http://www.w3.org/2000/svg" class="w-full h-full rounded-md shadow-xl transition-all duration-100 ease-out hover:scale-[1.05]">
                    <!-- Зеленый фон головы -->
                    <rect width="16" height="16" fill="#00A100"/>
                    
                    <!-- Черные глаза -->
                    <rect x="4" y="4" width="2" height="2" fill="#000"/>
                    <rect x="10" y="4" width="2" height="2" fill="#000"/>
                    
                    <!-- Черная область носа/рта -->
                    <rect x="6" y="6" width="4" height="4" fill="#000"/>
                    
                    <!-- Имитация разреза рта -->
                    <rect x="6" y="8" width="1" height="2" fill="#00A100"/>
                    <rect x="9" y="8" width="1" height="2" fill="#00A100"/>
                    
                    <!-- Дополнительные детали для тени/объема (опционально) -->
                    <rect x="15" y="0" width="1" height="16" fill="#006400" opacity="0.3"/>
                    <rect x="0" y="15" width="16" height="1" fill="#006400" opacity="0.3"/>
                </svg>
            `;
        }

        /**
         * Обновляет счетчик и спавнит нового Крипера.
         */
        function handleCreeperClick(event) {
            // Останавливаем всплытие события, чтобы не триггерить спавн Крипера через game-area
            event.stopPropagation(); 

            // Увеличиваем счет
            score++;
            scoreDisplay.textContent = score;

            // Удаляем текущего Крипера
            const creeperElement = event.currentTarget;
            creeperElement.remove();

            // Спавним нового Крипера
            spawnCreeper();
        }

        /**
         * Создает и размещает нового Крипера в случайном месте.
         */
        function spawnCreeper() {
            // Проверяем, есть ли уже Крипер. Если есть, удаляем его, чтобы спавнить только одного
            gameArea.querySelectorAll('.creeper').forEach(c => c.remove());

            const creeper = document.createElement('div');
            creeper.className = 'creeper';
            creeper.innerHTML = generateCreeperSVG();
            creeper.onclick = handleCreeperClick;

            const size = getCreeperActualSize();

            // Вычисляем максимальные координаты для спавна
            // gameArea.clientWidth и clientHeight дают размеры области без padding/border
            const maxX = gameArea.clientWidth - size;
            const maxY = gameArea.clientHeight - size;

            if (maxX <= 0 || maxY <= 0) {
                console.error("Игровая область слишком мала для спавна Крипера.");
                return;
            }

            // Генерируем случайные координаты
            const randomX = Math.floor(Math.random() * maxX);
            const randomY = Math.floor(Math.random() * maxY);

            // Устанавливаем позицию
            creeper.style.left = `${randomX}px`;
            creeper.style.top = `${randomY}px`;
            
            // Добавляем в игровую область
            gameArea.appendChild(creeper);
        }

        /**
         * Обработчик изменения размеров окна (для мобильных устройств).
         * Перезапускает спавн, чтобы Крипер не застрял за границами.
         */
        let resizeTimeout;
        window.addEventListener('resize', () => {
            clearTimeout(resizeTimeout);
            resizeTimeout = setTimeout(() => {
                // Только если Крипер существует, спавним его заново
                if (gameArea.querySelector('.creeper')) {
                    spawnCreeper();
                }
            }, 250);
        });

        // Запуск игры при загрузке страницы
        window.onload = function() {
            spawnCreeper();
        };

    </script>
</body>
</html>

