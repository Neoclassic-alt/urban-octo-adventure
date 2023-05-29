<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  data(): {
    name: string,
    age: number,
    children: {[id: number]: {name: string, age: number}},
    hasInfo: boolean
  } {
    return {
      name: "",
      age: 0,
      children: {},
      hasInfo: false
    }
  },
  methods: {
    plural(forms: string[], n: number) {
	    let idx;
	    if (n % 10 === 1 && n % 100 !== 11) {
	        idx = 0; // many
	    } else if (n % 10 >= 2 && n % 10 <= 4 && (n % 100 < 10 || n % 100 >= 20)) {
	        idx = 1; // few
	    } else {
	        idx = 2; // one
	    }
	    return forms[idx] || '';
    }
  },
  mounted() {
    const name = localStorage.getItem("name")
    const age = localStorage.getItem("age")
    const children = localStorage.getItem("children")
    if(name !== null && age !== null && children !== null) {
      this.name = name
      this.age = Number(age)
      this.children = JSON.parse(children)
      this.hasInfo = true
    }
  }
})
</script>

<template>
  <main class="container">
    <template v-if="hasInfo">
      <h3>Персональные данные</h3>
      <p><b>{{ name }}, {{ age }} {{ plural(['год', 'года', 'лет'], age) }}</b></p>
      <template v-if="Object.keys(children).length > 0">
        <h3>Дети</h3>
        <p v-for="(child, id) in children" :key="id" class="child">
          <b>{{ child.name }}, {{ child.age }} {{ plural(['год', 'года', 'лет'], child.age) }}</b>
        </p>
      </template>
      
    </template>
    <p v-else><i>Информация отсутствует</i></p>
  </main>
</template>

<style>
.container {
  width: 620px;
  margin: 0 auto;
}
.child {
  padding: 10px 20px;
  background: #F1F1F1;
  border-radius: 5px;
  width: fit-content;
  margin-block-end: 0;
  margin-block-start: 0;
  margin-bottom: 10px;
}
</style>