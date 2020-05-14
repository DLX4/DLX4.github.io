---
title: 使用angualrJS 1.2.x构造SPA应用
tags:
  - angularJS
  - 前端
url: 272.html
id: 272
categories:
  - 未分类
date: 2019-11-08 14:44:49
---

很多中后台项目（比如xxx管理系统）前端经常采用下面这样的架构。

![](http://106.54.113.128/wordpress/wp-content/uploads/2019/11/image-2.png)

点击不同的菜单项，加载不同的功能主页。

如何使用angularJS快速开发类似架构的项目呢？

**第一种方法，可以尝试使用angular-route模块。**

**主页面（index.html）：**

    <!DOCTYPE html>
    <html lang="en">
    <head>
    ...
    <script type="text/javascript">
    var app = angular.module('xx-app',[])
    app.config(function($routeProvider) {
      $routeProvider
        .when('/xx/xx', {
          templateUrl : 'xx.html',
          controller : 'xxController'
        })
    </script>
    </head>
    
    <div class="header">...</div>
    <div class="content">
    导航菜单
    <div class="side-box pull-left" id="sideBar"> 
    ...
    <a href="#/xx/xx">功能1</a></li>
    ...
    </div> 
    功能主页
    <div ng-view ></div>
    </div> 

**功能1页面（xx.html）**

`<div>....</div>`

**功能1controller（xxController.js）**

    var app = angular.module(...)

**第二种方法，使用ng-include动态加载页面。**

**index.html**

    <div ng-include="'header.html'">...</div>
    导航菜单
    <div ng-include="'menu.html'">...</div> 
    功能主页
    <div ng-include="'main.html'"></div>
    </div> 

**main.html**

    <div ng-controller="menu">
       <div ng-include="menu.cur.url"></div>
    </div>

**menu.js**

    app.controller('menu', [
      '$scope', 'menuProvider',
      function ($scope,  menuProvider) {
        $scope.menu =  menuProvider ;
      }
    ]);
    
    app.provider('menua', function () {});