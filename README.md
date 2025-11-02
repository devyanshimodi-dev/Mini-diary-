# Mini-diary-
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Mini Diary</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header>
    <h1>Mini Diary</h1>
    <p class="subtitle">Write. Save. Remember. ✨</p>
  </header>

  <main>
    <section class="editor">
      <input id="entry-title" placeholder="Title (optional)" />
      <textarea id="entry-content" placeholder="Write your entry here..." rows="8"></textarea>

      <div class="controls">
        <button id="save-btn">Save Entry</button>
        <button id="export-btn">Export Selected as .md</button>
        <button id="clear-btn">Delete All (clear local storage)</button>
      </div>
    </section>

    <section class="list">
      <h2>Your Entries</h2>
      <ul id="entries-list"></ul>
    </section>
  </main>

  <footer>
    <small>Private in your browser • Works offline • Tip: export entries to commit on GitHub</small>
  </footer>

  <template id="entry-template">
    <li class="entry">
      <div class="meta">
        <strong class="title"></strong>
        <time class="date"></time>
      </div>
      <p class="preview"></p>
      <div class="entry-actions">
        <button class="open-btn">Open</button>
        <button class="edit-btn">Edit</button>
        <button class="delete-btn">Delete</button>
        <button class="export-btn">Export</button>
      </div>
    </li>
  </template>

  <script src="script.js"></script>
</body>
</html>
:root{
  --bg:#0f0f12;
  --card:#111317;
  --muted:#9aa0a6;
  --accent:#ffd369;
  --text:#e8eef6;
  --gap:14px;
  font-family: "Segoe UI", Roboto, Arial, sans-serif;
}
*{box-sizing:border-box}
body{
  margin:0;
  padding:24px;
  background:linear-gradient(180deg,#0b0b0d, #0f1114);
  color:var(--text);
  min-height:100vh;
}
header{ text-align:center; margin-bottom:18px; }
h1{ margin:0; font-size:28px; letter-spacing:0.3px; }
.subtitle{ margin:6px 0 0; color:var(--muted); font-size:13px }

main{
  display:grid;
  grid-template-columns: 1fr 380px;
  gap:20px;
  align-items:start;
  max-width:1100px;
  margin: 0 auto;
}
.editor{
  background:var(--card);
  padding:16px;
  border-radius:12px;
  box-shadow: 0 6px 18px rgba(0,0,0,0.6);
}
.editor input, .editor textarea{
  width:100%;
  padding:10px;
  margin-bottom:10px;
  background:#0b0b0d;
  color:var(--text);
  border:1px solid #222428;
  border-radius:8px;
  outline:none;
}
.controls{ display:flex; gap:8px; flex-wrap:wrap }
.controls button{
  background:transparent;
  border:1px solid #2b2f33;
  padding:8px 12px;
  border-radius:8px;
  color:var(--text);
  cursor:pointer;
}
.controls button#save-btn{
  background:var(--accent);
  color:#111;
  border:none;
}
.controls button:hover{ transform:translateY(-1px) }

.list{
  background:var(--card);
  padding:12px;
  border-radius:12px;
  height:fit-content;
  max-height:72vh;
  overflow:auto;
}
.list h2{ margin:6px 0 12px; font-size:16px; color:var(--muted) }

.entry{
  padding:10px;
  border-radius:8px;
  margin-bottom:10px;
  background:linear-gradient(180deg,#0b0b0d,#0f1418);
  border:1px solid #202428;
}
.meta{ display:flex; justify-content:space-between; gap:10px; align-items:center }
.title{ color:var(--accent); }
.date{ color:var(--muted); font-size:12px }
.preview{ color:var(--text); margin:8px 0 10px; white-space:pre-wrap; max-height:3.6em; overflow:hidden; text-overflow:ellipsis; }
.entry-actions{ display:flex; gap:6px; flex-wrap:wrap }
.entry-actions button{ background:transparent; border:1px solid #2b2f33; padding:6px 8px; border-radius:6px; color:var(--text); cursor:pointer; font-size:13px }

footer{ text-align:center; margin-top:18px; color:var(--muted); font-size:13px }
@media(max-width:880px){
  main{ grid-template-columns:1fr; }
}
// Simple diary using localStorage
const LS_KEY = 'mini-diary-entries-v1';

function uid() { return Date.now().toString(36) + Math.random().toString(36).slice(2,8); }

function loadEntries() {
  try {
    const raw = localStorage.getItem(LS_KEY);
    return raw ? JSON.parse(raw) : [];
  } catch(e) { return []; }
}

function saveEntries(entries) {
  localStorage.setItem(LS_KEY, JSON.stringify(entries));
}

function renderList() {
  const list = document.getElementById('entries-list');
  list.innerHTML = '';
  const entries = loadEntries().sort((a,b)=> b.date - a.date);
  const tpl = document.getElementById('entry-template');
  entries.forEach(entry => {
    const node = tpl.content.cloneNode(true);
    const li = node.querySelector('li');
    li.dataset.id = entry.id;
    node.querySelector('.title').textContent = entry.title || '(No title)';
    node.querySelector('.date').textContent = new Date(entry.date).toLocaleString();
    node.querySelector('.preview').textContent = entry.content.slice(0,220);
    // buttons
    node.querySelector('.open-btn').addEventListener('click', ()=> openEntry(entry.id));
    node.querySelector('.edit-btn').addEventListener('click', ()=> editEntry(entry.id));
    node.querySelector('.delete-btn').addEventListener('click', ()=> { deleteEntry(entry.id); });
    node.querySelector('.export-btn').addEventListener('click', ()=> exportEntry(entry.id));
    list.appendChild(node);
  });
  if(entries.length === 0){
    list.innerHTML = '<p style="color:var(--muted)">No entries yet — write something lovely 💛</p>';
  }
}

function addEntry(title, content) {
  const entries = loadEntries();
  entries.push({ id: uid(), title: title || '', content, date: Date.now() });
  saveEntries(entries);
  renderList();
}

function editEntry(id) {
  const entries = loadEntries();
  const e = entries.find(x=>x.id===id);
  if(!e) return;
  document.getElementById('entry-title').value = e.title;
  document.getElementById('entry-content').value = e.content;
  // When saving, replace instead of adding
  document.getElementById('save-btn').dataset.editing = id;
  document.getElementById('save-btn').textContent = 'Save Changes';
}

function openEntry(id) {
  const e = loadEntries().find(x=>x.id===id);
  if(!e) return;
  alert(`Title: ${e.title || '(No title)'}\nDate: ${new Date(e.date).toLocaleString()}\n\n${e.content}`);
}

function deleteEntry(id) {
  if(!confirm('Delete this entry?')) return;
  const entries = loadEntries().filter(x=>x.id !== id);
  saveEntries(entries);
  renderList();
}

function exportEntry(id) {
  const e = loadEntries().find(x=>x.id===id);
  if(!e) return;
  const md = `# ${e.title || 'Untitled'}\n\n*Date: ${new Date(e.date).toISOString()}*\n\n${e.content}\n`;
  const blob = new Blob([md], { type: 'text/markdown' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `${(e.title || 'entry').replace(/\s+/g,'_').slice(0,40)}_${e.date}.md`;
  a.click();
  URL.revokeObjectURL(url);
}

function clearAll() {
  if(!confirm('Delete all entries from this browser?')) return;
  localStorage.removeItem(LS_KEY);
  renderList();
}

/* Bind UI */
document.getElementById('save-btn').addEventListener('click', ()=>{
  const title = document.getElementById('entry-title').value.trim();
  const content = document.getElementById('entry-content').value.trim();
  if(!content){ alert('Please write something first.'); return; }
  const saveBtn = document.getElementById('save-btn');
  if(saveBtn.dataset.editing){
    const id = saveBtn.dataset.editing;
    const entries = loadEntries();
    const idx = entries.findIndex(x=>x.id===id);
    if(idx>-1){
      entries[idx].title = title;
      entries[idx].content = content;
      entries[idx].date = Date.now();
      saveEntries(entries);
    }
    saveBtn.removeAttribute('data-editing');
    saveBtn.textContent = 'Save Entry';
  } else {
    addEntry(title, content);
  }
  document.getElementById('entry-title').value = '';
  document.getElementById('entry-content').value = '';
});

document.getElementById('clear-btn').addEventListener('click', clearAll);
document.getElementById('export-btn').addEventListener('click', ()=>{
  // If one entry selected for editing, export it, else export all as one markdown file
  const editing = document.getElementById('save-btn').dataset.editing;
  if(editing){
    exportEntry(editing);
    return;
  }
  const entries = loadEntries();
  if(entries.length === 0){ alert('No entries to export'); return; }
  const md = entries.map(e=>`# ${e.title || 'Untitled'}\n\n*Date: ${new Date(e.date).toISOString()}*\n\n${e.content}\n\n---\n`).join('\n');
  const blob = new Blob([md], { type: 'text/markdown' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `mini-diary-export_${Date.now()}.md`;
  a.click();
  URL.revokeObjectURL(url);
});

renderList();
