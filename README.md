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
        #interactive.viewport {
            width: 100%;
            height: 300px;
            border: 1px solid #ccc;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            margin: 10px;
            cursor: pointer;
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
            color: #856404;
        }
    </style>
    <!-- Подключаем библиотеку для сканирования -->
    <script type="text/javascript" src="https://unpkg.com/html5-qrcode@2.3.8/minified/html5-qrcode.min.js"></script>
</head>
<body>
    <h1>Сканер штрих-кодов</h1>
    <p>Наведите камеру на штрих-код товара</p>

    <div id="scanner-container">
        <div id="reader"></div>
    </div>

    <button onclick="startScanner()">Запустить сканер</button>
    <button onclick="stopScanner()">Остановить сканер</button>

    <div>Или введите код вручную:</div>
    <input type="text" id="manual-barcode" placeholder="Введите штрих-код">
    <button onclick="searchByManualInput()">Найти</button>

    <div id="result"></div><script>
    let html5Qrcode = null;
    const apiUrl = 'http://your-1c-server.example.com/barcode/barcodeapi/ПолучитьДанныеПоШтрихКоду';
    let isScanning = false;

    // Функция запуска сканера (упрощенная и надежная)
    async function startScanner() {
        const resultDiv = document.getElementById('result');
        const readerDiv = document.getElementById('reader');
        
        if (isScanning) {
            resultDiv.innerHTML = '<p>Сканер уже запущен</p>';
            return;
        }

        try {
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
                supportedScanTypes: [Html5QrcodeScanType.SCAN_TYPE_QR_CODE, Html5QrcodeScanType.SCAN_TYPE_BARCODE]
            };

            // Запускаем сканер
            await html5Qrcode.start(
                { facingMode: "environment" }, // Используем заднюю камеру
                config,
                onScanSuccess,  // Функция при успешном сканировании
                onScanFailure   // Функция при ошибке сканирования
            );

            isScanning = true;
            resultDiv.innerHTML = '<p class="success">Сканер запущен! Наведите камеру на штрих-код</p>';

        } catch (error) {
            console.error('Ошибка запуска сканера:', error);
            resultDiv.innerHTML = `<p class="error">Ошибка: ${error.message}</p>`;
        }
    }

    // Функция при успешном сканировании
    async function onScanSuccess(decodedText, decodedResult) {
        console.log(`Отсканировано: ${decodedText}`);
        await stopScanner();
        await searchBarcode(decodedText);
    }

    // Функция при ошибке сканирования (игнорируем большинство ошибок)
    function onScanFailure(error) {
        // Можно закомментировать, чтобы не засорять консоль
        // console.log('Ошибка сканирования (нормально):', error);
    }

    // Функция остановки сканера
    async function stopScanner() {
        if (html5Qrcode && isScanning) {
            try {
                await html5Qrcode.stop();
                html5Qrcode.clear();
                isScanning = false;
                document.getElementById('reader').innerHTML = '';
            } catch (error) {
                console.error('Ошибка остановки сканера:', error);
            }
        }
    }

    // Функция поиска по штрих-коду
    async function searchBarcode(barcode) {
        const resultDiv = document.getElementById('result');
        resultDiv.innerHTML = '<p class="loading">Ищем товар...</p>';

        // Для теста - заглушка, пока не настроен сервер 1С
        // ЗАКОММЕНТИРУЙТЕ этот блок, когда будете подключать реальный API
        setTimeout(() => {
            resultDiv.className = 'success';
            resultDiv.innerHTML = `
                <h3>ТОВАР НАЙДЕН (тестовые данные)</h3>
                <p><strong>Штрих-код:</strong> ${barcode}</p>
                <p><strong>Наименование:</strong> Тестовый товар</p>
                <p><strong>Артикул:</strong> TEST-${barcode}</p>
                <p><strong>Остаток:</strong> 100 шт.</p>
                <button onclick="startScanner()">Сканировать еще</button>
            `;
        }, 1000);

        // РАСКОММЕНТИРУЙТЕ этот блок, когда будет готов API 1С
        /*
        try {
            const url = `${apiUrl}?ШтрихКод=${encodeURIComponent(barcode)}`;
            const response = await fetch(url);
            
            if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
            
            const data = await response.json();
            
            if (data.Успех) {
                resultDiv.className = 'success';
                resultDiv.innerHTML = `
                    <h3>Товар найден:</h3>
                    <p><strong>Наименование:</strong> ${data.Наименование}</p>
                    <p><strong>Артикул:</strong> ${data.Артикул}</p>
                    <p><strong>Остаток:</strong> ${data.Остаток} шт.</p>
                    <p><strong>Штрих-код:</strong> ${barcode}</p>
                    <button onclick="startScanner()">Сканировать еще</button>
                `;
            } else {
                resultDiv.className = 'error';
                resultDiv.innerHTML = `
                    <p><strong>Товар не найден:</strong> ${data.Ошибка}</p>
                    <button onclick="startScanner()">Попробовать еще</button>
                `;
            }
        } catch (error) {
            resultDiv.className = 'error';
            resultDiv.innerHTML = `
                <p><strong>Ошибка соединения:</strong> ${error.message}</p>
                <button onclick="startScanner()">Попробовать еще</button>
            `;
        }
        */
    }

    // Поиск по ручному вводу
    function searchByManualInput() {
        const manualBarcode = document.getElementById('manual-barcode').value.trim();
        if (manualBarcode) {
            searchBarcode(manualBarcode);
        } else {
            alert('Введите штрих-код');
        }
    }

    // Останавливаем сканер при закрытии страницы
    window.addEventListener('beforeunload', stopScanner);
    
    // Не запускаем автоматически - даем пользователю контроль
    // window.onload = startScanner;
</script>
</body>
</html>
