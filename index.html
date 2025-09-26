<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Сканер штрих-кодов</title>
    <style>
        body {
            font-family: sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            text-align: center;
        }
        #status {
            padding: 10px;
            margin: 10px 0;
            border-radius: 5px;
        }
        .loading { background: #fff3cd; color: #856404; }
        .success { background: #d4edda; color: #155724; }
        .error { background: #f8d7da; color: #721c24; }
        button {
            padding: 10px 20px;
            font-size: 16px;
            margin: 10px;
        }
        #reader {
            width: 100%;
            max-width: 400px;
            height: 300px;
            border: 2px dashed #ccc;
            margin: 20px auto;
            display: flex;
            align-items: center;
            justify-content: center;
            background: #f9f9f9;
        }
    </style>
</head>
<body>
    <h1>Сканер штрих-кодов - Диагностика</h1>
    
    <div id="status" class="loading">Загрузка библиотеки...</div>
    
    <div id="reader">
        Камера будет здесь
    </div>
    
    <button onclick="startScanner()" id="scan-btn" disabled>Запустить сканер</button>
    <button onclick="stopScanner()" id="stop-btn" disabled>Остановить</button>
    
    <div id="result"></div>

    <script>
        let html5Qrcode = null;
        let isScanning = false;
        
        // Функция для загрузки библиотеки с нескольких CDN
        function loadScannerLibrary() {
            const statusDiv = document.getElementById('status');
            
            // Пробуем разные CDN
            const cdnUrls = [
                'https://unpkg.com/html5-qrcode@2.3.8/minified/html5-qrcode.min.js',
                'https://cdn.jsdelivr.net/npm/html5-qrcode@2.3.8/minified/html5-qrcode.min.js',
                'https://cdnjs.cloudflare.com/ajax/libs/html5-qrcode/2.3.8/html5-qrcode.min.js'
            ];
            
            function tryLoad(index) {
                if (index >= cdnUrls.length) {
                    statusDiv.className = 'error';
                    statusDiv.innerHTML = '❌ Все CDN недоступны. Проверьте интернет.';
                    return;
                }
                
                statusDiv.innerHTML = `Пробуем загрузить с: ${cdnUrls[index].split('/')[2]}`;
                
                const script = document.createElement('script');
                script.src = cdnUrls[index];
                script.onload = function() {
                    statusDiv.className = 'success';
                    statusDiv.innerHTML = '✅ Библиотека загружена!';
                    initializeScanner();
                };
                script.onerror = function() {
                    console.log(`CDN ${index + 1} не сработал`);
                    tryLoad(index + 1);
                };
                
                document.head.appendChild(script);
            }
            
            tryLoad(0);
        }
        
        function initializeScanner() {
            if (typeof Html5Qrcode === 'undefined') {
                document.getElementById('status').innerHTML = '❌ Библиотека не загрузилась';
                return;
            }
            
            document.getElementById('scan-btn').disabled = false;
            document.getElementById('status').innerHTML = '✅ Система готова. Нажмите "Запустить сканер"';
        }
        
        window.startScanner = async function() {
            const statusDiv = document.getElementById('status');
            const resultDiv = document.getElementById('result');
            
            if (isScanning) return;
            
            try {
                statusDiv.innerHTML = '🔄 Запускаем камеру...';
                
                html5Qrcode = new Html5Qrcode("reader");
                
                await html5Qrcode.start(
                    { facingMode: "environment" },
                    { 
                        fps: 10, 
                        qrbox: { width: 250, height: 150 } 
                    },
                    function(decodedText, decodedResult) {
                        // Успешное сканирование
                        resultDiv.innerHTML = `<p>Отсканировано: <strong>${decodedText}</strong></p>`;
                        stopScanner();
                    },
                    function(error) {
                        // Ошибки сканирования (игнорируем)
                    }
                );
                
                isScanning = true;
                document.getElementById('scan-btn').disabled = true;
                document.getElementById('stop-btn').disabled = false;
                statusDiv.innerHTML = '✅ Сканер активен. Наведите на штрих-код';
                
            } catch (error) {
                statusDiv.className = 'error';
                statusDiv.innerHTML = `❌ Ошибка: ${error.message}`;
            }
        };
        
        window.stopScanner = async function() {
            if (html5Qrcode && isScanning) {
                try {
                    await html5Qrcode.stop();
                    html5Qrcode.clear();
                    isScanning = false;
                    document.getElementById('scan-btn').disabled = false;
                    document.getElementById('stop-btn').disabled = true;
                    document.getElementById('status').innerHTML = '✅ Сканер остановлен';
                } catch (error) {
                    console.error('Ошибка остановки:', error);
                }
            }
        };
        
        // Загружаем библиотеку при старте
        loadScannerLibrary();
    </script>
</body>
</html>
