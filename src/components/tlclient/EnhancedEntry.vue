<template>
  <span class="EntryContainer">
    {{ TimeStamp }} :
    <span :style="TextStyle">
      {{ stext }}
    </span>
  </span>
</template>

<script lang="ts">
export default {
    name: "EnhancedEntry",
    props: {
        time: {
            type: Number,
            default: 0,
        },
        stext: {
            type: String,
            default: "",
        },
        cc: {
            type: String,
            default: "",
        },
        oc: {
            type: String,
            default: "",
        },
    },
    computed: {
        TimeStamp() {
            return new Date(this.time).toTimeString().split(" ")[0];
        },
        TextStyle() {
            return {
                "-webkit-text-fill-color": (this.cc === "") ? "unset" : this.cc,
                "-webkit-text-stroke-color": (this.oc === "") ? "unset" : this.oc,
                "-webkit-text-stroke-width": (this.oc === "") ? "0px" : "1px",
            };
        },
    },
};
</script>

<style>
.EntryContainer {
    font-weight: bold;
}
</style>
