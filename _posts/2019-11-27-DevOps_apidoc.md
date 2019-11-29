---
title: DevOps -- API Doc
---



> 在编写代码时，需要维护API文档，人工维护wiki/文档很容易造成文档没有及时更新。因此，可以使用编写注释的方式自动生成API Doc，常用的有Swagger,API Doc等，我们最终选择API Doc作为自动生成工具。



通过在函数头部添加以下注释就可以自动生成文档：

```
/**
 * @api {post} /gslb/node/ms MS Create
 * @apiDescription This API is used by MS to create MS.
 * @apiName CreateMS
 * @apiGroup Node Management
 * @apiHeader {String} Authorization PanoSToken (http://wiki.in.pano.im/display/PANO/Server+Token+and+EncyptedBody)
 * @apiHeaderExample {string} Header-Example:
 *     "Authorization": "PanoSToken xxxxxxxxxxx"
 * @apiParam {string} dc  Data Center Name
 * @apiParam {string} msid ms uuid
 * @apiParam {int64} group ms group
 * @apiParam {string} address ms address(IP or DNS)
 * @apiParam {int64} msLoad ms load(0-1000)
 * @apiParam {int64} step load to add when single user join
 * @apiParam {string} grayKey internal usage for upgrade
 * @apiParamExample {json} Request-Example:
 * {
 * "dc": "sh01dc1",
 * "msid": "abcd",
 * "group": 0,
 * "address": "1.2.3.4",
 * "msLoad": 200,
 * "step": 10,
 * "grayKey": "Key0.5",
 * }
 * @apiSuccess (Success 200) StatusOK  OK
 * @apiError   (Failed  400) StatusBadRequest Invalid Input Params
 * @apiError   (Failed  403) StatusForbidden Invalid Token
 * @apiError   (Failed  500) StatusInternalServerError Internal Server Error
 */
```

具体语法参照： https://apidocjs.com/



在GitLab CI中可以配置apidoc自动生成的步骤：

```
API Doc:
  stage: APIDOC
  script:
    - apidoc -i ./ -o /opt/output/documents/gslb -t /opt/tools/pano-apidoc-template
  tags:
    - build
  only:
    - merge_requests
```



然后使用Http服务器暴露API文档即可。此时代码中修改后，就会更新到API文档服务器中。