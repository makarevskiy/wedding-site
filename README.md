# Свадебный сайт Романа и Екатерины

Одностраничный свадебный сайт с таймером до события, формой подтверждения участия и списком гостей.

## 🎯 Особенности

- Адаптивный дизайн для всех устройств
- Таймер обратного отсчета до свадьбы (22.06.2026)
- Форма RSVP для подтверждения участия
- Список гостей с демо-данными
- Анимации и интерактивные элементы
- Календарь с выделенным днем свадьбы
- Информация о программе и дресс-коде

## 🚀 Как использовать

1. Перейдите на сайт: https://[ваш-username].github.io/wedding-site/
2. Заполните форму в разделе "RSVP"
3. Ваш ответ сохранится локально в браузере
4. Просмотрите список всех гостей

## 🛠 Технологии

- HTML5
- CSS3 (Flexbox, Grid, анимации)
- JavaScript (ES6+)
- LocalStorage для хранения данных

## 📁 Структура проекта

NeRoman, [17.01.2026 17:06]
<!-- ==== БЛОК ГОСТЕЙ ==== -->
<div class="container section">
    <div class="guests-section">
        <div class="guests-box">
            <h2 class="guests-title">Гости, подтвердившие участие</h2>
            <div class="guest-count" id="guestCount">
                <span id="guestStats">Загрузка реального списка...</span>
                <button onclick="loadGuests()" class="refresh-btn" title="Обновить">⟳</button>
            </div>
            <div class="guests-list" id="guestsList">
                <div class="loading">
                    <div class="loading-spinner"></div>
                    Загрузка общего списка гостей...
                </div>
            </div>
            <div class="connection-status" id="connectionStatus" style="text-align: center; margin-top: 20px; padding: 10px; border-radius: 8px; font-size: 12px; display: none;">
                <!-- Статус подключения -->
            </div>
        </div>
    </div>
</div>

<!-- ==== ФОРМА RSVP ==== -->
<div class="container section">
    <div class="rsvp-form">
        <h2 style="font-family: 'Bodoni Moda', serif; text-align: center; margin-top: 0;">RSVP</h2>
        <p style="font-size: 13px; text-align: center; margin-bottom: 25px;">Заполните форму, ваш ответ увидят все гости</p>
        
        <form id="rsvpForm">
            <label>Ваше имя *</label>
            <input type="text" id="guestName" placeholder="Имя Фамилия" required>
            
            <label>Количество человек *</label>
            <select id="guestCount" required>
                <option value="1">1 человек</option>
                <option value="2" selected>2 человека</option>
                <option value="3">3 человека</option>
                <option value="4">4 человека</option>
                <option value="5">5+ человек</option>
            </select>
            
            <label>Ваш ответ *</label>
            <select id="guestStatus" required>
                <option value="coming" selected>✅ Я приду</option>
                <option value="not_coming">❌ Не смогу прийти</option>
                <option value="maybe">🤔 Пока не знаю</option>
            </select>
            
            <div style="margin-top: 25px; padding: 15px; background: rgba(255, 133, 162, 0.05); border-radius: 10px; border-left: 3px solid var(--accent-rose);">
                <p style="margin: 0; font-size: 12px; color: #666; line-height: 1.4;">
                    💫 <strong>Работает на GitHub:</strong><br>
                    Ваш ответ сохранится в общий файл, который видят все гости.<br>
                    Обновляется автоматически каждые 30 секунд.
                </p>
            </div>
            
            <button type="submit" class="rsvp-btn" style="margin-top: 20px;">
                ОТПРАВИТЬ ОТВЕТ В ОБЩИЙ СПИСОК
            </button>
        </form>
        
        <!-- Сообщение об успехе -->
        <div id="successMessage" style="display: none; text-align: center; padding: 30px;">
            <div style="font-size: 64px; color: var(--accent-rose); margin-bottom: 20px;">🎉</div>
            <h3 style="font-family: 'Bodoni Moda'; color: var(--accent-rose);">Спасибо!</h3>
            <p id="successText" style="margin: 15px 0; font-size: 16px; line-height: 1.5;"></p>
            <p style="font-size: 12px; color: #666; margin: 20px 0;">
                ✅ Ваш ответ уже в общем списке!
            </p>
            <button onclick="resetForm()" style="
                margin-top: 20px;
                background: var(--accent-rose);
                color: white;
                border: none;
                padding: 12px 30px;
                border-radius: 25px;
                cursor: pointer;
                font-family: 'Montserrat';
                font-weight: 600;
            ">
                Отправить ещё ответ
            </button>
        </div>
    </div>
</div>

NeRoman, [17.01.2026 17:06]
<!-- Скрытый элемент для конфигурации -->
<div id="config" style="display: none;"
     data-owner="ВАШ_GITHUB_USERNAME"
     data-repo="НАЗВАНИЕ_РЕПОЗИТОРИЯ"
     data-file="guests.json"
     data-branch="main">
</div>

<script>
// ==== КОНФИГУРАЦИЯ ====
// ЗАМЕНИТЕ ЭТИ ДАННЫЕ НА СВОИ!
const CONFIG = {
    GITHUB_USERNAME: 'ВАШ_GITHUB_USERNAME',      // Например: 'ivanov'
    REPO_NAME: 'НАЗВАНИЕ_РЕПОЗИТОРИЯ',          // Например: 'wedding-site'
    BRANCH: 'main',
    FILE_PATH: 'guests.json',
    
    // GitHub Personal Access Token (обязательно!)
    // Создайте на: https://github.com/settings/tokens
    // Нужен scope: repo
    GITHUB_TOKEN: 'ВАШ_GITHUB_TOKEN'
};

// Безопасная проверка конфигурации
function checkConfig() {
    if (CONFIG.GITHUB_TOKEN.includes('ВАШ_')) {
        showError('Настройте GitHub Token в коде сайта');
        return false;
    }
    return true;
}

// ==== GITHUB API ФУНКЦИИ ====

// Получить текущий список гостей
async function getGuests() {
    if (!checkConfig()) return [];
    
    try {
        showStatus('🔄 Загрузка списка...', 'info');
        
        const url = https://api.github.com/repos/${CONFIG.GITHUB_USERNAME}/${CONFIG.REPO_NAME}/contents/${CONFIG.FILE_PATH};
        
        const response = await fetch(url, {
            headers: {
                'Authorization': token ${CONFIG.GITHUB_TOKEN},
                'Accept': 'application/vnd.github.v3+json'
            }
        });
        
        if (!response.ok) {
            if (response.status === 404) {
                // Файл не существует, создадим пустой массив
                return [];
            }
            throw new Error(GitHub API ошибка: ${response.status});
        }
        
        const data = await response.json();
        const content = atob(data.content); // Декодируем base64
        const guests = JSON.parse(content || '[]');
        
        showStatus('✅ Список загружен', 'success');
        return guests;
        
    } catch (error) {
        console.error('Ошибка загрузки гостей:', error);
        showStatus('⚠️ Ошибка загрузки', 'error');
        return [];
    }
}

// Добавить нового гостя
async function addGuest(name, count, status) {
    if (!checkConfig()) return false;
    
    try {
        showStatus('🔄 Отправка ответа...', 'info');
        
        // 1. Получаем текущий файл
        const fileUrl = https://api.github.com/repos/${CONFIG.GITHUB_USERNAME}/${CONFIG.REPO_NAME}/contents/${CONFIG.FILE_PATH};
        
        const fileResponse = await fetch(fileUrl, {
            headers: {
                'Authorization': token ${CONFIG.GITHUB_TOKEN},
                'Accept': 'application/vnd.github.v3+json'
            }
        });
        
        let currentContent = '[]';
        let sha = null;
        
        if (fileResponse.ok) {
            const fileData = await fileResponse.json();
            currentContent = atob(fileData.content);
            sha = fileData.sha;
        }
        
        // 2. Добавляем нового гостя
        const guests = JSON.parse(currentContent);
        
        // Проверяем дубликаты (по имени за последние 24 часа)
        const now = Date.now();
        const dayAgo = now - (24 * 60 * 60 * 1000);
        const recentDuplicate = guests.find(g => 
            g.name.toLowerCase() === name.toLowerCase() && 
            g.timestamp > dayAgo
        );
        
        if (recentDuplicate) {
            showError('Вы уже отправляли ответ за последние 24 часа');
            return false;
        }
        
        const newGuest = {
            id: 'guest_' + now,
            name: name.trim(),
            count: parseInt(count),
            status: status,
            date: new Date().toLocaleDateString('ru-RU'),
            time: new Date().toLocaleTimeString('ru-RU', {hour: '2-digit', minute:'2-digit'}),

NeRoman, [17.01.2026 17:06]
timestamp: now,
            ip: 'hidden' // Для приватности
        };
        
        guests.unshift(newGuest); // Добавляем в начало
        
        // 3. Обновляем файл на GitHub
        const updateResponse = await fetch(fileUrl, {
            method: 'PUT',
            headers: {
                'Authorization': token ${CONFIG.GITHUB_TOKEN},
                'Accept': 'application/vnd.github.v3+json',
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                message: Добавлен гость: ${name},
                content: btoa(unescape(encodeURIComponent(JSON.stringify(guests, null, 2)))),
                sha: sha,
                branch: CONFIG.BRANCH
            })
        });
        
        if (!updateResponse.ok) {
            throw new Error(Ошибка сохранения: ${updateResponse.status});
        }
        
        showStatus('✅ Ответ сохранён!', 'success');
        return true;
        
    } catch (error) {
        console.error('Ошибка добавления гостя:', error);
        showError('Ошибка отправки. Попробуйте позже.');
        return false;
    }
}

// ==== ОТОБРАЖЕНИЕ ====

// Загрузить и показать гостей
async function loadGuests() {
    const guests = await getGuests();
    displayGuests(guests);
}

// Показать список гостей
function displayGuests(guests) {
    const container = document.getElementById('guestsList');
    const stats = document.getElementById('guestStats');
    
    if (!guests || guests.length === 0) {
        container.innerHTML = 
            <div class="no-guests" style="text-align: center; padding: 40px;">
                <div style="font-size: 48px; margin-bottom: 15px;">🎊</div>
                <h3 style="color: var(--accent-rose);">Будьте первыми!</h3>
                <p>Пока никто не подтвердил участие.</p>
                <p><small>Заполните форму ниже, ваш ответ увидят все!</small></p>
            </div>
        ;
        stats.textContent = '0 ответов';
        return;
    }
    
    // Статистика
    const total = guests.length;
    const coming = guests.filter(g => g.status === 'coming').length;
    const maybe = guests.filter(g => g.status === 'maybe').length;
    const totalPeople = guests.reduce((sum, g) => sum + (g.count || 1), 0);
    
    stats.textContent = ${total} ответов • ${totalPeople} человек • ${coming} придут;
    
    // Генерация HTML
    let html = '';
    guests.forEach((guest, index) => {
        const statusClass = 
            guest.status === 'coming' ? 'status-coming' :
            guest.status === 'maybe' ? 'status-maybe' : 'status-not-coming';
        
        const statusIcon = 
            guest.status === 'coming' ? '✅' :
            guest.status === 'maybe' ? '🤔' : '❌';
        
        const statusText = 
            guest.status === 'coming' ? Придут ${guest.count > 1 ? guest.count + ' чел.' : ''} :
            guest.status === 'maybe' ? 'Пока не знает' : 'Не сможет';
        
        // Форматируем дату
        let timeText = guest.date || '';
        if (guest.time) {
            timeText +=  в ${guest.time};
        }
        
        // Проверяем, сегодня ли
        const today = new Date().toLocaleDateString('ru-RU');
        if (guest.date === today) {
            timeText = Сегодня в ${guest.time};
        }
        
        html += 
            <div class="guest-item" style="animation: slideIn 0.3s ${index * 0.05}s both;">
                <div>
                    <div class="guest-name">${guest.name}</div>
                    <div class="guest-date">${timeText}</div>
                </div>
                <span class="guest-status ${statusClass}">
                    ${statusIcon} ${statusText}
                </span>
            </div>
        ;
    });
    
    container.innerHTML = html;
}

// ==== ОБРАБОТКА ФОРМЫ ====

NeRoman, [17.01.2026 17:06]
document.getElementById('rsvpForm').addEventListener('submit', async function(e) {
    e.preventDefault();
    
    const name = document.getElementById('guestName').value.trim();
    const count = document.getElementById('guestCount').value;
    const status = document.getElementById('guestStatus').value;
    
    // Валидация
    if (!name || name.length < 2) {
        showError('Введите имя (минимум 2 буквы)');
        return;
    }
    
    if (name.length > 50) {
        showError('Имя слишком длинное (макс. 50 символов)');
        return;
    }
    
    // Блокируем форму
    const form = document.getElementById('rsvpForm');
    const submitBtn = form.querySelector('.rsvp-btn');
    const originalText = submitBtn.textContent;
    submitBtn.textContent = 'Отправка в общий список...';
    submitBtn.disabled = true;
    
    try {
        // Отправляем на GitHub
        const success = await addGuest(name, count, status);
        
        if (success) {
            // Показываем успех
            showSuccess(name, count, status);
            
            // Обновляем список
            setTimeout(loadGuests, 1500);
            
            // Прокручиваем к списку
            setTimeout(() => {
                document.querySelector('.guests-section').scrollIntoView({
                    behavior: 'smooth',
                    block: 'start'
                });
            }, 1000);
        } else {
            // Восстанавливаем форму
            submitBtn.textContent = originalText;
            submitBtn.disabled = false;
        }
        
    } catch (error) {
        console.error('Ошибка формы:', error);
        showError('Ошибка отправки');
        submitBtn.textContent = originalText;
        submitBtn.disabled = false;
    }
});

// Показать успешное сообщение
function showSuccess(name, count, status) {
    const form = document.getElementById('rsvpForm');
    const successDiv = document.getElementById('successMessage');
    const successText = document.getElementById('successText');
    
    // Скрываем форму, показываем успех
    form.style.display = 'none';
    successDiv.style.display = 'block';
