<template>
  <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <button
      class="navbar-toggler"
      type="button"
      data-toggle="collapse"
      data-target="#navModal"
      aria-controls="navModal"
      aria-expanded="false"
      aria-label="Toggle navigation"
    >
      <span class="navbar-toggler-icon"></span>
    </button>
    <router-link :to="{ name: 'home' }">
      <img alt="Vue logo" src="@/assets/logoR.png" style="width: 100px">
    </router-link>
    <div class="collapse navbar-collapse" id="navModal">
      <ul class="navbar-nav mr-auto mt-2 mt-lg-0">
        <li class="nav-item">
          <router-link :to="{ name: 'badges' }" :class="{ 'nav-link': true }">Badges</router-link>
        </li>
        <li class="nav-item">
          <router-link :to="{ name: 'stats' }" :class="{ 'nav-link': true }">Statistics</router-link>
        </li>
        <li class="nav-item">
          <router-link :to="{ name: 'rankings' }" :class="{ 'nav-link': true }">Ranking</router-link>
        </li>
        <li class="nav-item">
          <router-link :to="{ name: 'users' }" :class="{ 'nav-link': true }">Users</router-link>
        </li>
        <li class="nav-item">
          <router-link :to="{ name: 'tags' }" :class="{ 'nav-link': true }">Tags</router-link>
        </li>
        <li v-if="$store.state.users.loggedUser != null" class="nav-item">
          <a v-on:click="goToUserProfile()" class="nav-link pointer">Profile</a>
        </li>
        <li v-if="toBeOrNotToBeAdmin" class="nav-item">
          <router-link :to="{ name: 'backoffice' }" :class="{ 'nav-link': true }">Back-Office</router-link>
        </li>
        <li>
          <hr>
          <!-- Estilizar este hr, e confirmar se funciona mesmo -->
        </li>
      </ul>
      <ul v-if="$store.state.users.loggedUser === null" class="navbar-nav ml-auto">
        <li class="nav-item">
          <router-link :to="{ name: 'login' }" :class="{ 'nav-link': true }">Login</router-link>
        </li>
      </ul>
      <!-- <div  class="navbar-collapse collapse w-100 order-3 dual-collapse2"> -->
      <ul v-else class="navbar-nav ml-auto">
        <li class="nav-item">
          <img
            v-bind:src="loginUser.picture"
            alt="Ups"
            class="rounded-circle"
            style="height: 35px; width: auto;"
          >
        </li>
        <li class="nav-item">
          <div class="dropdown dropleft">
            <a
              class="nav-link"
              data-toggle="dropdown"
              role="button"
              style="height:20px;border-radius: 50%;"
            >
              <!-- O span vai se mostrar se houver notificações e o innerHTML do "sino" Vai ser o numero de notificações -->
              <span class="pointer">
                <i class="fas fa-bell" style="color: #60CAE2;">
                  {{
                  notificationsLength
                  }}
                </i>
              </span>
            </a>
            <div id="drops" class="dropdown-menu">
              <div
                v-for="(noti, cont) of loginUserNotifications"
                v-bind:key="cont"
                v-on:click="seenNotification(noti.id)"
              >
                <router-link
                  v-bind:to="{
                    name: 'thread',
                    params: { threadid: noti.idThread }
                  }"
                  class="dropdown-item"
                  v-bind:class="{ bgNoti: noti.visto, textNoti: noti.visto }"
                  v-if="!noti.visto"
                >
                  <i class="far fa-flag ii"></i>
                  <!-- <span class="userName">{{users.find(us => us.id == noti.idUserFirst).name}}</span> -->
                  <strong>{{noti.userInfo.name}}</strong>{{ noti.text }}
                </router-link>
              </div>
              <!-- <a href class="dropdown-item">oaoakodk</a>
              <a href class="dropdown-item">BOas</a>
              <span class="vermais text-right" v-on:click="verMais()">Ver Mais</span>-->
              <div class="dropdwon-divider"></div>
              <div id="dropdown-footer" class="dropdown-item">
                <ul class="list-inline viewPage">
                  <li class="list-inline-item helpers pointer viewPage">
                    <router-link
                      id="viewMore"
                      :to="{ name: 'notifications' }"
                      :class="{ 'nav-link': true }"
                    >View More</router-link>
                  </li>
                  <li
                    v-on:click="markAsRead()"
                    class="list-inline-item helpers pointer"
                  >Mark as read</li>
                  <!-- Isto podia ficar com um icone de mesnsagem aberta e com um helper -->
                </ul>
              </div>
            </div>
          </div>
        </li>
        <li class="nav-item pointer" @click="logout">
          <a class="nav-link pointer">Logout</a>
        </li>
      </ul>
      <!-- </div> -->
    </div>
  </nav>
</template>

<script>
import cookie from "cookie";
import Swal from "../../node_modules/sweetalert2/dist/sweetalert2.js";
import "../../node_modules/sweetalert2/src/sweetalert2.scss";
1;
export default {
  data() {
    return {
      numberOfNots: 10
    };
  },
  created() {
    // console.log(this.loginUser);
  },
  methods: {
    seenNotification(id) {
      console.log(id, "seenNotification");
      this.$store.commit("users/viewOrNotNotification", id);
    },
    markAsRead() {
      console.log("markasread");
      for (let i = 0; i < this.numberOfNots; i++) {
        if (
          this.$store.state.users.loggedUser.notifications[i].visto == false
        ) {
          this.$store.commit("users/viewOrNotNotification", this.$store.state.users.loggedUser.notifications[i].id);
        }
      }
    },
    logout() {
      /**
       * Pedido à API que deve apagar a cookie login,
       * e depois por o loggedUser a null
       */
      let parsedCookie = cookie.parse(document.cookie);
      let cookieSerialized = cookie.serialize("login", parsedCookie.login);

      Swal.fire({
        title: "De certeza que queres abandonar?",
        type: "question",
        showCancelButton: true,
        confirmButtonText: "Abandonar",
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33"
      }).then(result => {
        if (result.value) {
          this.$http({
            url: `http://${this.$store.getters.getIp}/auth-api/logout`,
            headers: {
              "x-access-token": parsedCookie.login
            }
          })
            .then(res => {
              if (res.data.auth == false) {
                /**
                 * Já apaga a cookie e faz logout,
                 * maaaaaaas, não envia a cookie automaticamente.........
                 */

                document.cookie = `login=; Path=/; Expires=Thu, 01 Jan 1970 00:00:01 GMT;`;
                this.$store.commit("users/unLoggedUser");
                this.$router.push({
                  name: "home"
                });
              }
            })
            .catch(err => {
              document.cookie = `login=; Path=/; Expires=Thu, 01 Jan 1970 00:00:01 GMT;`;
              this.$store.commit("users/unLoggedUser");
              this.$router.push({
                name: "home"
              });
              console.log(err, "Tá male");
            });
        }
      });

      console.log(parsedCookie, "parsedCookie");
      console.log(cookieSerialized, "logout");
    },
    goToUserProfile() {
      this.$router.push({
        name: "viewProfile",
        params: {
          userid: this.$store.state.users.loggedUser.id
        }
      });
    }
  },
  computed: {
    loginUser() {
      return this.$store.state.users.loggedUser;
    },
    notificationsLength() {
      if (this.$store.state.users.loggedUser != null) {
        if (
          this.$store.state.users.loggedUser.notifications.length > 0 &&
          this.$store.state.users.loggedUser.notifications[0] != null
        ) {
          return this.$store.state.users.loggedUser.notifications.filter(
            not => not.visto == false
          ).length;
        }
      }
    },
    toBeOrNotToBeAdmin() {
      if (
        this.$store.state.users.loggedUser != null &&
        this.$store.state.users.loggedUser.id == 1
      ) {
        console.log("É Admin, NAVBAr");
        return true;
      }

      return false;
    },
    loginUserNotifications() {
      if (
        this.$store.state.users.loggedUser.notifications.length > 0 &&
        this.$store.state.users.loggedUser.notifications[0] != null
      )
        return this.$store.state.users.loggedUser.notifications.filter(
          (not, cont) => {
            if (cont < this.numberOfNots) return true;
            return false;
          }
        );

      return [{ userInfo: {}}];
    }
  }
};
</script>

<style>
/* .navi {
  padding: 10px;
} */
.viewPage {
  color: black !important;
}
nav {
  margin-bottom: 2rem;
}
.pointer:hover {
  cursor: pointer;
}
#viewMore {
  color: black;
}
/* .navLogo {
  width: 100px;
} */
</style>
