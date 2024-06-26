---
date: 2023-06-09
share: true
title: Не возвращайте JSON массивы в API
description: Если в API вам нужно вернуть JSON массив, подумайте о том, чтобы завернуть его в объект, даже если кроме этого массива в объекте ничего не будет
slug: json-array-api
categories:
  - architecture
---


Если в API вам нужно вернуть JSON массив, подумайте о том, чтобы завернуть его в объект, даже если кроме этого массива в объекте ничего не будет. 
<!-- more -->

Например, вместо:
```json
["red", "green", "yellow"]
```

Лучше сделать:
```json
{
  "data": ["red", "green", "yellow"]
}
```

API имеет свойство меняться со временем и вы можете захотеть добавить какую-то информацию помимо массива. В первом примере это невозможно сделать, не сломав обратную совместимость. Во втором варианте таких проблем нет:
```json
{
  "data": ["red", "green", "yellow"],
  "shape": "circle"
}
```
