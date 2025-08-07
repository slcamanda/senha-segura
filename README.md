# senha-segura<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Gerador de Senhas Seguras</title>
  <link rel="stylesheet" href="styles.css" />
</head>
<body>
  <main>
    <h1>Gerador de Senhas Seguras com Entropia</h1>
    <div class="container">
      <label for="length">Tamanho da senha:</label>
      <input type="number" id="length" min="4" max="64" value="12" />

      <button id="generate">Gerar Senha</button>

      <div id="password-container" class="hidden">
        <p>Senha gerada:</p>
        <input type="text" id="password" readonly />
      </div>

      <div id="strength-container" class="hidden">
        <p>Força da senha:</p>
        <div id="strength-bar"></div>
        <p id="strength-text"></p>
      </div>

      <div id="time-container" class="hidden">
        <p>Tempo estimado para quebrar a senha:</p>
        <p id="break-time"></p>
      </div>
    </div>
  </main>
  <script src="script.js"></script>
</body>
</html>
body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: #1e1e2f;
  color: #f0f0f5;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  height: 100vh;
  margin: 0;
  padding: 2rem;
}

main {
  background: #292945;
  border-radius: 12px;
  padding: 2rem;
  width: 400px;
  box-shadow: 0 0 15px #5252f5aa;
}

h1 {
  text-align: center;
  margin-bottom: 1.5rem;
}

.container {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

input[type="number"] {
  width: 100%;
  padding: 0.5rem;
  font-size: 1rem;
  border-radius: 8px;
  border: none;
}

button {
  padding: 0.75rem;
  font-size: 1.1rem;
  background-color: #5252f5;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #4040d1;
}

#password {
  width: 100%;
  font-size: 1.2rem;
  padding: 0.5rem;
  border-radius: 8px;
  border: none;
  background-color: #3b3b66;
  color: #eee;
}

#strength-bar {
  height: 20px;
  border-radius: 10px;
  background-color: #444466;
  margin-top: 0.5rem;
  overflow: hidden;
  position: relative;
}

#strength-bar::after {
  content: '';
  display: block;
  height: 100%;
  width: 0%;
  background-color: #4caf50;
  transition: width 0.5s ease;
  border-radius: 10px;
}

#strength-text {
  margin-top: 0.5rem;
  font-weight: bold;
}

.hidden {
  display: none;
}
const lengthInput = document.getElementById('length');
const generateBtn = document.getElementById('generate');
const passwordInput = document.getElementById('password');
const passwordContainer = document.getElementById('password-container');
const strengthBar = document.getElementById('strength-bar');
const strengthText = document.getElementById('strength-text');
const strengthContainer = document.getElementById('strength-container');
const timeContainer = document.getElementById('time-container');
const breakTimeText = document.getElementById('break-time');

const alphabet = 
  'abcdefghijklmnopqrstuvwxyz' +
  'ABCDEFGHIJKLMNOPQRSTUVWXYZ' +
  '0123456789' +
  '!@#$%^&*()_+-=[]{}|;:,.<>?';

function generatePassword(length) {
  let pass = '';
  for (let i = 0; i < length; i++) {
    pass += alphabet.charAt(Math.floor(Math.random() * alphabet.length));
  }
  return pass;
}

function calculateEntropy(length, alphabetSize) {
  // Entropia = tamanho * log2(tamanho do alfabeto)
  return length * Math.log2(alphabetSize);
}

function calculateCrackTime(entropyBits) {
  // Suposição: 10^10 tentativas por segundo (computador poderoso)
  // Total de tentativas = 2^entropia
  const attemptsPerSecond = 1e10;
  const totalAttempts = Math.pow(2, entropyBits);
  const seconds = totalAttempts / attemptsPerSecond;

  if (seconds < 60) return `${Math.floor(seconds)} segundos`;
  const minutes = seconds / 60;
  if (minutes < 60) return `${Math.floor(minutes)} minutos`;
  const hours = minutes / 60;
  if (hours < 24) return `${Math.floor(hours)} horas`;
  const days = hours / 24;
  if (days < 365) return `${Math.floor(days)} dias`;
  const years = days / 365;
  if (years < 1000) return `${Math.floor(years)} anos`;
  return `mais de 1000 anos`;
}

function updateStrengthBar(entropy) {
  const maxEntropy = 128; // Define um limite máximo para a barra
  const percentage = Math.min((entropy / maxEntropy) * 100, 100);
  strengthBar.style.setProperty('--width', `${percentage}%`);
  strengthBar.querySelector('::after');
  strengthBar.style.setProperty('--width', `${percentage}%`);
  strengthBar.style.background = `linear-gradient(90deg, #4caf50 ${percentage}%, #444466 ${percentage}%)`;
  
  if (percentage < 40) {
    strengthText.textContent = 'Senha fraca';
    strengthBar.style.backgroundColor = '#f44336'; // vermelho
  } else if (percentage < 70) {
    strengthText.textContent = 'Senha média';
    strengthBar.style.backgroundColor = '#ff9800'; // laranja
  } else {
    strengthText.textContent = 'Senha forte';
    strengthBar.style.backgroundColor = '#4caf50'; // verde
  }
}

generateBtn.addEventListener('click', () => {
  const length = parseInt(lengthInput.value);
  if (isNaN(length) || length < 4 || length > 64) {
    alert('Digite um tamanho válido entre 4 e 64.');
    return;
  }
  const password = generatePassword(length);
  passwordInput.value = password;
  passwordContainer.classList.remove('hidden');

  const entropy = calculateEntropy(length, alphabet.length);
  updateStrengthBar(entropy);
  strengthContainer.classList.remove('hidden');

  const crackTime = calculateCrackTime(entropy);
  breakTimeText.textContent = crackTime;
  timeContainer.classList.remove('hidden');
});
