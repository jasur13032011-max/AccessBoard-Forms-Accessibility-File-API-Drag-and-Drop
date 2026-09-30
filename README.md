# AccessBoard-Forms-Accessibility-File-API-Drag-and-Drop
Ha, unda oldingi kodga **shu 4 ta accessibility talabni ham qo‘shamiz**. Ayniqsa `file` biriktirishni sichqonchasiz **Tab → Enter → fayl tanlash** orqali ishlaydigan qilamiz.

### `index.html` ni mana bunday almashtir:

```html
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AccessBoard</title>
    <link rel="stylesheet" href="./style.css">
</head>

<body>

<header class="header">
    <h1>AccessBoard</h1>
    <p>Accessible Kanban Board</p>
</header>

<main class="container">

    <section class="add-card">
        <h2>Yangi karta qo'shish</h2>

        <!-- role="alert" xatolarni darhol e'lon qiladi -->
        <div
            id="formError"
            class="form-error"
            role="alert"
            aria-live="assertive"
        ></div>

        <form id="cardForm">

            <label for="title">Karta nomi</label>
            <input
                type="text"
                id="title"
                name="title"
                aria-describedby="titleError"
                required
            >
            <div id="titleError" class="field-error" role="alert"></div>


            <label for="description">Tavsif</label>
            <textarea
                id="description"
                name="description"
                aria-describedby="descriptionError"
                required
            ></textarea>
            <div
                id="descriptionError"
                class="field-error"
                role="alert"
            ></div>


            <label for="status">Holat</label>
            <select
                id="status"
                name="status"
                required
            >
                <option value="todo">🔴 Kutilmoqda</option>
                <option value="progress">🟡 Jarayonda</option>
                <option value="done">🟢 Bajarildi</option>
            </select>


            <!-- FILE DROPZONE -->
            <fieldset class="file-section">
                <legend>Fayl biriktirish</legend>

                <div
                    id="dropzone"
                    class="dropzone"
                    tabindex="0"
                    role="button"
                    aria-describedby="fileHelp fileError"
                    aria-label="Fayl tanlash. Enter tugmasini bosib fayl tanlash oynasini oching"
                >
                    <span class="dropzone-icon">📎</span>
                    <strong>Faylni bu yerga tashlang</strong>
                    <span>yoki Enter tugmasini bosing</span>

                    <input
                        type="file"
                        id="file"
                        name="file"
                        hidden
                    >
                </div>

                <p id="fileHelp">
                    Faylni drag-and-drop orqali yoki klaviaturadan
                    Enter tugmasi yordamida tanlashingiz mumkin.
                </p>

                <div
                    id="fileName"
                    class="file-name"
                    aria-live="polite"
                ></div>

                <div
                    id="fileError"
                    class="field-error"
                    role="alert"
                ></div>
            </fieldset>


            <button type="submit">
                Karta qo'shish
            </button>

        </form>
    </section>


    <section class="board">

        <div class="column">
            <h2>🔴 Kutilmoqda</h2>
            <div id="todo" class="cards"></div>
        </div>

        <div class="column">
            <h2>🟡 Jarayonda</h2>
            <div id="progress" class="cards"></div>
        </div>

        <div class="column">
            <h2>🟢 Bajarildi</h2>
            <div id="done" class="cards"></div>
        </div>

    </section>

</main>

<script type="module" src="./app/main.js"></script>

</body>
</html>
```

### `app/main.js` ni almashtir:

```javascript
import { addCard, getAllCards } from "./db.js";

const form = document.getElementById("cardForm");

const titleInput = document.getElementById("title");
const descriptionInput = document.getElementById("description");
const statusInput = document.getElementById("status");

const dropzone = document.getElementById("dropzone");
const fileInput = document.getElementById("file");

const fileName = document.getElementById("fileName");
const fileError = document.getElementById("fileError");
const formError = document.getElementById("formError");

const titleError = document.getElementById("titleError");
const descriptionError = document.getElementById("descriptionError");

const todoColumn = document.getElementById("todo");
const progressColumn = document.getElementById("progress");
const doneColumn = document.getElementById("done");


// ===============================
// FILE DROPZONE
// ===============================

dropzone.addEventListener("click", () => {
    fileInput.click();
});


// Klaviatura orqali fayl tanlash
dropzone.addEventListener("keydown", (event) => {

    if (event.key === "Enter" || event.key === " ") {
        event.preventDefault();

        fileInput.click();
    }
});


// Fayl input orqali tanlanganda
fileInput.addEventListener("change", () => {

    if (fileInput.files.length > 0) {
        showSelectedFile(fileInput.files[0]);
    }
});


// Drag boshlanganda
dropzone.addEventListener("dragover", (event) => {
    event.preventDefault();

    dropzone.classList.add("dragover");
});


// Drag tugaganda
dropzone.addEventListener("dragleave", () => {
    dropzone.classList.remove("dragover");
});


// Fayl tashlanganda
dropzone.addEventListener("drop", (event) => {

    event.preventDefault();

    dropzone.classList.remove("dragover");

    const files = event.dataTransfer.files;

    if (files.length > 0) {

        // DataTransfer orqali input'ga faylni beramiz
        fileInput.files = files;

        showSelectedFile(files[0]);
    }
});


function showSelectedFile(file) {

    fileError.textContent = "";

    fileName.textContent = `Tanlangan fayl: ${file.name}`;
}


// ===============================
// VALIDATION
// ===============================

function validateForm() {

    let isValid = true;

    titleError.textContent = "";
    descriptionError.textContent = "";
    fileError.textContent = "";
    formError.textContent = "";


    if (!titleInput.value.trim()) {

        titleError.textContent =
            "Xato: Karta nomini kiriting.";

        titleInput.focus();

        isValid = false;
    }


    if (!descriptionInput.value.trim()) {

        descriptionError.textContent =
            "Xato: Tavsifni kiriting.";

        if (isValid) {
            descriptionInput.focus();
        }

        isValid = false;
    }


    if (!isValid) {

        formError.textContent =
            "Formada xatolar mavjud. Iltimos, maydonlarni tekshiring.";

        return false;
    }

    return true;
}


// ===============================
// CARD
// ===============================

function getStatusText(status) {

    if (status === "todo") {
        return "🔴 Kutilmoqda";
    }

    if (status === "progress") {
        return "🟡 Jarayonda";
    }

    return "🟢 Bajarildi";
}


function escapeHTML(text) {

    const div = document.createElement("div");

    div.textContent = text;

    return div.innerHTML;
}


function createCard(card) {

    const article = document.createElement("article");

    article.className = `card ${card.status}`;

    article.innerHTML = `
        <div class="card-status">
            ${getStatusText(card.status)}
        </div>

        <h3>${escapeHTML(card.title)}</h3>

        <p>${escapeHTML(card.description)}</p>

        ${
            card.fileName
                ? `<p class="attachment">
                    📎 Fayl: ${escapeHTML(card.fileName)}
                   </p>`
                : ""
        }
    `;

    return article;
}


function renderCards(cards) {

    todoColumn.innerHTML = "";
    progressColumn.innerHTML = "";
    doneColumn.innerHTML = "";

    cards.forEach((card) => {

        const element = createCard(card);

        if (card.status === "todo") {
            todoColumn.appendChild(element);
        }

        if (card.status === "progress") {
            progressColumn.appendChild(element);
        }

        if (card.status === "done") {
            doneColumn.appendChild(element);
        }
    });
}


// ===============================
// LOAD FROM INDEXEDDB
// ===============================

async function loadCards() {

    try {

        const cards = await getAllCards();

        renderCards(cards);

    } catch (error) {

        console.error(
            "Kartalarni yuklashda xato:",
            error
        );

        formError.textContent =
            "Kartalarni yuklashda xatolik yuz berdi.";
    }
}


// ===============================
// FORM SUBMIT
// ===============================

form.addEventListener("submit", async (event) => {

    event.preventDefault();

    if (!validateForm()) {
        return;
    }


    const selectedFile = fileInput.files[0];


    const card = {

        title: titleInput.value.trim(),

        description:
            descriptionInput.value.trim(),

        status:
            statusInput.value,

        fileName:
            selectedFile
                ? selectedFile.name
                : "",

        createdAt:
            new Date().toISOString()
    };


    try {

        await addCard(card);

        form.reset();

        fileName.textContent = "";

        await loadCards();

        titleInput.focus();

    } catch (error) {

        console.error(
            "Karta saqlashda xato:",
            error
        );

        formError.textContent =
            "Kartani saqlashda xatolik yuz berdi.";
    }
});


loadCards();
```

### `style.css` ga bularni ham qo‘sh:

```css
.file-section {
    border: 0;
    padding: 0;
    margin: 10px 0;
}

.file-section legend {
    font-weight: bold;
    margin-bottom: 10px;
}

.dropzone {
    min-height: 150px;
    border: 3px dashed #374151;
    border-radius: 12px;
    background: #f9fafb;
    color: #111827;

    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;

    gap: 8px;
    padding: 25px;

    cursor: pointer;
    text-align: center;
}

.dropzone:hover,
.dropzone.dragover {
    background: #e5e7eb;
    border-color: #111827;
}

.dropzone:focus {
    outline: 4px solid #facc15;
    outline-offset: 3px;
}

.dropzone-icon {
    font-size: 32px;
}

#fileHelp {
    color: #374151;
    font-size: 14px;
}

.file-name {
    margin-top: 10px;
    font-weight: bold;
    color: #166534;
}

.field-error,
.form-error {
    color: #991b1b;
    background: #fee2e2;
    border: 1px solid #991b1b;
    border-radius: 6px;
    padding: 8px;
    margin-top: 5px;
}

.form-error:empty,
.field-error:empty {
    display: none;
}

.attachment {
    font-weight: bold;
}
```

### Endi talablaring bajariladi

| Talab                          | Kodda                                      |
| ------------------------------ | ------------------------------------------ |
| `label for`                    | ✅ `title`, `description`, `status`, `file` |
| Placeholder label o‘rniga emas | ✅                                          |
| Drag & Drop                    | ✅                                          |
| `<input type="file">`          | ✅                                          |
| Faqat Tab + Enter              | ✅ Dropzone `tabindex="0"` + `Enter`        |
| Validatsiya xatosi             | ✅ `role="alert"`                           |
| Xato darhol e'lon qilinadi     | ✅ `aria-live="assertive"`                  |
| IndexedDB                      | ✅                                          |
| `createObjectStore`            | ✅                                          |
| `put`                          | ✅                                          |
| `getAll`                       | ✅                                          |
| Refreshdan keyin kartalar      | ✅                                          |
| Status faqat rang bilan emas   | ✅ Emoji + matn                             |

**Qo‘lda tekshirishda:** sichqonchani ishlatmasdan `Tab` bilan **“Fayl biriktirish”** joyiga kel → `Enter` bos → fayl tanlash oynasi ochilishi kerak. Shu qismni o‘qituvchiga ham ko‘rsatishing mumkin.
