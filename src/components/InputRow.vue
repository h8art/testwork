<template lang="pug">
  .row
    number-block(v-for='number,index in data.numbers',:key='index+""+number.value+""+ind+""+number.color',:rowIndex='ind',:value='number',:operator='data.operators[index]',:index='index',@depend='updateDepend',@changeBlock='changeBlock')
    input.input(@keyup.delete="onDelete",v-model='value',@keyup.enter='parseInput',v-autowidth="{minWidth: '20px', comfortZone: 0}")
    drag.result(:transfer-data="{ data:data.result,rowIndex: ind,isResult:true }",:style='{ "background-color":data.result.color }') {{data.result.value}}
</template>
<script>
import NumberBlock from './NumberBlock.vue'
export default {
  props: ['data','ind'],
  data(){
    return{
      value: '',
    }
  },
  components: { NumberBlock },
  methods:{
    onDelete(){
      if(!this.value){
        this.$emit('delete',this.ind)
      }
    },
    changeBlock(payload){
      this.$emit('changeBlock',payload)
    },
    updateDepend(payload){
      payload.rowIndex=this.ind
      this.$emit('depend',payload)
    },
    parseInput(){
      const payload = {
        val: this.value,
        ind: this.ind
      }
      this.$emit('parse',payload)
      this.value=''
    }
  }
}
</script>
<style lang="sass" scoped>
.row
  display: flex
  align-items: center
  background-color: rgba(255,255,255,0.2)
  padding: 10px 20px
  margin-bottom: 5px
  transition: box-shadow .3s
  &:hover
    box-shadow: 0px 0px 10px rgba(0,0,0,0.3)
.result
  padding: 5px 10px
  border-radius: 10px
  transition: background-color .3s
  margin-left: 10px
.input
  border-width: 0
  font-size: 20px
  background-color: #f7797d
  
</style>
