<template>
  <div class="fixed top-0 right-0 bottom-0 left-0 flex justify-center items-center">
    <div @click="$router.push(`/cars/edit/${ maintenance.car.id }`)" class="absolute top-0 right-0 bottom-0 left-0 bg-black opacity-75"></div>
    <div class="z-10 w-full bg-white dark:bg-gray-900 shadow-2xl" :style="{ maxWidth: '1400px' }">
      <form
        @submit.prevent="save"
        class="overflow-auto"
        :style="{ maxHeight: '70vh' }"
      >
        <table class="w-full table-auto divide-y dark:divide-gray-700" :style="{ minWidth: '1000px' }">
          <thead class="text-left text-gray-400 dark:text-gray-500">
            <tr class="divide-x dark:divide-gray-700">
              <td class="p-2">
                <select
                  v-model="add.part_type"
                  class="font-semibold text-black appearance-none outline-none cursor-pointer bg-transparent dark:text-gray-200" required
                >
                  <option value="" disabled selected>Pièce</option>
                  <option
                    v-for="partType in partTypes"
                    :key="partType.id"
                    :value="partType.id"
                  >{{ partType.name }}</option>
                </select>
              </td>
              <!-- <td class="p-2">
                <input
                  v-model="add.date"
                  type="text" required
                  placeholder="Date (jj/mm/aaaa)"
                  pattern="\d{1,2}/\d{1,2}/\d{4}"
                  class="w-full focus:outline-none bg-transparent dark:placeholder-gray-500 dark:text-white"
                />
              </td> -->
              <td class="p-2">
                <input
                  v-model="add.customer"
                  type="text" maxlength="255" required
                  placeholder="Fournisseur"
                  class="w-full focus:outline-none bg-transparent dark:placeholder-gray-500 dark:text-white"
                />
              </td>
              <td class="p-2">
                <input
                  v-model="add.reference"
                  type="text" maxlength="255" required
                  placeholder="Référence"
                  class="w-full focus:outline-none bg-transparent dark:placeholder-gray-500 dark:text-white"
                />
              </td>
              <td class="p-2">
                <input
                  v-model="add.amount"
                  type="number" max="999999" step=".01" required
                  placeholder="Quantité"
                  class="w-full text-right focus:outline-none bg-transparent dark:placeholder-gray-500 dark:text-white"
                />
              </td>
              <td class="p-2">
                <input
                  v-model="add.publicPrice"
                  type="number" max="999999" step=".01"
                  placeholder="Tarif public"
                  class="w-full text-right focus:outline-none bg-transparent dark:placeholder-gray-500 dark:text-white"
                />
              </td>
              <td class="p-2">
                <input
                  v-model="add.purchasePrice"
                  type="number" max="999999" step=".01"
                  placeholder="Tarif achat remisé"
                  class="w-full text-right focus:outline-none bg-transparent dark:placeholder-gray-500 dark:text-white"
                />
              </td>
              <td class="p-2">
                <input
                  v-model="add.customerPrice"
                  type="number" max="999999" step=".01" required
                  placeholder="Tarif client"
                  class="w-full text-right focus:outline-none bg-transparent dark:placeholder-gray-500 dark:text-white"
                />
              </td>
              <td class="p-2">
                <button class="block w-full text-center">
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 mx-auto text-gray-400 dark:text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6" />
                  </svg>
                </button>
              </td>
            </tr>
          </thead>
          <tbody class="divide-y dark:divide-gray-700">
            <tr
              v-for="part in maintenance.parts"
              :key="part.id"
              class="divide-x dark:divide-gray-700"
            >
              <td class="p-2 font-semibold">{{ setPartType(part.part_type) }}</td>
              <!-- <td class="p-2 capitalize">{{ setDate(part.date) }}</td> -->
              <td class="p-2">{{ part.customer }}</td>
              <td class="p-2">{{ part.reference }}</td>
              <td class="p-2 text-right">{{ part.amount }}</td>
              <td class="p-2 text-right">{{ part.publicPrice }} €</td>
              <td class="p-2 text-right">{{ part.purchasePrice }} €</td>
              <td class="p-2 text-right">{{ part.customerPrice }} €</td>
              <td @click="erase(part.id)" class="p-2 text-center cursor-pointer">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mx-auto text-gray-400 dark:text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </td>
            </tr>
          </tbody>
        </table>
      </form>
      <div class="flex divide-x dark:divide-gray-700">
        <!-- <div class="flex-1 block p-3 border-t dark:border-gray-700 text-gray-400 dark:text-gray-500 text-center cursor-pointer">Télécharger</div> -->
        <div
          v-if="role === 'super_admin'"
          @click="clear(maintenance.id)"
          class="flex-1 block p-3 border-t dark:border-gray-700 text-red-400 text-center cursor-pointer"
        >Supprimer cette intervention</div>
        <div class="flex-1 p-3 border-t dark:border-gray-700 text-center">Coût total de {{ totalPrice }}€</div>
      </div>
    </div>
  </div>
</template>

<script>
import api from '../api'

export default {
  emits: ['update'],
  data() {
    return {
      maintenance: {},
      partTypes: [],
      role: localStorage.getItem('role'),
      add: {
        maintenance: this.$route.params.maintenance,
        part_type: '',
        customer: '',
        reference: '',
        amount: '',
        publicPrice: '',
        purchasePrice: '',
        customerPrice: '',
      }
    }
  },
  methods: {
    get() {
      api.get(`maintenances/${ this.$route.params.maintenance }`).then(res => {
        this.maintenance = res.data
      })
    },
    setPartType(id) {
      return this.partTypes.find(partType => partType.id == id).name
    },
    setDate(slug) {
      let date = new Date(slug)
      return date.toLocaleDateString('fr-FR', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      })
    },
    save() {
      this.add.publicPrice = this.add.publicPrice === '' ? 0 : this.add.publicPrice
      this.add.purchasePrice = this.add.purchasePrice === '' ? 0 : this.add.purchasePrice
      this.add.customerPrice = this.add.customerPrice === '' ? 0 : this.add.customerPrice
      api.post('/parts', this.add).then(() => {
        this.get()
        for (let x in this.add) {
          this.add[x] = ''
        }
      })
    },
    erase(id) {
      api.delete(`/parts/${ id }`).then(() => this.get())
    },
    clear(id) {
      api.delete(`/maintenances/${ id }`).then(() => {
        this.maintenance.parts.forEach(part => this.erase(part.id))
        this.cars()
        this.$router.push(`/cars/edit/${ this.maintenance.car.id }`)
      })
    },
    cars() {
      api.get('/cars').then(res => {
        localStorage.setItem('cars', JSON.stringify(res.data))      
        this.$emit('update')
      })
    }
  },
  computed: {
    totalPrice() {
      let price = 0
      if (Object.keys(this.maintenance).length) {
        this.maintenance.parts.forEach(part => {
          price = price + (part.customerPrice * part.amount)
        })
      }
      return price
    }
  },
  mounted() {
    api.get('/part-types').then(res => {
      this.partTypes = res.data
    })
    this.get()
  }
}
</script>