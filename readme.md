## 日历组件

### 简单介绍

- 组件需要依赖 tailwindcss ，请自行在项目适配；
- 本组件依赖 uni-app 内置组件`swiper`组件
- nvue 页面中的`swiper`组件部分功能不支持，详情见[uni-app 内置组件：swiper](https://uniapp.dcloud.net.cn/component/swiper.html)
- 通过 uni_modules 引用组件，在页面 template 中即可直接使用，无需在页面中 import 和注册 components；
- 可设置显示收缩按钮，收缩后只显示一个星期的日期，展开后显示一个月的日期；可以通过触摸屏幕左右滑动切换月份或切换年；
- 本组件农历转换使用的 js 是 [@1900-2100 区间内的公历、农历互转](https://github.com/jjonline/calendar.js)
- 若有插件导入失败，重启编辑器；

### API

#### 属性说明

| 属性名            | 类型      | 默认值    | 说明                              |
| ----------------- | --------- | --------- | --------------------------------- |
| defaultStart      | number    | monday    | 周几为每周的第一天，可选值：[0~7] |
| headerLineColor   | string    | #008080   | 头部线颜色                        |
| lastMonthIcon     | urlString |           | 头部切换上月图标                  |
| nextMonthIcon     | urlString |           | 头部切换下月图标                  |
| lastYearIcon      | urlString |           | 头部切换上年图标                  |
| nextYearIcon      | urlString |           | 头部切换下年图标                  |
| currentDayBgColor | string    | #008080   | 当前日期背景色颜色                |
| festivalColor     | string    | #008080   | 重要节日文字颜色                  |
| currentDayColor   | string    | #008080   | 当前日期文字颜色                  |
| clickItemColor    | string    | #00808182 | 点击日期背景颜色                  |
| viewType          | string    | month     | 视图类型 可选值['month','week']   |
| showHeader        | boolean   | true      | 是否显示头部导航                  |

注意：

1. 农历日期、节日、标记日期，只会显示其一，优先级 : 标记 > 节日 > 农历日期；

#### 事件说明

| 事件名        | 说明                 | 返回值                                |
| ------------- | -------------------- | ------------------------------------- |
| handleCurrent | 点击日期项触发该函数 | 可通过回调参数 e 拿到该日期的详细信息 |
| handleChange  | 日历组件月份切换     | 可通过回调参数 e 拿到切换后的当前年月 |

1. 方法使用：在组件上添加 `@handleCurrent="handleClick" ` 事件，事件名可自行定义<br>
2. 方法使用：在组件上添加 `@handleChange="handleChange" ` 事件，事件名可自行定义<br>

#### @handleCurrent 返回值 e 说明

| 值            | 类型    | 说明                                                                                                                     |
| ------------- | ------- | ------------------------------------------------------------------------------------------------------------------------ |
| currentColor  | String  | 选中颜色                                                                                                                 |
| year          | number  | 年                                                                                                                       |
| month         | Number  | 阳历月                                                                                                                   |
| day           | Number  | 阳历日                                                                                                                   |
| type          | Number  | 月份所属                                                                                                                 |
| isCurrenMonth | boolean | 是否本月                                                                                                                 |
| lunarDate     | Object  | 农历信息，包含农历日期、节日、生肖等 详情见 [@1900-2100 区间内的公历、农历互转](https://github.com/jjonline/calendar.js) |

### 基本用法

在`template`中使用组件

```


<template>
  <AtomCalendar @handleCurrent="handleClick" />
</template>

```

在`script`中使用

```
<script setup lang="ts">
import AtomCalendar from "@/components/atom-calendar.vue";
const handleClick = (e: any) => {
  console.log(e, "接收参数");
};
</script>
```
