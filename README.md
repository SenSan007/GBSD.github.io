<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Игра 21</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #2c3e50;
            color: white;
            text-align: center;
        }
        #game {
            margin-top: 20px;
        }
        button {
            margin: 5px;
            padding: 10px;
            font-size: 16px;
        }
        .card {
            font-size: 24px;
            margin: 5px;
        }
        .red {
            color: red;
        }
    </style>
</head>
<body>
    <h1>Ебаный должник-лудик </h1>
    <div id="game">
	<p><b>БАНК</b><p>
        <p>Задолженность: <span id="debt">0</span></p>
		<p>Сумма кредита: <input type="number" id="loanAmount" value="100" min="100" max="500000"></p>
		<button onclick="takeLoan()">Взять кредит</button>
        <button onclick="repayLoan()">Вернуть кредит</button>
        <p>Ставка: <input type="number" id="bet" value="100" min="1" max="1000"></p>
		<p>Ваши рубли: <span id="money">1000</span></p>
        <button onclick="startGame()">Начать игру</button>
        <button onclick="hit()" disabled>Взять карту</button>
        <button onclick="stand()" disabled>Стоп</button>
		<div id="playerCards" class="cards"></div>
        <div id="dealerCards" class="cards"></div>
        <p id="result"></p>
    </div>

    <script>
        let money = 1000;
        let debt = 0;
        let bet = 100;
        let playerTotal = 0;
        let dealerTotal = 0;
        let playerCards = [];
        let dealerCards = [];
        let gameActive = false;
        const interestRate = 0.03; // 3%

        const suits = ['♥️', '♦️', '♣️', '♠️'];
        const values = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A'];

        function getRandomCard() {
            const value = Math.floor(Math.random() * values.length);
            const suit = Math.floor(Math.random() * suits.length);
            return { value: values[value], suit: suits[suit] };
        }

        function cardValue(card) {
            if (card.value === 'A') return 11; 
            if (['K', 'Q', 'J'].includes(card.value)) return 10;
            return parseInt(card.value);
        }

        function takeLoan() {
            const loanAmount = parseInt(document.getElementById('loanAmount').value);
            if (loanAmount >= 100 && loanAmount <= 500000) {
                debt += Math.round(loanAmount);
                money += Math.round(loanAmount);
                alert(`Вы взяли кредит на ${loanAmount} рублей.`);
                updateCardsDisplay();
            } else {
                alert("Сумма кредита должна быть от 100 до 500000 рублей.");
            }
        }

        function repayLoan() {
            if (debt === 0) {
                alert("У вас нет задолженности.");
                return;
            }

            const repaymentAmount = Math.min(debt, money);
            debt -= repaymentAmount;
            money -= repaymentAmount;

            alert(`Вы вернули ${repaymentAmount} рублей кредита.`);
            updateCardsDisplay();
        }

        function startGame() {
            if (money <= 0 && debt === 0) {
                alert("У вас недостаточно средств для начала игры.");
                return;
            }

            playerCards = [];
            dealerCards = [];
            playerTotal = 0;
            dealerTotal = 0;
            gameActive = true;

            bet = parseInt(document.getElementById('bet').value);
            if (bet > money + debt) {
                alert("Недостаточно средств для ставки!");
                return;
            }

            updateCardsDisplay();
            toggleButtons(true);
            document.getElementById('result').innerText = '';
        }

        function hit() {
            if (!gameActive) return;

            // Начисление процентов на задолженность перед взятием карты
            if (debt > 0) {
                debt = Math.round(debt * (1 + interestRate));
            }

            const newCard = getRandomCard();
            playerCards.push(newCard);
            playerTotal += cardValue(newCard);
            updateCardsDisplay();

            if (playerTotal > 21) {
                document.getElementById('result').innerText += "\nВы проиграли!";
                updateMoneyAfterRound(false);
                gameActive = false;
                toggleButtons(false);
            }
        }

        function stand() {
            if (!gameActive) return;

            while (dealerTotal < 17) {
                const newCard = getRandomCard();
                dealerCards.push(newCard);
                dealerTotal += cardValue(newCard);
            }

            updateCardsDisplay();
            checkWinner();
        }

        function updateCardsDisplay() {
            document.getElementById('playerCards').innerHTML = 'Ваши карты: ' + playerCards.map(card => getCardHTML(card)).join(' ');
            document.getElementById('dealerCards').innerHTML = 'Карты дилера: ' + dealerCards.map(card => getCardHTML(card)).join(' ');
            document.getElementById('result').innerText = `Ваши очки: ${playerTotal}, Очки дилера: ${dealerTotal}`;
            document.getElementById('money').innerText = Math.round(money);
            document.getElementById('debt').innerText = Math.round(debt);
        }

        function getCardHTML(card) {
            const colorClass = (card.suit === '♥️' || card.suit === '♦️') ? 'red' : '';
            return `<span class="card ${colorClass}">${card.value}${card.suit}</span>`;
        }

        function checkWinner() {
            if (dealerTotal > 21) {
                document.getElementById('result').innerText += "\nВы выиграли!";
                updateMoneyAfterRound(true);
            } else if (playerTotal > dealerTotal) {
                document.getElementById('result').innerText += "\nВы выиграли!";
                updateMoneyAfterRound(true);
            } else if (playerTotal < dealerTotal) {
                document.getElementById('result').innerText += "\nВы проиграли!";
                updateMoneyAfterRound(false);
            } else {
                document.getElementById('result').innerText += "\nНичья!";
            }

            gameActive = false;
            toggleButtons(false);
        }

        function updateMoneyAfterRound(win) {
            if (win) {
                money += Math.round(bet);
            } else {
                if (money >= bet) {
                    money -= Math.round(bet);
                } else {
                    debt += Math.round(bet);
                }
            }
            document.getElementById('money').innerText = Math.round(money);
        }

        function toggleButtons(active) {
            document.querySelector('button[onclick="hit()"]').disabled = !active;
            document.querySelector('button[onclick="stand()"]').disabled = !active;
        }

        function resetGame() {
            money = 1000;
            debt = 0;
            document.getElementById('money').innerText = Math.round(money);
            document.getElementById('debt').innerText = Math.round(debt);
            document.getElementById('result').innerText = '';
            document.getElementById('playerCards').innerHTML = '';
            document.getElementById('dealerCards').innerHTML = '';
            document.getElementById('loanAmount').value = 100; // Сбросить значение кредита
        }
    </script>
</body>
</html>
