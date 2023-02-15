---
title: Go语言常用框架
date: 2023-02-13 14:42:18
tags: [Go]
excerpt: Go语言常用框架
categories: Go
index_img: /img/index_img/3.png
banner_img: /img/banner_img/background30.jpg
---

<!-- 19.png background31.png -->


## gin框架

一个web框架

## gorm框架

一个ORM框架，能够实现go代码->数据库的表

## gorm-gen框架

能够实现数据库的表->go代码

## 入门测试程序

```go
package main

import (
	"gin/models"
	"github.com/gin-gonic/gin"
	"gorm.io/driver/mysql"
	"gorm.io/gen"
	"gorm.io/gorm"
	"net/http"
)

const dsn = "root:123456@tcp(127.0.0.1:3306)/cloud-disk?charset=utf8mb4&parseTime=True&loc=Local"

// 进行三个测试

// 1. 测试使用gorm框架生成mysql

// 2. 测试使用gen生成gin代码

// 3. 测试gin框架的router

func main() {

	// 操作思路
	// 使用gorm生成数据库
	// 如果需要修改数据库结构，可以修改数据库后重新运行gen
	// 也可以使用gorm进行修改后重新生成数据库

	// 连接数据库
	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
	if err != nil {
		panic(err)
	}
	db.AutoMigrate(models.GinTest{})

	// 设置gen
	g := gen.NewGenerator(gen.Config{
		OutPath:      "./gen/dal",
		ModelPkgPath: "./models",
		Mode:         gen.WithoutContext | gen.WithDefaultQuery | gen.WithQueryInterface,
	})
	g.UseDB(db)

	//生成数据库
	db.AutoMigrate(models.GinTest{})
	//生成go
	g.ApplyBasic(g.GenerateAllTable()...)
	g.Execute()

	//gin测试
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"msg": "success",
			"data": map[string]interface{}{
				"back": "hello world",
			},
		})
	})
	r.Run(":8080")
}

```