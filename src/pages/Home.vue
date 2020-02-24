<template>
  <div class="wrapper-content wrapper-content--fixed">
    <section>
      <div class="container">
        <!-- errors -->
        <div class="error" v-if="error" style="margin-bottom: 20px;">
          <p>{{ error }}</p>
        </div>

        <!-- search -->
        <search
          :value="search"
          placeholder = "Type username..."
          @search="search = $event" />
        <!-- buttons -->
        <button v-if="!repos" class="btn btnPrimary" @click="getRepos">Search!</button>
        <button v-else class="btn btnPrimary" @click="getRepos">Search Again!</button>

        <!-- wrapper -->
        <div class="repos__wrapper" v-if="repos">
          <div class="repos-user">
            <img class="repos-user-img" :src="user_info.avatar_url" :alt="user_info.name">
            <div class="repos-user-info">
              <p class="repos-user__name">{{ user_info.name }}</p>
              <p class="repos-user__repos">
                <strong>{{ user_info.public_repos }}</strong>
                repositories
              </p>
            </div>
          </div>
          <!-- item -->
          <table>

            <thead>
              <tr>
                <th @click="sort('name')">Name &#8595;</th>
                <th @click="sort('stargazers_count')">Stars &#8595;</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="repo in reposSort" :key="repo.id">
                <td>
                  <a class="link" target="_blank" :href="repo.html_url">{{repo.name}}</a>
                </td>
                <td>
                  {{ repo.stargazers_count }} ⭐
                </td>
              </tr>
            </tbody>

          </table>
          <!-- pagination -->
          <div class="button-list">
            <div class="btn btnPrimary" @click="prevPage"> &#8592; </div>
            <div class="btn btnPrimary" @click="nextPage"> &#8594; </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
  import search from '@/components/Search.vue';
  import axios from 'axios'
  export default {
    components: { search },
    data () {
      return {
        search: '',
        error: null,
        repos: null,
        user_info: null,
        currentSort: 'name',
        currentSortDir: 'asc',
        page: {
          current: 1,
          length: 5
        }
      }
    },
    computed: {
      reposSort () {
        if(this.repos) {
          return this.repos.sort((a, b) => {
            let mod = 1
            if (this.currentSortDir === 'desc') {
              mod = -1
            }
            if (a[this.currentSort] < b[this.currentSort]) {
              return -1 * mod
            }
            if (a[this.currentSort] > b[this.currentSort]) {
              return 1 * mod
            }
            return 0
          }).filter((row, index) => {
            let start = (this.page.current - 1) * this.page.length
            let end = this.page.current * this.page.length
            if(index >= start && index < end) {
              return true
            }
          })

        }
      }
    },
    methods: {
      getRepos () {
        axios.all([
          axios.get(`https://api.github.com/users/${this.search}/repos`),
          axios.get(`https://api.github.com/users/${this.search}`)])
            .then(axios.spread((res, user_info) => {
              this.repos = res.data
              this.user_info = user_info.data
              this.error = null
            }))
            .catch(err => {
              console.log(err)
              this.repos = null
              this.error = 'can`t find this user'
            })
      },
      sort (e) {
        if(e === this.currentSort) {
          this.currentSortDir = this.currentSortDir === 'asc' ? 'desc' : 'asc'
        }
        this.currentSort = e
      },

      // Pagination
      prevPage () {
        if (this.page.current > 1) {
          this.page.current --
        }
      },
      nextPage () {
        if ((this.page.current * this.page.length) < this.repos.length) {
          this.page.current ++
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  .container {
    display: flex;
    align-items: center;
    flex-direction: column;
  }

  .repos__wrapper {
    width: 400px;
    margin: 30px 0;
  }

  .repos-item {

  }

  .repos-info {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 10px;
    padding: 10px 0;
    border-bottom: 1px solid #dbdbdb;
  }

  button {
    margin-top: 40px;
  }

  .repos-user {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 15px 0 30px;
  }

  .repos-user-img {
    width: 120px;
    flex: 0 0 120px;
    border-radius: 50%;
  }

  .repos-user-info {
    text-align: right;

    .repos-user__repos {
      font-size: 13px;
      strong {
        font-size: 15px;
      }
    }
  }

  table {
    width: 100%;
  }

  td {
    padding: 20px 0 10px;
    border-bottom: 1px solid #dbdbdb;
    vertical-align: middle;
  }

  td:nth-child(2n),
  th:nth-child(2) {
    text-align: right;
  }

  .button-list {
    width: 100%;
    text-align: center;
    padding: 20px 0;

    .btn {
      border-radius: 60px;
      margin: 0 20px;
    }
  }


</style>
