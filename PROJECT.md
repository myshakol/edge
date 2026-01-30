# 🎴 Приложение для вытягивания карт (Card Drawer) - Полное описание архитектуры

## Обзор проекта

**Тип:** Одностраничное веб-приложение (Single Page Application - SPA)
**Технология:** Vanilla JavaScript (без фреймворков)
**Структура:** Single HTML файл с встроенными CSS и JavaScript
**Назначение:** Интерактивное приложение для случайного вытягивания карт из выбранной колоды с разными типами карточек (М/Ж) и уровнями сложности

---

## 📋 СТРУКТУРА HTML

### Основная иерархия:
```
body
  └─ .main-wrapper (основной контейнер)
      ├─ .sidebar (левая сидбар панель)
      │  ├─ Логотип EDGE
      │  ├─ Меню навигации (ИГРАТЬ, КОЛОДЫ и т.д.)
      │  └─ Список игровых сценариев
      │
      └─ .main-content (основной контент)
         ├─ .game-area (игровая область)
         │  ├─ .info-panel (панель информации)
         │  ├─ .game-display (основной экран игры)
         │  └─ .controls (управление)
         │
         └─ Модальные окна
            ├─ #colorPaletteModal (выбор колоды по цвету)
            └─ #confirmModal (подтверждение действия)
```

---

## 🎨 CSS СИСТЕМА

### Основные принципы дизайна

1. **Двухколоночный макет** - сидбар слева + основное содержимое справа
2. **Flexbox для раскладки** - все контейнеры используют flexbox для гибкого расположения
3. **Полноэкранное приложение** - использует 100vh для заполнения всей высоты окна
4. **Модальные окна** - наложены поверх всего контента с полупрозрачным фоном
5. **Color-coded UI** - цвета (G, Y, R, B) используются для визуальной идентификации

### Ключевые CSS классы

#### `.main-wrapper`
- Основной контейнер приложения
- `display: flex` - размещает сидбар и контент рядом
- `height: 100vh` - занимает всю высоту окна браузера

#### `.sidebar`
- Левая панель навигации
- `width: 240px` или фиксированная ширина
- Содержит логотип, меню и список сценариев
- **Назначение:** Навигация и выбор игровых режимов

#### `.main-content`
- Основной контент справа
- `flex: 1` - занимает оставшееся место после сидбара
- Содержит всю игровую логику

#### `.game-area`
- Контейнер для игровой области
- `display: flex; flex-direction: column` - раскладывает элементы вертикально

#### `.info-panel`
- Панель информации в верхней части
- Отображает выбранную колоду, уровень, цвет
- **Пример:** "зеленая / основная / --"

#### `.game-display`
- Центральная часть, где отображается карта/задание
- Может быть пустым или содержать текст вытянутой карты
- **Размер:** Большой и видный центр экрана

#### `.controls`
- Панель управления с кнопками
- **Кнопки:**
  - "карта М" - вытянуть карту мужского персонажа
  - "карта Ж" - вытянуть карту женского персонажа
  - "карта Ж ○" - вытянуть карту с бонусом/спецефектом
  - "уровень +" - повысить уровень

#### `.color-palette` или `#colorPaletteModal`
- Модальное окно для выбора колоды по цвету
- Содержит цветные кнопки: G, Y, R, B
- `position: fixed; z-index: 1000` - поверх всего
- `display: none` по умолчанию, видна при нажатии на цвет

#### `.confirm-modal` или `#confirmModal`
- Модальное окно подтверждения
- Текст: "Вы уверены?"
- Кнопки: "Отмена" и "ОК"
- **Использование:** Перед сбросом или другим важным действием

#### Кнопки управления
- Все кнопки имеют одинаковый стиль
- `padding`, `border-radius`, `cursor: pointer`
- Визуальные эффекты при наведении (hover)
- Разные размеры для разных кнопок

---

## 🔧 JAVASCRIPT ЛОГИКА

### 1. STATE MANAGEMENT - Управление состоянием

```javascript
let gameState = {
    currentDeckColor: 'зеленая',      // Текущий цвет колоды (зеленая, основная, и т.д.)
    currentDeckType: 'основная',      // Тип колоды (основная, экстра и т.д.)
    currentLevel: '--',                // Уровень сложности
    currentGender: null,               // Выбранный пол (М или Ж)
    currentCard: null,                 // Текущая вытянутая карта (объект с текстом)
    timerInterval: null,               // ID интервала таймера
    elapsedTime: 0,                   // Прошедшее время в секундах
    gameActive: true                  // Активна ли игра
};
```

**Структура отличается от обычной карточной игры:**
- Нет массива карт для кликания
- Нет Set кликнутых карт
- Вместо этого: одна вытянутая карта за раз
- Фокус на **случайном выборе** из большой колоды

### 2. КОНФИГУРАЦИЯ КОЛОД И УРОВНЕЙ

```javascript
const DECK_COLORS = ['зеленая', 'основная', 'красная', 'синяя'];
// Цвета для выбора палитры

const DECK_TYPES = {
    'зеленая': ['основная', 'экстра'],
    'основная': ['основная', 'фото-видео съемка 🎁'],
    // ... и так далее для каждого цвета
};

const LEVELS = ['--', 'Уровень 1', 'Уровень 2', 'Уровень 3', 'Уровень 4'];
// Уровни сложности
```

**Объяснение:**
- **DECK_COLORS** - доступные цвета колод
- **DECK_TYPES** - для каждого цвета список доступных типов колод
- **LEVELS** - список уровней сложности

### 3. ДАННЫЕ КАРТОЧЕК

```javascript
const DECK_DATA = {
    'зеленая': {
        'основная': [
            { text: 'Текст карты 1', gender: 'M' },
            { text: 'Текст карты 2', gender: 'Ж' },
            // ... много карт
        ],
        'экстра': [
            // Другие карты
        ]
    },
    'основная': {
        'основная': [
            // Карты основной колоды
        ]
    },
    // ... и так далее
};
```

**Структура карты:**
- **`text`** - текст задания/сценария на карте
- **`gender`** - пол: 'M' (мужская) или 'Ж' (женская)

**Особенность:** Каждая карта в колоде может быть либо для М, либо для Ж

---

## ⏱️ ТАЙМЕР

### `startTimer()`
```javascript
function startTimer() {
    gameState.timerInterval = setInterval(() => {
        gameState.elapsedTime++;
        updateTimerDisplay();
    }, 1000);
}
```

**Что происходит?**
- Таймер начинает отсчет времени
- Каждую секунду увеличивается счетчик
- Обновляется отображение на экране

### `stopTimer()`
```javascript
function stopTimer() {
    if (gameState.timerInterval) {
        clearInterval(gameState.timerInterval);
        gameState.timerInterval = null;
    }
}
```

**Зачем нужен?** Остановить таймер при выходе или сбросе

### `resetTimer()`
```javascript
function resetTimer() {
    stopTimer();
    gameState.elapsedTime = 0;
    updateTimerDisplay();
}
```

**Отличие от stopTimer:** Дополнительно сбрасывает счетчик на 0

### `updateTimerDisplay()`
```javascript
function updateTimerDisplay() {
    const timerElement = document.getElementById('timer');
    const minutes = Math.floor(gameState.elapsedTime / 60);
    const seconds = gameState.elapsedTime % 60;
    timerElement.textContent = 
        `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
}
```

**Что это делает?**
- Преобразует секунды в формат MM:SS
- `padStart(2, '0')` добавляет ведущие нули (например, "00:05" вместо "0:5")
- Обновляет текст таймера на экране

---

## 🎮 ОСНОВНЫЕ ФУНКЦИИ ИГРЫ

### `drawCard(gender)`

```javascript
function drawCard(gender) {
    // 1. Получить текущую колоду
    const deckData = DECK_DATA[gameState.currentDeckColor]?.[gameState.currentDeckType];
    
    if (!deckData || deckData.length === 0) {
        alert('Колода пуста!');
        return;
    }
    
    // 2. Отфильтровать карты по полу
    const filteredCards = deckData.filter(card => card.gender === gender);
    
    if (filteredCards.length === 0) {
        alert(`Карт ${gender} типа нет в этой колоде!`);
        return;
    }
    
    // 3. Выбрать случайную карту
    const randomIndex = Math.floor(Math.random() * filteredCards.length);
    gameState.currentCard = filteredCards[randomIndex];
    
    // 4. Отобразить карту
    displayCard(gameState.currentCard);
    
    // 5. Сохранить выбранный пол
    gameState.currentGender = gender;
}
```

**Пошагово:**
1. Получить карты из выбранной колоды
2. Отфильтровать по полу (М или Ж)
3. Выбрать случайную карту через `Math.random()`
4. Показать карту на экране
5. Запомнить, какой пол был выбран

**Почему фильтрация?** В одной колоде могут быть карты разных полов, нужно выбрать правильные

### `displayCard(card)`

```javascript
function displayCard(card) {
    const gameDisplay = document.querySelector('.game-display');
    gameDisplay.textContent = card.text;
    gameDisplay.style.opacity = 1;
    // Возможны анимации появления карты
}
```

**Что это делает?**
- Показывает текст карты в центре экрана
- Добавляет визуальный эффект (например, появление)

### `changeDeckColor(color)`

```javascript
function changeDeckColor(color) {
    gameState.currentDeckColor = color;
    // Обновить доступные типы колод для этого цвета
    updateDeckTypesList();
}
```

**Что это делает?**
- Изменить цвет выбранной колоды
- Обновить список доступных типов для этого цвета

### `changeDeckType(type)`

```javascript
function changeDeckType(type) {
    gameState.currentDeckType = type;
    // Может быть автоматический сброс текущей карты
}
```

**Что это делает?** Изменить тип (подколоду) внутри цвета

### `changeLevel(newLevel)`

```javascript
function changeLevel(newLevel) {
    gameState.currentLevel = newLevel;
    // Может влиять на фильтрацию или сложность карт
}
```

**Что это делает?** Изменить уровень сложности

### `resetGame()`

```javascript
function resetGame() {
    resetTimer();
    gameState.currentCard = null;
    gameState.currentGender = null;
    document.querySelector('.game-display').textContent = '';
    // Очистить экран
}
```

**Что это делает?** Полностью очистить игру и таймер

---

## 🎨 ПАЛИТРА ЦВЕТОВ (ВЫБОР КОЛОДЫ)

### `showColorPalette()`

```javascript
function showColorPalette() {
    const modal = document.getElementById('colorPaletteModal');
    modal.style.display = 'flex';  // или 'block', в зависимости от CSS
}
```

**Что происходит?**
- Показывает модальное окно с выбором цветов (G, Y, R, B)
- Пользователь может выбрать цвет колоды

### `closeColorPalette()`

```javascript
function closeColorPalette() {
    const modal = document.getElementById('colorPaletteModal');
    modal.style.display = 'none';
}
```

**Просто скрывает модальное окно**

---

## 📦 МОДАЛЬНЫЕ ОКНА

### `showConfirmModal(message, onConfirm)`

```javascript
function showConfirmModal(message, onConfirm) {
    const modal = document.getElementById('confirmModal');
    document.querySelector('#confirmModal .message').textContent = message;
    
    const okButton = modal.querySelector('.ok-button');
    okButton.onclick = () => {
        onConfirm();
        closeConfirmModal();
    };
    
    modal.style.display = 'flex';
}
```

**Что это делает?**
1. Показывает модальное окно с сообщением
2. При клике "ОК" выполняет переданную функцию
3. Закрывает модаль после подтверждения

### `closeConfirmModal()`

```javascript
function closeConfirmModal() {
    const modal = document.getElementById('confirmModal');
    modal.style.display = 'none';
}
```

---

## 🚀 ИНИЦИАЛИЗАЦИЯ

```javascript
document.addEventListener('DOMContentLoaded', () => {
    // Инициализировать UI
    updateInfoPanel();
    resetTimer();
    // Остальная инициализация
});
```

**Когда это вызывается?**
- Когда DOM полностью загружен
- Браузер готов к взаимодействию

---

## 📊 FLOW ДИАГРАММА ВЫТЯГИВАНИЯ КАРТЫ

```
┌─────────────────────────┐
│   Пользователь кликает  │
│  на "карта М" или "Ж"   │
└──────────────┬──────────┘
               │
               ▼
┌──────────────────────────────────┐
│  drawCard(gender)                │
│  - Получить текущую колоду       │
│  - Отфильтровать по полу         │
│  - Выбрать случайную карту       │
└──────────────┬───────────────────┘
               │
               ▼
        Карта выбрана?
         /          \
       ДА           НЕТ
       │             │
       ▼             ▼
  displayCard()  alert('Нет карт!')
       │
       ▼
┌─────────────────────┐
│  Карта показана на  │
│  экране             │
└─────────────────────┘
```

---

## 🔐 ЗАЩИТЫ И ПРОВЕРКИ

| Проверка | Где | Зачем |
|----------|-----|-------|
| `if (!deckData)` | drawCard | Проверить, что колода существует |
| `if (filteredCards.length === 0)` | drawCard | Проверить, что есть карты нужного пола |
| `if (gameState.timerInterval)` | stopTimer | Не вызывать clearInterval без интервала |
| Выбор цвета из G,Y,R,B | UI | Ограничить выбор валидными цветами |

---

## 📝 РЕЗЮМЕ

**Это специализированное приложение для случайного выбора:**
- ✅ Простой и интуитивный интерфейс
- ✅ Модульная система колод (цвет + тип)
- ✅ Разделение по полам (М/Ж)
- ✅ Таймер для отслеживания времени
- ✅ Модальные окна для подтверждения и выбора
- ✅ Адаптивный дизайн со сидбаром

**Основной поток:**
1. Выбрать цвет колоды (палитра)
2. Выбрать тип колоды
3. Выбрать уровень (опционально)
4. Нажать "карта М" или "карта Ж"
5. Увидеть вытянутую карту
6. Повторить шаги 4-5 или выбрать новую колоду
