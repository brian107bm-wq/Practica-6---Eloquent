<template>
  <div>
    <h1>Catálogo</h1>

    <input v-model="busqueda" placeholder="Buscar producto"/>

    <select v-model="categoriaSeleccionada">
      <option value="">Todas las categorías</option>

      <option v-for="categoria in categorias"
        :key="categoria.id"
        :value="categoria.id"
      >{{ categoria.nombre }}</option>
    </select>

    <div v-for="producto in productosPaginados"
      :key="producto.id">

      <img v-if="producto.imagen_url"
        :src="producto.imagen_url"
        :alt="producto.nombre"
        width="200"
      />

      <h3>{{ producto.nombre }}</h3>
      <p>${{ producto.precio }}</p>

      <router-link :to="`/catalogo/${producto.id}`">Ver detalle</router-link>
      <br><br>
      <button @click="carrito.agregar(producto)">

        <template v-if="carrito.cantidadDeProducto(producto.id) > 0">
          En carrito
          (
          {{
            carrito.cantidadDeProducto(producto.id)
          }}
          )
        </template>
        <template v-else>Agregar al carrito</template>
      </button>
      <hr>
    </div>

    <button @click="pagina--" :disabled="pagina === 1">Anterior</button>
    <button @click="pagina++">Siguiente</button>

  </div>
</template>

<script setup>
import axios from 'axios'
import {ref, computed, onMounted } from 'vue'
import { getProductos } from '@/services/productoService'
import { useCarritoStore } from '@/stores/carrito'

const carrito = useCarritoStore()
const productos = ref([])
const categorias = ref([])
const categoriaSeleccionada = ref('')
const busqueda = ref('')

const pagina = ref(1)
const porPagina = 10
const productosPaginados = computed(() => {

  const inicio =
    (pagina.value - 1)
    * porPagina
  return productosFiltrados.value.slice(
    inicio,
    inicio + porPagina
  )
})

const cargarCategorias = async () => {
  const response = await axios.get(
    'http://127.0.0.1:8000/api/categorias'
  )

  categorias.value = response.data.data
}

const productosFiltrados = computed(() =>
  productos.value.filter(p => {
    const coincideNombre =
      p.nombre
       .toLowerCase()
       .includes(
         busqueda.value.toLowerCase()
       )

    const coincideCategoria =
      !categoriaSeleccionada.value
      ||
      p.categoria_id ==
      categoriaSeleccionada.value

    return coincideNombre
      && coincideCategoria
  })
)

onMounted(async () => {
  const response = await getProductos()
  productos.value = response.data
  await cargarCategorias()
})

</script>