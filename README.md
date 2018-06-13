# verifyReceipt
内购验证

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
