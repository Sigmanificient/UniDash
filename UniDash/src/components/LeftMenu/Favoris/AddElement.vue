<template>
  <div class="mask-bg flex flex-center">
    <div class="add-favori-form gradient-border flex">
      <div class="content width-100 flex flex-column">
        <form @submit="addFavoris" class="flex flex-column flex-between width-100 height-100 list-none" ref="input-list">
          <div>
            <Box>
              <label>NAME :</label>
              <input class="input hover" ref="favoriName" type="text">
            </Box>
            <Box>
              <label>MODULE :</label>
              <select v-bind:value="moduleName" v-model="moduleName" class="input width-100">
                <option class="input" v-for="module in loadModules" :value="module.vuePath" :key="module.vuePath">{{ module.name }}</option>
              </select>
            </Box>
            <Box v-for="(input, index) in modules.forms" :key="index" v-show="!input.hidden">
              <label>{{ input.label.toUpperCase() }} :</label>
              <input class="input hover" :class="input.label" :type="input.type" :value="input.value">
            </Box>
          </div>
          <Box>
            <div class="flex add-close-button">
              <button class="input width-100 hover" @click="addFavorisClose">Close</button>
              <button class="input width-100 hover" type="submit">Add</button>
            </div>
          </Box>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import {computed} from "vue";
import {useStore} from "vuex";
import Box from "../../Custom/Box.vue";

export default {
  name: "AddElement",
  components: {
    Box
  },
  data() {
    let moduleName = "IframeElement";
    let modules = this.loadModules.find(module => module.vuePath === moduleName);
    return {
      moduleName,
      modules
    }
  },
  setup() {
    const store = useStore();
    const favorisFolderList = computed(() => store.state.favorisFolderList);
    const loadModules = computed(() => store.state.loadModules);

    function updateFavorisFolderList(newList) {
      store.commit('updateFavorisFolderList', newList);
    }

    return {
      updateFavorisFolderList,
      favorisFolderList,
      loadModules
    };
  },
  methods: {
    getParent(name) {
      let p = this.$parent;
      while(typeof p !== 'undefined') {
        if (p.$options.name == name) {
          return p;
        } else {
          p = p.$parent;
        }
      }
      return false;
    },
    getModule() {
      const loadModules = this.$store.state.loadModules;
      let module = loadModules.find(module => module.vuePath === this.moduleName);
      return module;
    },
    addFavoris(event) {
      event.preventDefault();

      let favorisFolderList = this.favorisFolderList;
      let favorisFolder = favorisFolderList.find(favorisFolder => favorisFolder.id === this.favorisFolder.id)
      let parent = this.getParent("FavorisFolder");

      let newFavori = {
        name: parent.$refs["add-favori-form"].$refs["favoriName"].value,
        data: {
          module: this.moduleName,
        }
      }

      for (const index in this.getModule().forms) {
        const input = this.getModule().forms[index];
        newFavori.data[input.label] = this.$refs["input-list"].getElementsByClassName(input.label)[0].value;
      }

      favorisFolder.list.push(newFavori);
      favorisFolderList.find(favorisFolder => favorisFolder.id === this.favorisFolder.id).list = favorisFolder.list

      this.updateFavorisFolderList(favorisFolderList);

      let favoriForm = this.getParent("FavorisFolder").$refs["add-favori-form"].$el;
      favoriForm.classList.add("hidden");
    },
    addFavorisClose() {
      this.getParent("FavorisFolder").$refs["add-favori-form"].$el.classList.add("hidden");
    }
  },
  props: {
    favorisFolder: {
      type: Object,
      required: true
    }
  },
  watch: {
    moduleName() {
      this.modules = this.getModule();
    }
  }
}
</script>

<style scoped lang="scss">
@import "src/style";

.add-close-button {
  gap: $light-len;
}

.mask-bg {
  position: absolute;
  width: 100vw;
  height: 100vh;

  top: 0;
  left: 0;

  background-color: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(1rem);

  z-index: 99998;

  .add-favori-form {
    width: 50vw;
    height: 75vh;

    z-index: 99999;

    .content {
      padding: $default-len;
      background-color: $gray-color;
      overflow: scroll;
      form * {
        margin-bottom: $medium-min-len;
      }
    }
  }
}
</style>