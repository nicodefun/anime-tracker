<script setup>
import { ref, computed, onMounted } from "vue";
const query = ref("");
const search_results = ref([]);
const input = ref(null);

const my_anime = ref([]);
const my_anime_asc = computed(() => {
  return my_anime.value.sort((a, b) => {
    return a.title.localeCompare(b.title);
  });
});

const searchName = async () => {
  if (!query.value) {
    search_results.value = [];
    console.log("no search value");
    return;
  }
  const url = `https://api.jikan.moe/v4/anime?q=${query.value}`;
  await fetch(url)
    .then((res) => res.json())
    .then((res) => {
      search_results.value = res.data;
      // console.log(res);
    });
  // console.log(search_results.value);
};
const anime_push = (anime) => {
  my_anime.value.push({
    id: anime.mal_id,
    title: anime.title,
    image: anime.images.jpg.image_url,
    total_episodes: anime.episodes,
    watched_episodes: 0,
  });
  localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};

const addAnime = (anime) => {
  search_results.value = [];
  if (my_anime.value.length === 0) {
    anime_push(anime);
  } else {
    let mapped = my_anime.value.map((item) => item.id);
    if (mapped.includes(anime.mal_id)) {
      console.log("exist");
    } else {
      anime_push(anime);
    }
  }
};

const increaseWatch = (anime) => {
  anime.watched_episodes++;
  localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};
const decreaseWatch = (anime) => {
  anime.watched_episodes--;
  localStorage.setItem("my_anime", JSON.stringify(my_anime.value));
};

onMounted(() => {
  my_anime.value = JSON.parse(localStorage.getItem("my_anime") || []);
});
</script>

<template>
  <div>
    <h1>Anime Tracker</h1>
    <section class="search-container">
      <form @submit.prevent="searchName">
        <input
          type="text"
          placeholder="Search for an anime..."
          v-model="query"
          ref="input"
        />
        <button class="button" type="submit">Search</button>
      </form>
      <div class="serarch-results" v-if="search_results.length > 0">
        <div class="result" v-for="result in search_results">
          <img :src="result.images.jpg.image_url" :alt="result.title" />
          <div class="details">
            <h3>{{ result.title }}</h3>
            <p :title="result.synopsis" v-if="result.synopsis">
              {{ result.synopsis.slice(0, 100) }}
            </p>
            <span class="flex-1"></span>
            <button class="button" @click="addAnime(result)">
              Add to my anime
            </button>
          </div>
        </div>
      </div>
    </section>
    <section class="my-anime" v-if="my_anime.length > 0">
      <h2>My Anime</h2>
      <div v-for="anime in my_anime_asc" class="anime">
        <img :src="anime.image" alt="anime.title" />
        <h3>{{ anime.title }}</h3>
        <div class="flex-1"></div>
        <span class="episodes">
          {{ anime.watched_episodes }} / {{ anime.total_episodes }}
        </span>
        <button
          class="button"
          v-if="anime.total_episodes !== anime.watched_episodes"
          @click="increaseWatch(anime)"
        >
          +
        </button>
        <button
          class="button"
          v-if="anime.watched_episodes > 0"
          @click="decreaseWatch(anime)"
        >
          -
        </button>
      </div>
    </section>
  </div>
</template>

<style scoped>
h1 {
  text-align: center;
  margin-bottom: 1.5rem;
}
form {
  display: flex;
  max-width: 480px;
  margin: 0 auto 1.5rem;
}
form input {
  appearance: none;
  outline: none;
  border: none;
  background: white;

  display: block;
  color: #888;
  font-size: 1.125rem;
  padding: 0.5rem 1rem;
  width: 100%;
}
.button {
  appearance: none;
  outline: none;
  cursor: pointer;
  border: none;
  background: none;
  font-family: inherit;
  display: block;
  padding: 0.5rem 1rem;
  font-size: 1.125rem;
  font-weight: 600;
  text-transform: uppercase;
  transition: 0.4s;
  color: white;
  background-size: 200%;
  background-image: linear-gradient(
    to right,
    rgba(117, 198, 135, 1) 43%,
    rgba(28, 135, 45, 1) 100%
  );
}
button:hover {
  background-position: right;
}
.serarch-results {
  background-color: #fff;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  max-height: 480px;
  overflow-y: scroll;
  margin-bottom: 1.5rem;
}
.result {
  display: flex;
  margin: 1rem;
  border: 1px solid #ccc;
  border-radius: 0.5rem;
  transition: 0.4s;
}
.result img {
  width: 100px;
  border-radius: 0.5rem;
  margin-right: 1rem;
  height: 130px;
}
.details {
  flex: 1 1 0%;
  display: flex;
  align-items: flex-start;
  flex-direction: column;
}
.flex-1 {
  flex: 1 1 0%;
}
.details h3 {
  font-size: 1.25rem;
  margin-bottom: 0.5rem;
}
.details p {
  font-size: 0.875rem;
  margin-bottom: 0.5rem;
  line-height: 1.4rem;
}
.details .button {
  margin-left: auto;
}
.my-anime h2 {
  color: grey;
  font-weight: 400;
  margin-bottom: 1.5rem;
}
.my-anime .anime {
  display: flex;
  align-items: center;
  margin-bottom: 1.5rem;
  background-color: #fff;
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
.anime img {
  width: 72px;
  height: 72px;
  object-fit: cover;
  border-radius: 1rem;
  margin-right: 1rem;
}
.anime h3 {
  color: #888;
  font-size: 1.125rem;
  width: 50%;
}
.anime .episodes {
  margin-right: 1rem;
  color: #888;
}
.anime .button:first-of-type {
  margin-right: 1rem;
}
.anime .button:last-of-type {
  margin-right: 0rem;
}
</style>
