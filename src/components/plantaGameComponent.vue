<template>
  <div class="container">
    <!-- Pantalla inicial -->
    <div v-if="!juegoActivo" class="initial-screen">
      <header>
        <h1>Quiz: Planta Virtual</h1>
        <p class="subtitle">Responde Verdadero o Falso para ayudar a tu planta a crecer</p>
      </header>
      
      <div class="welcome-section">
        <div class="initial-actions">
          <button class="btn-start-main" @click="empezarJuego">
            Comenzar Juego
          </button>
        </div>
      </div>
      
      <!-- líderes -->
      <LeaderboardGame2 :scores="leaderboard"></LeaderboardGame2>
    </div>

    <!-- Juego principal-->
    <div v-else>
      <header>
        <h1>Quiz: Planta Virtual</h1>
        <p class="subtitle">Responde Verdadero o Falso para ayudar a tu planta a crecer</p>
      </header>
      
      <div class="game-container">
        <div class="plant-status">
          <div class="plant-visual">
            <div class="plant" :class="plantState">
              <img :src="plantImage" :alt="plantName" class="plant-img">
            </div>
            <h2>{{ plantName }}</h2>
          </div>
          
          <div class="stats">
            <div class="stat health">
              <div class="stat-name">Salud</div>
              <div class="stat-value">{{ plant.health }}%</div>
              <div class="progress-bar">
                <div class="progress" :style="{ width: plant.health + '%' }"></div>
              </div>
            </div>
            
            <div class="stat growth">
              <div class="stat-name">Crecimiento</div>
              <div class="stat-value">{{ Math.min(plant.growth, 100) }}%</div>
              <div class="progress-bar">
                <div class="progress" :style="{ width: Math.min(plant.growth, 100) + '%' }"></div>
              </div>
            </div>
            
            <div class="stat score">
              <div class="stat-name">Puntuación</div>
              <div class="stat-value">{{ score }}/{{ questions.length }}</div>
              <div class="progress-bar">
                <div class="progress" :style="{ width: scoreProgress + '%' }"></div>
              </div>
            </div>
          </div>
        </div>
        
        <div v-if="!gameOver" class="quiz-section">
          <div class="quiz-info">
            <div class="current-question">{{ currentQuestion.question }}</div>
            <div class="question-counter">Pregunta {{ currentQuestionIndex + 1 }} de {{ questions.length }}</div>
          </div>
          
          <div class="cards-container">
            <div 
              class="truth-card" 
              @click="checkAnswer(true)" 
              :class="{ 
                disabled: isChecking || gameOver,
                selected: selectedAnswer === true,
                correct: showFeedback && currentQuestion.correctAnswer === true,
                incorrect: showFeedback && selectedAnswer === true && currentQuestion.correctAnswer === false
              }">
              <div class="card-text">VERDADERO</div>
              <div v-if="showFeedback" class="feedback-icon">
                <span v-if="currentQuestion.correctAnswer === true">✓</span>
                <span v-else-if="selectedAnswer === true">✗</span>
              </div>
            </div>
            
            <div 
              class="false-card" 
              @click="checkAnswer(false)" 
              :class="{ 
                disabled: isChecking || gameOver,
                selected: selectedAnswer === false,
                correct: showFeedback && currentQuestion.correctAnswer === false,
                incorrect: showFeedback && selectedAnswer === false && currentQuestion.correctAnswer === true }">
             
              <div class="card-text">FALSO</div>
              <div v-if="showFeedback" class="feedback-icon">
                <span v-if="currentQuestion.correctAnswer === false">✓</span>
                <span v-else-if="selectedAnswer === false">✗</span>
              </div>
            </div>
          </div>

          <div v-if="showFeedback" class="explanation">
            <p><strong>Explicación:</strong> {{ currentQuestion.explicacion }}</p>
          </div>
        </div>

        <!-- Game Over -->
        <div v-else class="game-over-section">
          <div class="game-over-card">
            <h2 class="game-over-title">¡Juego Terminado!</h2>
            
            <div class="final-results">
              <div class="result-item">
                <span class="result-label">Puntuación Final:</span>
                <span class="result-value">{{ score }}/{{ questions.length }}</span>
              </div>
              
              <div class="result-item">
                <span class="result-label">Porcentaje de Aciertos:</span>
                <span class="result-value">{{ Math.round((score / questions.length) * 100) }}%</span>
              </div>
              
              <div class="result-item">
                <span class="result-label">Etapa Alcanzada:</span>
                <span class="result-value">{{ plantName }}</span>
              </div>
              
              <div class="result-item">
                <span class="result-label">Crecimiento Final:</span>
                <span class="result-value">{{ Math.round(Math.min(plant.growth, 100)) }}%</span>
              </div>
            </div>

            <div class="performance-message" :class="performanceClass">
              <h3>{{ performanceTitle }}</h3>
              <p>{{ performanceDescription }}</p>
            </div>

            <!-- Sección para guardar puntuación -->
            <div class="score-actions">
              <button class="btn-save" @click="guardarPuntuacion" :disabled="puntuacionGuardada || !usuarioLogueado">
                {{ !usuarioLogueado ? 'Inicia sesión para guardar' : puntuacionGuardada ? 'Puntuación Guardada' : 'Guardar Puntuación' }}
              </button>
              <p v-if="!usuarioLogueado" class="login-warning">
                Debes iniciar sesión para guardar tu puntuación
              </p>
            </div>
            
            <div class="game-over-buttons">
              <button @click="volverAlInicio" class="btn-primary">
                Volver al Inicio
              </button>
              <button @click="reintentarJuego" class="btn-warning">Jugar Otra Vez</button>
              <button class="btn-info" @click="mostrarLeaderboard = true">
                Ver posiciones
              </button>
            </div>
          </div>
          <!-- Modal de la tabla líderes -->
          <div v-if="mostrarLeaderboard" class="modal-overlay">
            <div class="modal-content leaderboard-modal">
              <button class="modal-close-btn" @click="mostrarLeaderboard = false">×</button>
              <h2>Tabla de Posiciones</h2>
              <div class="leaderboard-container">
                <LeaderboardGame2 :scores="leaderboard" :showAciertos="true" />
              </div>
              <div class="modal-actions">
                <button class="btn-primary" @click="mostrarLeaderboard = false">
                  Cerrar
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import LeaderboardGame2 from '@/components/leaderboard-Game2.vue';

import etapaInicial from '@/assets/img/etapaInicial.png'
import etapa1 from '@/assets/img/etapa1.png';
import etapa2 from '@/assets/img/etapa2.png';
import etapa3 from '@/assets/img/etapa3.png';
import etapa4 from '@/assets/img/etapa4.png';
import etapa5 from '@/assets/img/etapa5.png';
import etapa6 from '@/assets/img/etapa6.png';
import etapa7 from '@/assets/img/etapa7.png';
import etapa8 from '@/assets/img/etapa8.png';
import etapa9 from '@/assets/img/etapa9.png';
import etapa10 from '@/assets/img/etapa10.png';

export default {
  components: {
    LeaderboardGame2
  },
  data() {
    return {
      juegoActivo: false,
      plant: {
        health: 100,
        growth: 0
      },
      score: 0,
      questionsAnswered: 0,
      message: 'Cargando preguntas...',
      messageType: '',
      currentQuestionIndex: 0,
      questions: [],
      isChecking: false,
      selectedAnswer: null,
      showFeedback: false,
      gameOver: false,
      puntuacionGuardada: false,
      leaderboard: [],
      usuarioLogueado: false,
      usuarioActual: null,
      gameStartTime: null,
      mostrarLeaderboard: false, 
      
      plantImages: {
        tierra: etapaInicial,        
        semilla: etapa1,             
        germinacion: etapa2,         
        plantulaJoven: etapa3,       
        primerasHojas: etapa4,       
        crecimientoVegetativo: etapa5, 
        desarrolloHojasTallo: etapa6, 
        ramificacion: etapa7,        
        floracion: etapa8,           
        fructificacion: etapa9,      
        maduracion: etapa10,         
      },
    };
  },
  computed: {
    plantState() {
      if (this.plant.health >= 70) return 'healthy';
      if (this.plant.health >= 40) return 'warning';
      if (this.plant.health >= 20) return 'sick';
      return 'critical';
    },
    plantImage() {
      const growth = Math.min(this.plant.growth, 100);
      if (growth >= 95) return this.plantImages.maduracion;
      if (growth >= 85) return this.plantImages.fructificacion;
      if (growth >= 75) return this.plantImages.floracion;
      if (growth >= 65) return this.plantImages.ramificacion;
      if (growth >= 55) return this.plantImages.desarrolloHojasTallo;
      if (growth >= 45) return this.plantImages.crecimientoVegetativo;
      if (growth >= 35) return this.plantImages.primerasHojas;
      if (growth >= 25) return this.plantImages.plantulaJoven;
      if (growth >= 15) return this.plantImages.germinacion;
      if (growth >= 5) return this.plantImages.semilla;
      if (growth >= 0) return this.plantImages.tierra; 
      return this.plantImages.tierra;
    },
    plantName() {
      const growth = Math.min(this.plant.growth, 100);
      if (growth >= 95) return 'Maduración';
      if (growth >= 85) return 'Fructificación';
      if (growth >= 75) return 'Floración';
      if (growth >= 65) return 'Ramificación';
      if (growth >= 55) return 'Desarrollo Hojas/Tallo';
      if (growth >= 45) return 'Crecimiento Vegetativo';
      if (growth >= 35) return 'Primeras Hojas Verdes';
      if (growth >= 25) return 'Plántula Joven';
      if (growth >= 15) return 'Germinación';
      if (growth >= 5) return 'Semilla Brotando';
      if (growth >= 0) return 'Tierra Preparada'; 
      return 'Tierra Preparada';
    },
    currentQuestion() {
      return this.questions[this.currentQuestionIndex] || { question: 'Cargando...' };
    },
    scoreProgress() {
      return this.questionsAnswered > 0 ? (this.score / this.questionsAnswered) * 100 : 0;
    },
    performanceClass() {
      const percentage = (this.score / this.questions.length) * 100;
      if (percentage >= 80) return 'excellent';
      if (percentage >= 60) return 'good';
      if (percentage >= 40) return 'average';
      return 'poor';
    },
    performanceTitle() {
      const percentage = (this.score / this.questions.length) * 100;
      if (percentage >= 80) return '¡Excelente Cuidado! ';
      if (percentage >= 60) return 'Buen Trabajo ';
      if (percentage >= 40) return 'Necesita Mejorar';
      return 'Cuidado Insuficiente ';
    },
    performanceDescription() {
      const percentage = (this.score / this.questions.length) * 100;
      if (percentage >= 80) {
        return 'Has demostrado un excelente conocimiento sobre el medio ambiente. ¡Tu planta ha crecido saludablemente!';
      } else if (percentage >= 60) {
        return 'Tienes buen conocimiento, pero hay áreas que puedes mejorar para un mejor cuidado de tus plantas.';
      } else if (percentage >= 40) {
        return 'Tu planta no ha crecido apropiadamente. Revisa los informaciones reales sobre el medio ambiente.';
      } else {
        return 'No has podido ayudar a tu planta a crecer adecuadamente. Es importante aprender más sobre el medio ambiente';
      }
    },
    tiempoJuego() {
      if (!this.gameStartTime) return 0;
      return Math.floor((Date.now() - this.gameStartTime) / 1000);
    }
  },
  methods: {
    verificarAutenticacion() {
      const token = localStorage.getItem('token');
      const usuario = localStorage.getItem('usuario');
      
      if (token && usuario) {
        this.usuarioLogueado = true;
        this.usuarioActual = JSON.parse(usuario);
      } else {
        this.usuarioLogueado = false;
        this.usuarioActual = null;
      }
    },
    
    volverAlInicio() {
      this.juegoActivo = false;
      this.gameOver = false;
      this.cargarLeaderboard();
    },

    reintentarJuego() {
      this.reiniciarJuego();
      this.juegoActivo = true;
    },

    empezarJuego() {
      this.verificarAutenticacion();
      // Reiniciar el estado del juego antes de comenzar para evitar valores residuales
      if (typeof this.reiniciarJuego === 'function') this.reiniciarJuego();
      this.juegoActivo = true;
      this.gameStartTime = Date.now();
    },

    async cargarLeaderboard() {
      try {
        const response = await axios.get(`${process.env.VUE_APP_API_URL}/api/juego2/leaderboard`);
        this.leaderboard = response.data;
      } catch (error) {
        console.error('Error cargando leaderboard:', error);
      }
    },

    async guardarPuntuacion() {
      if (!this.usuarioLogueado) {
        alert('Por favor inicia sesión para guardar tu puntuación');
        return;
      }

      try {
        const token = localStorage.getItem('token');
        
        const puntuacionData = {
          usuario_id: this.usuarioActual.id,
          puntuacion: this.score,
          crecimiento_final: Math.round(Math.min(this.plant.growth, 100)), 
          aciertos: this.score,
          total_preguntas: this.questions.length,
          etapa_alcanzada: this.plantName,
          tiempo_juego: this.tiempoJuego
        };

        const response = await axios.post(`${process.env.VUE_APP_API_URL}/api/juego2/puntuacion`, puntuacionData, {
          headers: {
            'Authorization': `Bearer ${token}`
          }
        });

        if (response.data.success) {
          this.puntuacionGuardada = true;
          this.cargarLeaderboard();
        }
      } catch (error) {
        console.error('Error guardando puntuación:', error);
        alert(`Error ${error.response?.status}: ${error.response?.data?.error || 'Error al guardar la puntuación'}`);
      }
    },

    async loadQuestions() {
      try {
        const response = await axios.get(`${process.env.VUE_APP_API_URL}/api/juego2/questions`);
        this.questions = response.data;
        this.message = 'Responde Verdadero o Falso para ayudar a tu planta a crecer';
      } catch (error) {
        console.error('Error cargando preguntas:', error);
        this.message = 'Error cargando las preguntas. Recarga la página.';
        this.messageType = 'negative';
      }
    },

    async checkAnswer(userAnswer) {
      if (this.isChecking || this.gameOver) return;
      
      this.isChecking = true;
      this.selectedAnswer = userAnswer;
      this.questionsAnswered++;

      try {
        const response = await axios.post(`${process.env.VUE_APP_API_URL}/api/juego2/check-answer`, {
          questionId: this.currentQuestion.id,
          userAnswer
        });
        
        const { isCorrect, explicacion } = response.data;
        
        this.currentQuestion.correctAnswer = response.data.correctAnswer;
        this.currentQuestion.explicacion = explicacion;
        this.showFeedback = true;

        if (isCorrect) {
          this.score++;
          
          // Solo crecimiento base cuando acierta
          this.plant.growth = Math.min(100, this.plant.growth + 10);
          this.plant.health = Math.min(100, this.plant.health + 5);
          
          this.message = `¡Correcto! +10% de crecimiento`;
          this.messageType = 'positive';

        } else {
          this.streak = 0;
          
          let healthLoss = 8;
          if (this.plant.health < 30) healthLoss = 5;
          if (this.plant.health < 15) healthLoss = 3;
          
          this.plant.health = Math.max(0, this.plant.health - healthLoss);
          this.message = `Incorrecto. -${healthLoss}% salud`;
          this.messageType = 'negative';

          if (this.plant.health < 30) {
            this.message += ' | ¡Cuida tu planta!';
          }
        }

        setTimeout(() => {
          if (this.currentQuestionIndex === this.questions.length - 1) {
            this.endGame();
          } else {
            this.nextQuestion();
          }
        }, 3500);
        
      } catch (error) {
        console.error('Error verificando respuesta:', error);
        this.message = 'Error verificando la respuesta. Intenta de nuevo.';
        this.messageType = 'negative';
        this.isChecking = false;
        this.showFeedback = false;
        this.selectedAnswer = null;
      }
    },
    
    nextQuestion() {
      this.currentQuestionIndex++;
      this.message = 'Elige Verdadero o Falso para la siguiente pregunta';
      this.messageType = '';
      this.isChecking = false;
      this.showFeedback = false;
      this.selectedAnswer = null;

      if (this.plant.health < 50 && this.plant.health > 0) {
        const recovery = 2;
        this.plant.health = Math.min(100, this.plant.health + recovery);
        this.message += ` | Tu planta se recupera +${recovery}% salud`;
      }

      if (this.plant.health <= 0) {
        this.endGame(true);
      }
    },

    endGame(plantDied = false) {
      this.gameOver = true;
      this.isChecking = true;

      if (plantDied) {
        this.message = '¡Tu planta ha muerto por falta de cuidados!';
        this.messageType = 'negative';
      } else {
        const growth = Math.min(this.plant.growth, 100);
        if (growth >= 70) {
          this.message = '¡Felicidades! Has logrado que tu planta florezca completamente.';
          this.messageType = 'positive';
        } else if (growth >= 40) {
          this.message = 'Buen trabajo, tu planta ha crecido considerablemente.';
          this.messageType = 'positive';
        } else if (growth >= 20) {
          this.message = 'Tu planta ha crecido, pero no alcanzó su máximo potencial.';
          this.messageType = 'warning';
        } else {
          this.message = 'Tu planta no ha crecido apropiadamente. Necesita más cuidados.';
          this.messageType = 'negative';
        }
      }
    },

    reiniciarJuego() {
      this.plant = {
        health: 100,
        growth: 0
      };
      this.score = 0;
      this.streak = 0;
      this.questionsAnswered = 0;
      this.currentQuestionIndex = 0;
      this.message = '¡Nuevo juego! Responde Verdadero o Falso para hacer crecer tu planta.';
      this.messageType = '';
      this.selectedAnswer = null;
      this.showFeedback = false;
      this.isChecking = false;
      this.gameOver = false;
      this.puntuacionGuardada = false;
      this.gameStartTime = Date.now();
      this.loadQuestions();
    }
  },
  mounted() {
    this.verificarAutenticacion();
    this.loadQuestions();
    this.cargarLeaderboard();
  }
};
</script>
<style scoped>
.initial-screen {
  max-width: 1200px;
  margin: 0 auto;
  margin-top: 10%;
  padding: 0 20px;
}

.initial-screen header {
  text-align: center;
  margin-top: 30px;
  margin-bottom: 30px;
  color: rgb(45, 56, 44);
}

.initial-screen h1 {
  font-size: 2.5rem;
  margin-bottom: 10px;
}

.initial-screen .subtitle {
  font-size: 1.2rem;
  opacity: 0.9;
}

.welcome-section {
  margin-bottom: 30px;
}

.initial-actions {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin: 25px 0;
}

.btn-start-main {
  background: #4CAF50;
  color: white;
  border: none;
  padding: 15px 30px;
  border-radius: 25px;
  cursor: pointer;
  font-weight: 600;
  font-size: 1.1rem;
  transition: background-color 0.3s;
}

.btn-start-main:hover {
  background: #45a049;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  margin-top: 10%;
  padding: 0 20px;
}

header {
  text-align: center;
  margin-top: 30px;
  margin-bottom: 30px;
  color: rgb(45, 56, 44);
}

h1 {
  font-size: 2.5rem;
  margin-bottom: 10px;
}

.subtitle {
  font-size: 1.2rem;
  opacity: 0.9;
}

.game-container {
  display: flex;
  flex-direction: column;
  gap: 30px;
  max-width: 1000px;
  margin: 0 auto;
}

.plant-status {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 25px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
  border: 2px solid #e8f5e8;
  order: 1;
}

.plant-visual {
  text-align: center;
  margin-bottom: 20px;
}

.plant {
  margin-bottom: 10px;
  transition: all 0.5s ease;
}

.plant-img {
  width: 120px;
  height: 120px;
  object-fit: contain;
  transition: all 0.5s ease;
}

.plant-visual h2 {
  margin-top: 15px;
  color: #2E7D32;
  font-size: 1.3rem;
  font-weight: bold;
}

.stats {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
}

.stat {
  background: linear-gradient(135deg, #f8fff8, #e8f5e8);
  padding: 15px;
  border-radius: 12px;
  text-align: center;
  border: 1px solid #c8e6c9;
}

.stat-name {
  font-weight: bold;
  margin-bottom: 8px;
  color: #2E7D32;
  font-size: 0.9rem;
}

.stat-value {
  font-size: 1.2rem;
  font-weight: bold;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
  color: #1b5e20;
}

.streak-fire {
  color: #ff6b35;
  font-size: 1.1em;
}

.progress-bar {
  height: 10px;
  background: #e0e0e0;
  border-radius: 6px;
  margin-top: 8px;
  overflow: hidden;
  border: 1px solid #ddd;
}

.progress {
  height: 100%;
  border-radius: 6px;
  transition: width 0.5s;
}

.health .progress {
  background: linear-gradient(90deg, #4CAF50, #66bb6a);
}

.growth .progress {
  background: linear-gradient(90deg, #FF9800, #ffb74d);
}

.score .progress {
  background: linear-gradient(90deg, #2196F3, #64b5f6);
}

/* Sección del quiz */
.quiz-section {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 25px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
  border: 2px solid #e8f5e8;
  order: 2;
}

.quiz-info {
  background: linear-gradient(135deg, #e8f5e8, #f1f8e9);
  border-radius: 15px;
  padding: 25px;
  margin-bottom: 25px;
  text-align: center;
  border: 1px solid #c8e6c9;
}

.current-question {
  font-size: 1.4rem;
  margin-bottom: 15px;
  font-weight: bold;
  color: #2E7D32;
  line-height: 1.4;
}

.question-counter {
  font-size: 1rem;
  color: #666;
  font-weight: 600;
}

/* Cards de verdadero/falso */
.cards-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-bottom: 25px;
}

.truth-card, .false-card {
  height: 160px;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
  text-align: center;
  position: relative;
  border: 3px solid transparent;
}

.truth-card {
  background: linear-gradient(135deg, #4CAF50, #2E7D32);
  color: white;
}

.false-card {
  background: linear-gradient(135deg, #F44336, #C62828);
  color: white;
}

.truth-card:hover, .false-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.2);
}

.truth-card:active, .false-card:active {
  transform: translateY(-2px);
}

.truth-card.disabled, .false-card.disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.truth-card.selected, .false-card.selected {
  transform: scale(1.03);
  box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.9);
  border: 3px solid rgba(255, 255, 255, 0.8);
}

.truth-card.correct, .false-card.correct {
  box-shadow: 0 0 0 3px #4CAF50, 0 0 25px rgba(76, 175, 80, 0.5);
  border: 3px solid #4CAF50;
}

.truth-card.incorrect, .false-card.incorrect {
  box-shadow: 0 0 0 3px #F44336, 0 0 25px rgba(244, 67, 54, 0.5);
  border: 3px solid #F44336;
}

.feedback-icon {
  position: absolute;
  top: 12px;
  right: 12px;
  font-size: 1.5rem;
  font-weight: bold;
  background: rgba(255, 255, 255, 0.9);
  width: 35px;
  height: 35px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #2E7D32;
}

.false-card .feedback-icon {
  color: #C62828;
}

.card-text {
  font-size: 1.6rem;
  font-weight: bold;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

.explanation {
  background: linear-gradient(135deg, #f1f8e9, #e8f5e8);
  border-radius: 12px;
  padding: 20px;
  margin-top: 20px;
  border-left: 5px solid #4CAF50;
  border: 1px solid #c8e6c9;
}

.explanation p {
  margin: 0;
  color: #333;
  line-height: 1.5;
  font-size: 1rem;
}

.explanation strong {
  color: #2E7D32;
}

.game-message {
  margin-top: 20px;
  padding: 18px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 12px;
  text-align: center;
  font-weight: bold;
  min-height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s;
  font-size: 1.1rem;
  border: 2px solid;
}

.positive {
  color: #4CAF50;
  background: rgba(76, 175, 80, 0.1);
  border-color: rgba(76, 175, 80, 0.3);
}

.negative {
  color: #F44336;
  background: rgba(244, 67, 54, 0.1);
  border-color: rgba(244, 67, 54, 0.3);
}

/* Game Over Section */
.game-over-section {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 30px;
}

.game-over-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 40px;
  text-align: center;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
  max-width: 500px;
  width: 100%;
}

.game-over-title {
  color: #2E7D32;
  font-size: 2.2rem;
  margin-bottom: 30px;
  border-bottom: 3px solid #4CAF50;
  padding-bottom: 15px;
}

.final-results {
  background: #f8f9fa;
  border-radius: 15px;
  padding: 20px;
  margin-bottom: 25px;
}

.result-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #e9ecef;
}

.result-item:last-child {
  border-bottom: none;
}

.result-label {
  font-weight: bold;
  color: #495057;
}

.result-value {
  font-weight: bold;
  color: #2E7D32;
  font-size: 1.1rem;
}

.performance-message {
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 25px;
  border-left: 5px solid;
}

.performance-message.excellent {
  background: rgba(76, 175, 80, 0.1);
  border-left-color: #4CAF50;
  color: #2E7D32;
}

.performance-message.good {
  background: rgba(33, 150, 243, 0.1);
  border-left-color: #2196F3;
  color: #1565C0;
}

.performance-message.average {
  background: rgba(255, 152, 0, 0.1);
  border-left-color: #FF9800;
  color: #EF6C00;
}

.performance-message.poor {
  background: rgba(244, 67, 54, 0.1);
  border-left-color: #F44336;
  color: #C62828;
}

.performance-message h3 {
  margin: 0 0 10px 0;
  font-size: 1.4rem;
}

.performance-message p {
  margin: 0;
  line-height: 1.5;
}

.score-actions {
  margin: 20px 0;
  text-align: center;
}

.btn-save {
  background: #2196F3;
  color: white;
  border: none;
  padding: 12px 25px;
  border-radius: 25px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  width: 100%;
  max-width: 250px;
  font-weight: 600;
}

.btn-save:hover:not(:disabled) {
  background: #1976D2;
  transform: translateY(-2px);
}

.btn-save:disabled {
  background: #ccc;
  cursor: not-allowed;
  transform: none;
}

.login-warning {
  color: #ff6b6b;
  font-size: 0.9rem;
  margin-top: 10px;
  text-align: center;
}

.game-over-buttons {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
  justify-content: center;
}

.btn-primary, .btn-warning, .btn-info {
  padding: 12px 24px;
  border-radius: 25px;
  cursor: pointer;
  font-weight: 600;
  border: none;
  font-size: 1rem;
  transition: all 0.3s ease;
  min-width: 150px;
}

.btn-primary {
  background: #4CAF50;
  color: white;
}

.btn-warning {
  background: #FF9800;
  color: white;
}

.btn-info {
  background: #2196F3;
  color: white;
}

.btn-primary:hover, .btn-warning:hover, .btn-info:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

/* Modals */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 1rem;
}

.modal-content {
  background: white;
  border-radius: 12px;
  max-width: 800px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  padding: 2rem;
  position: relative;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.2);
}

.modal-close-btn {
  position: absolute;
  top: 15px;
  right: 15px;
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: #7f8c8d;
  transition: color 0.2s;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-close-btn:hover{
  color: #e74c3c;
}

.leaderboard-modal {
  max-width: 600px;
}

.leaderboard-container {
  margin: 20px 0;
  max-height: 400px;
  overflow-y: auto;
}

.modal-content h2 {
  padding-right: 40px; 
}

.modal-actions {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-top: 15px;
  flex-wrap: wrap;
}

/* Desktop (1024px+) - Vista side-by-side */
@media (min-width: 1024px) {
  .game-container {
    flex-direction: row;
    align-items: flex-start;
  }
  
  .plant-status {
    flex: 0 0 350px;
    position: sticky;
    top: 20px;
  }
  
  .quiz-section {
    flex: 1;
  }
}

/* Tablets (768px - 1023px) */
@media (max-width: 1023px) and (min-width: 768px) {
  .game-container {
    flex-direction: row;
    align-items: flex-start;
  }
  
  .plant-status {
    flex: 0 0 300px;
  }
  
  .quiz-section {
    flex: 1;
  }
  
  .plant-img {
    width: 100px;
    height: 100px;
  }
  
  .stats {
    grid-template-columns: 1fr;
  }
}

/* Móviles (hasta 767px) - Una columna, diseño vertical */
@media (max-width: 767px) {
  .game-container {
    gap: 20px;
  }
  
  .plant-status,
  .quiz-section {
    padding: 20px;
  }
  
  .plant-img {
    width: 100px;
    height: 100px;
  }
  
  .stats {
    grid-template-columns: repeat(2, 1fr);
    gap: 12px;
  }
  
  .stat {
    padding: 12px;
  }
  
  .stat-name {
    font-size: 0.85rem;
  }
  
  .stat-value {
    font-size: 1.1rem;
  }
  
  .truth-card, .false-card {
    height: 140px;
    padding: 15px;
  }
  
  .card-text {
    font-size: 1.4rem;
  }
  
  .current-question {
    font-size: 1.2rem;
  }
}

/* Móviles pequeños (480px - 599px) */
@media (max-width: 599px) {
  .container,
  .initial-screen {
    padding: 0 15px;
  }
  
  h1 {
    font-size: 2rem;
  }
  
  .subtitle {
    font-size: 1.1rem;
  }
  
  .plant-status,
  .quiz-section {
    padding: 18px;
    border-radius: 16px;
  }
  
  .plant-img {
    width: 90px;
    height: 90px;
  }
  
  .stats {
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
  }
  
  .stat {
    padding: 10px;
  }
  
  .truth-card, .false-card {
    height: 130px;
    padding: 12px;
  }
  
  .card-text {
    font-size: 1.3rem;
  }
  
  .current-question {
    font-size: 1.1rem;
  }
}

/* Móviles muy pequeños (hasta 479px) */
@media (max-width: 479px) {
  .container,
  .initial-screen {
    padding: 0 10px;
  }
  
  h1 {
    font-size: 1.8rem;
  }
  
  .plant-status,
  .quiz-section {
    padding: 15px;
    border-radius: 14px;
  }
  
  .plant-img {
    width: 80px;
    height: 80px;
  }
  
  .plant-visual h2 {
    font-size: 1.1rem;
  }
  
  .stats {
    grid-template-columns: 1fr;
    gap: 8px;
  }
  
  .truth-card, .false-card {
    height: 120px;
    padding: 10px;
  }
  
  .card-text {
    font-size: 1.2rem;
  }
  
  .current-question {
    font-size: 1rem;
    line-height: 1.3;
  }
  
  .cards-container {
    gap: 15px;
  }
  
  .game-over-buttons {
    flex-direction: column;
    align-items: center;
  }
  
  .btn-primary, .btn-warning, .btn-info {
    width: 100%;
    max-width: 250px;
  }
}

/* Orientación landscape en móviles */
@media (max-height: 500px) and (orientation: landscape) {
  .plant-img {
    width: 70px;
    height: 70px;
  }
  
  .truth-card, .false-card {
    height: 100px;
  }
  
  .card-text {
    font-size: 1.1rem;
  }
  
  .stats {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>