<template>
  <main class="tree container">
    <div class="tree-viewer" v-if="metaData">
      <div class="col-1">
        <h1>{{ $t("tree.title") }}</h1>
        <div class="meta-info">
          <table>
            <tr>
              <td>{{ $t("tree.uploaded") }}</td>
              <td>
                <avatar
                  v-if="metaData.uploader.uuid !== '00000000-0000-0000-0000-000000000000'"
                  :id="metaData.uploader.uuid"
                  :title="false"
                />
                {{ metaData.uploader.name }}
              </td>
            </tr>
            <tr>
              <td :title="$t('tree.started')">
                {{ $t("tree.time") }}
              </td>
              <td>{{ metaData.time }}</td>
            </tr>
            <tr v-if="metaData.root">
              <td title="Root">Root</td>
              <td>
                <code>{{ metaData.root }}</code>
              </td>
            </tr>
            <tr v-if="metaData.referenceUser">
              <td :title="$t('tree.user')">
                {{ $t("tree.user") }}
              </td>
              <td>
                <avatar :id="metaData.referenceUser.uuid" :name="metaData.referenceUser.name" />
                {{ metaData.referenceUser.name }}
              </td>
            </tr>
          </table>
        </div>
        <button @click="expandTree">
          <font-awesome icon="plus-square" />
          {{ $t("tree.expand") }}
        </button>
        <button @click="collapseTree">
          <font-awesome icon="minus-square" />
          {{ $t("tree.collapse") }}
        </button>
      </div>
      <div class="col-2">
        <div>
          <branch
            v-for="(branch, node) in treeData"
            :branch-data="branch"
            :node="node"
            :key="node"
          />
        </div>
      </div>
    </div>
    <div v-else class="tool-intro">
      <div>
        <img alt="LuckPerms logo" src="../assets/logo.svg" />
        <div class="text">
          <h1>LuckPerms</h1>
          <p>{{ $t("tree.title") }}</p>
          <template v-if="!errors.load && !errors.unsupported">
            <router-link to="/treeview/demo">
              <button class="button demo-button">
                {{ $t("tools.demo") }}
              </button>
            </router-link>
            <p>{{ $t("tree.home.generate") }}</p>
            <ul>
              <li>
                <code>/lp tree [{{ $t("tree.home.scope") }}] [{{ $t("tree.home.player") }}]</code>
              </li>
              <li>{{ $t("tree.home.url") }}</li>
            </ul>
          </template>
          <div v-else class="error">
            <template v-if="errors.load">
              <h3>{{ $t("editor.error.title") }}</h3>
              <p>{{ $t("editor.error.info") }}</p>
              <i18n path="editor.error.new" tag="p">
                <template #command>
                  <code>/lp editor</code>
                </template>
              </i18n>
            </template>

            <template v-if="errors.unsupported">
              <h3>{{ $t("editor.unsupported.title") }}</h3>
              <i18n path="editor.unsupported.info" tag="p">
                <template #download>
                  <router-link to="/download">
                    {{ $t("editor.unsupported.download") }}
                  </router-link>
                </template>
              </i18n>
            </template>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import Avatar from "../components/Avatar.vue";
import Branch from "../components/Tree/Branch.vue";
import updateSession from "@/util/session";

export default {
  metaInfo: {
    title: "Tree",
  },
  components: {
    Avatar,
    Branch,
  },
  computed: {
    treeData() {
      const { tree } = this.$store.getters;

      if (tree.data?.tree) {
        return tree.data.tree;
      }
      return tree.data;
    },
    metaData() {
      return this.$store.state.tree?.metadata;
    },
    errors() {
      return this.$store.state.tree.errors;
    },
  },
  created() {
    if (this.treeData?.sessionId) return;
    updateSession(this.$route, "getTreeData");
  },
  methods: {
    expandTree() {
      this.$root.$emit("expandTree");
    },
    collapseTree() {
      this.$root.$emit("collapseTree");
    },
  },
  watch: {
    $route(route) {
      updateSession(route, "getTreeData");
    },
  },
};
</script>

<style lang="scss">
main.tree {
  display: flex;
  overflow-y: hidden;
}

.tree-viewer {
  width: 100%;
  height: 100%;
  max-height: 100%;
  display: flex;

  > .col-1 {
    flex: 0 0 30%;
    background: transparent;
    padding: 1rem;

    h1 {
      margin: 0;
      padding: 1rem;
      line-height: 1;
      background: rgba(255, 255, 255, 0.05);
      border-top-left-radius: 2px;
      border-top-right-radius: 2px;
    }

    .meta-info {
      background: $grey;
      padding: 1rem;
      border-bottom-left-radius: 2px;
      border-bottom-right-radius: 2px;
    }

    td:first-child {
      width: 40%;
    }

    button {
      background: rgba(0, 0, 0, 0.2);
      font: inherit;
      color: $brand-color;
      padding: 0.5rem 1rem;
      border: 0;
      margin-top: 1rem;
      margin-right: 1rem;
      cursor: pointer;

      svg {
        opacity: 0.5;
        margin-right: 0.5rem;
        color: #fff;
      }
    }
  }

  > .col-2 {
    flex: 0 0 70%;
    display: flex;
    padding: 1rem 1rem 1rem 0;

    > div {
      width: 100%;
      overflow: auto;
      padding-right: 1rem;

      > .branch {
        padding-left: 0;
      }
    }
  }
}
</style>
