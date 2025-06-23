balimoen-sound-upload/
├── server.js
├── package.json
├── uploads/         ← lege map (gewoon aanmaken)
└── public/
    └── index.html
    const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');
const app = express();
const PORT = process.env.PORT || 3000;

// Zorg dat de uploads-map bestaat
if (!fs.existsSync('./uploads')) {
  fs.mkdirSync('./uploads');
}

// File upload config
const storage = multer.diskStorage({
  destination: './uploads/',
  filename: (req, file, cb) => {
    cb(null, Date.now() + '-' + file.originalname);
  }
});
const upload = multer({ storage });

app.use(express.static('public'));
app.use('/uploads', express.static('uploads'));
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.post('/upload', upload.single('soundFile'), (req, res) => {
  const coinCategory = req.body.coinCategory || 'Onbekend';
  res.json({
    status: 'success',
    fileName: req.file.filename,
    originalName: req.file.originalname,
    coinCategory: coinCategory,
    url: `/uploads/${req.file.filename}`
  });
});

app.listen(PORT, () => {
  console.log(`Server draait op http://localhost:${PORT}`);
});
{
  "name": "balimoen-sound-upload",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "multer": "^1.4.5"
  }
  <!DOCTYPE html>
<html lang="nl">
<head>
  <meta charset="UTF-8">
  <title>Balimoen Sound Upload</title>
  <style>
    body {
      background-color: #0a0a40;
      color: white;
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 2rem;
    }
    .container {
      max-width: 500px;
      margin: auto;
      background: #1a1a60;
      padding: 20px;
      border-radius: 10px;
    }
    input, select, button {
      width: 100%;
      padding: 10px;
      margin: 10px 0;
      border: none;
      border-radius: 5px;
    }
    button {
      background-color: #00bfff;
      color: white;
      cursor: pointer;
    }
    audio {
      width: 100%;
      margin-top: 10px;
    }
    .result {
      margin-top: 20px;
      text-align: left;
    }
  </style>
</head>
<body>
  <h1>🎧 Balimoen Upload Center</h1>
  <p>“Laat je geluid horen!”</p>

  <div class="container">
    <form id="uploadForm">
      <label for="soundFile">Selecteer een geluid:</label>
      <input type="file" name="soundFile" id="soundFile" accept="audio/*" required>

      <label for="coinCategory">Kies een coin-categorie:</label>
      <select name="coinCategory" id="coinCategory" required>
        <option value="1-19 coins">1-19 coins</option>
        <option value="20-199 coins">20-199 coins</option>
        <option value="200-499 coins">200-499 coins</option>
        <option value="500-1000 coins">500-1000 coins</option>
        <option value="Galaxy">Galaxy</option>
        <option value="Interstellar">Interstellar</option>
        <option value="Universe">Universe</option>
        <option value="Tikkie">Tikkie</option>
        <option value="Abonnement">Abonnement</option>
      </select>

      <button type="submit">Uploaden</button>
    </form>

    <div class="result" id="result"></div>
  </div>

  <script>
    const uploadForm = document.getElementById('uploadForm');
    const resultDiv = document.getElementById('result');

    uploadForm.addEventListener('submit', async (e) => {
      e.preventDefault();
      const formData = new FormData(uploadForm);

      const res = await fetch('/upload', {
        method: 'POST',
        body: formData
      });

      const data = await res.json();
      if (data.status === 'success') {
        resultDiv.innerHTML = `
          <p><strong>Upload gelukt!</strong></p>
          <p><strong>Bestand:</strong> ${data.originalName}</p>
          <p><strong>Categorie:</strong> ${data.coinCategory}</p>
          <audio controls src="${data.url}"></audio>
          <p><a href="${data.url}" download>Klik hier om te downloaden</a></p>
        `;
      } else {
        resultDiv.textContent = "Er ging iets mis.";
      }
    });
  </script>
</body>
</html>
