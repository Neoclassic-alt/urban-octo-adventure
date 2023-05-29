<script lang="ts">
import { defineComponent } from 'vue';

const maxAge = 120;
const maxChildAge = 99;
const minDifference = 15; // минимальная разница в возрасте между родителем и ребёнком и минимальный возраст родителя

type Validate = {
  error: boolean,
  text: string
}

const nullChildValidation = {
  name: {
    error: false,
    text: "Пустое поле."
  }, 
  age: {
    error: false,
    text: "Пустое поле."
  }
}

export default defineComponent({
  data(): {
    name: string,
    age: number,
    children: {[id: number]: {name: string, age: number}},
    validation: {
      name: Validate;
      age: Validate;
      children: {[id: number]: {name: Validate, age: Validate}}
    },
    isValid: boolean,
    saved: boolean
  } {
    return {
      name: "",
      age: 0,
      children: {},
      validation: {
        name: {
          error: false,
          text: "Пустое поле."
        },
        age: {
          error: false,
          text: "Пустое поле."
        },
        children: {}
      },
      isValid: true,
      saved: false
    }
  },
  methods: {
    focus(e: Event) {
      if (e.target !== null){
        const target = e.target as HTMLElement
        target.closest(".field")!.querySelector("input")!.focus()
      }
    },
    addChild() { // добавление в конец
      const id = Math.floor(Math.random() * 1000000)
      this.children[id] = {name: "", age: 0}
      this.validation.children[id] = nullChildValidation
    },
    removeChild(id: number) {
      delete this.children[id]
      delete this.validation.children[id]
    },
    sendError(field: "name" | "age", text: string, children = false, childId = 0) {
      if (!children) {
        this.validation[field].error = true
        this.validation[field].text = text
      } else {
        this.validation.children[childId][field].error = true
        this.validation.children[childId][field].text = text
      }
      this.isValid = false
    },
    eraseValidation() {
      this.validation.name.error = false
      this.validation.age.error = false
      for (const id of Object.keys(this.children)) {
        this.validation.children[Number(id)].age.error = false
        this.validation.children[Number(id)].name.error = false
      }
      this.isValid = true
    },
    validate() {
      this.eraseValidation()
      if (this.name.length === 0) {
        this.sendError("name", "Пустое поле.")
      }
      if (this.age > maxAge) {
        this.sendError("age", `Возраст превышает ${maxAge} лет.`)
      }
      if (this.age < minDifference) {
        this.sendError("age", `Минимальный возраст родителя: ${minDifference} лет.`)
      }
      if (typeof this.age === "string"){
        this.sendError("age", `Возраст не является числом.`)
      }
      for (const [id, child] of Object.entries(this.children)) {
        if (child.name.length === 0) {
          this.sendError("name", "Пустое поле.", true, Number(id))
        }
        if (child.age > maxChildAge) {
          this.sendError("age", `Возраст ребёнка превышает ${maxChildAge} лет.`, true, Number(id))
        }
        if (this.age - child.age <= minDifference) {
          this.sendError("age", `Разница в возрасте между ребёнком и родителем должна быть не менее ${minDifference} лет.`, true, Number(id))
        }
        if (child.age < 0) {
          this.sendError("age", `Возраст ребёнка не может быть отрицательным.`, true, Number(id))
        }
        if (typeof child.age === "string"){
          this.sendError("age", `Возраст не является числом.`, true, Number(id))
        }
      }
    },
    save() {
      if (!this.isValid) {
        this.saved = false
        return
      }
      localStorage.setItem("name", this.name)
      localStorage.setItem("age", this.age.toString())
      localStorage.setItem("children", JSON.stringify(this.children))
      this.saved = true
    },
  },
  computed: {
    childrenCount() {
      return Object.keys(this.children).length
    }
  },
  mounted() {
    const name = localStorage.getItem("name")
    const age = localStorage.getItem("age")
    const children = localStorage.getItem("children")
    if(name !== null && age !== null && children !== null) {
      this.name = name
      this.age = Number(age)
      const parsedChildren = JSON.parse(children)
      for (const childId in parsedChildren) {
        this.validation.children[Number(childId)] = nullChildValidation
      }
      this.children = parsedChildren
    }
  }
})
</script>

<template>
  <main class="container">
    <h3>Персональные данные</h3>
    <div class="field" :class="{field__invalid: validation.name.error}" @click="focus">
      <p class="field__title">Имя</p>
      <input type="text" class="field__input" v-model.trim="name" />
    </div>
    <small class="field__error-text" v-show="validation.name.error">{{validation.name.text}}</small>
    <div class="field" :class="{field__invalid: validation.age.error}" @click="focus">
      <p class="field__title">Возраст</p>
      <input type="text" class="field__input" v-model.trim.number="age" />
    </div>
    <small class="field__error-text" v-show="validation.age.error">{{validation.age.text}}</small>
    <div class="fields__header">
      <h3>Дети (max. 5)</h3>
      <div 
        class="outline-button"
        @click="addChild"
        v-show="childrenCount < 5"
      >
        <span class="outline-button__plus">+</span>
        <span>Добавить ребёнка</span>
      </div>
    </div>
    <div class="fields-group" v-for="(child, id) in children" :key="id">
      <div class="fields-group__field">
        <div class="field" :class="{field__invalid: validation.children[id].name.error}" @click="focus">
          <p class="field__title">Имя</p>
          <input type="text" class="field__input" v-model.trim="children[id].name" />
        </div>
        <small class="field__error-text" v-show="validation.children[id].name.error">{{validation.children[id].name.text}}</small>
      </div>
      <div class="fields-group__field">
        <div class="field" :class="{field__invalid: validation.children[id].age.error}" @click="focus">
          <p class="field__title">Возраст</p>
          <input type="text" class="field__input" v-model.trim.number="children[id].age" />
        </div>
        <small class="field__error-text" v-show="validation.children[id].age.error">{{validation.children[id].age.text}}</small>
      </div>
      <a href="#" class="button-link" @click="removeChild(id)">Удалить</a>
    </div>
    <div class="fill-button" @click="() => {validate(); save()}">Сохранить</div>
    <span v-show="saved">Информация успешно сохранена</span>
  </main>
</template>

<style scoped>
.container {
  width: 620px;
  margin: 0 auto;
}
.field {
  border: 1px solid #F1F1F1;
  border-radius: 4px;
  padding: 8px 16px;
  margin-top: 10px;
  cursor: text;
}
.field__invalid {
  border: 1px solid #f51515;
}
.field__title {
  color: rgba(17, 17, 17, 0.48);
  margin-block-start: 0;
  margin-block-end: 0;
  margin-bottom: 3px;
}
.field__input {
  width: 100%;
  border: none;
  outline: none;
}
.field__error-text {
  color: #f51515;
}
.fields__header {
  display: flex;
  justify-content: space-between;
  margin-block-start: 0.7em;
  margin-block-end: 0.7em;
}
.outline-button {
  display: flex;
  flex-direction: row;
  align-items: center;
  padding: 0 20px;
  border: 2px solid #01A7FD;
  border-radius: 100px;
  color: #01A7FD;
  cursor: pointer;
  user-select: none;
}
.outline-button__plus {
  font-size: 28px;
  margin-right: 5px;
  font-weight: 600;
}
.fields-group {
  display: flex;
  justify-content: space-between;
  align-items: stretch;
  margin-bottom: 10px;
}
.fields-group__field {
  width: 270px;
  margin-right: 15px;
  margin-top: 0;
}
.button-link {
  text-decoration: none;
  color: #01A7FD;
  margin-top: 30px;
}
.fill-button {
  background: #01A7FD;
  border-radius: 100px;
  color: #fff;
  padding: 10px 20px;
  display: inline-block;
  cursor: pointer;
  user-select: none;
  margin-right: 20px;
}
</style>