<template>
  <div class="fixed top-0 right-0 bottom-0 left-0 flex justify-center items-center bg-white dark:bg-gray-900">
    <form
      @submit.prevent="login"
      method="post"
      class="w-full max-w-sm px-5"
    >
    <label for="email" class="block mb-1 text-gray-400 dark:text-gray-500 text-sm">E-mail</label>
    <input
      v-model="email"
      id="email"
      type="email"
      placeholder="Entrez votre adresse email"
      maxlength="255"
      required
      :class="[ error ? 'bg-red-50 border-red-200 dark:bg-gray-800 dark:border-red-800' : 'bg-gray-50 border-gray-200 dark:bg-gray-800 dark:border-gray-700' ]"
      class="block w-full h-10 px-2  border rounded font-light mb-4 focus:outline-none appearance-none"
    />
    <label for="password" class="block mb-1 text-gray-400 dark:text-gray-500 text-sm">Mot de passe</label>
    <input
      v-model="password"
      id="password"
      type="password"
      placeholder="Entrez votre mot de passe"
      maxlength="255"
      required
      :class="[ error ? 'bg-red-50 border-red-200 dark:bg-gray-800 dark:border-red-800' : 'bg-gray-50 border-gray-200 dark:bg-gray-800 dark:border-gray-700' ]"
      class="block w-full h-10 px-2 bg-gray-50 border border-gray-200 rounded font-light mb-6 focus:outline-none appearance-none"
    />
    <button type="submit" class="block w-full h-10 bg-red-50 border border-red-200 rounded text-red-400 font-medium hover:bg-red-100 duration-200 shadow dark:bg-gray-700 dark:border-gray-700 dark:text-red-400">Connexion</button>
  </form>
  </div>
</template>

<script>
import axios from 'axios'
import api from '../api'

export default {
  data() {
    return {
      email: '',
      password: '',
      error: false
    }
  },
  methods: {
    login() {
      axios.post(`${ process.env.VUE_APP_URL }/auth/local`, {
        identifier: this.email,
        password: this.password
      }).then(res => {
        localStorage.setItem('token', res.data.jwt)
        localStorage.setItem('role', res.data.user.role.type)
        this.$router.go()
      })
      .catch((err) => { this.error = true })
    }
  },
  beforeMount() {
    if (localStorage.getItem('token')) api.get('/compagnies').then(() => this.$router.push('/compagnies'))
  }
}
</script>