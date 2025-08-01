# -<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>自動ランダムジェネレーター</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* Light blue-gray background */
        }
        /* Custom styles for triangle rotation */
        .rotate-0 { transform: rotate(0deg); } /* Up */
        .rotate-90 { transform: rotate(90deg); } /* Right */
        .rotate-180 { transform: rotate(180deg); } /* Down */
        .rotate-270 { transform: rotate(270deg); } /* Left */

        /* Custom style for black circle */
        .bg-black-500 { background-color: #000; } /* Tailwind doesn't have black-500 by default */
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">
    <div class="bg-white p-8 rounded-xl shadow-2xl max-w-md w-full text-center">
        <h1 class="text-3xl font-bold text-gray-800 mb-6">自動ランダムジェネレーター</h1>
        <p class="text-gray-600 mb-8">カテゴリを選択し、「開始」ボタンで自動生成を開始します。</p>

        <!-- カテゴリ選択 -->
        <div class="mb-8 p-4 bg-blue-50 rounded-lg shadow-inner">
            <h2 class="text-xl font-semibold text-gray-700 mb-4">カテゴリを選択:</h2>
            <div class="flex flex-wrap justify-center gap-4">
                <label class="inline-flex items-center">
                    <input type="radio" class="form-radio text-blue-600 h-5 w-5" name="category" value="color" checked>
                    <span class="ml-2 text-gray-700 font-medium">色</span>
                </label>
                <label class="inline-flex items-center">
                    <input type="radio" class="form-radio text-green-600 h-5 w-5" name="category" value="number">
                    <span class="ml-2 text-gray-700 font-medium">数字</span>
                </label>
                <label class="inline-flex items-center">
                    <input type="radio" class="form-radio text-purple-600 h-5 w-5" name="category" value="direction">
                    <span class="ml-2 text-gray-700 font-medium">方向</span>
                </label>
            </div>
        </div>

        <!-- コントロールボタン -->
        <div class="flex justify-center gap-4 mb-8">
            <button id="startBtn" class="bg-indigo-600 hover:bg-indigo-700 text-white font-semibold py-3 px-8 rounded-lg shadow-md transition duration-300 ease-in-out transform hover:scale-105">
                開始
            </button>
            <button id="stopBtn" class="bg-red-500 hover:bg-red-600 text-white font-semibold py-3 px-8 rounded-lg shadow-md transition duration-300 ease-in-out transform hover:scale-105" disabled>
                停止
            </button>
        </div>

        <!-- 結果表示エリア -->
        <div id="outputDisplay" class="bg-gray-100 text-gray-800 text-5xl font-extrabold py-10 rounded-lg shadow-inner flex items-center justify-center min-h-[150px]">
            <!-- 結果がここに表示されます -->
            <span class="text-gray-400 text-lg">カテゴリを選択して開始</span>
        </div>
    </div>

    <script>
        // DOM要素の取得
        const categoryRadios = document.querySelectorAll('input[name="category"]');
        const startBtn = document.getElementById('startBtn');
        const stopBtn = document.getElementById('stopBtn');
        const outputDisplay = document.getElementById('outputDisplay');

        let intervalId = null; // setIntervalのIDを保持する変数
        let selectedCategory = 'color'; // 初期選択カテゴリ
        let isDisplayingResult = false; // 結果表示中かどうかのフラグ
        const displayDuration = 1500; // 結果表示時間 (1.5秒)
        const blankDuration = 1500;  // ブランク時間 (1.5秒)
        const totalCycleTime = displayDuration + blankDuration; // 合計サイクル時間 (3秒)

        // ランダムな色、数字、方向のリストを定義
        const colors = [
            'red', 'blue', 'yellow', 'black' // 黒を追加
        ];
        // 方向は回転角度で表現
        const directions = [
            { name: '上', rotate: 'rotate-0' },    // 0deg
            { name: '右', rotate: 'rotate-90' },   // 90deg
            { name: '下', rotate: 'rotate-180' },  // 180deg
            { name: '左', rotate: 'rotate-270' }   // 270deg
        ];

        /**
         * 指定されたカテゴリに基づいてランダムな結果を生成し、表示を更新します。
         * @param {string} category - 生成するカテゴリ ('color', 'number', 'direction')
         */
        function generateRandomOutput(category) {
            outputDisplay.innerHTML = ''; // 前の結果をクリア
            outputDisplay.classList.remove('text-gray-400', 'text-lg'); // 結果待ちのスタイルを削除
            outputDisplay.classList.remove('text-5xl'); // 数字以外の表示のために一時的にサイズを調整

            switch (category) {
                case 'color':
                    const randomColor = colors[Math.floor(Math.random() * colors.length)];
                    const colorCircle = document.createElement('div');
                    // Tailwindのカラークラスを使用。黒はカスタムCSSで定義
                    colorCircle.className = `w-24 h-24 rounded-full shadow-lg ${randomColor === 'black' ? 'bg-black-500' : `bg-${randomColor}-500`}`;
                    outputDisplay.appendChild(colorCircle);
                    break;
                case 'number':
                    const randomNumber = Math.floor(Math.random() * 10) + 1; // 1から10までのランダムな数字
                    outputDisplay.textContent = randomNumber;
                    outputDisplay.classList.add('text-5xl'); // 数字は大きく表示
                    break;
                case 'direction':
                    const randomDirection = directions[Math.floor(Math.random() * directions.length)];
                    const svgContainer = document.createElementNS("http://www.w3.org/2000/svg", "svg");
                    svgContainer.setAttribute('width', '80');
                    svgContainer.setAttribute('height', '80');
                    svgContainer.setAttribute('viewBox', '0 0 100 100');
                    svgContainer.classList.add('text-gray-800', randomDirection.rotate); // Tailwindの回転クラスを適用

                    const polygon = document.createElementNS("http://www.w3.org/2000/svg", "polygon");
                    polygon.setAttribute('points', '50,15 85,85 15,85'); // 上向きの三角形の座標
                    polygon.setAttribute('fill', 'currentColor');

                    svgContainer.appendChild(polygon);
                    outputDisplay.appendChild(svgContainer);
                    break;
                default:
                    outputDisplay.textContent = "エラー";
            }
        }

        /**
         * カテゴリ選択ラジオボタンの状態を更新します。
         * @param {boolean} disabled - 無効にする場合は true, 有効にする場合は false
         */
        function setCategoryRadiosDisabled(disabled) {
            categoryRadios.forEach(radio => {
                radio.disabled = disabled;
            });
        }

        // 自動生成のメインロジック
        function startGenerationCycle() {
            if (isDisplayingResult) {
                // 結果を表示するフェーズ
                generateRandomOutput(selectedCategory);
                isDisplayingResult = false;
            } else {
                // ブランクを表示するフェーズ
                outputDisplay.innerHTML = '<span class="text-gray-400 text-lg">次へ...</span>';
                outputDisplay.classList.add('text-gray-400', 'text-lg');
                outputDisplay.classList.remove('text-5xl'); // 数字表示のフォントサイズを元に戻す
                isDisplayingResult = true;
            }
        }

        // 開始ボタンのイベントリスナー
        startBtn.addEventListener('click', () => {
            // 選択されたカテゴリを取得
            categoryRadios.forEach(radio => {
                if (radio.checked) {
                    selectedCategory = radio.value;
                }
            });

            // 既にインターバルが動いている場合は停止
            if (intervalId) {
                clearInterval(intervalId);
            }

            // 初回表示
            isDisplayingResult = true; // 次のサイクルで結果を表示するように設定
            startGenerationCycle(); // 最初の結果を即時表示

            // 3秒間隔で自動生成サイクルを開始 (1.5秒表示 + 1.5秒ブランク)
            intervalId = setInterval(startGenerationCycle, displayDuration); // displayDurationは1.5秒なので、1.5秒ごとに切り替わる

            // UIの状態を更新
            startBtn.disabled = true;
            stopBtn.disabled = false;
            setCategoryRadiosDisabled(true); // カテゴリ選択を無効化
        });

        // 停止ボタンのイベントリスナー
        stopBtn.addEventListener('click', () => {
            clearInterval(intervalId); // 自動生成を停止
            intervalId = null; // IDをリセット
            isDisplayingResult = false; // フラグをリセット

            // UIの状態を更新
            startBtn.disabled = false;
            stopBtn.disabled = true;
            setCategoryRadiosDisabled(false); // カテゴリ選択を有効化
            outputDisplay.innerHTML = '<span class="text-gray-400 text-lg">停止しました</span>'; // 停止時のメッセージ
            outputDisplay.classList.add('text-gray-400', 'text-lg'); // スタイルを戻す
            outputDisplay.classList.add('text-5xl'); // 数字表示のフォントサイズを元に戻す
        });

        // 初期表示
        outputDisplay.innerHTML = '<span class="text-gray-400 text-lg">カテゴリを選択して開始</span>';
    </script>
</body>
</html>
