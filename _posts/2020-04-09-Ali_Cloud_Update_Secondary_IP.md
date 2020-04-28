---
title: DevOps -- Ali Cloud update secondary ip
---

可以使用API对Secondary IP进行Assign/Unassign动作：

```
package main

import (
	"fmt"
	"github.com/aliyun/alibaba-cloud-sdk-go/services/ecs"
	"gopkg.in/alecthomas/kingpin.v2"
	"os"
)

type ECSClient struct {
	client *ecs.Client
}
func (c *ECSClient) AssignPrivateIp  (eni string, ip string) error{
	req := ecs.CreateAssignPrivateIpAddressesRequest()
	req.Scheme = "https"
	req.NetworkInterfaceId = eni
	req.PrivateIpAddress = &[]string{ip}
	_, err := c.client.AssignPrivateIpAddresses(req)
	return err
}
func (c *ECSClient) UnassignPrivateIp(eni string, ip string) error{
	req := ecs.CreateUnassignPrivateIpAddressesRequest()
	req.Scheme = "https"
	req.NetworkInterfaceId = eni
	req.PrivateIpAddress = &[]string{ip}
	_, err := c.client.UnassignPrivateIpAddresses(req)
	return err
}



func main() {
	var (
		region = kingpin.Flag(
			"region",
			"ali cloud region name",
		).Default("cn-shanghai").String()
		accessKeyId = kingpin.Flag(
			"accessKeyId",
			"ali cloud access key id",
		).Default("").String()
		accessKeySecret = kingpin.Flag(
			"accessKeySecret",
			"ali cloud access Key Secret",
		).Default("").String()
		eni = kingpin.Flag(
			"eni",
			"eni interface",
		).Default("").String()
		vip = kingpin.Flag(
			"vip",
			"vip address",
		).Default("").String()
		action = kingpin.Flag(
			"action",
			"assign / unassign",
		).Default("").String()
	)
	kingpin.HelpFlag.Short('h')
	kingpin.Parse()

	client, err := ecs.NewClientWithAccessKey(*region, *accessKeyId, *accessKeySecret)

	if err != nil {
		fmt.Printf("Initial ecs client failed(%v)\n", err)
		os.Exit(-1)
	}

	c := ECSClient{client:client}

	if *action == "assign" {
		err = c.AssignPrivateIp(*eni, *vip)
		if err != nil {
			fmt.Printf("AssignPrivateIp Failed, err(%v)\n", err.Error())
			os.Exit(-1)
		}
	} else if *action == "unassign" {
		err = c.UnassignPrivateIp(*eni, *vip)
		if err != nil {
			fmt.Printf("UnassignPrivateIp Failed, err(%v)\n", err.Error())
			os.Exit(-1)
		}
	} else {
		fmt.Printf("Command unsupported (%v)\n", *action)
		os.Exit(-1)
	}
	os.Exit(0)
}

```


