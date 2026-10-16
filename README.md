# MOUSE-<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>イチゴジャムをきれいにペロペロ！子どもの口の体操アプリ</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts: M PLUS Rounded 1p -->
    <link href="https://fonts.googleapis.com/css2?family=M+PLUS+Rounded+1p:wght@700;900&display=swap" rel="stylesheet">
    <!-- FontAwesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- MediaPipe Face Mesh for Face & Mouth Tracking -->
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/face_mesh.js" crossorigin="anonymous"></script>
    <style>
        body {
            font-family: 'M PLUS Rounded 1p', sans-serif;
            user-select: none;
            -webkit-user-select: none;
            touch-action: none;
            overflow: hidden;
        }
        .text-shadow-lg {
            text-shadow: 0 4px 6px rgba(0, 0, 0, 0.3), 0 1px 3px rgba(0, 0, 0, 0.2);
        }
        .box-glow {
            box-shadow: 0 0 25px rgba(244, 63, 94, 0.6);
        }
        @keyframes float {
            0%, 100% { transform: translateY(0px) rotate(0deg); }
            50% { transform: translateY(-10px) rotate(2deg); }
        }
        .animate-float {
            animation: float 3s ease-in-out infinite;
        }
        @keyframes celebrate {
            0% { transform: scale(0.5) rotate(0deg); opacity: 0; }
            50% { transform: scale(1.1) rotate(5deg); opacity: 1; }
            100% { transform: scale(1) rotate(0deg); opacity: 1; }
        }
        .animate-celebrate {
            animation: celebrate 0.8s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }
    </style>
</head>
<body class="bg-gradient-to-b from-rose-100 via-pink-50 to-amber-50 text-slate-800 h-screen w-screen flex flex-col justify-between relative overflow-hidden">

    <!-- スタート画面オーバーレイ -->
    <div id="startScreen" class="absolute inset-0 z-50 flex flex-col items-center justify-center bg-rose-950/85 backdrop-blur-md p-6 transition-opacity duration-500">
        <div class="bg-white/95 border-8 border-rose-400 rounded-3xl p-8 max-w-lg w-full text-center space-y-6 shadow-2xl box-glow animate-float">
            <div class="inline-block bg-rose-500 text-white text-sm px-5 py-2 rounded-full font-black uppercase tracking-widest shadow-md">
                🍓 口の体操あそび 🍓
            </div>
            <h1 class="text-3xl md:text-5xl font-black text-rose-600 tracking-wide leading-tight">
                順番にジャムを<br>ペロペロしよう！
            </h1>
            <p class="text-slate-600 text-sm md:text-base leading-relaxed px-2 font-bold">
                カメラに顔をうつしてね！<br>
                アゴ、左、右、鼻の下、そして口のまわりへと<strong class="text-rose-500">順番にジャムが登場</strong>するよ。<br>舌を上手に動かしてきれいにしよう♪
            </p>
            <div class="pt-2">
                <button id="startButton" class="relative group overflow-hidden rounded-full bg-gradient-to-r from-rose-500 via-pink-500 to-amber-500 px-12 py-5 text-2xl md:text-3xl font-black text-white shadow-2xl transition-all transform active:scale-95 hover:scale-105 border-4 border-yellow-200">
                    <span class="relative z-10 flex items-center justify-center gap-3">
                        <i class="fa-solid fa-camera text-yellow-200"></i> はじめる！
                    </span>
                </button>
            </div>
            <p class="text-xs text-rose-400 font-bold">※カメラへのアクセスを許可してね</p>
        </div>
    </div>

    <!-- お祝い画面（リザルト）オーバーレイ -->
    <div id="resultScreen" class="absolute inset-0 z-50 flex flex-col items-center justify-center bg-rose-950/85 backdrop-blur-md p-6 hidden opacity-0 transition-opacity duration-500">
        <div class="bg-white border-8 border-yellow-400 rounded-3xl p-8 max-w-md w-full text-center space-y-6 shadow-2xl animate-celebrate">
            <div class="text-6xl animate-bounce">✨🍓👏👑🍓✨</div>
            <h2 class="text-3xl md:text-4xl font-black text-rose-600">よくできたね！</h2>
            <p class="text-slate-700 font-bold text-base md:text-lg">
                すべてのジャムをきれいに舐められたよ！<br>すごい舌の運動でした♪
            </p>
            <div class="pt-2">
                <button id="restartButton" class="w-full bg-gradient-to-r from-rose-500 to-amber-500 py-4 rounded-2xl font-black text-xl text-white shadow-lg border-4 border-yellow-200 hover:scale-105 transition-transform active:scale-95">
                    もういちどあそぶ
                </button>
            </div>
        </div>
    </div>

    <!-- ヘルパーメッセージ -->
    <div id="messageBox" class="absolute top-6 left-1/2 transform -translate-x-1/2 z-40 bg-rose-500 text-white px-6 py-3 rounded-full shadow-xl font-black text-sm md:text-base border-4 border-white opacity-0 pointer-events-none transition-opacity duration-300">
        ジャムを舌で舐めてきれいにしよう！
    </div>

    <!-- ヘッダー：ステップ表示 -->
    <header class="w-full max-w-4xl mx-auto px-4 py-4 z-10 flex justify-between items-center bg-white/80 backdrop-blur-md rounded-3xl mt-4 border-4 border-rose-200 shadow-lg">
        <div class="flex items-center gap-3">
            <div class="w-12 h-12 rounded-2xl bg-rose-500 flex items-center justify-center text-2xl shadow-md text-white">
                🍓
            </div>
            <div>
                <div id="stepTitle" class="text-xs text-rose-500 font-black">ステップ 1 / 5</div>
                <div id="stepInstruction" class="text-lg md:text-xl font-black text-rose-600">アゴの下のジャムを舐めよう！</div>
            </div>
        </div>
        <div class="flex items-center gap-3">
            <button id="resetJamButton" class="bg-amber-400 hover:bg-amber-500 text-slate-900 font-black px-4 py-2 rounded-2xl shadow-md border-2 border-white text-sm flex items-center gap-2 active:scale-95 transition-transform">
                <i class="fa-solid fa-rotate-right"></i> やりなおす
            </button>
        </div>
    </header>

    <!-- メインカメラ＆キャンバスエリア -->
    <main class="flex-1 w-full max-w-4xl mx-auto relative flex items-center justify-center px-4 my-3">
        <div class="relative w-full h-full max-h-[550px] bg-rose-950 rounded-3xl overflow-hidden border-8 border-white shadow-2xl flex items-center justify-center">
            <!-- カメラ映像用Video要素 -->
            <video id="cameraFeed" class="absolute inset-0 w-full h-full object-cover transform -scale-x-100 hidden" autoplay playsinline muted></video>
            
            <!-- 描画用Canvas -->
            <canvas id="outputCanvas" class="absolute inset-0 w-full h-full z-10 transform -scale-x-100"></canvas>
            
            <!-- ガイドメッセージ (中央下) -->
            <div id="guideFooterText" class="absolute bottom-4 left-1/2 transform -translate-x-1/2 z-20 bg-white/90 backdrop-blur-sm px-6 py-2 rounded-full border-2 border-rose-300 text-rose-600 font-black text-xs md:text-sm shadow-lg pointer-events-none">
                👅 舌を伸ばしてアゴのジャムを舐めよう！
            </div>
        </div>
    </main>

    <!-- フッター -->
    <footer class="w-full max-w-4xl mx-auto px-4 pb-4 z-10 text-center">
        <p class="text-xs font-bold text-rose-400">🍓 順番にジャムをペロペロ・お口の体操トレーニング 🍓</p>
    </footer>

    <!-- JavaScript 処理エンジン -->
    <script>
        const videoElement = document.getElementById('cameraFeed');
        const canvasElement = document.getElementById('outputCanvas');
        const canvasCtx = canvasElement.getContext('2d');

        let isGameActive = false;
        let currentStep = 0; // 0: アゴ, 1: 左口角, 2: 右口角, 3: 鼻の下, 4: 口の周りぐるり
        let stepJamOpacity = 1.0;

        const stepsInfo = [
            { title: "ステップ 1 / 5", instruction: "アゴの下のジャム", guide: "👅 舌を下へ伸ばしてアゴのジャムを舐めよう！" },
            { title: "ステップ 2 / 5", instruction: "左の口元のジャム", guide: "👅 舌を左に伸ばしてジャムを舐めよう！" },
            { title: "ステップ 3 / 5", instruction: "右の口元のジャム", guide: "👅 舌を右に伸ばしてジャムを舐めよう！" },
            { title: "ステップ 4 / 5", instruction: "鼻の下のジャム", guide: "👅 舌を上にぐっと伸ばして鼻の下を舐めよう！" },
            { title: "ステップ 5 / 5", instruction: "口のまわりぐるり", guide: "👅 舌で口のまわりをぐるっと1周舐め回そう！" }
        ];

        function showMessage(text, duration = 3000) {
            const box = document.getElementById('messageBox');
            box.innerText = text;
            box.style.opacity = '1';
            setTimeout(() => {
                box.style.opacity = '0';
            }, duration);
        }

        function initStep(step) {
            currentStep = step;
            stepJamOpacity = 1.0;
            document.getElementById('stepTitle').innerText = stepsInfo[step].title;
            document.getElementById('stepInstruction').innerText = stepsInfo[step].instruction;
            document.getElementById('guideFooterText').innerText = stepsInfo[step].guide;
        }

        function resizeCanvas() {
            const container = canvasElement.parentElement;
            canvasElement.width = container.clientWidth;
            canvasElement.height = container.clientHeight;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();
        initStep(0);

        // MediaPipe FaceMesh の初期化
        const faceMesh = new FaceMesh({
            locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/${file}`
        });

        faceMesh.setOptions({
            maxNumFaces: 1,
            refineLandmarks: true,
            minDetectionConfidence: 0.6,
            minTrackingConfidence: 0.6
        });

        faceMesh.onResults(onFaceResults);

        const camera = new Camera(videoElement, {
            onFrame: async () => {
                if (isGameActive) {
                    await faceMesh.send({ image: videoElement });
                }
            },
            width: 1280,
            height: 720
        });

        function onFaceResults(results) {
            canvasCtx.save();
            canvasCtx.clearRect(0, 0, canvasElement.width, canvasElement.height);

            // カメラ映像を描画
            canvasCtx.drawImage(results.image, 0, 0, canvasElement.width, canvasElement.height);

            if (results.multiFaceLandmarks && results.multiFaceLandmarks.length > 0) {
                const landmarks = results.multiFaceLandmarks[0];
                const w = canvasElement.width;
                const h = canvasElement.height;

                // 主要なランドマーク
                const chin = landmarks[152];       // アゴ先
                const upperLip = landmarks[13];    // 上唇
                const lowerLip = landmarks[14];    // 下唇
                const mouthLeft = landmarks[61];   // 口左端
                const mouthRight = landmarks[291]; // 口右端
                const noseTip = landmarks[1];      // 鼻の下（人中付近）

                // 継続的な舐め動作で不透明度を下げる
                const mouthOpenDist = Math.abs(upperLip.y - lowerLip.y);
                if (mouthOpenDist > 0.025 && stepJamOpacity > 0) {
                    stepJamOpacity -= 0.003; // しっかり舐め続けると減る
                    if (stepJamOpacity < 0) stepJamOpacity = 0;
                }

                // 現在のステップに応じた位置にジャムを描画
                if (currentStep === 0) {
                    // 1: アゴの下
                    const x = chin.x * w;
                    const y = (chin.y + 0.04) * h;
                    if (stepJamOpacity > 0.03) {
                        drawStrawberryJam(x, y, 20, stepJamOpacity, canvasCtx);
                    }
                } else if (currentStep === 1) {
                    // 2: 左の口元（少し内側に寄せるためオフセットを縮小）
                    const x = (mouthLeft.x - 0.02) * w;
                    const y = mouthLeft.y * h;
                    if (stepJamOpacity > 0.03) {
                        drawStrawberryJam(x, y, 20, stepJamOpacity, canvasCtx);
                    }
                } else if (currentStep === 2) {
                    // 3: 右の口元（少し内側に寄せるためオフセットを縮小）
                    const x = (mouthRight.x + 0.02) * w;
                    const y = mouthRight.y * h;
                    if (stepJamOpacity > 0.03) {
                        drawStrawberryJam(x, y, 20, stepJamOpacity, canvasCtx);
                    }
                } else if (currentStep === 3) {
                    // 4: 鼻の下
                    const x = noseTip.x * w;
                    const y = (noseTip.y + 0.03) * h;
                    if (stepJamOpacity > 0.03) {
                        drawStrawberryJam(x, y, 20, stepJamOpacity, canvasCtx);
                    }
                } else if (currentStep === 4) {
                    // 5: 口のまわりぐるり
                    const ringSpots = [
                        { x: upperLip.x * w, y: (upperLip.y - 0.03) * h },
                        { x: lowerLip.x * w, y: (lowerLip.y + 0.04) * h },
                        { x: (mouthLeft.x - 0.015) * w, y: mouthLeft.y * h },
                        { x: (mouthRight.x + 0.015) * w, y: mouthRight.y * h }
                    ];
                    ringSpots.forEach(spot => {
                        if (stepJamOpacity > 0.03) {
                            drawStrawberryJam(spot.x, spot.y, 18, stepJamOpacity, canvasCtx);
                        }
                    });
                }

                // ジャムが消えたら次のステップへ進む
                if (stepJamOpacity <= 0.03) {
                    if (currentStep < 4) {
                        currentStep++;
                        initStep(currentStep);
                        showMessage("すごい！次のジャムだよ！");
                    } else {
                        triggerVictory();
                    }
                }
            }

            canvasCtx.restore();
        }

        // イチゴジャムの描画関数
        function drawStrawberryJam(x, y, size, opacity, ctx) {
            ctx.save();
            ctx.globalAlpha = opacity;
            ctx.translate(x, y);

            const grad = ctx.createRadialGradient(-3, -3, 3, 0, 0, size);
            grad.addColorStop(0, '#f43f5e');
            grad.addColorStop(0.7, '#be123c');
            grad.addColorStop(1, '#881337');

            ctx.fillStyle = grad;
            ctx.beginPath();
            ctx.arc(0, 0, size, 0, Math.PI * 2);
            ctx.fill();

            // ツヤハイライト
            ctx.fillStyle = 'rgba(255, 255, 255, 0.6)';
            ctx.beginPath();
            ctx.arc(-size * 0.35, -size * 0.35, size * 0.25, 0, Math.PI * 2);
            ctx.fill();

            ctx.restore();
        }

        function startGame() {
            document.getElementById('startScreen').classList.add('opacity-0');
            setTimeout(() => {
                document.getElementById('startScreen').style.display = 'none';
            }, 500);

            isGameActive = true;
            resizeCanvas();
            initStep(0);
            camera.start();
            showMessage('アゴの下のジャムをペロペロしよう！');
        }

        function triggerVictory() {
            isGameActive = false;
            const resultScreen = document.getElementById('resultScreen');
            resultScreen.classList.remove('hidden');
            setTimeout(() => {
                resultScreen.classList.remove('opacity-0');
            }, 50);
        }

        document.getElementById('startButton').addEventListener('click', async () => {
            startGame();
        });

        document.getElementById('resetJamButton').addEventListener('click', () => {
            initStep(0);
            showMessage('はじめからやり直します！');
        });

        document.getElementById('restartButton').addEventListener('click', () => {
            const resultScreen = document.getElementById('resultScreen');
            resultScreen.classList.add('opacity-0');
            setTimeout(() => {
                resultScreen.classList.add('hidden');
                document.getElementById('startScreen').style.display = 'flex';
                document.getElementById('startScreen').classList.remove('opacity-0');
                initStep(0);
            }, 500);
        });
    </script>
</body>
</html>
