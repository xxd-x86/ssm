<template>
  <div class="notes-app">
    <h3>便签记事本</h3>
    
    <div class="notes-controls">
      <button @click="createNote" class="add-note-btn">
        <span class="plus-icon">+</span> 新建便签
      </button>
      <div class="search-container">
        <input 
          type="text" 
          v-model="searchText" 
          placeholder="搜索..." 
          class="search-input"
        />
      </div>
    </div>
    
    <div class="notes-empty" v-if="filteredNotes.length === 0">
      <p>{{ searchText ? '没有符合条件的便签' : '还没有便签，点击"新建便签"开始记录' }}</p>
    </div>
    
    <div class="notes-list">
      <div 
        v-for="note in filteredNotes" 
        :key="note.id" 
        class="note-card"
        :class="{ 'active': activeNoteId === note.id }"
        @click="selectNote(note.id)"
      >
        <div class="note-header">
          <div class="note-title">{{ note.title || '无标题' }}</div>
          <div class="note-actions">
            <button @click.stop="deleteNote(note.id)" class="delete-btn" title="删除">
              ×
            </button>
          </div>
        </div>
        <div class="note-preview">{{ note.content.substring(0, 100) }}{{ note.content.length > 100 ? '...' : '' }}</div>
        <div class="note-date">{{ formatDate(note.updatedAt) }}</div>
      </div>
    </div>
    
    <div class="note-editor" v-if="activeNote">
      <input 
        type="text" 
        v-model="activeNote.title" 
        placeholder="标题" 
        class="note-title-input"
        @input="saveNote"
      />
      <textarea 
        v-model="activeNote.content" 
        placeholder="在这里输入内容..." 
        class="note-content-input"
        @input="saveNote"
      ></textarea>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Notes',
  data() {
    return {
      notes: [],
      activeNoteId: null,
      activeNote: null,
      searchText: '',
      autosaveTimer: null
    };
  },
  computed: {
    filteredNotes() {
      if (!this.searchText) {
        return this.notes.sort((a, b) => b.updatedAt - a.updatedAt);
      }
      
      const lowercaseSearch = this.searchText.toLowerCase();
      return this.notes
        .filter(note => 
          note.title.toLowerCase().includes(lowercaseSearch) || 
          note.content.toLowerCase().includes(lowercaseSearch)
        )
        .sort((a, b) => b.updatedAt - a.updatedAt);
    }
  },
  mounted() {
    this.loadNotes();
    
    // 如果有便签，默认选中第一个
    if (this.notes.length > 0) {
      this.selectNote(this.notes[0].id);
    }
  },
  methods: {
    loadNotes() {
      const savedNotes = localStorage.getItem('notes');
      if (savedNotes) {
        try {
          this.notes = JSON.parse(savedNotes);
        } catch (e) {
          console.error('Failed to parse notes:', e);
          this.notes = [];
        }
      }
    },
    
    saveAllNotes() {
      localStorage.setItem('notes', JSON.stringify(this.notes));
    },
    
    createNote() {
      const newNote = {
        id: Date.now(),
        title: '',
        content: '',
        createdAt: Date.now(),
        updatedAt: Date.now()
      };
      
      this.notes.unshift(newNote);
      this.saveAllNotes();
      this.selectNote(newNote.id);
    },
    
    selectNote(noteId) {
      this.activeNoteId = noteId;
      this.activeNote = this.notes.find(note => note.id === noteId);
    },
    
    deleteNote(noteId) {
      if (confirm('确定要删除这条便签吗？')) {
        this.notes = this.notes.filter(note => note.id !== noteId);
        
        if (this.activeNoteId === noteId) {
          this.activeNoteId = null;
          this.activeNote = null;
          
          // 如果还有便签，选中第一个
          if (this.notes.length > 0) {
            this.selectNote(this.notes[0].id);
          }
        }
        
        this.saveAllNotes();
      }
    },
    
    saveNote() {
      if (this.autosaveTimer) {
        clearTimeout(this.autosaveTimer);
      }
      
      this.autosaveTimer = setTimeout(() => {
        const noteIndex = this.notes.findIndex(note => note.id === this.activeNoteId);
        
        if (noteIndex !== -1) {
          this.notes[noteIndex].updatedAt = Date.now();
          this.saveAllNotes();
        }
      }, 500);
    },
    
    formatDate(timestamp) {
      const date = new Date(timestamp);
      
      const today = new Date();
      const isToday = date.getDate() === today.getDate() 
        && date.getMonth() === today.getMonth()
        && date.getFullYear() === today.getFullYear();
      
      if (isToday) {
        return '今天 ' + date.getHours().toString().padStart(2, '0') + 
          ':' + date.getMinutes().toString().padStart(2, '0');
      }
      
      const yesterday = new Date();
      yesterday.setDate(today.getDate() - 1);
      const isYesterday = date.getDate() === yesterday.getDate() 
        && date.getMonth() === yesterday.getMonth()
        && date.getFullYear() === yesterday.getFullYear();
      
      if (isYesterday) {
        return '昨天 ' + date.getHours().toString().padStart(2, '0') + 
          ':' + date.getMinutes().toString().padStart(2, '0');
      }
      
      return (date.getMonth() + 1).toString().padStart(2, '0') + '-' + 
        date.getDate().toString().padStart(2, '0') + ' ' + 
        date.getHours().toString().padStart(2, '0') + ':' + 
        date.getMinutes().toString().padStart(2, '0');
    }
  }
};
</script>

<style scoped>
.notes-app {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  max-width: 900px;
  margin: 0 auto;
  height: 500px;
}

h3 {
  color: #333;
  font-size: 1.5rem;
  margin: 0;
  padding: 15px 20px;
  border-bottom: 1px solid #eee;
}

.notes-controls {
  display: flex;
  align-items: center;
  padding: 10px 20px;
  border-bottom: 1px solid #eee;
}

.add-note-btn {
  background-color: #4a90e2;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 12px;
  font-size: 14px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 5px;
  margin-right: 10px;
}

.add-note-btn:hover {
  background-color: #3a7bc8;
}

.plus-icon {
  font-size: 16px;
  font-weight: bold;
}

.search-container {
  flex: 1;
}

.search-input {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.notes-empty {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100px;
  color: #888;
  font-style: italic;
}

.notes-list {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  padding: 15px;
  overflow-y: auto;
  flex: 0 0 auto;
  max-height: 200px;
}

.note-card {
  background-color: #f9f9f9;
  border-radius: 6px;
  padding: 12px;
  width: calc(33.333% - 10px);
  min-width: 200px;
  cursor: pointer;
  transition: all 0.2s ease;
  border: 1px solid transparent;
}

.note-card:hover {
  background-color: #f0f0f0;
}

.note-card.active {
  border-color: #4a90e2;
  background-color: #eaf2fd;
}

.note-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.note-title {
  font-weight: 500;
  font-size: 16px;
  color: #333;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.note-preview {
  font-size: 14px;
  color: #666;
  margin-bottom: 8px;
  max-height: 60px;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}

.note-date {
  font-size: 12px;
  color: #999;
  text-align: right;
}

.delete-btn {
  background: none;
  border: none;
  color: #ccc;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  padding: 0 5px;
  line-height: 1;
}

.delete-btn:hover {
  color: #e74c3c;
}

.note-editor {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 15px;
  border-top: 1px solid #eee;
  overflow: hidden;
}

.note-title-input {
  border: none;
  font-size: 18px;
  font-weight: 500;
  margin-bottom: 10px;
  padding: 5px 0;
  border-bottom: 1px solid #eee;
}

.note-content-input {
  flex: 1;
  border: none;
  resize: none;
  font-size: 16px;
  line-height: 1.5;
  padding: 5px 0;
}

.note-title-input:focus, .note-content-input:focus {
  outline: none;
}

@media (max-width: 768px) {
  .note-card {
    width: calc(50% - 10px);
  }
}

@media (max-width: 576px) {
  .note-card {
    width: 100%;
  }
}

@media (max-width: 480px) {
  .notes-app {
    height: 600px;
  }
  
  .notes-controls {
    flex-direction: column;
    align-items: stretch;
  }
  
  .add-note-btn {
    margin-bottom: 10px;
    margin-right: 0;
  }
  
  .notes-list {
    max-height: 150px;
  }
  
  .note-card {
    width: 100%;
    padding: 10px;
  }
  
  .note-editor {
    padding: 10px;
  }
  
  .note-title-input {
    font-size: 16px;
  }
  
  .note-content-input {
    font-size: 14px;
  }
}
</style> 