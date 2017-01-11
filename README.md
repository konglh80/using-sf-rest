# 使用Salesforce REST API

  - 创建一个Connected App
  - User-Agent OAuth Authentication Flow
  - 调用Salesforce REST API
  - 使用Salesforce Workbench
  - 使用Refresh Token获取新的Access Token

## 创建一个Connected App
- 打开[Apps列表](https://ap2.salesforce.com/02u?retURL=%2Fui%2Fsetup%2FSetup%3Fsetupid%3DDevTools&setupid=TabSet)，点击新建一个Connected App。
![New Connected App](/images/new-connected-app-button.png)
- 填写`Connected App Name`，`API Name(自动生成)`，`Contact Email`。
![New Connected App](/images/new-connected-app.png)
- 勾选`API (Enable OAuth Settings)`部分的`Enable OAuth Settings`，并继续填写`Callback URL`，例如：`callback://oauth-success`。
- 选择`Selected OAuth Scopes`，包括`Access and manage your data (api)`，`Access your basic information (id, profile, email, address, phone)`，`Perform requests on your behalf at any time (refresh_token, offline_access)`，其余可不填，页面下方找到并点击`Save`按钮。
![New Connected App](/images/new-connected-app-consumer-key.png)
- 创建成功后会进入`Connected App Detail`页面，其中我们唯一需要的是`Consumer Key`。

*PS：新建的Connected App需要等几分钟后生效，立即使用`Consumer Key`有时会提示无效。*

*[官方文档供参考](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_defining_remote_access_applications.htm)。*

## User-Agent OAuth Authentication Flow
调用REST API前需要引导用户进行OAuth认证以获得Access Token，下面介绍Salesforce User-Agent OAuth Authentication Flow。
- 将用户引导至`https://login.salesforce.com/services/oauth2/authorize`，并在URL中附加如下参数。（Sandbox环境认证URL是`https://test.salesforce.com/services/oauth2/authorize`）
1. `response_type=token`
2. `client_id=<上一步创建Connected App后得到的Consumer Key>`
3. `redirect_uri=callback://oauth-success`（此处`redirect_uri`与创建Connected App时填写的`Callback URL`一致）
```
// 样例
https://login.salesforce.com/services/oauth2/authorize?response_type=token&client_id=3MVG9ZL0ppGP5UrAPuXwIS5TUnERw1UfZvUBMHr_8v0cOpSCUJ64aH8pVxZx9ek6JivYkaKns..vafP7rFfcr&redirect_uri=callback://oauth-success
```
- 接下来用户需要在Salesforce标准登录页面输入自己的用户名和密码完成认证过程，认证成功后会跳转至`callback://oauth-success`页面，并会将`access_token/refresh_token/instance_url`等信息附加到URL中。
```
// 样例
callback://oauth-success#access_token=00D28000000HjOH%21AQgAQPTO8.7zlmIe9EOXhdAzkXvQLdnj3EPCK1LkZPILEKoxMj0yz18nq6Hjns7sEzfkBaaLprVMRnOkEElgZsSV2_JEcunB&refresh_token=5Aep861TSESvWeug_wdvqFJuAURkIDcmWIctIHpXuYSqCDJ1uXoiCPLp_cpSjmwT6gu1lhCxIQNir9JM.wEHsxO&instance_url=https%3A%2F%2Fap2.salesforce.com&id=https%3A%2F%2Flogin.salesforce.com%2Fid%2F00D28000000HjOHEA0%2F00528000000Dp5CAAS&issued_at=1484124973986&signature=CeCJcczYoBZtXRHgIf%2BbiCcMvkwjRhhMsd0d1nybmT8%3D&scope=id+api+refresh_token&token_type=Bearer
```
*PS：由于access_token/refresh_token/instance_url等数据包含在URL中返回，某些情况会包含特殊字符，因此会被URI转码，需要进行URI解码后使用，确保无误。*

*[官方文档供参考](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_understanding_user_agent_oauth_flow.htm)*

## 使用Salesforce REST API
OAuth成功后返回的数据中，`instance_url`和`access_token`，用于直接访问Salesforce REST API。在向Salesforce发起REST请求之前需要做一些准备工作。

- 使用`instance_url`构建所访问的ORG的REST API的基础URL，所有的针对Salesforce发起的REST请求都基于此URL。
```
<instance_url>/services/data/v37.0
```
- 设置HTTP请求的Header。
```
'Content-Type' = 'application/json'
'Authorization' = 'Bearer ' + access_token
```

### 使用query API

使用query API查询Salesforce中的各类型数据。
例子：
```
HTTP Method: GET
URL: <instance_url>/services/data/v37.0/query?q=SELECT id,name,profile.name FROM user WHERE username='konglh80@gmail.com'
```
*参考：
[Executes the specified SOQL query](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/resources_query.htm)
[Execute a SOQL Query](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/dome_query.htm)*

### 创建记录
通过REST API创建一条Moblor_Tack__TimeCard__c记录。
例子：
`<instance_url>/services/data/v37.0/sobjects/<Object API Name>/`
```
HTTP Method: POST
URL: <instance_url>/services/data/v37.0/sobjects/Moblor_Tack__TimeCard__c/
Body: 
{
  "Moblor_Tack__TeamMember__c":"a2fN0000000qOM0IAM",
  "Moblor_Tack__Description__c":"Time card description",
  "Moblor_Tack__Hours__c":"3"
}
```

*PS：创建时提供的数据中Moblor_Tack__TeamMember__c必须是当前登录人的ID，也就是只能为自己填写TimeCard。*

*参考：
[Create a Record](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/dome_sobject_create.htm)*

### 更新记录
通过REST API更新一条Moblor_Tack__TimeCard__c记录。
例子：
`<instance_url>/services/data/v37.0/sobjects/<Object API Name>/<Record ID>`
```
HTTP Method: PATCH
URL: <instance_url>/services/data/v37.0/sobjects/Moblor_Tack__TimeCard__c/a2gN0000000SCfAIAW
Body: 
{
  "Moblor_Tack__Hours__c":"1"
}
```

*参考：
[Update a Record](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/dome_update_fields.htm)*

### 删除记录
通过REST API删除一条Moblor_Tack__TimeCard__c记录。
例子：
`<instance_url>/services/data/v37.0/sobjects/<Object API Name>/<Record ID>`
```
HTTP Method: DELETE
URL: <instance_url>/services/data/v37.0/sobjects/Moblor_Tack__TimeCard__c/a2gN0000000SCfAIAW
```

*PS：Time Card不能被删除，请求会返回400错误。=。=！*

*参考：
[Delete a Record](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/dome_delete_record.htm)*

### 读取记录
通过REST API删除一条Moblor_Tack__TimeCard__c记录。
例子：
`<instance_url>/services/data/v37.0/sobjects/<Object API Name>/<Record ID>?fields=Field1,Field2...`
```
HTTP Method: GET
URL: <instance_url>/services/data/v37.0/sobjects/Moblor_Tack__TimeCard__c/a2gN0000000SCfAIAW?fields=Moblor_Tack__Description__c,Moblor_Tack__Hours__c
```
*PS：如果不指定任何字段，则查询所有字段的值（`<instance_url>/services/data/v37.0/sobjects/<Object API Name>/<Record ID>`）*

*参考：
[Get Field Values from a Standard Object Record](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/dome_get_field_values.htm)*

