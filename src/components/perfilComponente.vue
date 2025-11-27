<template>
    <!-- Mostrar contenido solo si está autenticado -->
    <template v-if="isAuthenticated">
        <div class="header-perfil">
            <h1><i class="fas fa-leaf"></i> Hola, {{ usuario.usuario }}</h1> 
        </div>
        
        <div class="contenido-perfil">
            <div class="seccion-derecha">
                <div class="seccion-habitos">
                    <h2>
                        <i class="fas fa-leaf"></i> 
                        Mis Hábitos ({{ habitosUsuario.length }}/{{ maxHabitos }})
                    </h2>
                    
                    <!-- Lista de hábitos cuando hay hábitos -->
                    <div v-if="habitosUsuario.length > 0" class="lista-habitos">
                        <div 
                            v-for="(habito, index) in habitosUsuario" 
                            :key="habito.id"
                            class="item-habito"
                            :style="{ borderLeft: `4px solid ${obtenerColorBadge(index)}` }">
                        
                            <div class="habito-header">
                                <h3>{{ habito.title }}</h3>
                                <button 
                                    @click="eliminarHabito(index)"
                                    class="btn-eliminar"
                                    title="Eliminar hábito">
                              
                                    Eliminar
                                </button>
                            </div>
                            
                            <p class="habito-descripcion">{{ habito.desc }}</p>
                            
                            <div class="habito-info">
                                <small class="fecha-agregado">
                                    <i class="fas fa-calendar-plus"></i>
                                    Agregado el {{ formatearFecha(habito.fechaAgregado) }}
                                </small>
                            </div>
                        </div>
                    </div>
                    
                    <!-- Mensaje cuando no hay hábitos -->
                    <div v-else class="sin-habitos">
                        <i class="fas fa-seedling fa-2x"></i>
                        <p>Aún no has agregado ningún hábito ecológico.</p>
                        <p>¡Ve a la sección de hábitos para comenzar!</p>
                    </div>
                    
                    <!-- Información del límite -->
                    <div v-if="habitosUsuario.length < maxHabitos" class="info-limite">
                        <small>
                            <i class="fas fa-info-circle"></i>
                            Puedes agregar {{ maxHabitos - habitosUsuario.length }} hábito(s) más
                        </small>
                    </div>
                    
                    <!-- Advertencia cuando se alcanza el límite -->
                    <div v-else-if="habitosUsuario.length === maxHabitos" class="limite-alcanzado">
                        <small>
                            <i class="fas fa-check-circle"></i>
                            Has alcanzado el límite máximo de hábitos
                        </small>
                    </div>
                </div>
                
                <EstadisticaComponente></EstadisticaComponente>
            </div>
       
            <div class="seccion-izquierda">
                <formsPerfilComponente></formsPerfilComponente>
            </div> 
        </div>
    </template>
</template>

<script>
import EstadisticaComponente from './estadisticaComponente.vue';
import formsPerfilComponente from './formsPerfilComponente.vue';
import axios from 'axios';

export default {
    components: {
        formsPerfilComponente, 
        EstadisticaComponente
    },
    
    data() {
        return {
            isAuthenticated: true, 
            usuario: {
                usuario: '',
                id: null  
            },
            usuarioId: null,
            habitosUsuario: [],
            maxHabitos: 3,
            cargando: false
        };
    },
        
        async mounted() {
            await this.obtenerPerfil();
            this.cargarHabitos();
            globalThis.addEventListener('perfilActualizado', this.onPerfilActualizado);
            globalThis.addEventListener('userUpdated', this.onPerfilActualizado);
            globalThis.addEventListener('habitoAgregado', this.onHabitoAgregado);
        },
            
        beforeUnmount() {
            globalThis.removeEventListener('perfilActualizado', this.onPerfilActualizado);
            globalThis.removeEventListener('userUpdated', this.onPerfilActualizado);   
            globalThis.removeEventListener('habitoAgregado', this.onHabitoAgregado);
        },
    methods: {    
        onPerfilActualizado() {
            this.obtenerPerfil();
        },
        
        obtenerToken() {
            const token = localStorage.getItem('token');
            return token;
        },
        
        async obtenerPerfil() {
            try {
                this.cargando = true;
                const token = this.obtenerToken();
                
                if (!token) {
                    console.warn('No hay token disponible');
                    this.usuario.usuario = 'Usuario';
                    return;
                }

                const response = await axios.get(`${process.env.VUE_APP_API_URL}/api/perfil`, {
                    headers: {
                        'Authorization': `Bearer ${token}`
                    }
                });
                
                if (response.data) {
                    if (response.data.user && response.data.user.usuario) {
                        this.usuario = response.data.user;
                        // IMPORTANTE: Establecer el usuarioId después de obtener el perfil
                        this.usuarioId = response.data.user.id || response.data.user.correo || response.data.user.usuario;
                        // Cargar hábitos con el nuevo usuarioId
                        this.cargarHabitos();
                    } else {
                        this.usuario.usuario = 'Usuario';
                    }
                }
                
            } catch (error) {
                console.error('Error al obtener perfil:', error);
                this.usuario.usuario = 'Usuario';
            } finally {
                this.cargando = false;
            }
        },
        
        /**
         * Cargar hábitos del localStorage (específicos del usuario)
         */
        cargarHabitos() {
            if (!this.usuarioId) {
                console.warn('No hay usuario identificado para cargar hábitos');
                this.habitosUsuario = [];
                return;
            }
            
            const claveHabitos = `userHabitos_${this.usuarioId}`;
            const habitos = JSON.parse(localStorage.getItem(claveHabitos) || '[]');
            this.habitosUsuario = habitos;
            console.log(`Hábitos cargados para usuario ${this.usuarioId}:`, habitos);
        },

        /**
         * Manejar cuando se agrega un nuevo hábito
         */
        onHabitoAgregado() {
            this.cargarHabitos();
        },

        /**
         * Eliminar un hábito específico
         */
        eliminarHabito(index) {
            if (!this.usuarioId) {
                alert('Error: No se puede eliminar el hábito');
                return;
            }
            
            if (confirm('¿Estás seguro de que quieres eliminar este hábito?')) {
                this.habitosUsuario.splice(index, 1);
                
                const claveHabitos = `userHabitos_${this.usuarioId}`;
                localStorage.setItem(claveHabitos, JSON.stringify(this.habitosUsuario));
                
                globalThis.dispatchEvent(new CustomEvent('habitoEliminado'));
            }
        },

        /**
         * Obtener el color del badge según el índice
         */
        obtenerColorBadge(index) {
            const colores = ['#4caf50', '#2196f3', '#ff9800'];
            return colores[index % colores.length];
        },
        
        /**
         * Formatear fecha para mostrar
         */
        formatearFecha(fechaISO) {
            const fecha = new Date(fechaISO);
            return fecha.toLocaleDateString('es-ES', {
                year: 'numeric',
                month: 'short', 
                day: 'numeric'
            });
        },
        
        redirectToLogin() {
            this.$router.push('/login');
        }
    }
}
</script>

<style scoped>
.caja-perfil {
    background-color: white;
    border-radius: 15px;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
    font-family: 'Poppins', sans-serif;
    max-width: 600px;
    width: 95%;
    margin: 0 auto;
    padding: 25px;
    box-sizing: border-box;
}

.header-perfil {
    color: rgb(113, 185, 85);
    padding: 25px;
    text-align: center;
    font-family: 'Poppins', sans-serif;  
    border-radius: 20px;
    max-width: 1000px;
    margin: 0 auto;
}

.header-perfil h1 {
  margin-top: 10%;
  font-size: 2.3rem;
  font-weight: bold;
  color: #0b6d11;
  display: inline-flex;
  align-items: center;
  gap: 10px; 
}

.contenido-perfil {
    display: grid;
    grid-template-columns: 2fr 1fr;
    gap: 30px;
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
}

.seccion-izquierda {
    display: flex;
    flex-direction: column;
    gap: 30px;
}

.seccion-derecha {
    display: grid;
    grid-template-rows: auto 1fr;
    gap: 30px;
}

.seccion-habitos {
    padding: 25px;
    background-color: #f9f9f9;
    border-radius: 15px;
    box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
    font-family: 'Poppins', sans-serif;
    position: relative;
}

.seccion-habitos h2 {
    color: #333;
    margin-top: 0;
    margin-bottom: 20px;
    padding-bottom: 12px;
    border-bottom: 2px solid #e0e0e0;
    display: flex;
    align-items: center;
    gap: 10px;
    font-size: 1.4rem;
}

.lista-habitos {
    display: flex;
    flex-direction: column;
    gap: 15px;
}

.item-habito {
    background: white;
    border: 1px solid #e0e0e0;
    border-radius: 12px;
    padding: 20px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    transition: all 0.3s ease;
    position: relative;
}

.item-habito:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 16px rgba(0,0,0,0.12);
}

.habito-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 12px;
    gap: 15px;
}

.habito-header h3 {
    margin: 0;
    color: #2e7d32;
    font-size: 1.2rem;
    flex: 1;
    line-height: 1.3;
}

.btn-eliminar {
    background: #FF4742;
    border: 1px solid #FF4742;
    border-radius: 8px;
    box-shadow: rgba(0, 0, 0, 0.1) 1px 2px 4px;
    box-sizing: border-box;
    color: #FFFFFF;
    cursor: pointer;
    font-size: 12px;
    font-weight: 600;
    line-height: 1;
    min-height: 35px;
    padding: 10px 14px;
    text-align: center;
    transition: all 0.2s ease;
    white-space: nowrap;
}

.btn-eliminar:hover {
    background: #d32f2f;
    transform: translateY(-1px);
    box-shadow: 0 4px 8px rgba(255, 71, 66, 0.3);
}

.habito-descripcion {
    color: #666;
    font-size: 0.95rem;
    line-height: 1.5;
    margin: 12px 0;
}

.habito-info {
    margin-top: 15px;
    padding-top: 12px;
    border-top: 1px solid #f0f0f0;
}

.fecha-agregado {
    color: #888;
    font-size: 0.85rem;
    display: flex;
    align-items: center;
    gap: 6px;
}

.sin-habitos {
    text-align: center;
    padding: 40px 30px;
    color: #666;
    background: #f8f9fa;
    border-radius: 12px;
    margin: 20px 0;
    border: 2px dashed #ddd;
}

.sin-habitos i {
    color: #4caf50;
    margin-bottom: 15px;
    opacity: 0.7;
}

.sin-habitos p {
    margin: 8px 0;
    line-height: 1.4;
}

.info-limite {
    background: #e8f5e9;
    padding: 12px 16px;
    border-radius: 8px;
    margin-top: 15px;
    color: #2e7d32;
    text-align: center;
    border: 1px solid #c8e6c9;
}

.limite-alcanzado {
    background: #fff3cd;
    padding: 12px 16px;
    border-radius: 8px;
    margin-top: 15px;
    color: #856404;
    text-align: center;
    border: 1px solid #ffeaa7;
}

.info-limite i,
.limite-alcanzado i {
    margin-right: 8px;
}

.contenido-historial {
    background-color: #f9f9f9;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.no-autenticado {
    text-align: center;
    padding: 50px;
    color: #666;
}

.btn-login {
    background: linear-gradient(to right, #58a21c, #49ae27);
    color: white;
    border: none;
    padding: 14px 28px;
    border-radius: 10px;
    cursor: pointer;
    font-size: 1rem;
    margin-top: 20px;
    transition: all 0.3s ease;
}

.btn-login:hover {
    background: linear-gradient(to right, #27ae60, #219653);
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(39, 174, 96, 0.3);
}

/* ========== MEDIA QUERIES RESPONSIVE ========== */

/* Tablets (768px - 1024px) - MANTENER 2 COLUMNAS */
@media (max-width: 1024px) {
    .contenido-perfil {
        grid-template-columns: 2fr 1fr;
        gap: 20px;
        padding: 0 15px;
    }
    
    .seccion-izquierda {
        gap: 20px;
    }
    
    .seccion-derecha {
        gap: 20px;
    }
    
    .header-perfil {
        padding: 20px;
    }
    
    .header-perfil h1 {
        font-size: 2.3rem;
    }
    
    .seccion-habitos {
        padding: 18px;
    }
    
    .seccion-habitos h2 {
        font-size: 1.2rem;
    }
    
    .item-habito {
        padding: 16px;
    }
}

/* Móviles grandes (600px - 767px) - MANTENER 2 COLUMNAS CON AJUSTES */
@media (max-width: 767px) {
    .contenido-perfil {
        grid-template-columns: 1fr 1fr;
        gap: 15px;
        padding: 0 10px;
    }
    
    .seccion-izquierda {
        gap: 15px;
    }
    
    .seccion-derecha {
        grid-template-rows: auto auto;
        gap: 15px;
    }
    
    .header-perfil {
        padding: 15px;
    }
    
    .header-perfil h1 {
        font-size: 2.3rem;
    }
    
    .seccion-habitos {
        padding: 15px;
        border-radius: 10px;
    }
    
    .seccion-habitos h2 {
        font-size: 1.1rem;
        flex-direction: row;
        align-items: center;
        gap: 8px;
    }
    
    .item-habito {
        padding: 12px;
        border-radius: 8px;
    }
    
    .habito-header {
        flex-direction: column;
        align-items: flex-start;
        gap: 10px;
    }
    
    .btn-eliminar {
        align-self: stretch;
        min-height: 35px;
        font-size: 11px;
        padding: 8px 12px;
    }
    
    .habito-header h3 {
        font-size: 1rem;
    }
    
    .habito-descripcion {
        font-size: 0.9rem;
    }
}

/* Móviles pequeños (480px - 599px) - MANTENER 2 COLUMNAS CON MÁS AJUSTES */
@media (max-width: 599px) {
    .caja-perfil {
        padding: 15px 10px;
        border-radius: 12px;
    }
    
    .contenido-perfil {
        grid-template-columns: 1fr 1fr;
        gap: 12px;
        padding: 0 8px;
    }
    
    .seccion-izquierda {
        gap: 12px;
    }
    
    .seccion-derecha {
        grid-template-rows: auto auto;
        gap: 12px;
    }
    
    .header-perfil {
        padding: 12px;
        border-radius: 15px;
    }
    
    .header-perfil h1 {
        font-size: 2.3rem;
    }
    
    .seccion-habitos {
        padding: 12px;
        border-radius: 8px;
    }
    
    .seccion-habitos h2 {
        font-size: 1rem;
        margin-bottom: 12px;
        padding-bottom: 8px;
    }
    
    .item-habito {
        padding: 10px;
        border-radius: 6px;
    }
    
    .habito-header h3 {
        font-size: 0.9rem;
    }
    
    .habito-descripcion {
        font-size: 0.85rem;
        margin: 8px 0;
    }
    
    .sin-habitos {
        padding: 20px 15px;
    }
    
    .info-limite,
    .limite-alcanzado {
        padding: 8px 10px;
        font-size: 0.8rem;
    }
    
    .btn-eliminar {
        min-height: 32px;
        font-size: 10px;
        padding: 6px 10px;
    }
}

/* Móviles muy pequeños (hasta 479px) - MANTENER 2 COLUMNAS MÍNIMAS */
@media (max-width: 479px) {
    .caja-perfil {
        padding: 12px 8px;
        margin: 8px auto;
        width: 98%;
    }
    
    .contenido-perfil {
        grid-template-columns: 1fr 1fr;
        gap: 10px;
        padding: 0 5px;
    }
    
    .seccion-izquierda {
        gap: 10px;
    }
    
    .seccion-derecha {
        grid-template-rows: auto auto;
        gap: 10px;
    }
    
    .header-perfil {
        padding: 10px;
        border-radius: 12px;
    }
    
    .header-perfil h1 {
        font-size: 2.3rem;
    }
    
    .seccion-habitos {
        padding: 10px 8px;
        border-radius: 6px;
    }
    
    .seccion-habitos h2 {
        font-size: 0.9rem;
        margin-bottom: 10px;
        padding-bottom: 6px;
    }
    
    .item-habito {
        padding: 8px;
        border-radius: 5px;
    }
    
    .habito-header {
        gap: 8px;
    }
    
    .habito-header h3 {
        font-size: 0.85rem;
    }
    
    .habito-descripcion {
        font-size: 0.8rem;
        margin: 6px 0;
    }
    
    .habito-info {
        margin-top: 8px;
        padding-top: 6px;
    }
    
    .fecha-agregado {
        font-size: 0.75rem;
    }
    
    .sin-habitos {
        padding: 15px 10px;
    }
    
    .sin-habitos i {
        font-size: 1.2rem;
    }
    
    .btn-eliminar {
        min-height: 30px;
        font-size: 9px;
        padding: 5px 8px;
    }
}

@media (max-height: 500px) and (orientation: landscape) {
    .contenido-perfil {
        grid-template-columns: 1fr 1fr;
        gap: 12px;
    }
    
    .seccion-derecha {
        grid-template-rows: auto auto;
    }
    
    .seccion-habitos {
        padding: 10px;
    }
    
    .item-habito {
        padding: 8px;
    }
    
    .habito-header {
        flex-direction: row;
        align-items: center;
    }
    
    .btn-eliminar {
        min-height: 28px;
        font-size: 9px;
        padding: 4px 6px;
    }
}

/* Ajustes específicos para pantallas muy estrechas */
@media (max-width: 360px) {
    .contenido-perfil {
        grid-template-columns: 1fr 1fr;
        gap: 8px;
    }
    
    .seccion-habitos h2 {
        font-size: 0.85rem;
    }
    
    .habito-header h3 {
        font-size: 0.8rem;
    }
    
    .btn-eliminar {
        font-size: 8px;
        padding: 4px 6px;
    }
}
</style>
