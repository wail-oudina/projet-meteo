<script>
export default {
  name: "SearchComponent",
  data(){
    return{
      autocomplete_data : {},
      search_input : "",
      isSearchActivated : true,
      show_resultbox : false,
      searched_city_data : ""
    }
  },
  methods:{
    // Méthode pour gérer le clic en dehors du composant
    onclickOutside(){
      this.isSearchActivated = false
    },
    // Méthode pour gérer les événements clavier de l'input
    onKeyDown(event) {
      if (/^[a-zA-Z]$/.test(event.key) || event.key === "Backspace") {

        this.request_autocomplete_data()
      }
    },
    // Méthode pour effectuer la requête d'autocomplétion
    request_autocomplete_data(){
      // Configuration de la requête
      let options = {
        method: 'GET',
        url: 'https://weatherapi-com.p.rapidapi.com/search.json',
        params: {q: this.search_input},
        headers: {
          'X-RapidAPI-Key': '30aad44501msh65bbb52a41657d8p1ad31bjsn2ec80c8ab81e',
          'X-RapidAPI-Host': 'weatherapi-com.p.rapidapi.com'
        }
      };
      // Si la longueur de l'entrée est inférieure à 2, ne fait rien
      if (this.search_input.length < 2){
          return
        }
      // Effectue la requête et met à jour les données
      this.axios.request(options).then( (response) => {

        this.autocomplete_data = response.data
        this.show_resultbox = this.autocomplete_data.length > 0


      } )
    },
    // Méthode pour mettre à jour les données de la ville recherchée
    affectResult(result){
      this.searched_city_data = result
      this.$emit('update:searched_city_data', this.searched_city_data)
      this.show_resultbox = false

    },
    // Méthode pour obtenir la position actuelle de l'utilisateur
    getLocation(){
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition( (position) => {

          let latitude = position.coords.latitude;
          let longitude = position.coords.longitude;

          //this.affectResult(latitude + "," + longitude)
          this.search_input = latitude + "," + longitude
          this.request_autocomplete_data()
          
          
        });
      } else { 
        window.alert("Geolocation is not supported by this browser.");
      }
    },
  },
  props:{
      searched_city_data: Object
  },
  emits: ['update:searched_city_data'],
  
  
}
</script>

<template>


  <div  v-if="isSearchActivated" v-click-outside="onclickOutside" class="d-flex p-2 justify-content-center">
    <input @keydown="onKeyDown" v-model="search_input" autocomplete="off" class="form-control  mx-3 w-25 color" width="200px" id="search-input"   type="search" placeholder="Search a city.  Ex : Paris" aria-label="Search">
    <button @click="this.searched_city_data = getLocation()"  class="btn btn-outline-primary " >
      <svg style="color:white" width="20" height="20" xmlns="http://www.w3.org/2000/svg" class="ionicon" viewBox="0 0 512 512"><title>Location</title><path d="M256 48c-79.5 0-144 61.39-144 137 0 87 96 224.87 131.25 272.49a15.77 15.77 0 0025.5 0C304 409.89 400 272.07 400 185c0-75.61-64.5-137-144-137z" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="32"/><circle cx="256" cy="192" r="48" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="32"/></svg>
    </button>
  </div>
  <div v-else class="d-flex p-2 justify-content-center">
    <button @click="isSearchActivated = !isSearchActivated" class="btn btn-outline-primary " type="submit">
      <svg xmlns="http://www.w3.org/2000/svg" class="ionicon" viewBox="0 0 512 512" style="color:white;">
        <title>Search</title>
        <path d="M221.09 64a157.09 157.09 0 10157.09 157.09A157.1 157.1 0 00221.09 64z" fill="none" stroke="currentColor" stroke-miterlimit="10" stroke-width="32"/><path fill="none" stroke="currentColor" stroke-linecap="round" stroke-miterlimit="10" stroke-width="32" d="M338.29 338.29L448 448"/>
      </svg>
    </button>
  </div>  

  
  <div  v-if="show_resultbox && isSearchActivated" class="resultbox result-container d-flex flex-column justify-content-center ">


    <div v-for="result in autocomplete_data" @click="affectResult(result)"  class="result-item px-2 py-1 ">{{result.name }}, {{ result.country }}</div>
    

  </div>

</template>
<style>

.resultbox{
  background-color: #D8E1FF;
  z-index: 2;
}
.v-enter-active, .v-leave-active {
  transition: opacity 1s ease;
}
</style>
