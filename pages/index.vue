<template>
  <section class="app">
    <div class="app__wrapper">
      <button
        v-if="!isUsersLoaded"
        class="app__btn btn"
        autofocus
        @click="getUsers"
      >
        Показать пользователей
      </button>
      <div v-if="isUsersLoaded">
        <users-table
          :users="users"
          :content="content"
          :popup-data="popupData"
        ></users-table>
      </div>
    </div>
  </section>
</template>

<script>
import usersTable from '../components/data-table.vue'

export default {
  name: 'HomePage',
  components: {
    usersTable,
  },
  data() {
    return {
      users: [],
      isUsersLoaded: false,
      content: [
        {
          displaydText: 'Name',
          id: 'fullname',
        },
        {
          displaydText: 'Username',
          id: 'uname',
        },
        {
          displaydText: 'Company',
          id: 'company',
        },
        {
          displaydText: 'E-mail',
          id: 'email',
        },
        {
          displaydText: 'State',
          id: 'state',
        },
      ],
      popupData: [
        {
          title: 'Name',
          id: 'fullname',
        },
        {
          id: 'address',
          title: 'Address',
        },
      ],
    }
  },
  methods: {
    async getUsers() {
      const API_URL =
        'http://www.filltext.com/?rows=1000&id={index}&fullname={firstName}~{lastName}&company={business}&email={email}&uname={username}&address={addressObject}'
      const users = await this.$axios.$get(API_URL)
      this.users = users
      this.isUsersLoaded = true
    },
  },
}
</script>

<style lang="scss" scoped>
.app {
  padding: 60px;
  &__wrapper {
    width: fit-content;
    margin: 0 auto;
  }
}
.btn {
  display: block;
  margin: {
    left: auto;
    right: auto;
  }
  padding: calc(0.5em - 1px) 1em;
  background: {
    color: #48c774;
  }
  border: 1px solid black;
  border: {
    color: transparent;
    radius: 4px;
  }
  font: {
    size: 25px;
  }
  color: #fff;
  letter-spacing: normal;
  word-spacing: normal;
  text-transform: none;
  text-indent: 0px;
  text-shadow: none;
  cursor: pointer;
}
.pagination {
  justify-content: flex-start;
  &__btn {
    margin: {
      left: 0.5em;
      right: 0.5em;
    }
    font: {
      size: 14px;
    }
  }
}
</style>
