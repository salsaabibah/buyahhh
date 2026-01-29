<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Traditional Game on Modern Device</title>
    <style>
        :root {
            --bg-color: #ffffff;
            --accent-color: #333;
        }

        body {
            font-family: 'Comic Sans MS', 'Chalkboard SE', cursive;
            background-color: var(--bg-color);
            color: var(--accent-color);
            transition: background-color 0.4s ease;
            margin: 0;
            overflow: hidden; /* Mencegah scroll karena doodle banyak */
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
        }

        /* Container untuk doodle yang tersebar */
        #doodle-canvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: -1;
            pointer-events: none;
            display: grid;
            grid-template-columns: repeat(5, 1fr);
            grid-template-rows: repeat(5, 1fr);
            opacity: 0.3; /* Agar doodle tidak mengganggu teks */
            font-size: 2rem;
            padding: 20px;
        }

        .container {
            background: rgba(255, 255, 255, 0.9);
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            text-align: center;
            z-index: 10;
        }

        #webcam-container {
            margin: 20px 0;
            border: 5px solid var(--accent-color);
            border-radius: 15px;
            overflow: hidden;
            line-height: 0;
        }

        #label-container div {
            margin: 5px 0;
            padding: 8px;
            border-radius: 10px;
            background: #eee;
            font-weight: bold;
            min-width: 200px;
        }

        button {
            background-color: #333;
            color: white;
            border: none;
            padding: 15px 30px;
            font-size: 1.2rem;
            border-radius: 50px;
            cursor: pointer;
            transition: 0.3s;
        }

        button:hover {
            transform: scale(1.1);
            background-color: #000;
        }
    </style>
</head>
<body>

    <div id="doodle-canvas"></div>

    <div class="container">
        <h1>Traditional Game on Modern Device</h1>
        <button type="button" onclick="init()">Mulai Game</button>
        <div id="webcam-container"></div>
        <div id="label-container"></div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>

    <script type="text/javascript">
        const URL = "https://teachablemachine.withgoogle.com/models/IgPFn3Wcq/";

        let model, webcam, labelContainer, maxPredictions;
        const doodleCanvas = document.getElementById("doodle-canvas");

        async function init() {
            const modelURL = URL + "model.json";
            const metadataURL = URL + "metadata.json";

            model = await tmImage.load(modelURL, metadataURL);
            maxPredictions = model.getTotalClasses();

            const flip = true;
            webcam = new tmImage.Webcam(250, 250, flip);
            await webcam.setup();
            await webcam.play();
            window.requestAnimationFrame(loop);

            document.getElementById("webcam-container").appendChild(webcam.canvas);
            labelContainer = document.getElementById("label-container");
            for (let i = 0; i < maxPredictions; i++) {
                labelContainer.appendChild(document.createElement("div"));
            }
        }

        async function loop() {
            webcam.update();
            await predict();
            window.requestAnimationFrame(loop);
        }

        async function predict() {
            const prediction = await model.predict(webcam.canvas);
            
            let topClass = "";
            let topProb = 0;

            for (let i = 0; i < maxPredictions; i++) {
                const prob = prediction[i].probability;
                const percentage = (prob * 100).toFixed(1) + "%";
                labelContainer.childNodes[i].innerHTML = `${prediction[i].className}: ${percentage}`;
                
                if (prob > topProb) {
                    topProb = prob;
                    topClass = prediction[i].className;
                }
            }

            // Update Tampilan jika probabilitas di atas 70%
            if (topProb > 0.7) {
                updateStyle(topClass);
            } else {
                updateStyle("Kosong");
            }
        }

        function updateStyle(label) {
            let bgColor = "#ffffff";
            let doodleIcon = "✨";
            const name = label.toLowerCase();

            if (name.includes("gunting")) {
                bgColor = "#3498db"; // Biru
                doodleIcon = "✂️";
            } else if (name.includes("batu")) {
                bgColor = "#95a5a6"; // Abu-abu
                doodleIcon = "🪨";
            } else if (name.includes("kertas")) {
                bgColor = "#e74c3c"; // Merah
                doodleIcon = "📄";
            } else {
                bgColor = "#ffffff";
                doodleIcon = "🎨";
            }

            document.body.style.backgroundColor = bgColor;
            
            // Mengisi canvas dengan doodle yang tersebar jika berubah class
            if (window.currentLabel !== label) {
                doodleCanvas.innerHTML = "";
                for (let i = 0; i < 25; i++) {
                    const span = document.createElement("span");
                    span.innerHTML = doodleIcon;
                    // Rotasi acak sedikit agar lebih terlihat seperti doodle tangan
                    span.style.transform = `rotate(${Math.random() * 40 - 20}deg)`;
                    span.style.textAlign = "center";
                    doodleCanvas.appendChild(span);
                }
                window.currentLabel = label;
            }
        }
    </script>
</body>
</html>
