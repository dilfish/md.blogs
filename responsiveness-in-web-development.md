---
title: "web 开发中的 responsiveness"
date: 2021-11-17T11:25:27+08:00
draft: false
---

## 概念

responsiveness 意思是可响应性，其含义是在不同尺寸设备上面显示不同的样子。原理是 css 有一个特性叫 media query，可以查询当前显示环境的宽度。

另外一个技术叫做 flaxbox，可以让一个 column 元素，在窄的设备上面堆起来，而在宽的设备上面横排。

## 文档
可以参考 bootstrap 和 bulma 的文档：
https://getbootstrap.com/docs/5.0/layout/breakpoints/
https://bulma.io/documentation/columns/basics/

另外我看了一个示意比较清晰的文章是：
https://www.freecodecamp.org/news/learn-the-bootstrap-4-grid-system-in-10-minutes-e83bfae115da/

## 问题
```html
  <!-- Stack the columns on mobile by making one full-width and the other half-width -->
  <div class="row">
    <div class="col-md-8">.col-md-8</div>
    <div class="col-6 col-md-4">.col-6 .col-md-4</div>
  </div>
```

感觉理解起来有点乱，在设备宽度大于 md 的时候，显示为 8 和 4，但是小于 md 的时候怎么显示不够明确。只能多试试猜测他这个默认的含义。
