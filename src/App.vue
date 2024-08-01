<script setup>
import { ref } from 'vue'
import Swal from 'sweetalert2'

const products = ref([
  {
    name: 'Producto 1',
    price: '150000',
    image: '/demo/images/ecommerce/product-list/product-list-4-1.png',
    available: 10
  },
  {
    name: 'Producto 2',
    price: '200000',
    image: '/demo/images/ecommerce/product-list/product-list-4-2.png',
    available: 5
  },
  {
    name: 'Producto 3',
    price: '120000',
    image: '/demo/images/ecommerce/product-list/product-list-4-3.png',
    available: 2
  },
  {
    name: 'Producto 4',
    price: '300000',
    image: '/demo/images/ecommerce/product-list/product-list-4-4.png',
    available: 1
  },
  {
    name: 'Producto 5',
    price: '270000',
    image: '/demo/images/ecommerce/product-list/product-list-4-5.png',
    available: 0
  },
  {
    name: 'Producto 6',
    price: '180000',
    image: '/demo/images/ecommerce/product-list/product-list-4-6.png',
    available: 0
  }
]);

const visible = ref(false)
const modalResumenVisible = ref(false)
const franquiciaTarjeta = ref(null)
const dataCompra = ref({
  producto: {},
  pago: {
    numeroTarjeta: null,
    envio: 15000,
    impuestos: 0,
    subtotal: 0,
    total: 0
  },
})
const cantidades = []

const formatearPesos = (numero) => {
  // Asegurarse de que el número sea de tipo numérico
  numero = Number(numero);

  // Configurar opciones para el formato de moneda colombiana
  const opcionesFormato = {
    style: 'currency',
    currency: 'COP',
    minimumFractionDigits: 0,
    maximumFractionDigits: 0
  };

  // Formatear el número como pesos colombianos
  const numeroFormateado = numero.toLocaleString('es-CO', opcionesFormato);

  return numeroFormateado;
}
const validarNumeroTarjeta = (event) => {
  if (event.target.value.length > 18) {
    dataCompra.value.pago.numeroTarjeta = event.target.value.toString().substring(0, 18)
  }
  const numeroTarjeta = event.target.value
  const patron = /^(51|52|53|54|55)/;
  if (numeroTarjeta.toString().startsWith("4")) {
    franquiciaTarjeta.value = 1
  } else if (patron.test(numeroTarjeta.toString())) {
    franquiciaTarjeta.value = 2
  } else {
    franquiciaTarjeta.value = null
  }
}
const validarFecha = (event) => {
  const fecha = event.target.value
  const mes = fecha.substring(0, 2)
  if (mes > 12 || mes < 1) {
    dataCompra.value.pago.fechaVencimiento = ''
  }
}
const agregarProducto = (producto) => {
  dataCompra.value.producto = producto
  dataCompra.value.producto.quantity = 1
  dataCompra.value.pago.subtotal = dataCompra.value.producto.price * dataCompra.value.producto.quantity
  dataCompra.value.pago.impuestos = dataCompra.value.pago.subtotal * 0.19
  dataCompra.value.pago.total = dataCompra.value.pago.subtotal + dataCompra.value.pago.envio + dataCompra.value.pago.impuestos
  visible.value = true
  for (let i = 1; i <= producto.available; i++) {
    cantidades.push(i)
  }
}
const pagar = () => {
  visible.value = false
  modalResumenVisible.value = true
}
const confirmarCompra = () => {
  modalResumenVisible.value = false
  Swal.fire({
    text: 'Pagando...',
    timer: 3000,
    didOpen: () => {
      Swal.showLoading();
    }
  }).then(() => {
    const numeroAleatorio = Math.floor(Math.random() * 2) + 1;
    if (numeroAleatorio === 1) {
      Swal.fire({
        title: "Exito",
        text: "Pagado con éxito",
        icon: "success"
      });
    } else {
      Swal.fire({
        title: "Error",
        text: "No se pudo realizar el pago",
        icon: "error"
      }).then(() => {
        visible.value = true
      });
    }
  });
}
const cancelarComprar = () => {
  dataCompra.value.producto = {}
  dataCompra.value.pago = {}
  visible.value = false
  modalResumenVisible.value = false
}
const onChangeCantidades = ({ value }) => {
  dataCompra.value.producto.quantity = parseInt(value)
  dataCompra.value.pago.subtotal = dataCompra.value.producto.price * dataCompra.value.producto.quantity
  dataCompra.value.pago.impuestos = dataCompra.value.pago.subtotal * 0.19
  dataCompra.value.pago.total = dataCompra.value.pago.subtotal + dataCompra.value.pago.envio + dataCompra.value.pago.impuestos
}
</script>

<template>
  <div class="card">
    <div class="text-900 font-medium text-4xl mb-4">Tienda de productos</div>
    <div class="grid -mt-3 -ml-3 -mr-3">
      <div v-for="(product, i) in products" :key="i" class="col-12 md:col-6 lg:col-3 mb-5 lg:mb-0">
        <div class="mb-3 relative">
          <img :src="product.image" class="w-full" :alt="i"/>
          <Button
              type="button"
              class="border-1 border-white border-round py-2 px-3 absolute bg-black-alpha-30 text-white inline-flex align-items-center justify-content-center hover:bg-black-alpha-40 transition-colors transition-duration-300 cursor-pointer"
              :style="{ bottom: '1rem', left: '1rem', width: 'calc(100% - 2rem)' }"
              @click="agregarProducto(product)"
          >
            <i class="pi pi-shopping-cart mr-3 text-base"></i>
            <span class="text-base">Pagar con tarjeta de crédito</span>
          </Button>
        </div>
        <div class="flex flex-column align-items-center">
          <span class="text-xl text-900 font-medium">{{ product.name }}</span>
          <span class="text-lg">{{ formatearPesos(product.price) }}</span>
          <span class="text-sm">Disponible: {{ product.available }}</span>
        </div>
      </div>
    </div>
  </div>
  <!-- Modal datos tarjeta -->
  <Dialog
      v-model:visible="visible"
      modal
      header="Datos de la tarjeta"
      :style="{ width: '30vw' }"
      :breakpoints="{ '1400px': '50vw', '575px': '90vw' }"
      :closable="false"
  >
    <form class="flex flex-column p-2 gap-2" @submit="pagar()">
      <div class="flex gap-2">
        <svg v-if="franquiciaTarjeta === 1" xmlns="http://www.w3.org/2000/svg" width="40" height="40"
             viewBox="0 0 1000 1000">
          <path fill="#1a1f71"
                d="M40 186c-22.087 0-40 17.908-40 40v548c0 22.091 17.922 40 40 40h920c22.087 0 40-17.908 40-40V226c0-22.091-17.922-40-40-40zm557.25 187.156c24.338 0 43.843 5.376 56.281 10.375l-8.5 53.469l-5.625-2.844c-11.587-4.999-26.47-9.81-47-9.469c-24.576 0-35.937 10.963-35.937 21.22c-.143 11.557 13.275 19.179 35.218 30.593c36.217 17.61 52.956 38.962 52.72 67.031c-.488 51.217-43.324 84.313-109.313 84.313c-28.153-.31-55.276-6.27-69.938-13.157l8.813-55.25l8.094 3.938c20.617 9.212 33.967 12.938 59.093 12.938c18.043 0 37.409-7.555 37.563-24.094c.117-10.8-8.095-18.5-32.532-30.594c-23.812-11.804-55.38-31.577-55.03-67.031c.372-47.961 44.063-81.438 106.093-81.438M88.437 381h101.344c13.647.516 24.665 4.91 28.469 19.719l21.844 112.75c.003.01-.004.051 0 .062l6.562 33.782L308.187 381h66.563l-98.938 243.438l-66.5.062l-52.937-196.438c31.498 16.687 58.314 35.994 73.844 62.563c-4.004-8.407-9.28-17.897-16.031-27.25c-7.862-10.892-24.841-24.955-31.938-30.938c-24.73-20.847-58.315-37.684-94.594-46.593zm312.782.281h65.094l-40.72 243.063H360.5zm367.656 0h49.313l51.625 243.063h-59.188s-5.872-27.924-7.781-36.438c-9.303 0-74.367-.093-81.688-.093c-2.477 6.58-13.437 36.53-13.437 36.53H640.75l94.719-222.874c6.706-15.838 18.137-20.188 33.406-20.188m4.844 65.375c-3.203 9.041-8.78 23.641-8.407 23c0 0-20.037 53.44-25.28 67.313l52.655-.032A191955.64 191955.64 0 0 0 778 467.408l-4.281-20.75z"/>
        </svg>
        <svg v-if="franquiciaTarjeta === 2" xmlns="http://www.w3.org/2000/svg" width="40" height="40"
             viewBox="0 0 256 199">
          <path
              d="M46.54 198.011V184.84c0-5.05-3.074-8.342-8.343-8.342c-2.634 0-5.488.878-7.464 3.732c-1.536-2.415-3.731-3.732-7.024-3.732c-2.196 0-4.39.658-6.147 3.073v-2.634h-4.61v21.074h4.61v-11.635c0-3.731 1.976-5.488 5.05-5.488c3.072 0 4.61 1.976 4.61 5.488v11.635h4.61v-11.635c0-3.731 2.194-5.488 5.048-5.488c3.074 0 4.61 1.976 4.61 5.488v11.635zm68.271-21.074h-7.463v-6.366h-4.61v6.366h-4.171v4.17h4.17v9.66c0 4.83 1.976 7.683 7.245 7.683c1.976 0 4.17-.658 5.708-1.536l-1.318-3.952c-1.317.878-2.853 1.098-3.951 1.098c-2.195 0-3.073-1.317-3.073-3.513v-9.44h7.463zm39.076-.44c-2.634 0-4.39 1.318-5.488 3.074v-2.634h-4.61v21.074h4.61v-11.854c0-3.512 1.536-5.488 4.39-5.488c.878 0 1.976.22 2.854.439l1.317-4.39c-.878-.22-2.195-.22-3.073-.22m-59.052 2.196c-2.196-1.537-5.269-2.195-8.562-2.195c-5.268 0-8.78 2.634-8.78 6.805c0 3.513 2.634 5.488 7.244 6.147l2.195.22c2.415.438 3.732 1.097 3.732 2.195c0 1.536-1.756 2.634-4.83 2.634c-3.073 0-5.488-1.098-7.025-2.195l-2.195 3.512c2.415 1.756 5.708 2.634 9 2.634c6.147 0 9.66-2.853 9.66-6.805c0-3.732-2.854-5.708-7.245-6.366l-2.195-.22c-1.976-.22-3.512-.658-3.512-1.975c0-1.537 1.536-2.415 3.951-2.415c2.635 0 5.269 1.097 6.586 1.756zm122.495-2.195c-2.635 0-4.391 1.317-5.489 3.073v-2.634h-4.61v21.074h4.61v-11.854c0-3.512 1.537-5.488 4.39-5.488c.879 0 1.977.22 2.855.439l1.317-4.39c-.878-.22-2.195-.22-3.073-.22m-58.833 10.976c0 6.366 4.39 10.976 11.196 10.976c3.073 0 5.268-.658 7.463-2.414l-2.195-3.732c-1.756 1.317-3.512 1.975-5.488 1.975c-3.732 0-6.366-2.634-6.366-6.805c0-3.951 2.634-6.586 6.366-6.805c1.976 0 3.732.658 5.488 1.976l2.195-3.732c-2.195-1.757-4.39-2.415-7.463-2.415c-6.806 0-11.196 4.61-11.196 10.976m42.588 0v-10.537h-4.61v2.634c-1.537-1.975-3.732-3.073-6.586-3.073c-5.927 0-10.537 4.61-10.537 10.976c0 6.366 4.61 10.976 10.537 10.976c3.073 0 5.269-1.097 6.586-3.073v2.634h4.61zm-16.904 0c0-3.732 2.415-6.805 6.366-6.805c3.732 0 6.367 2.854 6.367 6.805c0 3.732-2.635 6.805-6.367 6.805c-3.951-.22-6.366-3.073-6.366-6.805m-55.1-10.976c-6.147 0-10.538 4.39-10.538 10.976c0 6.586 4.39 10.976 10.757 10.976c3.073 0 6.147-.878 8.562-2.853l-2.196-3.293c-1.756 1.317-3.951 2.195-6.146 2.195c-2.854 0-5.708-1.317-6.367-5.05h15.587v-1.755c.22-6.806-3.732-11.196-9.66-11.196m0 3.951c2.853 0 4.83 1.757 5.268 5.05h-10.976c.439-2.854 2.415-5.05 5.708-5.05m114.372 7.025v-18.879h-4.61v10.976c-1.537-1.975-3.732-3.073-6.586-3.073c-5.927 0-10.537 4.61-10.537 10.976c0 6.366 4.61 10.976 10.537 10.976c3.074 0 5.269-1.097 6.586-3.073v2.634h4.61zm-16.903 0c0-3.732 2.414-6.805 6.366-6.805c3.732 0 6.366 2.854 6.366 6.805c0 3.732-2.634 6.805-6.366 6.805c-3.952-.22-6.366-3.073-6.366-6.805m-154.107 0v-10.537h-4.61v2.634c-1.537-1.975-3.732-3.073-6.586-3.073c-5.927 0-10.537 4.61-10.537 10.976c0 6.366 4.61 10.976 10.537 10.976c3.074 0 5.269-1.097 6.586-3.073v2.634h4.61zm-17.123 0c0-3.732 2.415-6.805 6.366-6.805c3.732 0 6.367 2.854 6.367 6.805c0 3.732-2.635 6.805-6.367 6.805c-3.951-.22-6.366-3.073-6.366-6.805"/>
          <path fill="#FF5F00" d="M93.298 16.903h69.15v124.251h-69.15z"/>
          <path fill="#EB001B"
                d="M97.689 79.029c0-25.245 11.854-47.637 30.074-62.126C114.373 6.366 97.47 0 79.03 0C35.343 0 0 35.343 0 79.029c0 43.685 35.343 79.029 79.029 79.029c18.44 0 35.343-6.366 48.734-16.904c-18.22-14.269-30.074-36.88-30.074-62.125"/>
          <path fill="#F79E1B"
                d="M255.746 79.029c0 43.685-35.343 79.029-79.029 79.029c-18.44 0-35.343-6.366-48.734-16.904c18.44-14.488 30.075-36.88 30.075-62.125c0-25.245-11.855-47.637-30.075-62.126C141.373 6.366 158.277 0 176.717 0c43.686 0 79.03 35.563 79.03 79.029"/>
        </svg>
        <InputText
            type="number"
            placeholder="Numero de tarjeta"
            class="w-full lg:w-1/2"
            @keydown="validarNumeroTarjeta($event)"
            v-model="dataCompra.pago.numeroTarjeta"
            :useGrouping="false"
        />
      </div>
      <InputText
          id="txtNombreTitular"
          placeholder="Nombre del titular"
          class="w-full"
          v-model="dataCompra.pago.nombreTitular"
      />
      <div class="flex w-full gap-2">
        <InputMask
            class="w-6"
            v-model="dataCompra.pago.fechaVencimiento"
            mask="99/99"
            placeholder="MM/YY"
            @keyup="validarFecha($event)"
        />
        <InputMask
            class="w-6"
            v-model="dataCompra.pago.cvc"
            mask="999"
            placeholder="CVC"
        />
      </div>
      <div class="flex w-full gap-2">
        <Button
            size="small"
            label="Cancelar"
            icon="pi pi-times"
            severity="danger"
            @click="cancelarComprar()"
            class="w-6"
        />
        <Button
            size="small"
            label="Comprar"
            icon="pi pi-shopping-cart"
            @click="pagar()"
            class="w-6"
        />
      </div>
    </form>
  </Dialog>
  <!-- Modal resumen -->
  <Dialog
      v-model:visible="modalResumenVisible"
      modal
      :showHeader="false"
      :style="{ width: '50vw' }"
      :breakpoints="{ '1199px': '75vw', '575px': '90vw' }"
      :closable="false"
  >
    <div class="text-900 font-bold text-4xl my-2">Orden exitosa 🚀</div>
    <div
        :style="{ height: '3px', background: 'linear-gradient(90deg, var(--primary-color) 0%, rgba(33, 150, 243, 0) 50%)' }"></div>

    <div class="flex flex-column sm:flex-row sm:align-items-center sm:justify-content-between py-5">
      <div class="mb-3 sm:mb-0">
        <span class="font-medium text-xl text-900 mr-2">Numero de orden:</span>
        <span class="font-medium text-xl text-blue-500">99999</span>
      </div>
    </div>
    <div class="border-round surface-border border-1">
      <ul class="list-none p-0 m-0">
        <li
            class="p-3 surface-border flex align-items-start sm:align-items-center"
        >
          <img :src="dataCompra.producto.image" class="w-3rem sm:w-8rem flex-shrink-0 mr-3 shadow-2" :alt="dataCompra.producto.name"/>
          <div class="flex flex-column">
            <span class="text-900 font-semibold text-xl mb-2">{{ dataCompra.producto.name }}</span>
            <span class="text-900 font-medium">Cantidad: </span>
            <Dropdown v-model="dataCompra.producto.quantity" :options="cantidades" class="w-full md:w-14rem p-inputtext-sm" @change="onChangeCantidades" />
          </div>
          <span class="text-900 font-medium text-lg ml-auto">{{ formatearPesos(dataCompra.producto.price) }}</span>
        </li>
      </ul>
    </div>
    <div class="flex flex-wrap mt-5 pb-3">
      <div class="w-full lg:w-6 pl-3">
        <span class="font-medium text-900">Datos envio</span>
        <div class="flex flex-column text-900 mt-3 mb-5">
          <span class="mb-1">{{ dataCompra.pago.nombreTitular }}</span>
        </div>

        <span class="font-medium text-900">Información de pago</span>
        <div class="flex align-items-center mt-3">
          <img src="/demo/images/ecommerce/ordersummary/visa.png" class="w-4rem mr-3" alt="visa"/>
          <div class="flex flex-column">
            <span class="text-900 mb-1">Tarjeta de crédito</span>
            <span class="text-900 font-medium">**** **** **** ****</span>
          </div>
        </div>
      </div>
      <div class="w-full lg:w-6 pl-3 lg:pl-0 lg:pr-3 flex align-items-end mt-5 lg:mt-0">
        <ul class="list-none p-0 m-0 w-full">
          <li class="mb-3">
            <span class="font-medium text-900">Resumen</span>
          </li>
          <li class="flex justify-content-between mb-3">
            <span class="text-900">Subtotal</span>
            <span class="text-900 font-medium text-lg">{{ formatearPesos(dataCompra.pago.subtotal) }}</span>
          </li>
          <li class="flex justify-content-between mb-3">
            <span class="text-900">Envío</span>
            <span class="text-900 font-medium text-lg">{{ formatearPesos(dataCompra.pago.envio) }}</span>
          </li>
          <li class="flex justify-content-between mb-3">
            <span class="text-900">Impuestos</span>
            <span class="text-900 font-medium text-lg">{{ formatearPesos(dataCompra.pago.impuestos) }}</span>
          </li>
          <li class="flex justify-content-between border-top-1 surface-border py-3">
            <span class="text-900 font-medium">Total</span>
            <span class="text-900 font-bold text-lg">{{ formatearPesos(dataCompra.pago.total) }}</span>
          </li>
        </ul>
      </div>
    </div>
    <div class="flex justify-content-end">
      <Button label="Confirmar" icon="pi pi-check" class="mt-5 mb-5" @click="confirmarCompra"></Button>
    </div>
  </Dialog>
</template>
