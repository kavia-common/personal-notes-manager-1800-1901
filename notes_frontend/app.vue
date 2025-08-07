<template>
  <div class="layout-root">
    <aside class="sidebar">
      <header class="sidebar-header">
        <span class="logo">Notes</span>
        <button class="add-btn" @click="createNewNote" title="New note">+</button>
      </header>
      <nav class="notes-list">
        <button
          v-for="note in notes"
          :key="note.id"
          :class="['note-nav-btn', note.id === currentNoteId ? 'selected' : '']"
          @click="selectNote(note.id)"
        >
          <div class="note-title-ellipsis">{{ note.title || 'Untitled' }}</div>
          <button
            class="delete-btn"
            title="Delete"
            @click.stop="deleteNote(note.id)"
            aria-label="Delete"
          >
            🗑
          </button>
        </button>
      </nav>
      <footer class="sidebar-footer">
        <span class="footer-txt">Personal Notes Manager</span>
      </footer>
    </aside>

    <main class="main-content">
      <section v-if="currentNote" class="note-detail">
        <input
          type="text"
          class="note-title"
          v-model="currentNote.title"
          placeholder="Title"
          @input="updateCurrentNote"
        />
        <textarea
          class="note-body"
          v-model="currentNote.body"
          placeholder="Write your note..."
          @input="updateCurrentNote"
        />
        <div class="note-actions">
          <button class="save-btn" @click="saveCurrentNote" :disabled="!dirty">Save</button>
        </div>
      </section>
      <section v-else class="empty-state">
        <div>
          <h2>No Note Selected</h2>
          <p>Select or create a note to begin.</p>
        </div>
      </section>
    </main>
  </div>
</template>

<script setup lang="ts">
// PUBLIC_INTERFACE
/**
 * Main Notes Manager App.vue
 * - Contains layout, CRUD operations, and state logic for notes.
 * - Responsive and minimal with a sidebar and editor area.
 */
import { ref, computed, watch, onMounted } from 'vue'

type Note = {
  id: string
  title: string
  body: string
  created: number
  updated: number
}

/**
 * Utility to check if code is running in a client/browser environment.
 */
function isClient() {
  return typeof window !== 'undefined' && typeof localStorage !== 'undefined'
}

// PUBLIC_INTERFACE
function loadNotes(): Note[] {
  if (!isClient()) return []
  const saved = localStorage.getItem('notes')
  if (!saved) return []
  try {
    return JSON.parse(saved) as Note[]
  } catch {
    return []
  }
}

const notes = ref<Note[]>([])
const currentNoteId = ref<string | null>(null)
const dirty = ref(false)

/**
 * SSR-Safe initial notes and reactive currentNoteId population.
 * - Only perform localStorage I/O on onMounted (client-side).
 * - Avoid any client API access during SSR.
 */
onMounted(() => {
  if (isClient()) {
    notes.value = loadNotes()
    if (notes.value.length > 0 && !currentNoteId.value) {
      currentNoteId.value = notes.value[0].id
    }
  }
})

const currentNote = computed<Note | null>(() => {
  if (!currentNoteId.value) return null
  return notes.value.find(n => n.id === currentNoteId.value) || null
})

function persistNotes() {
  if (isClient()) {
    localStorage.setItem('notes', JSON.stringify(notes.value))
  }
}

function selectNote(id: string) {
  if (dirty.value && !confirm('You have unsaved changes. Continue?')) return
  currentNoteId.value = id
  dirty.value = false
}

function createNewNote() {
  if (dirty.value && !confirm('You have unsaved changes. Continue?')) return
  const id = Date.now().toString(36) + Math.random().toString(36).slice(2)
  const note: Note = {
    id,
    title: '',
    body: '',
    created: Date.now(),
    updated: Date.now()
  }
  notes.value.unshift(note)
  persistNotes()
  currentNoteId.value = note.id
  dirty.value = false
}

function deleteNote(id: string) {
  if (!confirm('Delete this note?')) return
  const idx = notes.value.findIndex(n => n.id === id)
  if (idx >= 0) {
    notes.value.splice(idx, 1)
    persistNotes()
  }
  // Set a new current note
  if (currentNoteId.value === id) {
    currentNoteId.value = notes.value[0]?.id ?? null
    dirty.value = false
  }
}

function updateCurrentNote() {
  if (!currentNote.value) return
  currentNote.value.updated = Date.now()
  dirty.value = true
}

function saveCurrentNote() {
  if (!currentNote.value) return
  const idx = notes.value.findIndex(n => n.id === currentNote.value?.id)
  if (idx !== -1) {
    notes.value[idx] = { ...currentNote.value }
    persistNotes()
    dirty.value = false
  }
}

/**
 * Keep notes in localStorage on change, but only on client-side.
 */
if (typeof window !== "undefined") {
  watch(notes, persistNotes, { deep: true })
}
</script>

<style scoped>
:root {
  --primary: #4a90e2;
  --secondary: #22313f;
  --accent: #f5a623;
  --sidebar-width: 240px;
  --sidebar-width-mobile: 64vw;
  --bg: #fafcff;
  --bg-alt: #f6f8fa;
  --txt-main: #22313f;
  --txt-accent: #f5a623;
}
.layout-root {
  display: flex;
  height: 100vh;
  background: var(--bg);
  color: var(--txt-main);
  font-family: 'Inter', 'Segoe UI', Arial, sans-serif;
  box-sizing: border-box;
}
.sidebar {
  background: var(--secondary);
  color: white;
  width: var(--sidebar-width);
  min-width: 160px;
  display: flex;
  flex-direction: column;
  min-height: 0;
  transition: width 0.2s;
}
.sidebar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  padding: 1.5rem 1rem 0.75rem 1.25rem;
  border-bottom: 1px solid #34495e33;
}
.logo {
  font-weight: bold;
  font-size: 1.35rem;
  letter-spacing: 0.02em;
}
.add-btn {
  background: var(--accent);
  color: var(--secondary);
  font-size: 1.3rem;
  border: none;
  border-radius: 50%;
  padding: 0.25em 0.55em;
  cursor: pointer;
  transition: background 0.2s;
}
.add-btn:hover {
  background: #f8bb4a;
}
.notes-list {
  flex: 1 1 0%;
  overflow-y: auto;
  padding: 0.5rem 0.25rem;
  display: flex;
  flex-direction: column;
  gap: 2px;
}
.note-nav-btn {
  display: flex;
  align-items: center;
  justify-content: space-between;
  text-align: left;
  color: white;
  background: none;
  border: none;
  width: 100%;
  padding: 0.52rem 1.1rem 0.52rem 1.0rem;
  font-size: 1rem;
  cursor: pointer;
  border-radius: 5px;
  transition: background 0.12s;
  outline: none;
  gap: 0.6rem;
  position: relative;
}
.note-title-ellipsis {
  flex: 1 1 auto;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  min-width: 0;
}
.note-nav-btn.selected, .note-nav-btn:active, .note-nav-btn:hover {
  background: rgba(74, 144, 226, 0.19);
  color: var(--accent);
}
.delete-btn {
  background: none;
  border: none;
  color: #ef9292;
  border-radius: 2px;
  padding: 0 0.12em;
  font-size: 1.0em;
  cursor: pointer;
  opacity: 0.72;
  transition: opacity 0.15s;
}
.delete-btn:hover {
  color: #ec2b2b;
  opacity: 1;
}
.sidebar-footer {
  font-size: 0.9em;
  padding: 1rem 1.2rem 0.7rem 1.3rem;
  background: transparent;
  opacity: 0.72;
  text-align: left;
}
.footer-txt {
  font-size: 0.95em;
  font-weight: 300;
  color: #ccd6e6;
}

/* Main content styles */
.main-content {
  flex: 1 1 0%;
  display: flex;
  flex-direction: column;
  min-width: 0;
  padding: 0;
  background: var(--bg);
  justify-content: flex-start;
}

.note-detail {
  max-width: 720px;
  margin: 2.5rem auto 0 auto;
  padding: 2.5rem 2rem 1.5rem 2rem;
  background: var(--bg-alt);
  border-radius: 19px;
  box-shadow: 0 2px 12px 0 rgba(33,53,63,0.08);
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
  min-height: 59vh;
}
.note-title {
  font-size: 1.3rem;
  font-weight: 500;
  padding: 0.53rem 0.7rem;
  border: 1px solid #dde3ea;
  border-radius: 7px;
  margin-bottom: 0.2rem;
  outline: none;
  color: var(--txt-main);
  background: white;
  width: 100%;
}
.note-title:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 1.5px var(--primary)22;
}
.note-body {
  font-size: 1.13rem;
  min-height: 238px;
  line-height: 1.6;
  padding: 0.62em 0.8em;
  border: 1px solid #deeaf4;
  border-radius: 8px;
  background: white;
  resize: vertical;
  outline: none;
  color: var(--txt-main);
}
.note-body:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 1.5px var(--primary)17;
}

.note-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 0.7rem;
}
.save-btn {
  background: var(--primary);
  color: white;
  border: none;
  border-radius: 6px;
  padding: 0.44em 1.9em;
  font-size: 1.05em;
  font-weight: 500;
  cursor: pointer;
  box-shadow: 0 1.5px 5px 0 rgba(74,144,226,0.05);
  transition: background 0.17s, filter 0.17s;
}
.save-btn:disabled {
  opacity: 0.57;
  filter: grayscale(0.4);
  cursor: not-allowed;
}
.save-btn:not(:disabled):hover {
  background: #2377ce;
}
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #b2b6bb;
}
.empty-state h2 {
  margin: 0 0 0.6em 0;
  font-size: 2em;
  color: var(--txt-main);
}
@media (max-width: 800px) {
  .layout-root {
    flex-direction: column;
    height: 100dvh;
  }
  .sidebar {
    width: 100vw;
    min-width: unset;
    flex-direction: row;
    align-items: stretch;
    height: 4.2rem;
    position: relative;
    z-index: 10;
    border-right: none;
    border-bottom: 1px solid #35495e27;
    padding: 0;
  }
  .sidebar-header {
    flex: 1 1 auto;
    flex-direction: row;
    align-items: center;
    justify-content: flex-start;
    padding: 0 1.1rem;
    border-bottom: none;
    border-right: 1px solid #34495e13;
    min-width: 0;
  }
  .logo {
    font-size: 1.13rem;
    margin-right: 1rem;
    line-height: 1.3;
    white-space: nowrap;
  }
  .add-btn {
    font-size: 1.07rem;
  }
  .sidebar-footer {
    display: none;
  }
  .notes-list {
    flex: 3 3 auto;
    flex-direction: row;
    overflow-x: auto;
    overflow-y: hidden;
    padding: 0.6rem 0.33rem 0.4rem 0.46rem;
    gap: 0.23rem;
  }
  .note-nav-btn {
    min-width: 80px;
    font-size: 1em;
    padding: 0.55em 0.55em 0.5em 0.7em;
    max-width: 176px;
  }
}
@media (max-width: 600px) {
  .note-detail {
    margin: 1.4rem 0 0 0;
    padding: 1.1rem 0.3rem 1.6rem 0.3rem;
    border-radius: 11px;
  }
  .main-content {
    padding: 0;
  }
  .note-title {
    font-size: 1.1em;
    padding: 0.48rem 0.44rem;
  }
  .note-body {
    min-height: 110px;
    font-size: 1.05em;
    padding: 0.48em 0.45em;
    border-radius: 4px;
  }
  .note-actions {
    margin-top: 0.5rem;
  }
}
</style>
