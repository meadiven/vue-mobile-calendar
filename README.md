# vue-mobile-calendar

![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg) ![Github file size](https://img.shields.io/badge/size-18kb-brightgreen.svg)
![npm](https://img.shields.io/badge/npm-6.3.0-red.svg)
![node.js](https://img.shields.io/badge/node.js-v10.14.2-blue.svg)

> 基于 Vue 移动端日历组件
> 支持上下滑动切换月份功能

## How to use

```bash
npm i -S vue-date-mobile
```
global regisiter
```js
import Vue from 'vue'
import VueMobileCalendar from 'vue-date-mobile'
Vue.use(VueMobileCalendar)
```

OR in component

```js
import VueMobileCalendar from 'vue-date-mobile'

// in vue component

export default {
  components: { VueMobileCalendar }
}
```

```xml
<vue-mobile-calendar></vue-mobile-calendar>
```
## Props

| name               | type     | desc                          | default |
| ------------------ | -------- | ----------------------------- | ------- |
| selected           | String   | The current selected date     | null    |
| lunar              | Boolean  | Display lunar                 | false   |
| show               | Boolean  | Display the calendar          | true    |

## Event

| name    | params              | desc                           |
| ------- | ------------------- | ------------------------------ |
| @change | Select date object  | The day which user clicked     |
| @move   | Date object         | The first day in changed month |

Select date object eg:

```xml
<vue-mobile-calendar v-if="true" @move="changeMonth" @change="selectDay" :lunar="true"></vue-mobile-calendar>
```
```js
export default {
  components: { VueMobileCalendar },
  methods: {
      selectDay (date) {
        console.log(date.dateStr); // yyyy-mm-dd eg:2018-12-18
        console.log(date.showInLunar); // 十二
      },
      changeMonth (date) {
        console.log(date) // Date Object
      },
  }
}
```