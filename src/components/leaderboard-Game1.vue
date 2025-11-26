<template>
  <div class="leaderboard-section" v-if="scores.length > 0">
    <h3>Mejores Puntuaciones</h3>
    <div class="table-container">
      <table class="leaderboard-table">
        <thead>
          <tr>
            <th class="col-rank-enc">Posición</th>
            <th class="col-name-enc">Jugador</th>
            <th class="col-aciertos-enc" v-if="showAciertos">Aciertos</th>
            <th class="col-score-enc">Efectividad</th>
            <th class="col-time-enc">Tiempo</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(score, index) in scores" :key="score.id || index" class="leaderboard-row">
            <td class="col-rank">#{{ index + 1 }}</td>
            <td class="col-name">{{ score.usuario }}</td>
            <td class="col-aciertos" v-if="showAciertos">{{ score.aciertos }}/{{ score.total_residuos }}</td>
            <td class="col-score">{{ score.puntuacion }}%</td>
            <td class="col-time">{{ formatoTiempo(score.tiempo_segundos) }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: "LeaderboardGame1",
  props: {
    scores: {
      type: Array,
      required: true,
      default: () => []
    },
    showAciertos: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    formatoTiempo(segundos) {
      const minutos = Math.floor(segundos / 60);
      const segs = segundos % 60;
      return `${minutos}:${segs.toString().padStart(2, '0')}`;
    }
  }
}
</script>

<style scoped>
.leaderboard-section {
  margin: 20px auto 0 auto; 
  padding: 15px;
  border-top: 2px solid #24bf71;
  max-width: 500px; 
  width: 90%; 
}

.leaderboard-section h3 {
  margin-bottom: 15px;
  color: #2E7D32;
  text-align: center;
}

.table-container {
  max-height: 300px;
  overflow-y: auto;
  border-radius: 8px;
  padding: 10px;
}

.leaderboard-table {
  width: 100%;
  border-collapse: collapse;
  background: white;
  border-radius: 6px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.leaderboard-table thead {
  background: #2E7D32;
  color: white;
}

.leaderboard-table th {
  padding: 12px 8px;
  font-weight: bold;
  text-align: center;
  font-size: 0.9rem;
  border-bottom: 2px solid #1B5E20;
}

.leaderboard-table tbody tr {
  transition: background-color 0.2s;
}

.leaderboard-table tbody tr:nth-child(even) {
  background: #f8f9fa;
}

.leaderboard-table tbody tr:nth-child(odd) {
  background: #ffffff;
}

.leaderboard-table tbody tr:hover {
  background: #e8f5e8;
}

.leaderboard-table td {
  padding: 10px 8px;
  text-align: center;
  border-bottom: 1px solid #e0e0e0;
}

.col-rank {
  font-weight: bold;
  color: #2E7D32;
  width: 50px;
}

.col-name {
  text-align: left;
  font-weight: 500;
  color: #2E7D32;
  padding-left: 15px;
}

.col-aciertos {
  color: #666;
  font-weight: 500;
  width: 80px;
  font-size: 0.9rem;
}

.col-score {
  font-weight: bold;
  color: #FF9800;
  width: 70px;
}

.col-time {
  color: #666;
  font-weight: 500;
  width: 70px;
  font-size: 0.9rem;
}


@media (max-width: 768px) {
  .leaderboard-section {
    max-width: 450px;
    width: 95%;
  }
  
  .table-container {
    max-height: 350px;
  }
  
  .leaderboard-table {
    font-size: 0.85rem;
  }
  
  .leaderboard-table th,
  .leaderboard-table td {
    padding: 8px 4px;
  }
  
  .col-name {
    padding-left: 8px;
    max-width: 100px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  
  .col-aciertos {
    width: 70px;
    font-size: 0.8rem;
  }
  
  .col-score {
    width: 60px;
    font-size: 0.85rem;
  }
  
  .col-time {
    width: 60px;
    font-size: 0.8rem;
  }
  
  .col-rank {
    width: 40px;
  }
}

@media (max-width: 480px) {
  .leaderboard-section {
    max-width: 350px;
    width: 100%;
    padding: 10px;
  }
  
  .table-container {
    padding: 5px;
  }
  
  .leaderboard-table {
    font-size: 0.8rem;
  }
  
  .leaderboard-table th,
  .leaderboard-table td {
    padding: 6px 3px;
  }
  
  .col-name {
    max-width: 80px;
    padding-left: 5px;
  }
  
  .col-aciertos {
    width: 60px;
    font-size: 0.75rem;
  }
  
  .col-score {
    width: 50px;
    font-size: 0.8rem;
  }
  
  .col-time {
    width: 50px;
    font-size: 0.75rem;
  }
  
  .col-rank {
    width: 35px;
  }
}

</style>