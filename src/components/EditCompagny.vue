<template>
  <form
    @submit.prevent="save"
    method="post"
    class="fixed top-0 right-0 bottom-0 left-0 flex justify-center items-center"
  >
    <div @click="$router.push('/compagnies')" class="absolute top-0 right-0 bottom-0 left-0 bg-black opacity-60"></div>
    <div class="z-10 w-full max-w-screen-md bg-white dark:bg-gray-900 shadow-2xl">
      <div class="px-2 py-4 md:px-8 md:py-6 overflow-auto" :style="{ maxHeight: '70vh' }">
        <table class="w-full table-auto divide-y dark:divide-gray-700">
          <tbody>
            <tr>
              <td class="px-2 py-1 text-gray-400 dark:text-gray-500">Nom</td>
              <td class="px-2 py-1">
                <input
                  v-model="compagny.name"
                  type="text" maxlength="255" required
                  placeholder="Saisir un nom"
                  class="w-full focus:outline-none bg-transparent dark:placeholder-gray-500 dark:text-white"
                />
              </td>
            </tr>
            <tr>
              <td class="px-2 py-1 text-gray-400 dark:text-gray-500">Adresse</td>
              <td class="px-2 py-1">
                <input
                  v-model="compagny.adress"
                  type="text" maxlength="255"
                  placeholder="Saisir une adresse"
                  class="w-full focus:outline-none bg-transparent dark:placeholder-gray-500 dark:text-white"
                />
              </td>
            </tr>
            <tr>
              <td class="px-2 py-1 flex text-gray-400 dark:text-gray-500">Véhicules {{ compagny.cars ? `(${ compagny.cars.length })` : '' }}</td>
              <td class="px-2 py-1">
                <router-link
                  v-for="car in compagny.cars"
                  :key="car.id"
                  :to="{ path: `/cars/edit/${ car.id }` }"
                  class="flex justify-between max-w-xs"
                >
                  <strong class="font-semibold">{{ car.brand }} {{ car.model }}</strong>
                  <span>{{ car.totalPrice ? `${ car.totalPrice }€` : '' }}</span>
                </router-link>
                <p
                  v-if="totalAmount > 0"
                  class="flex justify-between max-w-xs"
                >
                  <strong class="font-semibold">Coût total</strong>
                  <span>{{ totalAmount }}€</span>
                </p>
                <router-link
                  :to="{ path: `/cars/add/${ compagny.id }` }"
                  class="block pb-1 text-gray-400 dark:text-gray-500"
                >Ajouter</router-link>
              </td>
            </tr>
            <tr>
              <td class="px-2 py-1 flex text-gray-400 dark:text-gray-500">Détail</td>
              <td class="px-2 py-1">
                <textarea
                  v-model="compagny.detail"
                  type="text" maxlength="3000" rows="3"
                  placeholder="Saisir un détail"
                  class="w-full focus:outline-none bg-transparent dark:placeholder-gray-500 dark:text-white"
                ></textarea>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="flex divide-x dark:divide-gray-700">
        <div @click="download" class="flex-1 block p-3 border-t dark:border-gray-700 text-gray-400 dark:text-gray-500 text-center cursor-pointer">Télécharger</div>
        <button class="flex-1 block p-3 border-t dark:border-gray-700 text-gray-400 dark:text-gray-500 focus:outline-none">Enregistrer</button>
        <div
          v-if="role === 'super_admin'"
          @click="erase"
          class="flex-1 block p-3 border-t dark:border-gray-700 text-red-400 text-center cursor-pointer"
        >Supprimer</div>
      </div>
    </div>
  </form>
</template>

<script>
import api from '../api'
import pdfMake from 'pdfmake/build/pdfmake'
import pdfFonts from 'pdfmake/build/vfs_fonts'
pdfMake.vfs = pdfFonts.pdfMake.vfs

export default {
  emits: ['fetch'],
  data() {
    return {
      compagny: {},
      role: localStorage.getItem('role'),
      cars: [],
      amount: 0
    }
  },
  methods: {
    save() {
      api.put(`/compagnies/${ this.compagny.id }`, this.compagny).then(() => {
        this.$emit('fetch')
        this.$router.push('/compagnies')
      })
    },
    erase() {
      api.delete(`/compagnies/${ this.compagny.id }`).then(() => {
        this.$emit('fetch')
        this.$router.push('/compagnies')
      })
    },
    setCompagny() {
      this.compagny = JSON.parse(localStorage.getItem('compagnies')).find(compagny => compagny.id == this.$route.params.id) || []
      api.get('/parts').then(res => {
        this.compagny.cars.forEach(car => {
          res.data.forEach(part => {
            if (part.maintenance.car === car.id) {
              if (!car.totalPrice) car.totalPrice = 0
              car.totalPrice = car.totalPrice + (part.customerPrice * part.amount)
            }
          })
        })
      })

    },
    download() {
      let body = [['Modèle', 'Numéro de série', 'Mise en service', 'Motorisation']]
      this.compagny.cars.forEach(car => body.push([car.model, car.serie, this.setDate(car.service), car.motor]))
      pdfMake.createPdf({
        pageSize: 'A4',
        pageOrientation: 'landscape',
        content: [
          { text: this.compagny.name, fontSize: 20, bold: true },
          { text: this.compagny.adress, fontSize: 15 },
          { text: this.compagny.detail, fontSize: 15 },
          {
            fontSize: 10,
            margin: [0, 30, 0, 0],
            layout: 'lightHorizontalLines',
            table: { headerRows: 1, widths: [ '*', 'auto', '*', '*', '*' ], body: body }
          }
        ]
      }).open()
    },
    setDate(slug) {
      let date = new Date(slug)
      return date.toLocaleDateString('fr-FR', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      })
    }
  },
  computed: {
    totalAmount() {
      this.amount = 0
      if (this.compagny.cars) {
        this.compagny.cars.forEach(car => { this.amount = this.amount + car.totalPrice })
      }
      return this.amount
    }
  },
  mounted() {
    this.setCompagny()
    api.get('/compagnies').then(res => {
      localStorage.setItem('compagnies', JSON.stringify(res.data))
      this.setCompagny()
    })
  }
}
</script>