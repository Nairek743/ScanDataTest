<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Сканер штрих-кодов 1С</title>
    <style>
        body {
            font-family: sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            text-align: center;
        }
        #scanner-container {
            width: 100%;
            max-width: 500px;
            margin: 20px auto;
            position: relative;
        }
        #reader {
            width: 100%;
            height: 300px;
            border: 1px solid #ccc;
            background-color: #f5f5f5;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            margin: 10px;
            cursor: pointer;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
        }
        button:hover {
            background-color: #0056b3;
        }
        button:disabled {
            background-color: #6c757d;
            cursor: not-allowed;
        }
        #manual-input {
            margin: 20px 0;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 5px;
        }
        #manual-barcode {
            padding: 8px;
            font-size: 16px;
            margin: 0 10px;
            width: 200px;
        }
        #result {
            margin-top: 20px;
            padding: 15px;
            border-radius: 5px;
            text-align: left;
        }
        .success {
            background-color: #d4edda;
            border: 1px solid #c3e6cb;
            color: #155724;
        }
        .error {
            background-color: #f8d7da;
            border: 1px solid #f5c6cb;
            color: #721c24;
        }
        .loading {
            background-color: #fff3cd;
            border: 1px solid #ffeaa7;
            color: #856404;
        }
        .info {
            background-color: #d1ecf1;
            border: 1px solid #bee5eb;
            color: #0c5460;
        }
        .scanner-active {
            border: 3px solid #28a745 !important;
        }
    </style>
    
    <!-- Подключаем библиотеку -->
    <script src="https://unpkg.com/html5-qrcode@2.3.8/minified/html5-qrcode.min.js"></script>
</head>
<body>
    <h1>📷 Сканер штрих-кодов</h1>
    <p>Наведите камеру на штрих-код товара для получения информации из 1С</p>

    <div id="scanner-container">
        <div id="reader">
            <div style="padding: 50px; text-align: center; color: #666;">
                Камера будет активирована после нажатия "Запустить сканер"
            </div>
        </div>
    </div>

    <div>
        <button onclick="startScanner()" id="start-btn">Запустить сканер</button>
        <button onclick="stopScanner()" id="stop-btn" disabled>Остановить сканер</button>
    </div>

    <div id="manual-input">
        <div>Или введите код вручную:</div>
        <input type="text" id="manual-barcode" placeholder="Введите штрих-код">
        <button onclick="searchByManualInput()">Найти</button>
    </div>

    <div id="result">
        <p class="info">Для начала работы нажмите "Запустить сканер" или введите штрих-код вручную</p>
    </div>

    <script>
        // Ждем полной загрузки страницы и библиотеки
        window.addEventListener('DOMContentLoaded', function() {
            initScanner();
        });

        function initScanner() {
            // Проверяем, загрузилась ли библиотека
            if (typeof Html5Qrcode === 'undefined') {
                console.error('Библиотека Html5Qrcode не загрузилась!');
                document.getElementById('result').innerHTML = 
                    '<p class="error">❌ Ошибка загрузки сканера. Проверьте интернет-соединение и обновите страницу.</p>';
                return;
            }

            console.log('✅ Библиотека Html5Qrcode загружена');
            
            let html5Qrcode = null;
            const apiUrl = 'http://your-1c-server.example.com/barcode/barcodeapi/ПолучитьДанныеПоШтрихКоду';
            let isScanning = false;

            // Функция запуска сканера
            window.startScanner = async function() {
                const resultDiv = document.getElementById('result');
                const readerDiv = document.getElementById('reader');
                const startBtn = document.getElementById('start-btn');
                const stopBtn = document.getElementById('stop-btn');
                
                if (isScanning) {
                    resultDiv.innerHTML = '<p class="info">Сканер уже запущен</p>';
                    return;
                }

                try {
                    resultDiv.innerHTML = '<p class="loading">🔄 Запускаем сканер... Разрешите доступ к камере</p>';

                    // Очищаем предыдущий сканер
                    if (html5Qrcode) {
                        await html5Qrcode.stop();
                        html5Qrcode.clear();
                    }

                    // Создаем экземпляр сканера
                    html5Qrcode = new Html5Qrcode("reader");
                    
                    // Настройки камеры
                    const config = {
                        fps: 10,
                        qrbox: { width: 250, height: 150 },
                        aspectRatio: 1.0
                    };

                    // Запускаем сканер
                    await html5Qrcode.start(
                        { facingMode: "environment" }, // Используем заднюю камеру
                        config,
                        onScanSuccess,
                        onScanFailure
                    );

                    isScanning = true;
                    readerDiv.classList.add('scanner-active');
                    startBtn.disabled = true;
                    stopBtn.disabled = false;
                    
                    resultDiv.innerHTML = '<p class="success">✅ Сканер запущен! Наведите камеру на штрих-код</p>';

                } catch (error) {
                    console.error('Ошибка запуска сканера:', error);
                    resultDiv.innerHTML = `<p class="error">❌ Ошибка: ${error.message}</p>`;
                    
                    if (error.message.includes('permission')) {
                        resultDiv.innerHTML += '<p>Разрешите доступ к камере в настройках браузера</p>';
                    }
                }
            };

            // Функция при успешном сканировании
            async function onScanSuccess(decodedText, decodedResult) {
                console.log(`✅ Отсканировано: ${decodedText}`);
                await stopScanner();
                await searchBarcode(decodedText);
            }

            // Функция при ошибке сканирования
            function onScanFailure(error) {
                // Игнорируем обычные ошибки сканирования (они нормальны)
                // console.log('Ошибка сканирования:', error);
            }

            // Функция остановки сканера
            window.stopScanner = async function() {
                const readerDiv = document.getElementById('reader');
                const startBtn = document.getElementById('start-btn');
                const stopBtn = document.getElementById('stop-btn');
                
                if (html5Qrcode && isScanning) {
                    try {
                        await html5Qrcode.stop();
                        html5Qrcode.clear();
                        isScanning = false;
                        readerDiv.classList.remove('scanner-active');
                        startBtn.disabled = false;
                        stopBtn.disabled = true;
                        
                        readerDiv.innerHTML = '<div style="padding: 50px; text-align: center; color: #666;">Камера отключена</div>';
                        
                    } catch (error) {
                        console.error('Ошибка остановки сканера:', error);
                    }
                }
            };

            // Функция поиска по штрих-коду
            window.searchBarcode = async function(barcode) {
                const resultDiv = document.getElementById('result');
                resultDiv.innerHTML = '<p class="loading">🔍 Ищем товар в базе 1С...</p>';

                // ТЕСТОВЫЕ ДАННЫЕ - удалите этот блок когда подключите API 1С
                setTimeout(() => {
                    resultDiv.className = 'success';
                    resultDiv.innerHTML = `
                        <h3>✅ ТОВАР НАЙДЕН</h3>
                        <p><strong>Штрих-код:</strong> ${barcode}</p>
                        <p><strong>Наименование:</strong> Тестовый товар №${barcode}</p>
                        <p><strong>Артикул:</strong> TEST-${barcode.substring(0, 6)}</p>
                        <p><strong>Остаток на складе:</strong> ${Math.floor(Math.random() * 100)} шт.</p>
                        <p><strong>Цена:</strong> ${Math.floor(Math.random() * 1000) + 100} руб.</p>
                        <div style="text-align: center; margin-top: 15px;">
                            <button onclick="startScanner()">Сканировать еще один код</button>
                        </div>
                    `;
                }, 1500);

                // РАСКОММЕНТИРУЙТЕ ДЛЯ РЕАЛЬНОГО API 1С:
                /*
                try {
                    const url = `${apiUrl}?ШтрихКод=${encodeURIComponent(barcode)}`;
                    const response = await fetch(url);
                    
                    if (!response.ok) throw new Error(`Ошибка HTTP: ${response.status}`);
                    
                    const data = await response.json();
                    
                    if (data.Успех) {
                        resultDiv.className = 'success';
                        resultDiv.innerHTML = `
                            <h3>✅ Товар найден:</h3>
                            <p><strong>Наименование:</strong> ${data.Наименование}</p>
                            <p><strong>Артикул:</strong> ${data.Артикул}</p>
                            <p><strong>Остаток:</strong> ${data.Остаток} шт.</p>
                            <p><strong>Штрих-код:</strong> ${barcode}</p>
                            <div style="text-align: center; margin-top: 15px;">
                                <button onclick="startScanner()">Сканировать еще</button>
                            </div>
                        `;
                    } else {
                        resultDiv.className = 'error';
                        resultDiv.innerHTML = `
                            <h3>❌ Товар не найден</h3>
                            <p><strong>Причина:</strong> ${data.Ошибка}</p>
                            <div style="text-align: center; margin-top: 15px;">
                                <button onclick="startScanner()">Попробовать еще</button>
                            </div>
                        `;
                    }
                } catch (error) {
                    resultDiv.className = 'error';
                    resultDiv.innerHTML = `
                        <h3>❌ Ошибка соединения</h3>
                        <p><strong>Подробности:</strong> ${error.message}</p>
                        <p>Проверьте подключение к серверу 1С</p>
                        <div style="text-align: center; margin-top: 15px;">
                            <button onclick="startScanner()">Попробовать еще</button>
                        </div>
                    `;
                }
                */
            };

            // Поиск по ручному вводу
            window.searchByManualInput = function() {
                const manualBarcode = document.getElementById('manual-barcode').value.trim();
                if (manualBarcode) {
                    // Останавливаем сканер если он запущен
                    if (isScanning) {
                        stopScanner();
                    }
                    searchBarcode(manualBarcode);
                } else {
                    alert('Пожалуйста, введите штрих-код');
                }
            };

            // Останавливаем сканер при закрытии страницы
            window.addEventListener('beforeunload', stopScanner);
            
            // Включаем отправку по Enter в поле ввода
            document.getElementById('manual-barcode').addEventListener('keypress', function(e) {
                if (e.key === 'Enter') {
                    searchByManualInput();
                }
            });

            console.log('✅ Сканер инициализирован, готов к работе!');
            document.getElementById('result').innerHTML = 
                '<p class="success">✅ Система готова к работе! Нажмите "Запустить сканер"</p>';
        }

        // Простая проверка загрузки библиотеки
        setTimeout(function() {
            if (typeof Html5Qrcode !== 'undefined') {
                console.log('✅ Библиотека Html5Qrcode загружена успешно');
            } else {
                console.log('❌ Библиотека Html5Qrcode не загрузилась');
                document.getElementById('result').innerHTML = 
                    '<p class="error">❌ Ошибка загрузки сканера. Обновите страницу.</p>';
            }
        }, 2000);
    </script>
</body>
</html>
