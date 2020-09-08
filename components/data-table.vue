<template>
  <div class="wrapper">
    <div class="search">
      <input v-model="searchQuery" class="search__input" type="text" />
      <button class="btn search__btn" @click="search(users)">search</button>
    </div>
    <table class="table">
      <tr class="table__row table-header">
        <th
          v-for="item in content"
          :key="item.id"
          class="table__cell table-header__cell"
          :class="[
            sortedBy === item.id && sortedTo === 'up'
              ? 'table-header__cell--up'
              : 'table-header__cell--down',
          ]"
        >
          <button class="table-header__btn" @click="sortData(item.id)">
            {{ item.displaydText }}
          </button>
        </th>
      </tr>
      <tr
        v-for="user in tableData"
        :key="user.id"
        class="table__row"
        @click="showPopup(user)"
      >
        <td v-for="item in content" :key="item.id" class="table__cell">
          {{ user[item.id] }}
        </td>
      </tr>
    </table>
    <div v-if="isPopupOpen" class="popup__outside">
      <div class="table__popup popup">
        <p v-for="item in popupData" :key="item.id" class="popup__text">
          {{ item.title }}: {{ popupItem[item.id] }}
        </p>
        <button class="popup__close" @click="isPopupOpen = false"></button>
      </div>
    </div>
    <div class="app__pagination pagination">
      <button
        class="pagination__btn btn"
        :class="[currentPage === 1 ? 'btn--disabled' : '']"
        :disabled="currentPage === 1"
        @click="prevPage"
      >
        Prev
      </button>
      <button
        v-if="currentPage !== 1"
        class="pagination__btn btn"
        @click="goToPage(1)"
      >
        1
      </button>
      <button class="pagination__btn btn btn--current" disabled>
        {{ currentPage }}
      </button>
      <button
        v-if="currentPage !== maxPage"
        class="pagination__btn btn"
        @click="goToPage(maxPage)"
      >
        {{ maxPage }}
      </button>
      <button
        class="pagination__btn btn"
        :class="[currentPage === maxPage ? 'btn--disabled' : '']"
        :disabled="currentPage === maxPage"
        @click="nextPage"
      >
        Next
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    users: {
      type: Array,
      required: true,
    },
    content: {
      type: Array,
      required: true,
    },
    popupData: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      offset: 25,
      currentUsers: [],
      isPopupOpen: false,
      popupItem: null,
      currentPage: 1,
      sortedBy: null,
      sortedTo: null,
      searchQuery: '',
    }
  },
  computed: {
    tableData() {
      const pageData = this.currentUsers.slice(
        (this.currentPage - 1) * this.offset,
        this.currentPage * this.offset
      )
      const formatedData = pageData.map((item) => this.formatUserData(item))
      return formatedData
    },
    maxPage() {
      if (this.currentUsers.length === 0) {
        return 1
      }
      return Math.ceil(this.currentUsers.length / this.offset)
    },
  },
  mounted() {
    this.currentUsers = this.users
  },
  methods: {
    formatUserData(item) {
      const newItem = {
        id: item.id,
      }
      for (const elem of this.content) {
        newItem[elem.id] = this.findKeyValue(item, elem.id)
      }
      for (const elem of this.popupData) {
        newItem[elem.id] = !Object.keys(newItem).includes(elem.id)
          ? this.getPopupValue(item, elem.id)
          : newItem[elem.id]
      }
      return newItem
    },
    findKeyValue(obj, key) {
      const keys = Object.keys(obj)
      const values = Object.values(obj)
      let value = null
      if (keys.includes(key)) {
        value = obj[key]
      } else {
        const objArr = values.filter((item) => typeof item === 'object')
        for (let i = 0; i < objArr.length; i++) {
          if (value != null) {
            break
          }
          value = this.findKeyValue(objArr[i], key)
        }
      }
      return value
    },
    showPopup(item) {
      this.popupItem = item
      this.clickOutside()
      this.isPopupOpen = true
    },
    clickOutside() {
      const outsideClicker = (event) => {
        if (
          event.target.className.includes('outside') ||
          event.key === 'Escape'
        ) {
          this.isPopupOpen = false
          document.removeEventListener('click', outsideClicker, true)
          document.removeEventListener('keydown', outsideClicker, true)
        }
      }
      document.addEventListener('click', outsideClicker, true)
      document.addEventListener('keydown', outsideClicker, true)
    },
    getPopupValue(obj, key) {
      let value = this.findKeyValue(obj, key)
      if (typeof value === 'object') {
        for (const item in value) {
          if (typeof value[item] === 'object') {
            delete value[item]
          }
        }
        value = Object.values(value).join(', ')
      }
      return value
    },
    prevPage() {
      this.currentPage--
    },
    nextPage() {
      this.currentPage++
    },
    goToPage(page) {
      this.currentPage = page
    },
    sortData(item) {
      if (this.sortedBy === item) {
        this.currentUsers.reverse()
        this.sortedTo = this.sortedTo === 'up' ? 'down' : 'up'
      } else {
        this.currentUsers.sort((a, b) => {
          const nameA = this.findKeyValue(a, item)
          const nameB = this.findKeyValue(b, item)
          if (nameA < nameB) return -1
          if (nameA > nameB) return 1
          return 0
        })
        this.sortedBy = item
        this.sortedTo = 'up'
      }
    },
    search() {
      this.currentPage = 1
      this.currentUsers = this.users.filter((item) => {
        return this.subSearch(item) === true
      })
    },
    subSearch(item) {
      for (const i in item) {
        if (typeof item[i] === 'object' && this.subSearch(item[i])) return true
        else if (
          item[i]
            .toString()
            .toLowerCase()
            .includes(this.searchQuery.toString().toLowerCase())
        ) {
          return true
        }
      }
    },
  },
}
</script>
<style lang="scss" scoped>
.table {
  table-layout: fixed;
  width: 100%;
  margin: {
    left: auto;
    right: auto;
    bottom: 40px;
  }
  &__row {
    cursor: pointer;
    &:hover {
      background: {
        color: #dddddd;
      }
      .table__cell {
        border-color: #808080;
      }
    }
  }
  &__cell {
    position: relative;
    border-width: 1px;
    word-break: break-all;
  }
}
.table-header {
  &__cell {
    padding: 0;
    &::before,
    &::after {
      content: '';
      position: absolute;
      top: 20px;
      right: 10px;
      //display: none;
      width: 15px;
      height: 2px;
      background: {
        color: #000;
      }
      transform: rotate(45deg);
    }
    &::after {
      transform: rotate(-45deg);
      right: 20px;
    }
    &--up {
      &::before {
        right: 10px;
        transform: rotate(-45deg);
      }
      &::after {
        transform: rotate(45deg);
      }
    }
    &:hover {
      &::after,
      &::before {
        display: block;
      }
    }
  }
  &__btn {
    display: block;
    width: 100%;
    padding: 0.5em 0.75em;
    background: {
      color: #fff;
    }
    border: none;
    outline: none;
    font: {
      size: 16px;
    }
    text-align: left;
    cursor: pointer;
    &:hover {
      background: {
        color: #616161;
      }
      color: #fff;
    }
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
  &--disabled {
    background: {
      color: grey;
    }
  }
  &--current {
    background: {
      color: #05ff5c;
    }
  }
}
.search {
  display: flex;
  margin: {
    bottom: 20px;
  }
  &__btn {
    margin: {
      left: 10px;
    }
    font: {
      size: 14px;
    }
  }
  &__input {
    padding: 5px;
    border: 1px solid black;
  }
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
.popup {
  position: absolute;
  top: 200px;
  left: calc(50% - 160px);
  width: 320px;
  height: auto;
  padding: 30px;
  background: {
    color: #ebe69f;
  }
  border: 1px solid #cec87d;
  box-shadow: 0 0 5px 2px #cec87d;
  &__outside {
    z-index: 1000;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100vh;
  }
  &__close {
    position: absolute;
    top: 10px;
    right: 10px;
    display: block;
    width: 15px;
    height: 15px;
    background: transparent;
    border: none;
    cursor: pointer;
    &::before,
    &::after {
      content: '';
      position: absolute;
      top: 5px;
      left: 0;
      display: block;
      width: 15px;
      height: 2px;
      background: black;
      transform: rotate(-45deg);
    }
    &::after {
      transform: rotate(45deg);
    }
  }
}
</style>
