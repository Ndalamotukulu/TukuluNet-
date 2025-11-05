# TukuluNet-
A movie website 
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>TukuluFlix</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #000;
      color: white;
      margin: 0;
      padding: 0;
      text-align: center;
    }
    .auth-container, .upload-form {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }
    input, textarea {
      margin: 8px 0;
      padding: 10px;
      border: none;
      border-radius: 4px;
      width: 250px;
    }
    button {
      margin: 8px 0;
      padding: 10px;
      border: none;
      border-radius: 4px;
      background-color: red;
      color: white;
      cursor: pointer;
      width: 270px;
      font-weight: bold;
    }
    #google-login { background-color: #4285F4; }
    #main-content { display: none; padding: 20px; }
    .movie {
      background: #111;
      margin: 10px auto;
      padding: 10px;
      border-radius: 8px;
      width: 300px;
    }
    img { width: 100%; border-radius: 8px; }
  </style>
</head>
<body>
  <!-- 🔐 Auth Section -->
  <div class="auth-container" id="auth-container">
    <h2>Welcome to TukuluFlix</h2>
    <input type="email" id="email" placeholder="Email" required />
    <input type="password" id="password" placeholder="Password" required />
    <button id="signup">Sign Up</button>
    <button id="login">Log In</button>
    <button id="google-login">Login with Google</button>
    <p id="auth-message"></p>
  </div>

  <!-- 🎬 Main Content -->
  <div id="main-content">
    <h1>TukuluFlix 🎥</h1>
    <button id="logout">Logout</button>
    <h3>Upload a Movie</h3>
    <div class="upload-form">
      <input type="text" id="movie-title" placeholder="Movie Title" required />
      <input type="text" id="movie-year" placeholder="Year" required />
      <textarea id="movie-desc" placeholder="Description" rows="3"></textarea>
      <input type="file" id="movie-image" accept="image/*" />
      <button id="upload-movie">Upload Movie</button>
      <p id="upload-message"></p>
    </div>
    <h2>🎞️ Movie List</h2>
    <div id="movies"></div>
  </div>

  <script type="module">
    // Firebase SDKs
    import { initializeApp } from "https://www.gstatic.com/firebasejs/12.4.0/firebase-app.js";
    import { 
      getAuth, 
      createUserWithEmailAndPassword, 
      signInWithEmailAndPassword, 
      signOut, 
      onAuthStateChanged,
      GoogleAuthProvider,
      signInWithPopup
    } from "https://www.gstatic.com/firebasejs/12.4.0/firebase-auth.js";
    import { 
      getFirestore, collection, addDoc, getDocs, serverTimestamp, query, orderBy 
    } from "https://www.gstatic.com/firebasejs/12.4.0/firebase-firestore.js";
    import { 
      getStorage, ref, uploadBytes, getDownloadURL 
    } from "https://www.gstatic.com/firebasejs/12.4.0/firebase-storage.js";

    // ✅ Firebase Config
    const firebaseConfig = {
      apiKey: "AIzaSyAg6QVwlLj1MJ25eiD1caeF_S21k_gzfXU",
      authDomain: "tukuluflix.firebaseapp.com",
      projectId: "tukuluflix",
      storageBucket: "tukuluflix.firebasestorage.app",
      messagingSenderId: "615822572297",
      appId: "1:615822572297:web:9586daf4a4df2c034bf1d8",
      measurementId: "G-H6GNWLR236"
    };

    // 🔥 Initialize Firebase
    const app = initializeApp(firebaseConfig);
    const auth = getAuth(app);
    const db = getFirestore(app);
    const storage = getStorage(app);
    const provider = new GoogleAuthProvider();

    // Elements
    const authContainer = document.getElementById('auth-container');
    const mainContent = document.getElementById('main-content');
    const signupBtn = document.getElementById('signup');
    const loginBtn = document.getElementById('login');
    const googleBtn = document.getElementById('google-login');
    const logoutBtn = document.getElementById('logout');
    const uploadBtn = document.getElementById('upload-movie');
    const authMsg = document.getElementById('auth-message');
    const uploadMsg = document.getElementById('upload-message');
    const moviesDiv = document.getElementById('movies');
    const emailInput = document.getElementById('email');
    const passwordInput = document.getElementById('password');

    // 🟢 Email Signup
    signupBtn.addEventListener('click', async () => {
      try {
        await createUserWithEmailAndPassword(auth, emailInput.value, passwordInput.value);
        authMsg.textContent = "Signup successful!";
      } catch (error) {
        authMsg.textContent = error.message;
      }
    });

    // 🔵 Email Login
    loginBtn.addEventListener('click', async () => {
      try {
        await signInWithEmailAndPassword(auth, emailInput.value, passwordInput.value);
        authMsg.textContent = "Login successful!";
      } catch (error) {
        authMsg.textContent = error.message;
      }
    });

    // 🟣 Google Login
    googleBtn.addEventListener('click', async () => {
      try {
        await signInWithPopup(auth, provider);
      } catch (error) {
        authMsg.textContent = error.message;
      }
    });

    // 🔴 Logout
    logoutBtn.addEventListener('click', async () => await signOut(auth));

    // 🟠 Upload Movie
    uploadBtn.addEventListener('click', async () => {
      const title = document.getElementById('movie-title').value;
      const year = document.getElementById('movie-year').value;
      const desc = document.getElementById('movie-desc').value;
      const imageFile = document.getElementById('movie-image').files[0];

      if (!title || !year || !imageFile) {
        uploadMsg.textContent = "Please fill in title, year, and choose an image.";
        return;
      }

      try {
        // Upload image to Firebase Storage
        const imageRef = ref(storage, `movies/${Date.now()}_${imageFile.name}`);
        await uploadBytes(imageRef, imageFile);
        const imageURL = await getDownloadURL(imageRef);

        // Save movie to Firestore
        await addDoc(collection(db, "movies"), {
          title,
          year,
          desc,
          imageURL,
          createdAt: serverTimestamp()
        });
        uploadMsg.textContent = "✅ Movie uploaded!";
        loadMovies(); // refresh list
      } catch (err) {
        uploadMsg.textContent = "Error: " + err.message;
      }
    });

    // 👁️ Watch Auth State
    onAuthStateChanged(auth, (user) => {
      if (user) {
        authContainer.style.display = "none";
        mainContent.style.display = "block";
        loadMovies();
      } else {
        authContainer.style.display = "flex";
        mainContent.style.display = "none";
      }
    });

    // 🎞️ Load Movies from Firestore
    async function loadMovies() {
      const q = query(collection(db, "movies"), orderBy("createdAt", "desc"));
      const snapshot = await getDocs(q);
      moviesDiv.innerHTML = "";
      snapshot.forEach((doc) => {
        const m = doc.data();
        const movieEl = document.createElement("div");
        movieEl.className = "movie";
        movieEl.innerHTML = `
          <img src="${m.imageURL}" alt="${m.title}">
          <h3>${m.title}</h3>
          <p>${m.year}</p>
          <p>${m.desc || ""}</p>
        `;
        moviesDiv.appendChild(movieEl);
      });
    }
  </script>
</body>
</html>
