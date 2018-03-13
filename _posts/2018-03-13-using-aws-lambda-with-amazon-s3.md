---
layout: post
title: "Using AWS Lambda with Amazon S3 (Node.js)"
description: ""
category: 
tags: [AWS, Lambda, S3]
---
{% include JB/setup %}

{: style="font-size: 16px;line-height: 2;"}

AWS Lambda提供serverless的運算服務，serverless意思是開發者不需要管server的任何資源，也就是說你不需要創建一個EC2 instance，然後還要煩惱auto scaling, HA等設定，AWS Lambda替開發者解決這些問題，它會自動scale。<br />
開發者只需要專心在code就好，你的code會透過`Event`來驅動，
這個`Event`可以來自於AWS上的服務，例如S3, API Gateway等服務，舉例來說你可以設定，當上傳檔案至S3時，S3會發送一個`Event`給你的code來回應這個`Event`。這篇文章就會以AWS上[建立縮圖的example](https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html)來說明。 <br />
而AWS Lambda的費用是取決於你的code執行時間，也就是說當沒有任何Event去trigger你的code時，是不需要收費的，在某些應用情況來說，是比EC2來得優惠。但AWS Lambda還是有它的缺點，因為你不需要也無法管server的任何設定/資源，在某些應用情況來說，替開發者帶來了些限制，所以當想要更有彈性，或者喜歡hands-on架構的開發者，或許就可以考慮EC2。

<!--more-->


---
### 1. Prerequisite
---

{: style="font-size: 16px;line-height: 1.6;"}

開始之前，請先去下載並且安裝AWS CLI，安裝及設置文件可以參考[這篇文章](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-bundle.html)。


<br />

---
### 2. 創建2個Bueckts，並且上傳一個測試檔案
---

{: style="font-size: 16px;line-height: 1.6;"}

首先先在S3上創建2個buckets，2個的名稱請自己取，但記得第2個名稱結尾一定要是resized，這篇的example分別為

{: style="font-size: 16px;line-height: 1.6;"}

- kenyang-image
- kenyang-imageresized

{: style="font-size: 16px;line-height: 1.6;"}

創建完成以後，在`kenyang-image`裡上傳一張圖片。

<br />


---
### 3. 創建Deployment Package
---

{: style="font-size: 16px;line-height: 1.6;"}

在這一步驟要創建一個`Deployment package`，它是一個zip檔案，而這zip檔案裡面包含了Lambda code + dependencies。

{: style="font-size: 16px;line-height: 1.6;"}

首先先安裝2個packages:

{: style="font-size: 16px;line-height: 1.6;"}

- aysnc: Async utility module
- gm: GraphicsMagick用來轉圖

```bash	
$ cd test-lambda-s3
$ npm install async gm
```

{: style="font-size: 16px;line-height: 1.6;"}

安裝完以後，創建一個`CreateThumbnail.js`，並把下面的code貼入。這份code是利用async去依序完成三個工作:

{: style="font-size: 16px;line-height: 1.6;"}

1. 從S3下載檔案至buffer
2. 轉檔案
3. 上傳檔案至S3

```javascript
// dependencies
var async = require('async');
var AWS = require('aws-sdk');
var gm = require('gm')
            .subClass({ imageMagick: true }); // Enable ImageMagick integration.
var util = require('util');

// constants
var MAX_WIDTH  = 100;
var MAX_HEIGHT = 100;

// get reference to S3 client 
var s3 = new AWS.S3();
 
exports.handler = function(event, context, callback) {
    // Read options from the event.
    console.log("Reading options from event:\n", util.inspect(event, {depth: 5}));
    var srcBucket = event.Records[0].s3.bucket.name;
    // Object key may have spaces or unicode non-ASCII characters.
    var srcKey    =
    decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, " "));  
    var dstBucket = srcBucket + "resized";
    var dstKey    = "resized-" + srcKey;

    // Sanity check: validate that source and destination are different buckets.
    if (srcBucket == dstBucket) {
        callback("Source and destination buckets are the same.");
        return;
    }

    // Infer the image type.
    var typeMatch = srcKey.match(/\.([^.]*)$/);
    if (!typeMatch) {
        callback("Could not determine the image type.");
        return;
    }
    var imageType = typeMatch[1];
    if (imageType != "jpg" && imageType != "png") {
        callback('Unsupported image type: ${imageType}');
        return;
    }

    // Download the image from S3, transform, and upload to a different S3 bucket.
    async.waterfall([
        function download(next) {
            // Download the image from S3 into a buffer.
            s3.getObject({
                    Bucket: srcBucket,
                    Key: srcKey
                },
                next);
        },
        function transform(response, next) {
            gm(response.Body).size(function(err, size) {
                // Infer the scaling factor to avoid stretching the image unnaturally.
                var scalingFactor = Math.min(
                    MAX_WIDTH / size.width,
                    MAX_HEIGHT / size.height
                );
                var width  = scalingFactor * size.width;
                var height = scalingFactor * size.height;

                // Transform the image buffer in memory.
                this.resize(width, height)
                    .toBuffer(imageType, function(err, buffer) {
                        if (err) {
                            next(err);
                        } else {
                            next(null, response.ContentType, buffer);
                        }
                    });
            });
        },
        function upload(contentType, data, next) {
            // Stream the transformed image to a different S3 bucket.
            s3.putObject({
                    Bucket: dstBucket,
                    Key: dstKey,
                    Body: data,
                    ContentType: contentType
                },
                next);
            }
        ], function (err) {
            if (err) {
                console.error(
                    'Unable to resize ' + srcBucket + '/' + srcKey +
                    ' and upload to ' + dstBucket + '/' + dstKey +
                    ' due to an error: ' + err
                );
            } else {
                console.log(
                    'Successfully resized ' + srcBucket + '/' + srcKey +
                    ' and uploaded to ' + dstBucket + '/' + dstKey
                );
            }

            callback(null, "message");
        }
    );
};
```

{: style="font-size: 16px;line-height: 1.6;"}

接著就把所有檔案zip起來，

```bash
$  zip -r9 ~/create-thumbnail.zip *
```

<br />


---
### 4. 替Lambda Function建立一個IAM Role
---

{: style="font-size: 16px;line-height: 1.6;"}

在這一步驟要創建一個IAM Role專門去執行我們的lambda function，

{: style="font-size: 16px;line-height: 2;"}

[1] 打開IAM console [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/). <br />
[2] 選擇【Create role】<br />
[3] 在【Select type of trusted entity】中，請選擇AWS service, 然後選Lambda，並點擊【Next Permissions】。（如下圖）
![trusted entity]({{site.url}}/assets/2018-03-13-1.png)

{: style="font-size: 16px;line-height: 2;"}

[4] 在【Filter: 】中輸入【AWSLambdaExecute】，並點擊【Next: Review】（如下圖）
![trusted entity]({{site.url}}/assets/2018-03-13-2.png)

{: style="font-size: 16px;line-height: 2;"}

[5] 在【Role name】中輸入想要的角色名稱，並且點擊【Create role】<br />
[6] 接著點擊打開剛剛創建的角色<br />
[7] 點擊右下角的【Add inline policy】<br />
[8] 在Service，點擊【Choose a service】，並且選擇【S3】<br />
[9] 在Actions，點擊【Select actions】，接著把【Write】展開，找到【PutObject】並且打勾<br />
[10] 在Resources，把右下角的【Any】checkbox勾起來<br />
[11] 點擊【Choose Review policy】，接著輸入Name，並且點擊【Create policy】<br />
[12] 在Summary的頁面中，記下Role ARN，待會我們會用到（如下圖）
![trusted entity]({{site.url}}/assets/2018-03-13-3.png)


<br />


---
### 5. 上傳Deployment Package
---

{: style="font-size: 16px;line-height: 2;"}

執行下面的指令把剛剛的Deployment package上傳上去，請修改幾個參數：

{: style="font-size: 16px;line-height: 2;"}

1. region: 這篇使用ap-northeast-1
2. role-arn:上面第三步驟所創建的role arn
3. runtime: 這篇使用nodejs6.10
4. zip-file: 你的zip檔案路徑

```bash
$ aws lambda create-function \
--region ap-northeast-1 \
--function-name CreateThumbnail \
--zip-file fileb://file-path/CreateThumbnail.zip \
--role role-arn \
--handler CreateThumbnail.handler \
--runtime nodejs6.10 \
--timeout 30 \
--memory-size 1024
```

{: style="font-size: 16px;line-height: 2;"}

成功的話會回傳一串JSON回來，請把FunctionArn記下來，待會會用到


<br />


---
### 6. 手動調用Lambda Function
---

{: style="font-size: 16px;line-height: 2;"}

上傳上去以後，就可以來進行`手動`測試，先創建一個`input.txt`，並把下面的內容貼進去，需修改

{: style="font-size: 16px;line-height: 2;"}

- awsRegion: 這篇使用ap-northeast-1
- name: 改成上面我們創建的bucket名稱，這篇是kenyang-image
- arn: 把後面的kenyang-image改成你的bucket名稱，這篇是kenyang-image
- key: 上傳上去的測試檔名

```json
{  
   "Records":[  
      {  
         "eventVersion":"2.0",
         "eventSource":"aws:s3",
         "awsRegion":"ap-northeast-1",
         "eventTime":"1970-01-01T00:00:00.000Z",
         "eventName":"ObjectCreated:Put",
         "userIdentity":{  
            "principalId":"AIDAJDPLRKLG7UEXAMPLE"
         },
         "requestParameters":{  
            "sourceIPAddress":"127.0.0.1"
         },
         "responseElements":{  
            "x-amz-request-id":"C3D13FE58DE4C810",
            "x-amz-id-2":"FMyUVURIY8/IgAtTv8xRjskZQpcIZ9KG4V5Wp6S7S/JRWeUWerMUE5JgHvANOjpD"
         },
         "s3":{  
            "s3SchemaVersion":"1.0",
            "configurationId":"testConfigRule",
            "bucket":{  
               "name":"kenyang-image",
               "ownerIdentity":{  
                  "principalId":"A3NL1KOZZKExample"
               },
               "arn":"arn:aws:s3:::kenyang-image"
            },
            "object":{  
               "key":"test.jpg",
               "size":1024,
               "eTag":"d41d8cd98f00b204e9800998ecf8427e",
               "versionId":"096fKKXTRTtl3on89fVO.nfljtsv6qko"
            }
         }
      }
   ]
}
```

{: style="font-size: 16px;line-height: 2;"}

接著就可以執行下面的指令進行測試，一樣要修改

{: style="font-size: 16px;line-height: 2;"}

- region: 這篇是使用ap-northeast-1
- payload: input.txt路徑

```
$ aws lambda invoke \
--invocation-type Event \
--function-name CreateThumbnail \
--region ap-northeast-1 \
--payload file:///file-path/input.txt \
outputfile.txt
```


{: style="font-size: 16px;line-height: 2;"}

然後就可以去bucket `kenyang-imageresized`看看有沒有產生縮圖。

<br />


---
### 7. 設定Amazon S3 發送Events給Lambda
---

#### 7-1. 替S3設定InvokeFunction的權限

{: style="font-size: 16px;line-height: 2;"}

這步驟是給予S3去調用Function的權限，且只有在下列個二個條件下才會去調用function:


{: style="font-size: 16px;line-height: 2;"}

1. 檔案被創建在你指定的bucket下
2. 且這bucket是由該lambda account所創建的，話句話說，如果把此bucket刪除，接著別的帳號用戶去創建相同名稱的bucket，並且上傳檔案，此時並不會去調用function.

{: style="font-size: 16px;line-height: 2;"}

執行下方的指令即可給予S3此權限，請替換下列參數：

{: style="font-size: 16px;line-height: 2;"}

- region: 這篇是使用ap-northeast-1
- staement-id: unique的ID即可
- source-arn: 把kenyang-image改為你的bucket名稱

```
$ aws lambda add-permission \
--function-name CreateThumbnail \
--region ap-northeast-1 \
--statement-id 1234 \
--action "lambda:InvokeFunction" \
--principal s3.amazonaws.com \
--source-arn arn:aws:s3:::kenyang-image
```

<br />

#### 7-2. 設置S3 notification

{: style="font-size: 16px;line-height: 2;"}

在這步驟要設定當上傳檔案至此bucket時，要發送notification，以及收到notification時，要去調用上面創建的AWS Lambda Function。

{: style="font-size: 16px;line-height: 2;"}

[1] 點擊該Bucket<br />
[2] 點擊【Properties】<br />
[3] 找到【Advanced settings】<br />
[4] 點擊【Events】<br />
[5] 點擊【Add notification】<br />
[6] 勾選【ObjectCreate (All)】<br />
[7] 在【Send to】中選擇【Lambda Function】<br />
[8] 在【Lambda】中找到創建的function【CreateThumbnail】<br />
[9] 點擊【Save】<br />


<br />

#### 7-3. 測試

{: style="font-size: 16px;line-height: 2;"}

完成上述步驟以後，此時就可以進行測試，上傳檔案至指定的bucket裡面，然後去另外一個對應的bucket中看看thumbnail是否有被創建。




















<br />
<br />
<br />













