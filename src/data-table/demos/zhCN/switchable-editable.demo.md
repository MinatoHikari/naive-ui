# 可切换的可编辑表格

不太简单，胜在好看

```html
<n-data-table
  :key="(row) => row.innerKey"
  :columns="columns"
  :data="data"
  virtual-scroll
  :pagination="false"
  max-height="500"
/>
```

```js
import { h, defineComponent, ref, nextTick } from 'vue'
import { NInput } from 'naive-ui'

let key = 0

const tableUtils = {
  insert: (tableData, insertData) => {
    insertData.key = key
    key = key + 1
    tableData.push(insertData)
  }
}

const dataSample = {
  name: 'John Brown',
  age: '32',
  address: 'New York No. 1 Lake Park'
}

const createData = () => {
  const tableData = []
  for (let i = 0; i < 100; i++) {
    tableUtils.insert(tableData, { ...dataSample })
  }

  return tableData
}

const ShowOrEdit = defineComponent({
  props: {
    value: [String, Number],
    onUpdateValue: [Function, Array]
  },
  setup (props) {
    const isEdit = ref(false)
    const inputRef = ref(null)
    const inputValue = ref(props.value)
    function handleOnClick () {
      isEdit.value = true
      nextTick(() => {
        inputRef.value.focus()
      })
    }
    function handleChange () {
      props.onUpdateValue(inputValue.value)
      isEdit.value = false
    }
    return () =>
      h(
        'div',
        {
          onClick: handleOnClick
        },
        isEdit.value
          ? h(NInput, {
            ref: inputRef,
            value: inputValue.value,
            onUpdateValue: (v) => {
              inputValue.value = v
            },
            onChange: handleChange,
            onBlur: handleChange
          })
          : props.value
      )
  }
})

export default defineComponent({
  setup () {
    const data = ref(createData())
    return {
      data,
      columns: [
        {
          title: 'Name',
          key: 'name',
          width: 150,
          render (row, index) {
            return h(ShowOrEdit, {
              value: row.name,
              onUpdateValue (v) {
                data.value[index].name = v
              }
            })
          }
        },
        {
          title: 'Age',
          key: 'age',
          width: 100,
          render (row, index) {
            return h(ShowOrEdit, {
              value: row.age,
              onUpdateValue (v) {
                data.value[index].age = v
              }
            })
          }
        },
        {
          title: 'Address',
          key: 'address',
          render (row, index) {
            return h(ShowOrEdit, {
              value: row.address,
              onUpdateValue (v) {
                data.value[index].address = v
              }
            })
          }
        }
      ],
      pagination: {
        pageSize: 10
      }
    }
  }
})
```
