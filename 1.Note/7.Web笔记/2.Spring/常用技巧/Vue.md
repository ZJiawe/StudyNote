# 1、 子组件向父组件传输值

#### 1.1 子组件

```vue
this.$emit('imgurl', response)
```

#### 1.2 父组件

```vue
<Upload ref="Upload" v-on:imgurl="getImgUrl" />
```

```js
    getImgUrl (val) {
      this.tableData.photo = val
    }
```

