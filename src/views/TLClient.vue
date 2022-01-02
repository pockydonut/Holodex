<template>
  <v-container fill-height>
    <div class="d-flex flex-column" style="height:100%; width:100%">
      <v-card min-width="100%" outlined>
        <span>menu</span>
      </v-card>
      <div class="d-flex align-stretch flex-row" style="height:100%">
        <v-card class="grow" min-height="100%">
          <span>TL part</span>
        </v-card>
        <v-card min-height="100%" outlined>
          <span>Synced Chat</span>
        </v-card>
      </div>

      <v-card min-width="100%" outlined>
        <span>TLmenu</span>
      </v-card>
    </div>
  </v-container>
</template>

<script lang="ts">
export default {
    name: "Tlclient",
    metaInfo() {
        // const vm = this;
        return {
            get title() {
                return "TLClient - Holodex";
            },
        };
    },
    components: {
    },
    data() {
        return {
        };
    },
    activated() {
    },
};
</script>

<style>
</style>
