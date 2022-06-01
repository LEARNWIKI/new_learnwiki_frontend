<template>
  <div class="home">
    <button @click="save">Сохранить</button>
    <VueFlow v-model="elements" class="basicflow" @connect="handleConnect" id="options-api">
      <Background/>
      <MiniMap/>
      <Controls/>
    </VueFlow>
    {{elements}}
  </div>
</template>

<script>
// @ is an alias to /src

import {Background, Controls, MiniMap, VueFlow, useVueFlow} from '@braks/vue-flow'
import axios from 'axios'

const {addEdges} = useVueFlow({id: 'options-api'})

export default {
  name: 'HomeView',
  data() {
    return {
      elements: []
    }
  },
  mounted() {
    axios
        .get('http://127.0.0.1:8000/api/node')
        .then((response) => {
          let data = response.data
          for (const i of data) {
            console.log(i)
            this.elements.push({
              id: i.id,
              label: i.label,
              position: {
                x: i.x,
                y: i.y
              },
            })
          }
        })
    axios
        .get('http://127.0.0.1:8000/api/link')
        .then((response) => {
          let data = response.data
          let edges = []
          for (const i of data) {
            console.log(i)
            edges.push({
              id: 'e' + i.id,
              source: i.from_links,
              target: i.to,
              type: 'straight'
            })
          }
          addEdges(edges)
        })
    console.log(this.elements)
  },
  components: {
    VueFlow,
    MiniMap,
    Background,
    Controls
  },
  methods: {
    handleConnect: (params) => {
      console.log("create edge")
      addEdges([params])
    },
    save() {
      console.log(this.elements)
      for (const elem of this.elements) {
        axios.put('http://127.0.0.1:8000/api/node/', {
          id: elem.id,
          label: elem.label,
          x: elem.position.x,
          y: elem.position.y,
          type_node: "G"
        })
      }
    }
  }
}
</script>

<style scoped>
@import '@braks/vue-flow/dist/style.css';
@import '@braks/vue-flow/dist/theme-default.css';

.home {
  margin: 0;
  height: 100%;
}
</style>
