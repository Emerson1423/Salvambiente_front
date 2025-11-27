<template>
    <div class="container-avatar">
        <div class="avatar-preview">
            <div class="avatar-container">
                <img :src="user.personajeBase" class="avatar-base" alt="Personaje base">
            </div>
        </div>

        <div v-if="seccionActual === 'personajes'" class="opciones-container">
            <h3>Elige un personaje</h3>
            <div class="opciones-grid">
                <div v-for="personaje in opcionesPersonajes" 
                     :key="personaje.id" 
                     class="opcion-item"
                     :class="{selected: user.personajeBase === personaje.image}"
                     @click="seleccionarPersonaje(personaje)">
                    <img :src="personaje.image" :alt="personaje.nombre" class="opcion-imagen-personaje">
                    <span class="opcion-nombre">{{ personaje.nombre }}</span>
                </div>
            </div>
            
            <button class="card-btn" @click="updateAvatar">Seleccionar</button>
        </div>
    </div>
</template>

<script>
export default {   
    data(){
        return {
            user:{
                personajeBase: require('@/assets/perfil/reemplazarP.png')
            },
            personajeSeleccionado: null,
            
            opcionesPersonajes: [
                { id: 1, image: require('@/assets/perfil/reciclarP.png'), nombre: "Recicla" },
                { id: 2, image: require('@/assets/perfil/reemplazarP.png'), nombre: "Reemplaza" },
                { id: 3, image: require('@/assets/perfil/reduceP.png'), nombre: "Reduce"},     
                { id: 4, image: require('@/assets/perfil/renuevaP.png'), nombre: "Renueva" },      
                { id: 5, image: require('@/assets/perfil/reutilizarP.png'), nombre: "Reutiliza" },    
            ],
            seccionActual: 'personajes'
        };
    },

    methods: {
        seleccionarPersonaje(personaje) {
            this.user.personajeBase = personaje.image;
            this.personajeSeleccionado = personaje;
        },
        
        async updateAvatar() {
            try {
                
                if (!this.user.personajeBase) {
                    alert('Por favor selecciona un personaje primero');
                    return;
                }
              
                const avatarData = {
                    personajeBase: this.user.personajeBase
                };
              localStorage.setItem('avatarPreferencias', JSON.stringify(avatarData));

              globalThis.dispatchEvent(new Event('storage'));
              console.log('Avatar guardado:', this.user.personajeBase);
                
            } catch (error) {
                console.error('Error al cambiar avatar:', error);
                alert('Error al cambiar el avatar');
            }
        },

        cargarPreferencias() {
            const guardado = localStorage.getItem('avatarPreferencias');
            if (guardado) {
                const datos = JSON.parse(guardado);
                this.user.personajeBase = datos.personajeBase || this.user.personajeBase;
            }
        }
    },
    
    mounted() {
        this.cargarPreferencias();
    }
}
</script>

<style scoped>
.container-avatar {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 15px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  font-family: 'Poppins', sans-serif;  
  margin: 20px 0;
  width: 100%;
  box-sizing: border-box;
}

.avatar-preview {
  margin-bottom: 20px;
  text-align: center;
}

.avatar-container {
  width: 120px;
  height: 160px;
  position: relative;
  border-radius: 12px;
  overflow: hidden;
  border: 3px solid #2ecc71;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
  margin: 0 auto;
}

.avatar-base {
  position: absolute;
  width: 100%;
  height: 100%;
  object-fit: contain;
  top: 0;
  left: 0;
}

.selector-seccion {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
  justify-content: center;
  width: 100%;
}

.selector-seccion button {
  padding: 10px 16px;
  border: none;
  border-radius: 8px;
  background-color: #e0e0e0;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.9rem;
  font-weight: 500;
}

.selector-seccion button:hover {
  background-color: #d0d0d0;
  transform: translateY(-2px);
}

.opciones-container {
  width: 100%;
  padding: 0 0 20px 0;
}

.opciones-container h3 {
  text-align: center;
  color: #2c3e50;
  margin-bottom: 25px;
  font-size: 1.3rem;
  font-weight: 600;
}

.opciones-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  margin-bottom: 30px;
  padding: 0 10px;
}

.opcion-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 15px 10px;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid transparent;
  background: white;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.08);
}

.opcion-item:hover {
  background-color: #f8f9fa;
  transform: translateY(-3px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.opcion-item.selected {
  border-color: #27ae60;
  background-color: #e8f5e9;
  transform: scale(1.02);
}

.opcion-imagen-personaje {
  width: 70px;
  height: 70px;
  object-fit: contain;
  margin-bottom: 10px;
  transition: transform 0.3s ease;
}

.opcion-item:hover .opcion-imagen-personaje {
  transform: scale(1.1);
}

.opcion-imagen {
  width: 50px;
  height: 50px;
  object-fit: contain;
  margin-bottom: 8px;
}

.opcion-nombre {
  font-size: 0.85rem;
  text-align: center;
  color: #555;
  font-weight: 500;
  margin-top: 5px;
}

.card-btn {
  background: #549d18;
  color: #fff;
  border: none;
  border-radius: 10px;
  padding: 14px 30px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  font-size: 1rem;
  display: block;
  margin: 30px auto 0 auto;
  min-width: 180px;
  box-shadow: 0 4px 8px rgba(84, 157, 24, 0.3);
}

.card-btn:hover {
  background: #52a973;
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(82, 169, 115, 0.4);
}

.card-btn:active {
  transform: translateY(0);
}

/* ========== MEDIA QUERIES RESPONSIVE ========== */

/* Tablets (768px - 1024px) */
@media (max-width: 1024px) {
  .container-avatar {
    padding: 18px;
    margin: 15px 0;
  }
  
  .avatar-container {
    width: 110px;
    height: 150px;
  }
  
  .opciones-grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 18px;
  }
  
  .opcion-imagen-personaje {
    width: 65px;
    height: 65px;
  }
  
  .card-btn {
    padding: 13px 28px;
    min-width: 170px;
  }
}

/* Móviles grandes (600px - 767px) */
@media (max-width: 767px) {
  .container-avatar {
    padding: 15px;
    margin: 12px 0;
    border-radius: 12px;
  }
  
  .avatar-container {
    width: 100px;
    height: 140px;
  }
  
  .opciones-container h3 {
    font-size: 1.2rem;
    margin-bottom: 20px;
  }
  
  .opciones-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 15px;
    padding: 0 5px;
  }
  
  .opcion-item {
    padding: 12px 8px;
    border-radius: 10px;
  }
  
  .opcion-imagen-personaje {
    width: 60px;
    height: 60px;
  }
  
  .opcion-nombre {
    font-size: 0.8rem;
  }
  
  .card-btn {
    padding: 12px 25px;
    min-width: 160px;
    font-size: 0.95rem;
    margin-top: 25px;
  }
}

/* Móviles pequeños (480px - 599px) */
@media (max-width: 599px) {
  .container-avatar {
    padding: 12px;
    margin: 10px 0;
    border-radius: 10px;
  }
  
  .avatar-preview {
    margin-bottom: 15px;
  }
  
  .avatar-container {
    width: 90px;
    height: 130px;
    border-width: 2px;
  }
  
  .opciones-container h3 {
    font-size: 1.1rem;
    margin-bottom: 18px;
  }
  
  .opciones-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 12px;
    padding: 0;
  }
  
  .opcion-item {
    padding: 10px 6px;
    border-radius: 8px;
  }
  
  .opcion-imagen-personaje {
    width: 55px;
    height: 55px;
    margin-bottom: 8px;
  }
  
  .opcion-nombre {
    font-size: 0.75rem;
  }
  
  .card-btn {
    padding: 11px 22px;
    min-width: 150px;
    font-size: 0.9rem;
    margin-top: 20px;
    border-radius: 8px;
  }
  
  .selector-seccion {
    gap: 8px;
    margin-bottom: 15px;
  }
  
  .selector-seccion button {
    padding: 8px 12px;
    font-size: 0.85rem;
  }
}

/* Móviles muy pequeños (hasta 479px) */
@media (max-width: 479px) {
  .container-avatar {
    padding: 10px;
    margin: 8px 0;
    border-radius: 8px;
  }
  
  .avatar-container {
    width: 80px;
    height: 120px;
  }
  
  .opciones-container h3 {
    font-size: 1rem;
    margin-bottom: 15px;
  }
  
  .opciones-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
  }
  
  .opcion-item {
    padding: 8px 4px;
    border-radius: 6px;
  }
  
  .opcion-imagen-personaje {
    width: 50px;
    height: 50px;
    margin-bottom: 6px;
  }
  
  .opcion-nombre {
    font-size: 0.7rem;
  }
  
  .card-btn {
    padding: 10px 20px;
    min-width: 140px;
    font-size: 0.85rem;
    margin-top: 15px;
    border-radius: 6px;
  }
  
  .selector-seccion {
    flex-direction: column;
    align-items: center;
    gap: 6px;
  }
  
  .selector-seccion button {
    width: 100%;
    max-width: 200px;
    padding: 9px 12px;
  }
}

/* Orientación landscape en móviles */
@media (max-height: 500px) and (orientation: landscape) {
  .container-avatar {
    padding: 15px;
    margin: 10px 0;
  }
  
  .avatar-container {
    width: 70px;
    height: 100px;
  }
  
  .opciones-grid {
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
    margin-bottom: 20px;
  }
  
  .opcion-item {
    padding: 8px 4px;
  }
  
  .opcion-imagen-personaje {
    width: 45px;
    height: 45px;
    margin-bottom: 5px;
  }
  
  .opcion-nombre {
    font-size: 0.7rem;
  }
  
  .card-btn {
    padding: 8px 16px;
    min-width: 120px;
    font-size: 0.8rem;
    margin-top: 15px;
  }
}

/* Pantallas muy estrechas (hasta 360px) */
@media (max-width: 360px) {
  .container-avatar {
    padding: 8px;
  }
  
  .avatar-container {
    width: 70px;
    height: 110px;
  }
  
  .opciones-grid {
    grid-template-columns: 2fr;
    gap: 8px;
  }
  
  .opcion-item {
    flex-direction: row;
    justify-content: flex-start;
    padding: 8px 12px;
    gap: 10px;
  }
  
  .opcion-imagen-personaje {
    width: 40px;
    height: 40px;
    margin-bottom: 0;
    flex-shrink: 0;
  }
  
  .opcion-nombre {
    font-size: 0.75rem;
    text-align: left;
    margin-top: 0;
  }
  
  .card-btn {
    width: 100%;
    max-width: 200px;
  }
}

/* Asegurar que las imágenes se carguen correctamente */
.avatar-base,
.opcion-imagen-personaje,
.opcion-imagen {
  image-rendering: -webkit-optimize-contrast;
  image-rendering: crisp-edges;
}

/* Mejoras de accesibilidad */
.opcion-item:focus {
  outline: 2px solid #27ae60;
  outline-offset: 2px;
}

.card-btn:focus {
  outline: 2px solid #2ecc71;
  outline-offset: 2px;
}
</style>