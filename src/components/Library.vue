<template>
  <div class="library">
    <header class="library-header">
      <div class="header-content">
        <div class="header-left">
          <h1>Bibliothèque UM6P</h1>
        </div>
        <div class="header-right">
          <input 
            v-model="searchQuery" 
            type="text" 
            placeholder="Rechercher par titre ou auteur..."
            class="search-input"
          >
          <select v-model="statusFilter" class="status-filter">
            <option value="">Tous les statuts</option>
            <option value="true">Disponible</option>
            <option value="false">Emprunté</option>
          </select>
        </div>
      </div>
    </header>

    <div class="books-grid">
      <div v-for="book in filteredBooks" 
           :key="book.id" 
           class="book-card"
           @click="showBookDetails(book)">
        <img :src="getImageUrl(book.id)" :alt="book.titre" class="book-image">
        <div class="book-info">
          <h3>{{ book.titre }}</h3>
          <p>{{ book.auteur }}</p>
          <p>Année: {{ book.annee }}</p>
          <p>Catégorie: {{ book.categorie }}</p>
          <div class="status-container">
            <span :class="['status', book.disponible ? 'disponible' : 'emprunté']">
              {{ book.disponible ? 'Disponible' : 'Emprunté' }}
            </span>
            <button 
              class="loan-button"
              :class="{ 'return-button': !book.disponible }"
              @click.stop="toggleLoan(book)"
            >
              {{ book.disponible ? 'Emprunter' : 'Retourner' }}
            </button>
          </div>
        </div>
      </div>
    </div>

    <div v-if="selectedBook" class="modal" @click="closeModal">
      <div class="modal-content" @click.stop>
        <button class="close-button" @click="closeModal">&times;</button>
        <div class="book-details-table">
          <div class="book-image-container">
            <img :src="getImageUrl(selectedBook.id)" :alt="selectedBook.titre" class="modal-image">
          </div>
          <table class="details-table">
            <tr>
              <th>Titre</th>
              <td>{{ selectedBook.titre }}</td>
            </tr>
            <tr>
              <th>Auteur</th>
              <td>{{ selectedBook.auteur }}</td>
            </tr>
            <tr>
              <th>Année</th>
              <td>{{ selectedBook.annee }}</td>
            </tr>
            <tr>
              <th>Catégorie</th>
              <td>{{ selectedBook.categorie }}</td>
            </tr>
            <tr>
              <th>Statut</th>
              <td><span :class="['status', selectedBook.disponible ? 'disponible' : 'emprunté']">
                {{ selectedBook.disponible ? 'Disponible' : 'Emprunté' }}
              </span></td>
            </tr>
            <tr>
              <th>Résumé</th>
              <td>{{ selectedBook.resume }}</td>
            </tr>
          </table>
        </div>
        <button 
          class="loan-button modal-loan-button"
          :class="{ 'return-button': !selectedBook.disponible }"
          @click="toggleLoan(selectedBook)"
        >
          {{ selectedBook.disponible ? 'Emprunter' : 'Retourner' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import bookImage1 from '@/assets/1.png'
import bookImage2 from '@/assets/2.jpeg'
import bookImage3 from '@/assets/3.jpeg'
import bookImage4 from '@/assets/4.png'
import bookImage5 from '@/assets/5.jpeg'
import bookImage6 from '@/assets/6.jpg'
import bookImage7 from '@/assets/7.jpeg'
import bookImage8 from '@/assets/8.jpg'
import bookImage9 from '@/assets/9.jpeg'
import bookImage10 from '@/assets/10.png'
import bookImage11 from '@/assets/11.jpg'
import bookImage12 from '@/assets/12.jpg'
import bookImage13 from '@/assets/13.png'
import bookImage14 from '@/assets/14.jpg'
import bookImage15 from '@/assets/15.jpg'
import bookImage16 from '@/assets/16.jpg'
import bookImage17 from '@/assets/17.jpeg'

export default {
  name: 'Library',
  data() {
    return {
      books: [],
      searchQuery: '',
      statusFilter: '',
      selectedBook: null,
      bookImages: {
        '1': bookImage1,
        '2': bookImage2,
        '3': bookImage3,
        '4': bookImage4,
        '5': bookImage5,
        '6': bookImage6,
        '7': bookImage7,
        '8': bookImage8,
        '9': bookImage9,
        '10': bookImage10,
        '11': bookImage11,
        '12': bookImage12,
        '13': bookImage13,
        '14': bookImage14,
        '15': bookImage15,
        '16': bookImage16,
        '17': bookImage17
      }
    }
  },
  computed: {
    filteredBooks() {
      return this.books.filter(book => {
        const matchesSearch = book.titre.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
                            book.auteur.toLowerCase().includes(this.searchQuery.toLowerCase())
        const matchesStatus = !this.statusFilter || book.disponible.toString() === this.statusFilter
        return matchesSearch && matchesStatus
      })
    },
    availableBooks() {
      return this.books.filter(book => book.disponible).length
    },
    borrowedBooks() {
      return this.books.filter(book => !book.disponible).length
    }
  },
  methods: {
    getImageUrl(bookId) {
      return this.bookImages[bookId] || this.books.find(b => b.id === bookId)?.image
    },
    async fetchBooks() {
      try {
        const response = await fetch('http://localhost:3000/livres')
        this.books = await response.json()
      } catch (error) {
        console.error('Erreur lors du chargement des livres:', error)
      }
    },
    showBookDetails(book) {
      this.selectedBook = book
    },
    closeModal() {
      this.selectedBook = null
    },
    async toggleLoan(book) {
      try {
        const newStatus = !book.disponible
        const response = await fetch(`http://localhost:3000/livres/${book.id}`, {
          method: 'PATCH',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({ disponible: newStatus })
        })
        
        if (response.ok) {
          book.disponible = newStatus
          if (this.selectedBook && this.selectedBook.id === book.id) {
            this.selectedBook.disponible = newStatus
          }
        }
      } catch (error) {
        console.error('Erreur lors de la modification du statut:', error)
      }
    }
  },
  mounted() {
    this.fetchBooks()
  }
}
</script>

<style scoped>
.library {
  padding: 0;
  background-color: #f5f5f5;
  min-height: 100vh;
}

.library-header {
  background-color: #424242;
  color: white;
  padding: 1rem 0;
  margin-bottom: 2rem;
}

.header-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header-left {
  display: flex;
  flex-direction: column;
}

.library-header h1 {
  margin: 0;
  font-size: 2rem;
  font-weight: 600;
}

.header-right {
  display: flex;
  gap: 1rem;
  align-items: center;
}

.search-input {
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  width: 250px;
  background-color: white;
}

.status-filter {
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  background-color: white;
  cursor: pointer;
  min-width: 150px;
}

.books-grid {
  max-width: 1200px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
  padding: 0 20px;
}

.book-card {
  border: 1px solid #ddd;
  border-radius: 4px;
  overflow: hidden;
  background-color: white;
  display: flex;
  flex-direction: column;
  height: 100%;
}

.book-image {
  width: 100%;
  height: 300px;
  object-fit: contain;
  background-color: #f5f5f5;
  padding: 10px;
}

.book-info {
  padding: 15px;
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}

.book-info h3 {
  margin: 0 0 10px 0;
  color: #333;
  font-size: 1.1rem;
}

.book-info p {
  margin: 4px 0;
  color: #666;
  font-size: 0.9rem;
}

.status-container {
  margin-top: auto;
  padding-top: 15px;
  border-top: 1px solid #eee;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.status {
  display: inline-block;
  padding: 6px 12px;
  border-radius: 4px;
  font-size: 0.9em;
  margin-bottom: 10px;
  width: 80%;
  text-align: center;
}

.status.disponible {
  background-color: #e0e0e0;
  color: #333;
}

.status.emprunté {
  background-color: #e0e0e0;
  color: #666;
}

.loan-button {
  width: 80%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #f5f5f5;
  color: #333;
  cursor: pointer;
  font-weight: 500;
}

.loan-button:hover {
  background-color: #e0e0e0;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 4px;
  max-width: 600px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  position: relative;
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  border: none;
  background: none;
  font-size: 20px;
  cursor: pointer;
  color: #666;
}

.book-image-container {
  text-align: center;
  background-color: #f5f5f5;
  padding: 10px;
  border-radius: 4px;
  margin-bottom: 20px;
}

.modal-image {
  width: 200px;
  height: 300px;
  object-fit: contain;
  background-color: #f5f5f5;
  padding: 10px;
}

.details-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.details-table th,
.details-table td {
  padding: 10px;
  text-align: left;
  border-bottom: 1px solid #eee;
}

.details-table th {
  background-color: #f5f5f5;
  color: #333;
  font-weight: 500;
  width: 30%;
}

.details-table td {
  width: 70%;
  color: #666;
}

.modal-loan-button {
  margin-top: 20px;
  width: 100%;
  padding: 8px;
  font-size: 14px;
}

@media (max-width: 768px) {
  .header-content {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }

  .header-left {
    align-items: center;
  }

  .header-right {
    flex-direction: column;
    width: 100%;
  }

  .search-input,
  .status-filter {
    width: 100%;
  }

  .book-image {
    height: 250px;
  }
  
  .modal-content {
    padding: 15px;
  }
  
  .modal-image {
    width: 150px;
    height: 250px;
  }
}
</style> 