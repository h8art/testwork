<template lang="pug">
.app
  .body
    button(@click='addNew').add + Add new
    input-row(v-for='input,index in inputs',:key='"row"+index',:data='input',:ind='index',@parse='parseInput',@depend='updateDepend',@changeBlock='changeBlock',@delete='onDelete')
</template>
<script>
import InputRow from './components/InputRow.vue'
export default {
  components: { InputRow },
  data(){
    return{
      inputs: []
    }
  },
  methods:{
    getRandomColor(){
      return "#" + Math.random().toString(16).slice(2, 8)
    },
    onDelete(payload){
      let input = this.inputs[payload]
      input.numbers.splice(input.numbers.length-1,1)
      input.operators.splice(input.operators.length-1,1)
      this.recalcResults()
    },
    async changeBlock(payload){
      console.log(payload)
      if(payload.childs.length>0){
        await payload.childs.map(child=>{
          this.inputs[child.row].numbers[child.ind].value=payload.value
        })
      }
      this.inputs[payload.rowIndex].numbers[payload.index].value=payload.value
      this.recalcResults()
    },
    parseInput(payload){
      const value = payload.val
      const ind = payload.ind
      let input = this.inputs[ind]
      Array.prototype.push.apply(input.numbers,value.split(/\D/g).filter(String).map((item)=>{
        return {
          value: item, 
          childs: [],
          color: '#D8D8D8',
          parent: null,
        }
      }))
      Array.prototype.push.apply(input.operators,value.replace(/[0-9]/g, '').split(''))
      this.recalcResults()
    },
    recalcResults(){
      this.inputs.map(input=>{
        var body = input.numbers.reduce((acc,number,index)=>{
          const operator = input.numbers.length-1!=index?input.operators[index]:''
          return acc+(number.value+operator)
        },'')
        const result =  eval(body)
        input.result.value = result
        if(input.result.childs.length>0){
          input.result.childs.map(child=>{
            this.inputs[child.row].numbers[child.ind].value=result
          })
        }
      })
    },
    updateSimpleBlockDepend(payload){
      let input = this.inputs[payload.rowIndex]
      let parentInput = this.inputs[payload.event.rowIndex]

      const parentIndex = payload.event.index
      const childIndex = payload.index

      let parentBlock = parentInput.numbers[parentIndex]
      let childBlock = input.numbers[childIndex]

      childBlock.value = payload.event.data.value
      if(parentBlock.color=='#D8D8D8'){
        const color = this.getRandomColor()
        childBlock.color = color
        parentBlock.color = color
      }else{
        childBlock.color = parentBlock.color
      }
      childBlock.parent={
        row: payload.event.rowIndex,
        ind: payload.event.index
      }
      parentBlock.childs.push({
        row: payload.rowIndex,
        ind: payload.index
      })
      this.recalcResults()
    },
    updateResultDepend(payload){
      console.log(payload)
      if(payload.rowIndex<=payload.event.rowIndex){
        alert('Результат должен находиться выше или в одной строке с зависимым значением')
      }else{
        let input = this.inputs[payload.rowIndex]
        const childIndex = payload.index
        let childBlock = input.numbers[childIndex]
        let parentBlock = this.inputs[payload.event.rowIndex].result

        if(parentBlock.color=='#D8D8D8'){
          const color = this.getRandomColor()
          childBlock.color = color
          parentBlock.color = color
        }else{
          childBlock.color = parentBlock.color
        }
        //if ind == -1 => isResult
        childBlock.value = payload.event.data.value.toString()
        childBlock.parent={
          row: payload.event.rowIndex,
          ind: -1
        }
        parentBlock.childs.push({
          row: payload.rowIndex,
          ind: payload.index
        })
        this.recalcResults()
      }
    },
    updateDepend(payload){
      payload.event.isResult?this.updateResultDepend(payload):this.updateSimpleBlockDepend(payload)
    },
    addNew(){
      const emptyBlock = {
        numbers: [],
        operators: [],
        result: {
          value: 0,
          childs: [],
          color: '#D8D8D8'
        }
      }
      this.inputs.push(emptyBlock)
    }
  }
}
</script>

<style lang="sass" scoped>
.app
  min-width: 100vw
  min-height: 100vh
  padding-top: 50px
  background-color: #E9E8E9
  .body
    width: 800px
    margin: 0 auto
.add
  border-width: 0
  border-radius: 40px
  padding: 20px 40px
  background-color: #38ef7d
  font-size: 25px
  color: #fff
  font-weight: bold
  transition: box-shadow .3s
  outline: none
  margin: 0 auto
  margin-bottom: 25px
  &:hover
    cursor: pointer
    box-shadow: 0px 0px 20px rgba(0,0,0,0.3)
</style>
