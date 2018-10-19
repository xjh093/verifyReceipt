# verifyReceipt
内购验证

# 越狱订单和正常订单对比:
- 越狱订单虽然状态返回是成功的，但是```in_app```这个参数是空的。
- 大概查了一下。iOS7以下是没有这个in_app参数的，iOS7以上是有的。因为现在App基本支持的起步都是iOS8 iOS9了，iOS7可以不用管了。

```
越狱订单receipt_data向苹果服务器校验后如下：
{
    "status": 0, 
    "environment": "Production", 
    "receipt": {
        "receipt_type": "Production", 
        "adam_id": 1377028992, 
        "app_item_id": 1377028992, 
        "bundle_id": "*******【敏感信息不给看】*******", 
        "application_version": "3", 
        "download_id": 80042231041057, 
        "version_external_identifier": 827853261, 
        "receipt_creation_date": "2018-07-23 07:30:45 Etc/GMT", 
        "receipt_creation_date_ms": "1532331045000", 
        "receipt_creation_date_pst": "2018-07-23 00:30:45 America/Los_Angeles", 
        "request_date": "2018-07-23 07:33:54 Etc/GMT", 
        "request_date_ms": "1532331234485", 
        "request_date_pst": "2018-07-23 00:33:54 America/Los_Angeles", 
        "original_purchase_date": "2018-07-01 12:16:21 Etc/GMT", 
        "original_purchase_date_ms": "1530447381000", 
        "original_purchase_date_pst": "2018-07-01 05:16:21 America/Los_Angeles", 
        "original_application_version": "3", 
        "in_app": [ ]
    }
}
```

```
苹果服务器向返回status结果，含义如下，其中为0时表示成功。
21000 App Store无法读取你提供的JSON数据
21002 收据数据不符合格式
21003 收据无法被验证
21004 你提供的共享密钥和账户的共享密钥不一致
21005 收据服务器当前不可用
21006 收据是有效的，但订阅服务已经过期。当收到这个信息时，解码后的收据信息也包含在返回内容中
21007 收据信息是测试用（sandbox），但却被发送到产品环境中验证
21008 收据信息是产品环境中使用，但却被发送到测试环境中验证
```

详见：https://www.jianshu.com/p/5cf686e92924

---

# 内购验证示例
```
- (void)xx_verify_test:(SKPaymentTransaction *)transaction
{
    NSError *error;
    NSDictionary *requestContents = @{@"receipt-data": [[JHIAPManager iapManager] jh_receiptString]};
    NSData *requestData = [NSJSONSerialization dataWithJSONObject:requestContents
                                                          options:0
                                                            error:&error];
    
    if (!requestData) { /* ... Handle error ... */
        DLog(@"error:%@",error);
    }
    
    // https://buy.itunes.apple.com/verifyReceipt
    NSURL *storeURL = [NSURL URLWithString:@"https://sandbox.itunes.apple.com/verifyReceipt"];
    NSMutableURLRequest *storeRequest = [NSMutableURLRequest requestWithURL:storeURL];
    [storeRequest setHTTPMethod:@"POST"];
    [storeRequest setHTTPBody:requestData];
    
    NSURLSession *session = [NSURLSession sharedSession];
    NSURLSessionDataTask *task = [session dataTaskWithRequest:storeRequest completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSDictionary *jsonResponse = [NSJSONSerialization JSONObjectWithData:data options:0 error:&error];
        if (jsonResponse) {
            DLog(@"jsonResponse:%@",jsonResponse);
            [[JHIAPManager iapManager] jh_finishTransaction:transaction];
            
            /**< 
             {
             environment = Sandbox;
             receipt =     {
                 "adam_id" = 0;
                 "app_item_id" = 0;
                 "application_version" = "1.1";
                 "bundle_id" = "com.haocold.xxxxx";
                 "download_id" = 0;
                 "in_app" =         (
                     {
                     "is_trial_period" = false;
                     "original_purchase_date" = "2017-09-18 05:46:11 Etc/GMT";
                     "original_purchase_date_ms" = 1505713571000;
                     "original_purchase_date_pst" = "2017-09-17 22:46:11 America/Los_Angeles";
                     "original_transaction_id" = 1000000335003766;
                     "product_id" = "com.haocold.xxxxx_6";
                     "purchase_date" = "2017-09-18 05:46:11 Etc/GMT";
                     "purchase_date_ms" = 1505713571000;
                     "purchase_date_pst" = "2017-09-17 22:46:11 America/Los_Angeles";
                     quantity = 1;
                     "transaction_id" = 1000000335003766;
                     }
                 );
                 "original_application_version" = "1.0";
                 "original_purchase_date" = "2013-08-01 07:00:00 Etc/GMT";
                 "original_purchase_date_ms" = 1375340400000;
                 "original_purchase_date_pst" = "2013-08-01 00:00:00 America/Los_Angeles";
                 "receipt_creation_date" = "2017-09-18 05:46:12 Etc/GMT";
                 "receipt_creation_date_ms" = 1505713572000;
                 "receipt_creation_date_pst" = "2017-09-17 22:46:12 America/Los_Angeles";
                 "receipt_type" = ProductionSandbox;
                 "request_date" = "2017-09-18 05:46:13 Etc/GMT";
                 "request_date_ms" = 1505713573975;
                 "request_date_pst" = "2017-09-17 22:46:13 America/Los_Angeles";
                 "version_external_identifier" = 0;
                 };
             status = 0;
             }
             */
        }
    }];
    [task resume];
}

```


# 过期

```
{
    status = 0;
    environment = Sandbox;
    receipt =     {
        "adam_id" = 0;
        "app_item_id" = 0;
        "application_version" = 1;
        "bundle_id" = "com.haocold.xxxxx";
        "download_id" = 0;
        "expiration_date" = "2018-10-19 06:54:54 Etc/GMT";                   // 过期参数
        "expiration_date_ms" = 1539932094000;                                // 过期参数
        "expiration_date_pst" = "2018-10-18 23:54:54 America/Los_Angeles";   // 过期参数
        "in_app" =         (
            //......
        )
        "organization_display_name" = "ACME Bad VPPReceipt Inc.";
        "original_application_version" = "1.0";
        "original_purchase_date" = "2013-08-01 07:00:00 Etc/GMT";
        "original_purchase_date_ms" = 1375340400000;
        "original_purchase_date_pst" = "2013-08-01 00:00:00 America/Los_Angeles";
        "receipt_creation_date" = "2018-10-19 06:54:54 Etc/GMT";
        "receipt_creation_date_ms" = 1539932094000;
        "receipt_creation_date_pst" = "2018-10-18 23:54:54 America/Los_Angeles";
        "receipt_type" = ProductionVPPSandbox;
        "request_date" = "2018-10-19 06:54:59 Etc/GMT";
        "request_date_ms" = 1539932099562;
        "request_date_pst" = "2018-10-18 23:54:59 America/Los_Angeles";
        "version_external_identifier" = 0;
}
```





