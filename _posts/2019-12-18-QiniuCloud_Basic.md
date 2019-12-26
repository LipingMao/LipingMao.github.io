---
title: Object Storage -- qiniu cloud basic
---


> 选用Qiniu云作为对象存储Provider，测试了业务服务器APP Server需要的最重要的两个接口。


例子如下：
```
package main

import (
	"context"
	"fmt"
	"github.com/qiniu/api.v7/v7/auth/qbox"
	"github.com/qiniu/api.v7/v7/storage"
	"net/http"
	"time"
)


// AccessKey & Secret
var accessKey = "$$$YOUR KEY$$$"
var secretKey = "$$$YOUR PASS$$$"
var mac = qbox.NewMac(accessKey, secretKey)

// Buckets which to be used
var buckets = []string{"pano-service1", "pano-service2"}
// CDN cname for speed up download
var domains = map[string]string {
	"pano-service1": "http://q2yuq6tkm.bkt.clouddn.com",
	"pano-service2": "http://q2yui4vau.bkt.clouddn.com",
}


// Storage config
func getCfg() storage.Config {
	// Config
	cfg := storage.Config{}
	cfg.Zone = &storage.ZoneHuanan
	cfg.UseHTTPS = false
	cfg.UseCdnDomains = false
	return cfg
}


// Upload Token
func getUploadToken(bucket string) string{
	putPolicy := storage.PutPolicy{
		Scope:               bucket,
	}
	return putPolicy.UploadToken(mac)
}
// Upload File
func uploadFile(localFile, bucket, target string) {
	fmt.Printf("Start to upload file, localFile %v, bucket %v, target %v\n", localFile, bucket, target)

	// Token
	upToken := getUploadToken(bucket)

	// Config
	cfg := getCfg()

	// Upload
	formUploader := storage.NewFormUploader(&cfg)
	ret := storage.PutRet{}
	err := formUploader.PutFile(context.Background(), &ret, upToken, target, localFile, nil)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("Finish to download, %v, %v\n", ret.Key,ret.Hash)
}

// Delete File
func deleteFile(bucket, target string) {
	fmt.Printf("Start to delete, bucket %v, target %v\n", bucket, target)
	cfg := getCfg()
	bucketManager := storage.NewBucketManager(mac, &cfg)
	err := bucketManager.Delete(bucket, target)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("Success delete, bucket %v, target %v\n", bucket, target)
}

// Delete File After Days
func deleteAfterDays(bucket, target string, days int) {
	fmt.Printf("Start to delete after days, bucket %v, target %v, days %v\n", bucket, target, days)
	cfg := getCfg()
	bucketManager := storage.NewBucketManager(mac, &cfg)
	err := bucketManager.DeleteAfterDays(bucket, target, days)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("Success delete, bucket %v, target %v\n", bucket, target)
}


// Download Link(CDN加速)
func downloadURL(bucket, target string) string {
	fmt.Printf("Start to generate download URL, bucket %v, target %v\n", bucket, target)

	deadline := time.Now().Add(time.Second * 3600).Unix() // 1小时有效
	privateAccessURL := storage.MakePrivateURL(mac, domains[bucket], target, deadline)
		http.StatusInternalServerError
	return privateAccessURL
}


// Test code
func main () {
	localFile := "/Users/limao/awscli-bundle.zip"
	target := "day3/awscli-bundle.zip"

	// Upload & Delete
	uploadFile(localFile, buckets[0],target)
	deleteFile(buckets[0], target)

	// Upload & Delete after 1 Day & generate download URL
	uploadFile(localFile, buckets[1],target)
	deleteAfterDays(buckets[1], target, 1)
	url := downloadURL(buckets[1], target)
	fmt.Printf("downloadurl: %v\n", url)
}
```
