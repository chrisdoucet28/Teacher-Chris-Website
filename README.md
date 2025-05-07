# Teacher-Chris-Website

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Teacher Chris - Upload Games</title>
  <style>
    body { font-family: sans-serif; padding: 20px; background: #f0f0f5; }
    h1 { text-align: center; }
    form, .game-entry { background: white; padding: 20px; margin: 20px auto; border-radius: 10px; max-width: 600px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
    input, textarea, button { width: 100%; margin-top: 10px; padding: 8px; }
    .game-entry { border-left: 5px solid #4caf50; }
    .game-entry h3 { margin-top: 0; }
  </style>
</head>
<body>
  <h1>Teacher Chris - 5th & 6th Grade Games</h1>


  <form id="uploadForm">
    <input type="text" id="title" placeholder="Game Title" required />
    <input type="number" id="printQty" placeholder="Recommended print quantity (for class of 25)" required />
    <textarea id="notes" placeholder="Optional notes..."></textarea>
    <label>PPT File:</label>
    <input type="file" id="pptFile" accept=".ppt,.pptx" required />
    <label>Printable File (PDF/DOC):</label>
    <input type="file" id="printFile" accept=".pdf,.doc,.docx" required />
    <button type="submit">Upload Game</button>
  </form>


  <div id="gameList"></div>


  <!-- Firebase JS SDK -->
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-storage-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-firestore-compat.js"></script>


  <script>
    const firebaseConfig = {
      apiKey: "AIzaSyD9KAz_Fx0BkPHTzeXc22iqTUX1pkmgWlM",
      authDomain: "teacherchris-4b6ea.firebaseapp.com",
      projectId: "teacherchris-4b6ea",
      storageBucket: "teacherchris-4b6ea.appspot.com",
      messagingSenderId: "120957541253",
      appId: "1:120957541253:web:5b3f94672c950d8a6e82ec",
      measurementId: "G-X7MY2DFPDW"
    };


    firebase.initializeApp(firebaseConfig);
    const storage = firebase.storage();
    const db = firebase.firestore();


    const form = document.getElementById("uploadForm");
    const gameList = document.getElementById("gameList");


    form.addEventListener("submit", async (e) => {
      e.preventDefault();


      const title = document.getElementById("title").value;
      const printQty = document.getElementById("printQty").value;
      const notes = document.getElementById("notes").value;
      const pptFile = document.getElementById("pptFile").files[0];
      const printFile = document.getElementById("printFile").files[0];


      const pptRef = storage.ref().child(`games/${title}/${pptFile.name}`);
      const printRef = storage.ref().child(`games/${title}/${printFile.name}`);


      const pptSnap = await pptRef.put(pptFile);
      const printSnap = await printRef.put(printFile);


      const pptURL = await pptSnap.ref.getDownloadURL();
      const printURL = await printSnap.ref.getDownloadURL();


      await db.collection("games").add({
        title, printQty, notes, pptURL, printURL,
        timestamp: firebase.firestore.FieldValue.serverTimestamp()
      });


      form.reset();
      loadGames();
    });


    async function loadGames() {
      gameList.innerHTML = "";
      const snapshot = await db.collection("games").orderBy("timestamp", "desc").get();
      snapshot.forEach(doc => {
        const game = doc.data();
        const div = document.createElement("div");
        div.className = "game-entry";
        div.innerHTML = `
          <h3>${game.title}</h3>
          <p><strong>PPT:</strong> <a href="${game.pptURL}" target="_blank">Download</a></p>
          <p><strong>Print File:</strong> <a href="${game.printURL}" target="_blank">Download</a></p>
          <p><strong>Recommended Copies:</strong> ${game.printQty}</p>
          <p>${game.notes || ""}</p>
        `;
        gameList.appendChild(div);
      });
    }


    loadGames();
  </script>
</body>
</html>
