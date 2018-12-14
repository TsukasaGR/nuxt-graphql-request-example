<template>
  <v-layout>
    <v-flex text-xs-center>
      <h1>GraphQLサンプルページ</h1>
      <v-btn color="primary" @click="getUsers">Usersを取得する</v-btn>
      <p class="title">Users</p>
      <div class="users">
        <div v-for="(user, index) in users" :key="index" class="user">
          <div v-if="editId !== user.id" class="user-box">
            <div class="user-row">
              <div class="key">name:</div>
              <div class="value">{{ user.name }}</div>
            </div>
            <div class="user-row">
              <div class="key">email:</div>
              <div class="value">{{ user.email }}</div>
            </div>
            <v-btn color="primary" @click="setEditValues(user)">編集する</v-btn>
          </div>
          <div v-else class="user-box">
            <div class="user-row">
              <div class="key">name:</div>
              <div class="value">
                <v-text-field label="Name" v-model="editName"/>
              </div>
            </div>
            <div class="user-row">
              <div class="key">email:</div>
              <div class="value">
                <v-text-field label="Email" v-model="editEmail"/>
              </div>
            </div>
            <v-btn color="primary" @click="updateUser()">更新する</v-btn>
          </div>
        </div>
      </div>
    </v-flex>
  </v-layout>
</template>

<script>
import { GraphQLClient } from 'graphql-request'

export default {
  data() {
    return {
      client: 0,
      endPoint: 'http://127.0.0.1:8000/graphql',
      users: [],
      editId: null,
      editName: null,
      editEmail: null
    }
  },
  created() {
    this.clientInitialize()
  },
  methods: {
    clientInitialize() {
      this.client = new GraphQLClient(this.endPoint, { headers: {} })
    },
    getUsers() {
      const query = `
        query users($count: Int!) {
          users(count: $count) {
            data {
              id
              name
              email
            }
          }
        }
      `
      const variables = { count: 10 }

      this.client.request(query, variables).then(data => (this.users = data.users.data))
    },
    updateUser() {
      const query = `
        mutation updateUser(
          $id: ID
          $name: String
          $email: String
        ) {
          updateUser(
            id: $id
            name: $name
            email: $email
          ) {
            id
            name
            email
          }
        }
      `
      const variables = { id: this.editId, name: this.editName, email: this.editEmail }

      this.client.request(query, variables).then(data => {
        this.getUsers()
      })

      this.resetEditValues()
    },
    setEditValues(user) {
      this.editId = user.id
      this.editName = user.name
      this.editEmail = user.email
    },
    resetEditValues() {
      this.editId = null
      this.editName = null
      this.editEmail = null
    }
  }
}
</script>
<style lang="stylus" scoped>
.title {
  font-size: 1.5em;
  font-weight: bold;
  padding-top: 20px;
}

.users {
  width: 100%;
  display: -webkit-flex;
  display: flex;
  -webkit-flex-wrap: wrap;
  flex-wrap: wrap;

  .user {
    width: 50%;
    border: 1px solid gray;
    padding: 5px;

    .user-box {
      .user-row {
        width: 100%;
        display: -webkit-flex;
        display: flex;
        -webkit-flex-wrap: wrap;
        flex-wrap: wrap;
        text-align: left;

        .key {
          flex: 1;
        }

        .value {
          flex: 3;
        }
      }
    }
  }
}
</style>
