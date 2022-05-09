<!-- Kod skapad av Simon Andersson i kursen DT162G vid Mittuniversitetet -->

<script>
// URL till webbtjänsten
const url = "https://dt162g-projekt-api.herokuapp.com/";

export default {
  // DATA - Koppla data till komponenten (App)
  data() {
    return {
      // Tom array för alla poster
      posts: [],
      // Visa spara-knappen i formuläret
      showSaveBtn: true,
      // Skapa tom variabel för id till PUT-anrop
      putid: "",
    };
  },

  // METODER
  methods: {
    // GET - hämta alla poster
    getPosts() {
      // Anropa GET från webbtjänsten för att hämta alla poster
      fetch(url)
        .then((response) => response.json())
        .then((data) => {
          // Lagra svarsdatan i posts för tillgång i appen
          this.posts = data.reverse();
        });
    },

    // DELETE - radera enskild post
    removePost(_id) {
      // Lagra medskickat id
      let id = _id.target.id;

      // Anropa webbtjänsten med verbet delete, skicka med id i url:en
      fetch(url + id, {
        method: "DELETE",
      })
        .then((response) => response.json())
        .then((data) => {
          // Lagra svarsdatan i posts för tillgång i appen
          this.posts = data.reverse();
        });
    },

    // POST - lagra ny post
    addPost() {
      // Skapa tom array att lagra inmatad data till
      let body = {};

      // Hämta inmatad data och lagra i arrayen
      body.purchaseTitle = this.newTitle;
      body.purchaseDate = this.newDate;
      body.purchaseCost = this.newCost;
      body.purchaseInfo = this.newInfo;

      // Anropa webbtjänsten med verbet POST för att lagra ny post
      fetch(url, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body),
      })
        .then((response) => response.json())
        .then((data) => {
          // Lagra svarsdatan i posts för tillgång i appen
          this.posts = data.reverse();
        });

      // Rensa inmatningsfälten
      this.newTitle = "";
      this.newDate = "";
      this.newCost = "";
      this.newInfo = "";
    },

    // Ladda upp och ändra enskild post
    changePost(_id) {
      // Lagra medskickat id för posten som ska ändras
      this.putid = _id.target.id;

      // Visa ändra-knappen och ge den medskickat id
      this.showSaveBtn = false;

      // Hämta vald post
      fetch(url + this.putid)
        .then((response) => response.json())
        .then((data) => {
          // Skriv ut datan till formuläret
          this.newTitle = data.purchaseTitle;
          this.newDate = new Date(data.purchaseDate).toLocaleDateString();
          this.newCost = data.purchaseCost;
          this.newInfo = data.purchaseInfo;

          // Uppdatera renderingen för att visa data i formuläret
          this.$forceUpdate();
        });
    },

    // PUT - ändra enskild post
    putPost() {
      // Skapa tom array att lagra inmatad data till
      let body = {};

      // Hämta inmatad data och lagra i arrayen
      body.purchaseTitle = this.newTitle;
      body.purchaseDate = this.newDate;
      body.purchaseCost = this.newCost;
      body.purchaseInfo = this.newInfo;

      // Anropa webbtjänsten med verbet PUT för att ändra posten, skicka med postens ID
      fetch(url + this.putid, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body),
      })
        .then((response) => response.json())
        .then((data) => {
          // Lagra svarsdatan i posts för tillgång i appen
          this.posts = data.reverse();
        });

      // Rensa inmatningsfälten
      this.newTitle = "";
      this.newDate = "";
      this.newCost = "";
      this.newInfo = "";

      // Visa spara-knappen igen
      this.showSaveBtn = true;

      // Nollställ PUT-id
      this.putid = "";
    },
  },
  mounted() {
    // Hämta in och tillgängliggör data från webbtjänsten vid laddning av appen
    this.getPosts();
  },
};
</script>

<template>
  <!-- Sidhuvud -->
  <header>
    <h1>Mina matinköp</h1>
  </header>

  <!-- Huvudinnehållet -->
  <main>

    <!-- Sektion för att mata in data/formulär -->
    <section id="inputsection">
      <form>
        <span>
          <label for="title">Rubrik</label>
          <input type="text" name="title" id="title" v-model="newTitle" />
          <br />
          <label for="date">Inköpsdatum</label>
          <input type="date" name="date" id="date" v-model="newDate" />
          <br />
          <label for="cost">Kostnad</label>
          <input type="number" name="cost" id="cost" v-model="newCost" />
          <br />
        </span>
        <span>
          <label for="info">Information</label>
          <textarea rows="6" name="info" id="info" v-model="newInfo" />
          <br />
          <input
            v-if="showSaveBtn"
            type="button"
            value="Spara"
            class="savebtn"
            @click="addPost"
          />
          <input
            v-else
            type="button"
            value="Ändra"
            class="changebtn"
            @click="putPost"
          />
          <br />
        </span>
      </form>
    </section>

    <!-- Sektion för utskrift av data -->
    <section id="outputsection">
      <!-- Artikel för att skriva ut respektive post, loopas ut med v-for -->
      <article v-for="post in posts" :key="post._id">
        <h2>{{ post.purchaseTitle }}</h2>
        <p class="date">
          {{ new Date(post.purchaseDate).toLocaleDateString() }}
        </p>
        <p class="cost">{{ post.purchaseCost }} kr</p>
        <p class="info">{{ post.purchaseInfo }}</p>
        <div>
          <button class="change" v-bind:id="post._id" @click="changePost">
            Ändra
          </button>
          |
          <button class="delete" v-bind:id="post._id" @click="removePost">
            Ta bort
          </button>
        </div>
      </article>
    </section>
  </main>

  <!-- Sidfot -->
  <footer>
    <p>
      Webbplats skapad av Simon Andersson som projektuppgift i kursen DT162G,
      JavaScriptbaserad webbutveckling
    </p>
  </footer>
</template>

<style>
@import "./assets/base.css";

#app {
  background-color: rgb(238, 238, 238);
  width: 100%;
  height: 100%;
  min-height: 100vh;
}

main {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

header {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 75px;
  background-color: white;
  width: 100%;
  box-shadow: 0 2px 5px 5px rgba(0, 0, 0, 0.015);
}

h1 {
  display: flex;
  align-items: center;
  gap: 15px;
}

button:hover {
  cursor: pointer;
}

section {
  background: white;
  border-radius: 10px;
  padding: 2%;
  margin: 4%;
  width: 90%;
  box-shadow: 0 2px 50px 150px rgba(0, 0, 0, 0.015);
}

article {
  background-color: rgb(240, 240, 240);
  padding: 5%;
  padding-bottom: 20%;
  border-radius: 10px;
  margin-bottom: 5%;
}

article * {
  margin: 2%;
}

.date {
  font-weight: 200;
  margin-top: -10px;
}

.cost {
  font-size: 1.75em;
  font-weight: 500;
}

article div {
  float: right;
  white-space: nowrap;
  margin-top: 7%;
}

article a {
  font-weight: 600;
}

.delete {
  color: var(--red);
}

#inputsection {
  max-height: 600px;
}

form {
  width: 100%;
}

form span {
  display: flex;
  flex-direction: column;
}

form input {
  padding: 3%;
}

#info {
  height: 100%;
  max-height: 150px;
}

.savebtn {
  background: var(--green);
  border: none;
  border-radius: 10px;
  padding: 5%;
  font-weight: 600;
  font-size: 1.25em;
}

.changebtn {
  background: var(--orange);
  border: none;
  border-radius: 10px;
  padding: 5%;
  font-weight: 600;
  font-size: 1.25em;
}

.savebtn,
.changebtn:hover {
  cursor: pointer;
}

footer {
  width: 100%;
  position: static;
  display: flex;
  justify-content: center;
  align-items: center;
  bottom: 0;
  padding: 0.5%;
  margin-top: 75px;
}

footer p {
  text-align: center;
  font-size: 0.75em;
}

/* DATORSTORLEK */
@media (min-width: 900px) {
  body {
    display: flex;
    place-items: center;
  }

  header {
    height: 75px;
    width: 100%;
  }

  section {
    max-width: 800px;
  }

  #inputsection {
    max-height: 325px;
    margin-bottom: 0;
  }

  #outputsection {
    display: flex;
    flex-wrap: wrap;
    padding-bottom: 0;
  }

  article {
    width: 45%;
    margin: auto;
    margin-bottom: 50px;
    padding-bottom: 5%;
  }

  article div {
    margin-bottom: -2%;
  }

  form {
    display: flex;
  }

  form span {
    margin-left: 25px;
    margin-right: 25px;
    min-width: 320px;
  }
}
</style>