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

    <div id="result"></div>

    <script>
        // Переменная для объекта сканера
        let html5QrcodeScanner = null;
        // АДРЕС ВАШЕГО СЕРВЕРА 1С! ЗАМЕНИТЕ НА СВОЙ!
        const apiUrl = 'http://your-1c-server.example.com/barcode/barcodeapi/ПолучитьДанныеПоШтрихКоду';

        // Функция запуска сканера
        function startScanner() {
            // Инициализируем сканер
            html5QrcodeScanner = new Html5QrcodeScanner(
                "reader", // ID контейнера
                { 
                    fps: 10, 
                    qrbox: { width: 250, height: 150 },
                    supportedScanTypes: [Html5QrcodeScanType.SCAN_TYPE_QR_CODE, Html5QrcodeScanType.SCAN_TYPE_BARCODE]
                },
                /* verbose= */ false
            );

            // Обработка успешного сканирования
            html5QrcodeScanner.render(onScanSuccess, onScanFailure);
        }

        // Функция остановки сканера
        function stopScanner() {
            if (html5QrcodeScanner) {
                html5QrcodeScanner.clear().catch(error => {
                    console.error("Failed to clear html5QrcodeScanner. ", error);
                });
                html5QrcodeScanner = null;
                document.getElementById('reader').innerHTML = '';
            }
        }

        // Функция, вызываемая при успешном сканировании
        async function onScanSuccess(decodedText, decodedResult) {
            // Останавливаем сканер после успешного сканирования
            stopScanner();
            // Выводим результат
            await searchBarcode(decodedText);
        }

        // Функция, вызываемая при ошибке сканирования
        function onScanFailure(error) {
            // Ошибки можно игнорировать, так как сканер будет работать до успеха
            // console.warn(`Code scan error = ${error}`);
        }

        // Поиск по штрих-коду (основная функция)
        async function searchBarcode(barcode) {
            const resultDiv = document.getElementById('result');
            resultDiv.innerHTML = '<p class="loading">Поиск...</p>';

            try {
                // Формируем URL с параметром
                const url = `${apiUrl}?ШтрихКод=${encodeURIComponent(barcode)}`;

                // Отправляем GET-запрос к API 1С
                const response = await fetch(url);

                if (!response.ok) {
                    throw new Error(`Ошибка HTTP: ${response.status}`);
                }

                const data = await response.json();

                // Формируем результат на основе ответа от 1С
                if (data.Успех) {
                    resultDiv.className = 'success';
                    resultDiv.innerHTML = `
                        <h3>Товар найден:</h3>
                        <p><strong>Наименование:</strong> ${data.Наименование}</p>
                        <p><strong>Артикул:</strong> ${data.Артикул}</p>
                        <p><strong>Остаток:</strong> ${data.Остаток} шт.</p>
                        <p><strong>Штрих-код:</strong> ${barcode}</p>
                    `;
                } else {
                    resultDiv.className = 'error';
                    resultDiv.innerHTML = `<p><strong>Товар не найден:</strong> ${data.Ошибка}</p>`;
                }

            } catch (error) {
                console.error('Ошибка:', error);
                resultDiv.className = 'error';
                resultDiv.innerHTML = `<p><strong>Произошла ошибка при запросе к серверу:</strong> ${error.message}</p>`;
            }

            // Через 3 секунды снова предлагаем запустить сканер
            setTimeout(() => {
                if (confirm('Сканировать еще один штрих-код?')) {
                    startScanner();
                }
            }, 3000);
        }

        // Функция для поиска по вручную введенному коду
        function searchByManualInput() {
            const manualBarcode = document.getElementById('manual-barcode').value.trim();
            if (manualBarcode) {
                searchBarcode(manualBarcode);
            } else {
                alert('Введите штрих-код');
            }
        }

        // Запускаем сканер при загрузке страницы
        window.onload = startScanner;
    </script>
</body>
</html>
