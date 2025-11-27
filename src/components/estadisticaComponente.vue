<template>
  <div v-if="estadisticas && estadisticas.tieneCalculos" class="stats-panel">
    <h2><i class="fas fa-chart-line"></i> Tus Estadísticas</h2>
    
    <!-- Tarjetas de resumen -->
    <div class="stats-cards">
      <div class="stat-card">
        <div class="stat-icon" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);">
          <i class="fas fa-calculator"></i>
        </div>
        <div class="stat-info">
          <p class="stat-label">Total Cálculos</p>
          <p class="stat-value">{{ estadisticas.totalCalculos }}</p>
        </div>
      </div>
      
      <div class="stat-card">
        <div class="stat-icon" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">
          <i class="fas fa-chart-bar"></i>
        </div>
        <div class="stat-info">
          <p class="stat-label">Promedio Emisiones</p>
          <p class="stat-value">{{ estadisticas.promedioEmisiones }} <span class="unit">kg CO₂</span></p>
        </div>
      </div>
      
      <div class="stat-card">
        <div class="stat-icon" style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);">
          <i class="fas fa-arrow-down"></i>
        </div>
        <div class="stat-info">
          <p class="stat-label">Mínimo</p>
          <p class="stat-value">{{ estadisticas.menorEmisiones }} <span class="unit">kg CO₂</span></p>
        </div>
      </div>
      
      <div class="stat-card">
        <div class="stat-icon" style="background: linear-gradient(135deg, #fa709a 0%, #fee140 100%);">
          <i class="fas fa-arrow-up"></i>
        </div>
        <div class="stat-info">
          <p class="stat-label">Máximo</p>
          <p class="stat-value">{{ estadisticas.mayorEmisiones }} <span class="unit">kg CO₂</span></p>
        </div>
      </div>
    </div>

    <!-- Gráficos -->
    <div class="charts-container">
      <!-- Gráfico de línea - Evolución mensual -->
      <div class="chart-wrapper">
        <h3><i class="fas fa-chart-line"></i> Evolución de Emisiones</h3>
        <div class="chart-container">
          <canvas ref="lineChart"></canvas>
        </div>
      </div>

      <!-- Gráfico de dona - Distribución por categorías -->
      <div class="chart-wrapper">
        <h3><i class="fas fa-chart-pie"></i> Distribución por Categoría</h3>
        <div class="chart-container">
          <canvas ref="doughnutChart"></canvas>
        </div>
        <div class="legend-custom">
          <div class="legend-item">
            <span class="legend-color" style="background: #4caf50;"></span>
            <span>Baja (&lt;50 kg)</span>
          </div>
          <div class="legend-item">
            <span class="legend-color" style="background: #ff9800;"></span>
            <span>Media (50-100 kg)</span>
          </div>
          <div class="legend-item">
            <span class="legend-color" style="background: #f44336;"></span>
            <span>Alta (&gt;100 kg)</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Información adicional -->
    <div class="info-footer">
      <div class="info-item">
        <i class="fas fa-calendar-check"></i>
        <div>
          <strong>Primer cálculo:</strong>
          <span>{{ formatearFecha(estadisticas.primerCalculo) }}</span>
        </div>
      </div>
      <div class="info-item">
        <i class="fas fa-calendar"></i>
        <div>
          <strong>Último cálculo:</strong>
          <span>{{ formatearFecha(estadisticas.ultimoCalculo) }}</span>
        </div>
      </div>
    </div>
  </div>

  <!-- Mensaje cuando no hay estadísticas -->
  <div v-else class="no-stats">
    <i class="fas fa-chart-line fa-3x"></i>
    <p>Aún no tienes estadísticas disponibles.</p>
    <p>¡Realiza tu primer cálculo de huella de carbono para ver tus estadísticas!</p>
  </div>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
  data() {
    return {
      estadisticas: null,
      evolucionMensual: [],
      distribucionCategorias: {},
      lineChart: null,
      doughnutChart: null
    }
  },
  
  async mounted() {
    await this.cargarEstadisticas();
  },
  
  beforeUnmount() {
    // Destruir gráficos al desmontar el componente
    if (this.lineChart) this.lineChart.destroy();
    if (this.doughnutChart) this.doughnutChart.destroy();
  },
  
  methods: {
    async cargarEstadisticas() {
      try {
        const token = localStorage.getItem('token');
        const response = await fetch(`${process.env.VUE_APP_API_URL}/api/estadisticas`, {
          headers: { 'Authorization': `Bearer ${token}` }
        });
        
        if (!response.ok) throw new Error('Error al cargar estadísticas');
        
        const data = await response.json();
        this.estadisticas = data.estadisticas;
        this.evolucionMensual = data.evolucionMensual || [];
        this.distribucionCategorias = data.distribucionCategorias || { baja: 0, media: 0, alta: 0 };
        
        // Esperar al siguiente tick para asegurar que los canvas estén renderizados
        this.$nextTick(() => {
          this.crearGraficos();
        });
        
      } catch (error) {
        console.error(error);
        this.$toast?.error('Error al cargar estadísticas');
      }
    },
    
    crearGraficos() {
      this.crearGraficoLinea();
      this.crearGraficoDona();
    },
    
    crearGraficoLinea() {
      if (!this.$refs.lineChart) return;
      
      const ctx = this.$refs.lineChart.getContext('2d');
      
      // Destruir gráfico anterior si existe
      if (this.lineChart) this.lineChart.destroy();
      
      // Preparar datos
      const labels = this.evolucionMensual.map(item => {
        const meses = ['Ene', 'Feb', 'Mar', 'Abr', 'May', 'Jun', 'Jul', 'Ago', 'Sep', 'Oct', 'Nov', 'Dic'];
        return `${meses[item.mes - 1]} ${item.anio}`;
      });
      
      const datos = this.evolucionMensual.map(item => item.emisiones);
      
      this.lineChart = new Chart(ctx, {
        type: 'line',
        data: {
          labels: labels,
          datasets: [{
            label: 'Emisiones (kg CO₂)',
            data: datos,
            borderColor: '#4caf50',
            backgroundColor: 'rgba(76, 175, 80, 0.1)',
            borderWidth: 3,
            fill: true,
            tension: 0.4,
            pointRadius: 5,
            pointHoverRadius: 7,
            pointBackgroundColor: '#4caf50',
            pointBorderColor: '#fff',
            pointBorderWidth: 2
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            legend: {
              display: false
            },
            tooltip: {
              backgroundColor: 'rgba(0, 0, 0, 0.8)',
              padding: 12,
              titleFont: {
                size: 14,
                weight: 'bold'
              },
              bodyFont: {
                size: 13
              },
              callbacks: {
                label: function(context) {
                  return `Emisiones: ${context.parsed.y} kg CO₂`;
                }
              }
            }
          },
          scales: {
            y: {
              beginAtZero: true,
              grid: {
                color: 'rgba(0, 0, 0, 0.05)'
              },
              ticks: {
                callback: function(value) {
                  return value + ' kg';
                }
              }
            },
            x: {
              grid: {
                display: false
              }
            }
          }
        }
      });
    },
    
    crearGraficoDona() {
      if (!this.$refs.doughnutChart) return;
      
      const ctx = this.$refs.doughnutChart.getContext('2d');
      
      // Destruir gráfico anterior si existe
      if (this.doughnutChart) this.doughnutChart.destroy();
      
      const datos = [
        this.distribucionCategorias.baja || 0,
        this.distribucionCategorias.media || 0,
        this.distribucionCategorias.alta || 0
      ];
      
      this.doughnutChart = new Chart(ctx, {
        type: 'doughnut',
        data: {
          labels: ['Baja', 'Media', 'Alta'],
          datasets: [{
            data: datos,
            backgroundColor: [
              '#4caf50',
              '#ff9800',
              '#f44336'
            ],
            borderWidth: 0,
            hoverOffset: 15
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            legend: {
              display: false
            },
            tooltip: {
              backgroundColor: 'rgba(0, 0, 0, 0.8)',
              padding: 12,
              titleFont: {
                size: 14,
                weight: 'bold'
              },
              bodyFont: {
                size: 13
              },
              callbacks: {
                label: function(context) {
                  const total = context.dataset.data.reduce((a, b) => a + b, 0);
                  const porcentaje = total > 0 ? ((context.parsed / total) * 100).toFixed(1) : 0;
                  return `${context.label}: ${context.parsed} (${porcentaje}%)`;
                }
              }
            }
          },
          cutout: '65%'
        }
      });
    },
    
    formatearFecha(fecha) {
      if (!fecha) return 'N/A';
      return new Date(fecha).toLocaleDateString('es-ES', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      });
    }
  }
}
</script>

<style scoped>
.stats-panel {
  background: #fff;
  padding: 25px;
  border-radius: 15px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  font-family: 'Poppins', sans-serif;
}

.stats-panel h2 {
  margin-top: 0;
  margin-bottom: 25px;
  color: #2c3e50;
  font-size: 1.8rem;
  display: flex;
  align-items: center;
  gap: 12px;
  padding-bottom: 15px;
  border-bottom: 3px solid #4caf50;
}

.stats-panel h2 i {
  color: #4caf50;
}

/* Tarjetas de estadísticas */
.stats-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  margin-bottom: 30px;
}

.stat-card {
  background: #fff;
  border-radius: 12px;
  padding: 20px;
  display: flex;
  align-items: center;
  gap: 15px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.stat-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.12);
}

.stat-icon {
  width: 60px;
  height: 60px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.stat-icon i {
  color: #fff;
  font-size: 1.5rem;
}

.stat-info {
  flex: 1;
}

.stat-label {
  margin: 0;
  font-size: 0.85rem;
  color: #7f8c8d;
  font-weight: 500;
}

.stat-value {
  margin: 5px 0 0 0;
  font-size: 1.8rem;
  font-weight: bold;
  color: #2c3e50;
}

.stat-value .unit {
  font-size: 0.7rem;
  font-weight: normal;
  color: #95a5a6;
}

/* Contenedor de gráficos */
.charts-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 25px;
  margin-bottom: 25px;
}

.chart-wrapper {
  background: #f8f9fa;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.chart-wrapper h3 {
  margin: 0 0 20px 0;
  color: #2c3e50;
  font-size: 1.2rem;
  display: flex;
  align-items: center;
  gap: 10px;
}

.chart-wrapper h3 i {
  color: #4caf50;
}

.chart-container {
  position: relative;
  height: 250px;
}

/* Leyenda personalizada */
.legend-custom {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-top: 20px;
  flex-wrap: wrap;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.9rem;
  color: #555;
}

.legend-color {
  width: 16px;
  height: 16px;
  border-radius: 4px;
  display: inline-block;
}

/* Footer con información adicional */
.info-footer {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 15px;
  padding: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 12px;
  color: #fff;
}

.info-item {
  display: flex;
  align-items: center;
  gap: 12px;
}

.info-item i {
  font-size: 1.5rem;
  opacity: 0.9;
}

.info-item div {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.info-item strong {
  font-size: 0.85rem;
  opacity: 0.9;
}

.info-item span {
  font-size: 1rem;
}

/* Sin estadísticas */
.no-stats {
  text-align: center;
  padding: 60px 30px;
  background: #f8f9fa;
  border-radius: 15px;
  color: #7f8c8d;
}

.no-stats i {
  color: #bdc3c7;
  margin-bottom: 20px;
}

.no-stats p {
  margin: 10px 0;
  font-size: 1.1rem;
  line-height: 1.6;
}

.no-stats p:first-of-type {
  font-weight: 600;
  color: #34495e;
  font-size: 1.3rem;
}

/* ========== RESPONSIVE ========== */

/* Tablets grandes */
@media (max-width: 1024px) {
  .stats-cards {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .charts-container {
    grid-template-columns: 1fr;
  }
}

/* Tablets */
@media (max-width: 768px) {
  .stats-panel {
    padding: 20px;
  }
  
  .stats-panel h2 {
    font-size: 1.5rem;
  }
  
  .stats-cards {
    grid-template-columns: 1fr 1fr;
    gap: 15px;
  }
  
  .stat-card {
    padding: 15px;
    flex-direction: column;
    text-align: center;
  }
  
  .stat-icon {
    width: 50px;
    height: 50px;
  }
  
  .stat-value {
    font-size: 1.5rem;
  }
  
  .chart-container {
    height: 220px;
  }
  
  .info-footer {
    padding: 15px;
  }
}

/* Móviles */
@media (max-width: 600px) {
  .stats-panel {
    padding: 15px;
    border-radius: 12px;
  }
  
  .stats-panel h2 {
    font-size: 1.3rem;
    margin-bottom: 20px;
  }
  
  .stats-cards {
    grid-template-columns: 1fr;
    gap: 12px;
  }
  
  .stat-card {
    flex-direction: row;
    text-align: left;
    padding: 12px;
  }
  
  .stat-icon {
    width: 45px;
    height: 45px;
  }
  
  .stat-icon i {
    font-size: 1.2rem;
  }
  
  .stat-value {
    font-size: 1.4rem;
  }
  
  .chart-wrapper {
    padding: 15px;
  }
  
  .chart-wrapper h3 {
    font-size: 1.1rem;
  }
  
  .chart-container {
    height: 200px;
  }
  
  .legend-custom {
    gap: 12px;
    font-size: 0.85rem;
  }
  
  .info-footer {
    grid-template-columns: 1fr;
    gap: 12px;
    padding: 15px;
  }
  
  .info-item {
    font-size: 0.9rem;
  }
  
  .no-stats {
    padding: 40px 20px;
  }
  
  .no-stats p {
    font-size: 1rem;
  }
}

/* Móviles pequeños */
@media (max-width: 480px) {
  .stats-panel {
    padding: 12px;
  }
  
  .stats-panel h2 {
    font-size: 1.2rem;
    gap: 8px;
  }
  
  .stat-card {
    padding: 10px;
  }
  
  .stat-icon {
    width: 40px;
    height: 40px;
  }
  
  .stat-label {
    font-size: 0.8rem;
  }
  
  .stat-value {
    font-size: 1.2rem;
  }
  
  .chart-container {
    height: 180px;
  }
  
  .legend-custom {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }
}

/* Landscape móvil */
@media (max-height: 500px) and (orientation: landscape) {
  .stats-panel {
    padding: 10px;
  }
  
  .stats-cards {
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
  }
  
  .stat-card {
    padding: 10px;
  }
  
  .chart-container {
    height: 150px;
  }
  
  .no-stats {
    padding: 20px;
  }
}
</style>