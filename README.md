<!DOCTYPE html> 
<html>
<head>
    <title>Тест камеры</title>
</head>
<body>
    <h1>Тест доступа к камере</h1>
    <button onclick="testCamera()">Проверить камеру</button>
    <div id="result"></div>

    <script>
        async function testCamera() {
            const resultDiv = document.getElementById('result');
            
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                resultDiv.innerHTML = '<p style="color:green">Камера доступна! ✅</p>';
                
                // Покажем видео с камеры
                const video = document.createElement('video');
                video.srcObject = stream;
                video.autoplay = true;
                resultDiv.appendChild(video);
                
            } catch (error) {
                resultDiv.innerHTML = `<p style="color:red">Ошибка: ${error.message}</p>`;
            }
        }
    </script>
</body>
</html>
