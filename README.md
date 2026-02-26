<!DOCTYPE html>
<html>
<head>
    <title>Y.G.V.</title>

    <style>
        body {
            margin: 0;
            font-family: Arial;
            background: #87CEFA; /* Light Blue */
            color: black;
            text-align: center;
        }

        h1 {
            font-size: 40px;
            margin-top: 20px;
            color: #003366;
        }

        .uploadBox {
            margin: 20px auto;
            padding: 20px;
            background: white;
            width: 60%;
            border-radius: 15px;
            box-shadow: 0 0 15px #3399ff;
        }

        input, button, select {
            padding: 10px;
            margin: 5px;
            border-radius: 8px;
            border: none;
            outline: none;
        }

        input, select {
            width: 200px;
        }

        button {
            background: #3399ff;
            cursor: pointer;
            font-weight: bold;
            color: white;
        }

        button:hover {
            background: #003366;
        }

        #searchBar {
            width: 300px;
            margin: 20px;
        }

        #videoList {
            display: flex;
            flex-wrap: wrap;
            justify-content: flex-start; /* LEFT SIDE */
            padding-left: 20px;
        }

        .videoCard {
            background: white;
            margin: 15px;
            padding: 15px;
            border-radius: 15px;
            width: 320px;
            box-shadow: 0 0 10px #3399ff;
            transition: 0.3s;
            text-align: left;
        }

        .videoCard:hover {
            transform: scale(1.03);
        }

        video {
            width: 100%;
            border-radius: 10px;
        }

        .deleteBtn {
            background: red;
            color: white;
        }

        .favorite {
            background: gold;
            color: black;
        }

        .sticker {
            display: inline-block;
            padding: 5px 10px;
            margin: 3px;
            border-radius: 10px;
            color: white;
            font-size: 12px;
        }

        .sticker button {
            background: transparent;
            color: white;
            border: none;
            cursor: pointer;
            margin-left: 5px;
        }
    </style>
</head>

<body>

<h1>Y.G.V.</h1>

<div class="uploadBox">
    <input type="file" id="videoUpload" accept="video/*">
    <input type="text" id="videoTitle" placeholder="Enter Title">
    <input type="password" id="videoPassword" placeholder="Password (optional)">
    <button onclick="saveVideo()">Upload</button>
</div>

<input type="text" id="searchBar" placeholder="Search videos..." onkeyup="displayVideos()">

<div id="videoList"></div>

<script>
let db;

let request = indexedDB.open("YGV_DB", 1);

request.onupgradeneeded = function(event) {
    db = event.target.result;
    db.createObjectStore("videos", { keyPath: "id", autoIncrement: true });
};

request.onsuccess = function(event) {
    db = event.target.result;
    displayVideos();
};

function saveVideo() {
    const file = document.getElementById("videoUpload").files[0];
    const title = document.getElementById("videoTitle").value;
    const password = document.getElementById("videoPassword").value;

    if (!file || !title) {
        alert("Select video and title!");
        return;
    }

    let transaction = db.transaction(["videos"], "readwrite");
    let store = transaction.objectStore("videos");

    store.add({
        title: title,
        video: file,
        password: password,
        stickers: [],
        favorite: false
    });

    transaction.oncomplete = function() {
        displayVideos();
    };
}

function deleteVideo(id) {
    let transaction = db.transaction(["videos"], "readwrite");
    let store = transaction.objectStore("videos");
    store.delete(id);

    transaction.oncomplete = function() {
        displayVideos();
    };
}

function toggleFavorite(id) {
    let transaction = db.transaction(["videos"], "readwrite");
    let store = transaction.objectStore("videos");
    let req = store.get(id);

    req.onsuccess = function() {
        let data = req.result;
        data.favorite = !data.favorite;
        store.put(data);
        displayVideos();
    };
}

function addSticker(id) {
    let text = prompt("Enter sticker text:");
    if (!text) return;

    let colors = ["red","blue","green","purple","orange"];
    let randomColor = colors[Math.floor(Math.random()*colors.length)];

    let transaction = db.transaction(["videos"], "readwrite");
    let store = transaction.objectStore("videos");
    let req = store.get(id);

    req.onsuccess = function() {
        let data = req.result;
        data.stickers.push({text:text,color:randomColor});
        store.put(data);
        displayVideos();
    };
}

function removeSticker(id,index){
    let transaction = db.transaction(["videos"], "readwrite");
    let store = transaction.objectStore("videos");
    let req = store.get(id);

    req.onsuccess = function() {
        let data = req.result;
        data.stickers.splice(index,1);
        store.put(data);
        displayVideos();
    };
}

function displayVideos() {
    const search = document.getElementById("searchBar").value.toLowerCase();
    const videoList = document.getElementById("videoList");
    videoList.innerHTML = "";

    let transaction = db.transaction(["videos"], "readonly");
    let store = transaction.objectStore("videos");
    let request = store.openCursor();

    request.onsuccess = function(event) {
        let cursor = event.target.result;
        if (cursor) {

            if (cursor.value.title.toLowerCase().includes(search)) {

                if(cursor.value.password){
                    let input = prompt("Enter password for: " + cursor.value.title);
                    if(input !== cursor.value.password){
                        cursor.continue();
                        return;
                    }
                }

                let videoURL = URL.createObjectURL(cursor.value.video);

                let stickersHTML = "";
                cursor.value.stickers.forEach((s,i)=>{
                    stickersHTML += `
                        <span class="sticker" style="background:${s.color}">
                            ${s.text}
                            <button onclick="removeSticker(${cursor.value.id},${i})">x</button>
                        </span>
                    `;
                });

                videoList.innerHTML += `
                    <div class="videoCard">
                        <h3>${cursor.value.title}</h3>
                        <video controls src="${videoURL}"></video>
                        <div>${stickersHTML}</div>
                        <button onclick="addSticker(${cursor.value.id})">Add Sticker</button>
                        <button class="favorite" onclick="toggleFavorite(${cursor.value.id})">
                            ${cursor.value.favorite ? "★ Favorited" : "☆ Favorite"}
                        </button>
                        <button class="deleteBtn" onclick="deleteVideo(${cursor.value.id})">Delete</button>
                    </div>
                `;
            }

            cursor.continue();
        }
    };
}
</script>

</body>
</html>
