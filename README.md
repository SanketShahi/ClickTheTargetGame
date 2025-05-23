<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Click the Target Game</title>
    <!-- AdMob SDK with App ID -->
    <script async src="https://www.googletagmanager.com/gtag/js?id=ca-app-pub-8181154383865920~1127745801"></script>
    <script src="https://imasdk.googleapis.com/js/sdkloader/ima3.js"></script>
    <style>
        body {
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #f0f0f0;
            font-family: Arial, sans-serif;
        }

        #game-container {
            width: 600px;
            height: 400px;
            background-color: white;
            border: 2px solid #333;
            position: relative;
            overflow: hidden;
        }

        #target {
            width: 50px;
            height: 50px;
            background-color: #ff4444;
            border-radius: 50%;
            position: absolute;
            cursor: pointer;
            transition: all 0.2s ease;
        }

        #score {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 20px;
        }

        #high-score {
            position: absolute;
            top: 40px;
            left: 10px;
            font-size: 20px;
        }

        #timer {
            position: absolute;
            top: 10px;
            right: 10px;
            font-size: 20px;
        }

        #start-button {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 10px 20px;
            font-size: 18px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }

        #start-button:hover {
            background-color: #45a049;
        }

        #reward-button {
            position: absolute;
            top: 60%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 10px 20px;
            font-size: 18px;
            background-color: #ff9800;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            display: none;
        }

        #reward-button:hover {
            background-color: #e68900;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <div id="score">Score: 0</div>
        <div id="high-score">High Score: 0</div>
        <div id="timer">Time: 30</div>
        <div id="target" style="display: none;"></div>
        <button id="start-button">Start Game</button>
        <button id="reward-button">Watch Ad for +30s</button>
    </div>

    <script>
        // Element references
        const target = document.getElementById('target');
        const scoreDisplay = document.getElementById('score');
        const highScoreDisplay = document.getElementById('high-score');
        const timerDisplay = document.getElementById('timer');
        const startButton = document.getElementById('start-button');
        const rewardButton = document.getElementById('reward-button');

        // Game state variables
        let score = 0;
        let highScore = 0;
        let timeLeft = 30;
        let gameActive = false;
        let timerInterval;
        let isRewardRound = false;

        // AdMob rewarded ad setup
        let rewardedAd;

        function initializeAds() {
            rewardedAd = new google.ima.RewardedAd(
                'ca-app-pub-8181154383865920/9253679657', // Ad Unit ID
                'ca-app-pub-8181154383865920~1127745801', // App ID
                { adTagUrl: 'https://pubads.g.doubleclick.net/gampad/ads?...' } // Optional, AdMob provides default
            );
            rewardedAd.load();
        }

        window.onload = initializeAds;

        // Move target to random position
        function moveTarget() {
            if (gameActive) {
                const maxX = 550;
                const maxY = 350;
                const newX = Math.floor(Math.random() * maxX);
                const newY = Math.floor(Math.random() * maxY);
                target.style.left = newX + 'px';
                target.style.top = newY + 'px';
            }
        }

        // Target click handler
        target.addEventListener('click', () => {
            if (gameActive) {
                score++;
                scoreDisplay.textContent = `Score: ${score}`;
                moveTarget();
            }
        });

        // Start game handler
        startButton.addEventListener('click', () => {
            if (!gameActive) {
                gameActive = true;
                startButton.style.display = 'none';
                rewardButton.style.display = 'none';
                target.style.display = 'block';
                score = 0;
                timeLeft = 30;
                isRewardRound = false;
                scoreDisplay.textContent = `Score: ${score}`;
                timerDisplay.textContent = `Time: ${timeLeft}`;
                moveTarget();

                timerInterval = setInterval(() => {
                    timeLeft--;
                    timerDisplay.textContent = `Time: ${timeLeft}`;
                    if (timeLeft <= 0) {
                        clearInterval(timerInterval);
                        gameActive = false;
                        target.style.display = 'none';
                        startButton.style.display = 'block';
                        rewardButton.style.display = 'block';
                        if (!isRewardRound && score > highScore) {
                            highScore = score;
                            highScoreDisplay.textContent = `High Score: ${highScore}`;
                        } else if (isRewardRound && score >= 50 && score > highScore) {
                            highScore = score;
                            highScoreDisplay.textContent = `High Score: ${highScore}`;
                        }
                        alert(`Game Over! Your score: ${score}`);
                    }
                }, 1000);
            }
        });

        // Reward ad handler
        rewardButton.addEventListener('click', () => {
            if (rewardedAd && rewardedAd.isLoaded()) {
                rewardedAd.show();
                rewardedAd.onAdEvent((event) => {
                    if (event.type === google.ima.AdEvent.Type.COMPLETED) {
                        // Ad watched, grant reward
                        timeLeft = 30;
                        timerDisplay.textContent = `Time: ${timeLeft}`;
                        gameActive = true;
                        isRewardRound = true;
                        target.style.display = 'block';
                        rewardButton.style.display = 'none';
                        startButton.style.display = 'none';

                        timerInterval = setInterval(() => {
                            timeLeft--;
                            timerDisplay.textContent = `Time: ${timeLeft}`;
                            if (timeLeft <= 0) {
                                clearInterval(timerInterval);
                                gameActive = false;
                                target.style.display = 'none';
                                startButton.style.display = 'block';
                                rewardButton.style.display = 'block';
                                if (isRewardRound && score >= 50 && score > highScore) {
                                    highScore = score;
                                    highScoreDisplay.textContent = `High Score: ${highScore}`;
                                }
                                alert(`Game Over! Your score: ${score}`);
                            }
                        }, 1000);
                    }
                });
            } else {
                alert("Ad not ready, please try again.");
            }
        });
    </script>
</body>
</html>
