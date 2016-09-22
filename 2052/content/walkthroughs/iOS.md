# 在 iOS 应用中调用 Microsoft Graph

在本文中，我们将了解连接你的应用程序与 Office 365，以及调用 Microsoft Graph API 至少需要完成的任务。我们使用 [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) 中的代码，说明需要在你的应用中实现的主要概念。此示例介绍了以下核心基础知识：如何使用 Microsoft Azure Active Directory (AAD) 进行身份验证，以及如何使用 Microsoft Graph API 对 Office 365 邮件服务进行简单的服务调用（发送邮件）。建议你克隆或下载此存储库中的项目（本文配套）。 


本文引用了 [iOS 和 OSX 适用的 Microsoft Azure Active Directory Authentication Library (ADAL)](https://github.com/AzureAD/azure-activedirectory-library-for-objc)，并且 [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) 示例使用此库进行身份验证。请参阅此存储库，详细了解 iOS 项目中的使用和实现情况。


## 概述

若要调用 Microsoft Graph API，您的 iOS 应用必须完成以下任务：

1. 在 Microsoft Azure Active Directory (AD) 中注册应用程序。
2. 向 Azure AD 请求并获取访问令牌。
3. 在对 Microsoft Graph API 提出的 REST 请求中使用访问令牌。 



## 在 Azure Active Directory 中注册应用程序

您需要先注册应用程序并设置 Microsoft Graph 服务的使用权限，然后才能开始使用 Office 365。只需单击几下，即可使用[应用程序注册工具](https://dev.office.com/app-registration)将应用程序注册为访问用户的工作或学校帐户。
若要进行管理，您需要转到 [Microsoft Azure 管理门户](https://manage.windowsazure.com)

您也可以参阅**在 Azure AD 中手动注册应用以便其能访问 Office 365 API** 一文中的[在 Azure 管理门户中注册本机应用](https://msdn.microsoft.com/en-us/office/office365/howto/add-common-consent-manually)部分，了解关于如何手动注册应用的说明，并注意以下细节：

* 若要进行注册，您需要提供重定向 URI。这一所需值用于指定用户在成功通过身份验证后，被重定向到哪里。如果您没有指定正确的重定向 URI，则身份验证请求会失败。
* 注册时，必须向应用授予适用于 **Microsoft Graph** 的**以登录用户身份发送邮件的权限**。  


请记下 Azure 应用程序的“**配置**”页中的下列值。

* 客户端 ID
* 重定向 URI

您需要使用这些值在应用中配置 OAuth 流。 

## 向 Azure AD 请求并获取访问令牌

若要请求并获取用于调用 Microsoft Graph API 的访问令牌，您可以使用 **iOS 和 OSX 适用的 Microsoft Azure Active Directory Authentication Library (ADAL)** 提供的 [acquireAuthTokenWithResource:clientId:redirectUri:completionBlock:](https://github.com/AzureAD/azure-activedirectory-library-for-objc)。此 SDK 为应用程序提供了 Microsoft Azure AD 的全部功能，其中包括 OAuth2 的行业标准协议支持、Web API 集成（含用户级同意）和双因素身份验证支持。

此方法提取以下参数：

1. **resourceID** - 这是要访问的相应资源。例如，由于我们想访问 Microsoft Graph API，因此该值为“https://graph.microsoft.com”。
2. **clientID** - 在您完成 AAD 注册时，用于标识应用的值。
3. **redirectURI** - 再强调一次，这一所需值用于指定用户在成功通过身份验证后，被重定向到哪里。


首先，您需要指定身份验证上下文。这就定义了您要从中获得访问令牌的颁发机构。在我们的示例中，它来自于 AAD 租户，您需要对它进行声明：

    @property (strong,    nonatomic) ADAuthenticationContext *context;

然后，初始化它，并指定颁发机构的位置 (https://login.microsoftonline.com/common)：

    self.context = [ADAuthenticationContext authenticationContextWithAuthority:self.authority]; 


在 [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) 示例中，我们创建了单一身份验证类 (**AuthenticationManager**) 以方便本文演示，此类进行了初始化，并指定了颁发机构和所需参数。再强调一次，此类只是一个关于如何处理身份验证工作流的示例。相关代码段为： 



    - (void)acquireAuthTokenWithResource:(NSString *)resourceID
                            clientID:(NSString*)clientID
                         redirectURI:(NSURL*)redirectURI
                          completion:(void (^)(ADAuthenticationError *error))completion {
    
    NSLog(@"acquireAuthTokenWithResource");
    [self.context acquireTokenWithResource:resourceID
                                  clientId:clientID
                               redirectUri:redirectURI
                           completionBlock:^(ADAuthenticationResult *result) {
                               NSLog(@"Completion");
                               
                               if (result.status !=AD_SUCCEEDED){
                                   NSLog(@"error");
                                   completion(result.error);
                               }
                               
                               else{
                                   NSLog(@"complete!");
                                   self.accessToken = result.accessToken;
                                   self.userID = result.tokenCacheStoreItem.userInformation.userId;
                                   completion(nil);
                               }
                           }];


当此应用首次运行时，身份验证管理器会向颁发机构发送请求，以便将你重定向到登录页。 
您需要提供凭据，然后响应中会包含身份验证结果。 
如果成功，它还会包含刷新令牌和访问令牌。 
当此应用程序第二次运行时，假设您没有清除令牌缓存和 Cookie，身份验证管理器会使用缓存中的访问令牌或刷新令牌，对客户端请求进行身份验证。 
如果您需要获取访问令牌，那么这会生成服务调用。 


## 在对 Microsoft Graph API 提出的请求中使用访问令牌

使用访问令牌，您的应用可以对 Microsoft Graph API 提出身份验证请求。您的应用必须将访问令牌附加到 HTTP 请求头的**授权**下。

[ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) 示例使用 Microsoft Graph API 中的 sendMail 终结点发送电子邮件。再强调一次，在我们的示例中，我们创建了单一身份验证类 (AuthenticationManager)，此类进行了初始化，并指定了访问令牌。我们需要使用访问令牌才能构造我们的请求。



    - (void)sendMailREST {
    
    AuthenticationManager *authManager = [AuthenticationManager sharedInstance];

    //Helper method used to construct the email message
    NSData *postData = [self mailContent];
    
    //Microsoft Graph API endpoint for sending mail
    NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:[NSURL URLWithString:@"https://graph.microsoft.com/v1.0/me/microsoft.graph.sendmail"]];

    [request setHTTPMethod:@"POST"];
    [request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
    [request setValue:@"application/json, text/plain, */*" forHTTPHeaderField:@"Accept"];
    
    // Access token required for request header
    NSString *authorization = [NSString stringWithFormat:@"Bearer %@", authManager.accessToken];
    [request setValue:authorization forHTTPHeaderField:@"Authorization"];
    [request setHTTPBody:postData];

    NSURLConnection *conn = [[NSURLConnection alloc] initWithRequest:request delegate:self];
    
    if(conn) {
        NSLog(@"Connection Successful");
    } else {
        NSLog(@"Connection could not be made");
    }
    
    [conn start];

如您所见，响应由 NSURLConnection 代理（即 NSURLConnectionDelegate 和 NSURLConnectionDataDelegate）进行处理。

## 后续步骤

如果访问令牌已过期或即将过期，那么你可以使用 ADAuthenticationContext 的 **acquireTokenSilentWithResource:clientId:redirectUri:completionBlock:** 来获取新的访问令牌。[ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) 示例中介绍了它的用法。另外，你还可以找到用于清除令牌缓存和已存储 Cookie 的代码。  

Microsoft Graph API 的功能非常强大，统一了可用于与各种 Microsoft 数据进行交互的 API。请查看 [API 参考](http://graph.microsoft.io/docs/api-reference/v1.0)，了解您还可以使用 Microsoft Graph API 完成什么任务。

