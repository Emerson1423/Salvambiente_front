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

<style>
.container {
  max-width: 1100px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.header-noticias {
  text-align: center;
  margin-bottom: 30px;
}

.header-noticias h1 {
  margin-top: 10%;
  font-size: 2.3rem;
  font-weight: bold;
  color: #0b6d11;
  display: inline-flex;
  align-items: center;
  gap: 10px;
}

.header-noticias .icono-planta {
  width: 90px;
  height: auto;
}

.header-noticias hr {
  width: 60%;
  margin: 10px auto;
  border: none;
  border-top: 1px solid #aaa;
}

.header-noticias p {
  max-width: 700px;
  margin: 0 auto;
  color: #444;
  font-size: 1rem;
  line-height: 1.6;
}

.loading {
  text-align: center;
  color: #555;
  font-size: 1.1rem;
  padding: 40px 0;
}

.noticias-wrapper {
  background-color: #6ebb6b; 
  padding: 30px;           
  border-radius: 15px;      
  margin-bottom: 20px;      
}

.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr); 
  gap: 20px;
}

.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  background-color: #fff;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  transition: transform 0.2s;
  display: flex;
  flex-direction: column;
  height: 100%;
}

.card:hover {
  transform: translateY(-5px);
}

.card-img {
  width: 100%;
  height: 180px;
  object-fit: cover;
}

.card-content {
  padding: 20px;
  flex-grow: 1;
  display: flex;
  flex-direction: column;
}

.card-content h2 {
  font-size: 18px;
  margin-bottom: 12px;
  line-height: 1.4;
  color: #2c5530;
  font-weight: 600;
  word-wrap: break-word;
  word-break: normal;
  overflow-wrap: break-word;
  hyphens: auto;
  display: block;
  overflow: visible;
}

.card-content small {
  display: block;
  margin-bottom: 15px;
  color: #666;
  font-size: 12px;
  font-weight: 500;
}

.btn-link {
  color: #2c7b33;
  text-decoration: none;
  font-weight: bold;
  padding: 8px 16px;
  border: 2px solid #2c7b33;
  border-radius: 5px;
  text-align: center;
  transition: all 0.3s ease;
  margin-top: auto;
  display: inline-block;
  width: fit-content;
}

.btn-link:hover {
  background-color: #2c7b33;
  color: white;
  text-decoration: none;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
  margin-top: 20px;
}

.pagination button {
  width: 40px;
  height: 40px;
  border-radius: 8px;
  border: none;
  background-color: #307b2c;
  color: white;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.pagination button:hover:not(:disabled) {
  background-color: #256d39;
}

.pagination button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.page-number {
  padding: 8px 14px;
  border-radius: 6px;
  background: #e5e5e5;
  font-weight: bold;
  color: #333;
  font-size: 14px;
}

/* Tablets grandes (900px - 768px) */
@media (max-width: 900px) {
  .container {
    padding: 15px;
  }
  
  .header-noticias h1 {
    font-size: 2.3rem;
  }
  
  .header-noticias .icono-planta {
    width: 80px;
  }
  
  .header-noticias p {
    font-size: 0.95rem;
  }
  
  .noticias-wrapper {
    padding: 25px;
  }
  
  .grid {
    grid-template-columns: repeat(2, 1fr); /* 2 columnas en tablets */
    gap: 18px;
  }
  
  .card-content {
    padding: 18px;
  }
  
  .card-content h2 {
    font-size: 17px;
    line-height: 1.35;
  }
  
  .card-img {
    height: 170px;
  }
  
  .btn-link {
    padding: 7px 14px;
    font-size: 14px;
  }
  
  .pagination button {
    width: 38px;
    height: 38px;
    font-size: 16px;
  }
}

/* Tablets pequeñas (768px - 600px) */
@media (max-width: 768px) {
  .header-noticias h1 {
    font-size: 2.3rem;
  }
  
  .header-noticias .icono-planta {
    width: 70px;
  }
  
  .header-noticias hr {
    width: 70%;
  }
  
  .header-noticias p {
    font-size: 0.9rem;
  }
  
  .noticias-wrapper {
    padding: 20px;
    border-radius: 12px;
  }
  
  .grid {
    gap: 16px;
  }
  
  .card-content {
    padding: 16px;
  }
  
  .card-content h2 {
    font-size: 16px;
    line-height: 1.3;
  }
  
  .card-img {
    height: 160px;
  }
  
  .card-content small {
    font-size: 11px;
  }
  
  .btn-link {
    padding: 7px 14px;
    font-size: 13px;
  }
  
  .loading {
    font-size: 1rem;
    padding: 30px 0;
  }
}

/* Móviles grandes (600px - 480px) */
@media (max-width: 600px) {
  .container {
    padding: 12px;
  }
  
  .header-noticias {
    margin-bottom: 25px;
  }
  
  .header-noticias h1 {
    font-size: 2.3rem;
    flex-direction: column;
    gap: 8px;
  }
  
  .header-noticias .icono-planta {
    width: 60px;
  }
  
  .header-noticias hr {
    width: 80%;
    margin: 8px auto;
  }
  
  .header-noticias p {
    font-size: 0.85rem;
    line-height: 1.5;
  }
  
  .noticias-wrapper {
    padding: 15px;
    border-radius: 10px;
  }
  
  .grid {
    grid-template-columns: 1fr; 
    gap: 15px;
  }
  
  .card-content {
    padding: 15px;
  }
  
  .card-content h2 {
    font-size: 17px; 
    line-height: 1.4;
    margin-bottom: 10px;
  }
  
  .card-img {
    height: 200px; 
  }
  
  .card-content small {
    font-size: 12px;
    margin-bottom: 12px;
  }
  
  .btn-link {
    padding: 8px 16px;
    font-size: 14px;
    width: 100%; 
    text-align: center;
  }
  
  .pagination {
    gap: 12px;
    margin-top: 25px;
  }
  
  .pagination button {
    width: 36px;
    height: 36px;
    font-size: 16px;
  }
  
  .page-number {
    padding: 6px 12px;
    font-size: 13px;
  }
  
  .loading {
    font-size: 0.95rem;
    padding: 25px 0;
  }
}

/* Móviles pequeños (480px - 380px) */
@media (max-width: 480px) {
  .container {
    padding: 10px;
  }
  
  .header-noticias h1 {
    font-size: 2.3rem;
  }
  
  .header-noticias .icono-planta {
    width: 50px;
  }
  
  .header-noticias p {
    font-size: 0.8rem;
  }
  
  .noticias-wrapper {
    padding: 12px;
  }
  
  .grid {
    gap: 12px;
  }
  
  .card-content {
    padding: 12px;
  }
  
  .card-content h2 {
    font-size: 16px;
    line-height: 1.35;
  }
  
  .card-img {
    height: 180px;
  }
  
  .card-content small {
    font-size: 11px;
  }
  
  .btn-link {
    padding: 7px 14px;
    font-size: 13px;
  }
  
  .pagination {
    gap: 10px;
  }
  
  .pagination button {
    width: 34px;
    height: 34px;
    font-size: 14px;
  }
  
  .page-number {
    padding: 5px 10px;
    font-size: 12px;
  }
}

/* Móviles muy pequeños (menos de 380px) */
@media (max-width: 380px) {
  .header-noticias h1 {
    font-size: 2.3rem;
  }
  
  .header-noticias .icono-planta {
    width: 45px;
  }
  
  .card-content h2 {
    font-size: 15px;
  }
  
  .card-img {
    height: 160px;
  }
  
  .btn-link {
    padding: 6px 12px;
    font-size: 12px;
  }
  
  .pagination button {
    width: 32px;
    height: 32px;
    font-size: 13px;
  }
}
</style>