<template>
  <v-app>
    <app-bar />
    <v-main class="px-md-0 px-3">
      <v-fade-transition>
        <router-view />
      </v-fade-transition>
      <vue-progress-bar />
    </v-main>
    <app-footer />
    <tour />
  </v-app>
</template>

<script>
import AppFooter from "./components/core/AppFooter.vue";
import AppBar from "./components/core/AppBar.vue";
import Tour from "./components/core/Tour.vue";
import store from "@/store/";
import ajax from "./auth/ajax";

export default {
  name: "App",
  components: { AppBar, AppFooter, Tour },
  data() {
    return {
      interval: 0
    };
  },
  beforeDestroy() {
    clearInterval(this.interval);
  },
  mounted() {
    //  [App.vue specific] When App.vue is finish loading finish the progress bar
    this.$Progress.finish();

    this.$i18n.locale =
      store.getters.getLanguagePreference === null
        ? "en"
        : store.getters.getLanguagePreference;

    if (this.firstVisit) {
      store.dispatch("setTour", { lang: this.$i18n.locale });
      this.$tours["appTour"].start();
    }

    this.interval = setInterval(() => {
      if (this.loggedInUser && navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(position => {
          ajax
            .post("user_locations", {
              latitude: position.coords.latitude,
              longitude: position.coords.longitude,
              user_id: this.loggedInUser._id
            })
            .catch(function(err) {
              console.log(err);
            });
        });
      }
    }, 10 * 60 * 1000);
  },
  created() {
    store.dispatch("fillCountriesList", { lang: this.$i18n.locale });
    store.dispatch("setCurrentCountry");
    this.$Progress.start();
    this.$router.beforeEach((to, from, next) => {
      if (to.meta.progress !== undefined) {
        let meta = to.meta.progress;
        this.$Progress.parseMeta(meta);
      }
      this.$Progress.start();

      next();
    });
    this.$router.afterEach(() => {
      this.$Progress.finish();
    });
  },
  computed: {
    firstVisit: () => store.getters.getFirstVisit
  },
  watch: {
    "$i18n.locale"(newValue) {
      store.dispatch("fillCountriesList", { lang: newValue });
      store.dispatch("setEthiopia", { lang: newValue });
      store.dispatch("setLanguagePreference", { lang: newValue });
      store.dispatch("setGraphDescriptions", { lang: newValue });
      store.dispatch("setAboutDescriptions", { lang: newValue });
      store.dispatch("setPrivacyPolicy", { lang: newValue });
      store.dispatch("setAboutActions", { lang: newValue });
      store.dispatch("setInformation", { lang: newValue });
      store.dispatch("setActions", { lang: newValue });
      store.dispatch("setStates", { lang: newValue });
      store.dispatch("setAllSymptoms", { lang: newValue });
      store.dispatch("setLearningPaths", {
        age_group: "Adults",
        lang: newValue
      });
      store.dispatch("setSelfSymptomUser", {
        userId: this.loggedInUser._id,
        demo: false,
        lang: newValue
      });
      store.dispatch("setSymptomHistory", {
        userId: this.loggedInUser._id,
        lang: this.$i18n.locale
      });
    }
  }
};
</script>

<style>
html {
  scroll-behavior: smooth;
  scroll-margin-top: 50px;
}
.v-step__arrow {
  color: white;
}

.shadow-sm {
  box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.1) !important;
}

.shadow {
  box-shadow: 0 0.35rem 0.75rem rgba(0, 0, 0, 0.1) !important;
}

.shadow-lg {
  box-shadow: 0 1rem 3rem rgba(0, 0, 0, 0.15) !important;
}

.shadow-none {
  box-shadow: none !important;
}

.shadow-in {
  box-shadow: 0 0 5px #dededede inset !important;
}

.v-event,
.v-image__image,
.v-card--shaped,
.v-snack__wrapper,
.v-menu__content,
.v-alert,
.v-dialog--active {
  border-radius: 20px 3px 20px 3px !important;
}

.v-pagination__item,
.v-pagination__navigation {
  border-radius: 10px 3px 10px 3px !important;
  border: 1px solid rgba(0, 0, 0, 0.06);
  box-shadow: 0 0.2rem 0.2rem rgba(0, 0, 0, 0.05) !important;
}

.v-skeleton-loader__bone {
  border-radius: 10px 3px 10px 3px !important;
}
</style>
