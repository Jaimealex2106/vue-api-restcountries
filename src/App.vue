<script setup>
import { ref, computed, onMounted } from 'vue'

// Variables reactivas
const countries = ref([])
const loading = ref(true)
const error = ref(false)
const busqueda = ref('')

// Consumo de API (SampleAPIs)
const fetchCountries = async () => {
  try {
    const response = await fetch('https://api.sampleapis.com/countries/countries')
    if (!response.ok) throw new Error('Error en la red')
    countries.value = await response.json()
  } catch (err) {
    error.value = true
    console.error(err)
  } finally {
    loading.value = false
  }
}

// Filtro computado protegido
const paisesFiltrados = computed(() => {
  if (!Array.isArray(countries.value)) return []
  return countries.value.filter(pais => 
    pais.name.toLowerCase().includes(busqueda.value.toLowerCase())
  )
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
      <input type="text" v-model="busqueda" placeholder="Buscar país por nombre..." />
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
        <div v-for="pais in paisesFiltrados" :key="pais.name" class="card">
          <!-- Uso de ?. para evitar errores si falta la bandera -->
          <img :src="pais.media?.flag" :alt="'Bandera de ' + pais.name" class="flag" />
          <h2>{{ pais.name }}</h2>
          <p><strong>Capital:</strong> {{ pais.capital }}</p>
          <p><strong>Moneda:</strong> {{ pais.currency }}</p>
          <!-- Validación con operador ternario para la población -->
          <p><strong>Población:</strong> {{ pais.population ? pais.population.toLocaleString() : 'N/A' }}</p>
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