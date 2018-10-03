<template lang="pug">
.tab
  drop(@dragover="over = true",@dragleave="over = false",@drop='updateConnection($event,index)')
    drag.value(:transfer-data="{ data:value,index,rowIndex,isResult:false }",:style='{ "background-color":over?"#EFBBB7":value.color }') 
      input(v-model='currValue',@blur='updateValue(index)',v-autowidth="{minWidth: '20px', comfortZone: 0}")
  .operator {{operator}}
</template>
<script>
export default {
  props: ['value','operator','index','rowIndex'],
  data(){
    return{
      over: false,
      currValue: this.value.value
    }
  },
  methods:{
    updateValue(index){
      let payload = {
        rowIndex: this.rowIndex,
        index,
        ...this.value,
      }
      payload.value=this.currValue
      this.$emit('changeBlock',payload)
    },
    updateConnection(event,index){
      const payload = {
        event,
        index
      }
      this.over=false
      this.$emit('depend',payload)
    }
  },
  watch: {
    'value.value':function(newVal){
      this.$nextTick(()=>{
        this.currValue=newVal
      })
      
    }
  }
  
}
</script>
<style lang="sass" scoped>
.tab
  display: flex
  align-items: center
  .value
    padding: 5px 10px
    border-radius: 10px
    transition: all .3s
    input
      border-width: 0px
      background-color: transparent
      font-size: 15px
      outline: none
  .operator
    margin: 0 10px
    font-weight: bold
</style>

