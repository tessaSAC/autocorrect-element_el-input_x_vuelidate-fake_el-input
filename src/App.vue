<template>
<div>
    <DiscoDefineNewColumn
      :columnName="generatedColumnName"
      :columnLabel="generatedColumnLabel"
      @newColumnName="newColumnName = $event"
      @newColumnLabel="newColumnLabel = $event"
      @isValid="isValid = $event"
    />
    <br />
    <Explanation />
</div>
</template>

<script>
import DiscoDefineNewColumn from "./components/DiscoDefineNewColumn"
import Explanation from "./components/Explanation"

export default {
  name: "App",
  components: {
    DiscoDefineNewColumn,
    Explanation
  },
  data() {
    return {
      newColumnName: '',
      newColumnLabel: ''
    }
  },
  computed: {
    generatedColumnName() {
      return `name_${ Math.ceil(Math.random() * 10) }`
    },
    generatedColumnLabel() {
      return `${ new Date() }`
    }
  }
}
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
