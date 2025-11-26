<template>
  <div class="container">
    <div class="header-noticias">
      <h1>
        Noticias Ambientales
        <img src="@/assets/icons/plant-icon.gif" alt="icono planta" class="icono-planta" />
      </h1>
      <hr />
      <p>
        Entérate de lo que está pasando en el mundo ambiental. En esta sección
        encontrarás noticias locales y globales que afectan nuestro planeta y cómo
        podemos ayudar a cambiar la situación.
      </p>
    </div>

    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>Cargando noticias ambientales...</p>
    </div>

    <div v-else-if="noticias.length === 0" class="no-noticias">
      <p>No se pudieron cargar las noticias. Intenta más tarde.</p>
    </div>

    <div v-else>
      <div class="noticias-wrapper">
        <div class="grid">
          <div v-for="noticia in noticiasPaginadas" :key="noticia.url" class="card">
            <img :src="noticia.image" :alt="noticia.title" class="card-img">
            <div class="card-content">
              <h2>{{ noticia.title }}</h2>
              <p>{{ noticia.description }}</p>
              <small>{{ formatearFecha(noticia.publishedAt) }}</small>
              <a :href="noticia.url" target="_blank" rel="noopener noreferrer" class="btn-link">
                Leer más →
              </a>
            </div>
          </div>
        </div>
      </div>

      <div class="pagination">
        <button @click="prevPage" :disabled="paginaActual === 1">
          ◀
        </button>
        <span class="page-number">Página {{ paginaActual }} de {{ totalPaginas }}</span>
        <button @click="nextPage" :disabled="paginaActual === totalPaginas">
          ▶
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'NoticiasApi',
  data() {
    return {
      noticias: [],
      loading: true,
      paginaActual: 1,
      noticiasPorPagina: 6
    }
  },
  computed: {
    noticiasPaginadas() {
      const inicio = (this.paginaActual - 1) * this.noticiasPorPagina
      return this.noticias.slice(inicio, inicio + this.noticiasPorPagina)
    },
    totalPaginas() {
      return Math.ceil(this.noticias.length / this.noticiasPorPagina)
    }
  },
  methods: {
    async fetchNoticias() {
      const URL = `${process.env.VUE_APP_API_URL}/api/noticias`
      
      console.log('📡 Obteniendo noticias desde:', URL)
      
      try {
        const response = await axios.get(URL)
        console.log('✅ Respuesta recibida:', response.data)
        
        // Filtrar y eliminar duplicados
        const noticiasConImagen = response.data.articles.filter(n => n.image && n.title && n.description)
        
        // Eliminar duplicados por título
        const noticiasUnicas = this.eliminarDuplicados(noticiasConImagen)
        
        this.noticias = noticiasUnicas
        console.log(`✅ ${this.noticias.length} noticias únicas cargadas`)
        
      } catch (error) {
        console.error('❌ Error al cargar noticias:', error)
        console.error('Detalles:', error.response?.data)
        this.noticias = []
      } finally {
        this.loading = false
      }
    },
    
    eliminarDuplicados(noticias) {
      const vistos = new Set()
      return noticias.filter(noticia => {
        // Normalizar título para comparación (sin espacios extras, minúsculas)
        const tituloNormalizado = noticia.title.trim().toLowerCase()
        
        if (vistos.has(tituloNormalizado)) {
          return false // Ya existe
        }
        
        vistos.add(tituloNormalizado)
        return true
      })
    },
    
    formatearFecha(fecha) {
      const opciones = { 
        year: 'numeric', 
        month: 'long', 
        day: 'numeric' 
      }
      return new Date(fecha).toLocaleDateString('es-ES', opciones)
    },
    
    nextPage() {
      if (this.paginaActual < this.totalPaginas) {
        this.paginaActual++
        window.scrollTo({ top: 0, behavior: 'smooth' })
      }
    },
    
    prevPage() {
      if (this.paginaActual > 1) {
        this.paginaActual--
        window.scrollTo({ top: 0, behavior: 'smooth' })
      }
    }
  },
  mounted() {
    this.fetchNoticias()
  }
}
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Poppins', Arial, sans-serif;
}

.header-noticias {
  text-align: center;
  margin-bottom: 40px;
}

.header-noticias h1 {
  font-size: 2.5rem;
  font-weight: bold;
  color: #0b6d11;
  display: inline-flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 15px;
}

.header-noticias .icono-planta {
  width: 90px;
  height: auto;
}

.header-noticias hr {
  width: 60%;
  margin: 15px auto;
  border: none;
  border-top: 2px solid #6ebb6b;
}

.header-noticias p {
  max-width: 700px;
  margin: 0 auto;
  color: #555;
  font-size: 1.1rem;
  line-height: 1.6;
}

.loading {
  text-align: center;
  padding: 60px 20px;
}

.spinner {
  width: 50px;
  height: 50px;
  border: 5px solid #f3f3f3;
  border-top: 5px solid #6ebb6b;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 20px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.no-noticias {
  text-align: center;
  padding: 60px 20px;
  background-color: #f8f9fa;
  border-radius: 12px;
  margin: 20px 0;
}

.no-noticias p {
  font-size: 1.2rem;
  color: #666;
  margin: 0;
}

.noticias-wrapper {
  background: linear-gradient(135deg, #6ebb6b 0%, #5aa857 100%);
  padding: 40px;
  border-radius: 20px;
  margin-bottom: 30px;
  box-shadow: 0 8px 24px rgba(106, 187, 107, 0.3);
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 25px;
}

.card {
  border: none;
  border-radius: 12px;
  overflow: hidden;
  background-color: #fff;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  display: flex;
  flex-direction: column;
}

.card:hover {
  transform: translateY(-8px);
  box-shadow: 0 8px 20px rgba(0,0,0,0.15);
}

.card-img {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

.card-content {
  padding: 20px;
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}

.card-content h2 {
  font-size: 1.2rem;
  margin-bottom: 12px;
  color: #0b6d11;
  font-weight: 600;
  line-height: 1.4;
}

.card-content p {
  font-size: 0.95rem;
  margin-bottom: 15px;
  color: #444;
  line-height: 1.6;
  flex-grow: 1;
}

.card-content small {
  display: block;
  margin-bottom: 15px;
  color: #888;
  font-size: 0.85rem;
}

.btn-link {
  color: #0b6d11;
  text-decoration: none;
  font-weight: 600;
  font-size: 0.95rem;
  display: inline-flex;
  align-items: center;
  gap: 5px;
  transition: all 0.3s ease;
  align-self: flex-start;
}

.btn-link:hover {
  color: #6ebb6b;
  gap: 10px;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
  margin-top: 30px;
}

.pagination button {
  width: 45px;
  height: 45px;
  border-radius: 10px;
  border: none;
  background: linear-gradient(135deg, #6ebb6b 0%, #5aa857 100%);
  color: white;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(106, 187, 107, 0.3);
}

.pagination button:hover:not(:disabled) {
  background: linear-gradient(135deg, #5aa857 0%, #4d9449 100%);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(106, 187, 107, 0.4);
}

.pagination button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  transform: none;
}

.page-number {
  padding: 10px 20px;
  border-radius: 8px;
  background: #f0f0f0;
  font-weight: 600;
  color: #333;
  font-size: 0.95rem;
}

/* Responsive */
@media (max-width: 768px) {
  .header-noticias h1 {
    font-size: 2rem;
  }
  
  .noticias-wrapper {
    padding: 20px;
  }
  
  .grid {
    grid-template-columns: 1fr;
    gap: 20px;
  }
  
  .noticiasPorPagina {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 480px) {
  .header-noticias h1 {
    font-size: 1.5rem;
    flex-direction: column;
  }
  
  .header-noticias .icono-planta {
    width: 60px;
  }
}
</style>