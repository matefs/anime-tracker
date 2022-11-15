

<script setup>
import { ref, onMounted, computed } from 'vue'
const query = ref('')
const my_anime = ref([])
const search_results = ref([])
const my_anime_asc = computed(() => {
	return my_anime.value.sort((a, b) => {
		return a.title.localeCompare(b.title)
	})
})

const searchAnime = () => {
	const url = `https://api.jikan.moe/v4/anime?q=${query.value}`
	fetch(url)
		.then(res => res.json())
		.then(data => {
			search_results.value = data.data
        if(data.data == 0){
        alert('No anime found with this name')
        }
    })
}

const handleInput = (e) => {
	if (!e.target.value) {
		search_results.value = []
	}
}

const addAnime = (anime) => {
	search_results.value = []
	query.value = ''
	my_anime.value.push({
		id: anime.mal_id,
		title: anime.title,
		image: anime.images.jpg.image_url,
		total_episodes: anime.episodes,
		watched_episodes: 0,
	})
	localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}

const increaseWatch = (anime) => {
	anime.watched_episodes++
	localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}

const decreaseWatch = (anime) => {
	anime.watched_episodes--
	localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
} 

const deleteAnimeItem = (anime,my_anime_asc) => {
	my_anime_asc = my_anime_asc.filter(animeItem => animeItem.id != anime.id)	
	my_anime.value = my_anime.value.filter(animeItem => animeItem.id != anime.id)
	console.log(my_anime.value, my_anime_asc)	
}

onMounted(() => {
	my_anime.value = JSON.parse(localStorage.getItem('my_anime')) || []
})


</script>

<template>
	<main>
		<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-Zenh87qX5JnK2Jl0vWa8Ck2rdkQ2Bzep5IDxbcnCeuOxjzrPF/et3URy9Bv1WTRi" crossorigin="anonymous">

<div class="container text-center">
		<h1>My Anime Tracker</h1>

		<form @submit.prevent="searchAnime" class="form mb-3">
			<input type="text form-control form-control-lg" placeholder="Search for an anime..." v-model="query" @input="handleInput" />
			<button type="submit" class="button btn btn-primary">Search</button>
		</form>

		<div class="results" v-if="search_results.length > 0">
			<div v-for="anime in search_results" class="result card">
				<img :src="anime.images.jpg.image_url" class="card-img-top" />
				<div class="details card-body">
					<h3 class="card-title">{{ anime.title }}</h3>
					<p class='card-text' :title="anime.synopsis" v-if="anime.synopsis">{{ anime.synopsis.slice(0, 120) }}...</p>
					<span class="flex-1"></span>
					<button @click="addAnime(anime)" class="btn btn-primary button">Add to My Anime</button>
				</div>
			</div>
		</div>


		
			<h2 class="h2">My Anime List</h2>
		<div class="myanime container" v-if="my_anime.length > 0">

			<div v-for="anime in my_anime_asc" class="anime card">
				<img class="card-img-top" :src="anime.image" />
				<h3 class="card-title">{{ anime.title }}</h3>
				<div class="flex-1"></div>
				<span class="episodes">{{ anime.watched_episodes }} / {{ anime.total_episodes }}</span>
				<button 
					v-if="anime.total_episodes !== anime.watched_episodes" 
					@click="increaseWatch(anime)" class="btn btn-primary button">+</button>
				<button 
					v-if="anime.watched_episodes > 0"
					@click="decreaseWatch(anime)" class="btn btn-danger button">-</button>
				<button class="btn" @click="deleteAnimeItem(anime,my_anime_asc)">Remove </button>
			</div>
		</div>
  
  
</div> <!-- fim container -->  
  </main>
</template>


<style>

.results {
	overflow-y: scroll;
	max-height: 480px;
	display:flex;

	flex-wrap: wrap;
	justify-content: center;
}

.card{
	width:17rem;
	margin: 1em;
}

.myanime{
	display:flex;
	flex-wrap:wrap;
	justify-content: center;
}

</style>
