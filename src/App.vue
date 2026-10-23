<script setup>
import { ref, computed, onMounted } from 'vue'

// Variables reactivas
const countries = ref([])
const loading = ref(true)
const error = ref(false)
const busqueda = ref('')

// Consumo de API directo y sin bloqueos
const fetchCountries = async () => {
  try {
    // Usamos el endpoint público v3.1 (sin proxy y sin API Key). 
    // El getTime() evita que el navegador recicle errores viejos de la caché.
    const response = await fetch(`https://restcountries.com/v3.1/all?timestamp=${new Date().getTime()}`);
    
    if (!response.ok) throw new Error('Error en la red');
    
    const data = await response.json();
    
    // Asignamos la lista completa de más de 250 países
    countries.value = Array.isArray(data) ? data : [];
    
    console.log("Países cargados exitosamente:", countries.value.length);
  } catch (err) {
    error.value = true;
    console.error("Fallo al cargar la API:", err);
  } finally {
    loading.value = false;
  }
}

// Filtro computado protegido
const paisesFiltrados = computed(() => {
  if (!Array.isArray(countries.value)) return [];
  
  return countries.value.filter(pais => {
    const nombrePais = pais.name?.common || '';
    return nombrePais.toLowerCase().includes(busqueda.value.toLowerCase());
  });
})

// Llamada en onMounted
onMounted(() => {
  fetchCountries()
})
</script>

<template>
  <main class="container">
    <h1>Explorador de Países</h1>

    <!-- v-model para búsqueda -->
    <div class="search-box">
      <input 
        type="text" 
        v-model="busqueda" 
        placeholder="Buscar país por nombre..." 
      />
    </div>

    <!-- v-if: Indicador de carga -->
    <div v-if="loading" class="loading">
      <p>⏳ Cargando países...</p>
    </div>

    <!-- v-if: Manejo de errores -->
    <div v-else-if="error" class="error">
      <p>❌ No fue posible obtener los datos de la API.</p>
    </div>

    <!-- Contenedor principal -->
    <div v-else>
      <div v-if="paisesFiltrados.length === 0">
        <p>No se encontraron resultados para "{{ busqueda }}"</p>
      </div>
      
      <div class="grid">
        <!-- v-for: Lista de países -->
        <div v-for="pais in paisesFiltrados" :key="pais.cca3" class="card">
          <!-- v-bind (:src y :alt) -->
          <img :src="pais.flags.svg" :alt="'Bandera de ' + pais.name.common" class="flag" />
          <h2>{{ pais.name.common }}</h2>
          <p><strong>Capital:</strong> {{ pais.capital ? pais.capital[0] : 'N/A' }}</p>
          <p><strong>Región:</strong> {{ pais.region }}</p>
          <p><strong>Población:</strong> {{ pais.population.toLocaleString() }}</p>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
.container { font-family: sans-serif; max-width: 1000px; margin: 0 auto; padding: 20px; }
h1 { text-align: center; }
.search-box { display: flex; justify-content: center; margin-bottom: 20px; }
input { padding: 10px; width: 300px; border: 1px solid #ccc; border-radius: 5px; }
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 20px; }
.card { border: 1px solid #ddd; border-radius: 8px; padding: 15px; text-align: center; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
.flag { width: 100%; height: 150px; object-fit: cover; border-radius: 4px; }
.loading, .error { text-align: center; font-size: 1.2rem; font-weight: bold; }
.error { color: red; }
</style>