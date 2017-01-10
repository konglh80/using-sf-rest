# 使用Salesforce REST API

  - 创建一个Connected App
  - User-Agent OAuth Authentication Flow
  - 调用Salesforce REST API
  - Production与Sandbox的差别


## 创建一个Connected App
- 打开[Apps列表](https://ap2.salesforce.com/02u?retURL=%2Fui%2Fsetup%2FSetup%3Fsetupid%3DDevTools&setupid=TabSet)，点击新建一个Connected App。
![New Connected App](/images/new-connected-app-button.png)
- 填写`Connected App Name`，`API Name(自动生成)`，`Contact Email`。
![New Connected App](/images/new-connected-app.png)
- 勾选`API (Enable OAuth Settings)`部分的`Enable OAuth Settings`，并继续填写`Callback URL`，其值固定为`https://login.salesforce.com/services/oauth2/success`。
- 选择`Selected OAuth Scopes`，包括`Access and manage your data (api)`，`Access your basic information (id, profile, email, address, phone)`，`Perform requests on your behalf at any time (refresh_token, offline_access)`，其余可不填，页面下方找到并点击`Save`按钮。
![New Connected App](/images/new-connected-app-consumer-key.png)
- 创建成功后会进入`Connected App Detail`页面，其中我们唯一需要的是`Consumer Key`

*PS：新建的Connected App需要等几分钟后生效。*
*[官方文档供参考](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_defining_remote_access_applications.htm)。*

## User-Agent OAuth Authentication Flow
调用REST API前需要引导用户进行OAuth认证以获得Access Token，下面介绍Salesforce User-Agent OAuth Authentication Flow。
- 将用户引导至`https://login.salesforce.com/services/oauth2/authorize`，并在URL中附加如下参数。
1. `response_type=token`
2. `client_id=<上一步创建Connected App后得到的Consumer Key>`
3. `redirect_uri=https://login.salesforce.com/services/oauth2/success`
```
// 样例
https://login.salesforce.com/services/oauth2/authorize?response_type=token&client_id=3MVG9ZL0ppGP5UrAPuXwIS5TUnERw1UfZvUBMHr_8v0cOpSCUJ64aH8pVxZx9ek6JivYkaKns..vafP7rFfcr&redirect_uri=https://login.salesforce.com/services/oauth2/success
```
- 接下来用户会在Salesforce标准登录页面输入自己的用户名和密码完成认证过程，认证成功后会跳转至`https://login.salesforce.com/services/oauth2/success`页面，并会将Access Token等数据附加到URL中
```
// 样例
https://login.salesforce.com/services/oauth2/success#access_token=00Dx0000000BV7z%21AR8AQBM8J_xr9kLqmZIRyQxZgLcM4HVi41aGtW0qW3JCzf5xdTGGGSoVim8FfJkZEqxbjaFbberKGk8v8AnYrvChG4qJbQo8&refresh_token=5Aep8614iLM.Dq661ePDmPEgaAW9Oh_L3JKkDpB4xReb54_pZfVti1dPEk8aimw4Hr9ne7VXXVSIQ%3D%3D
```
*PS：由于Access Token/Refresh Token等数据包含在URL中返回，某些情况会包含特殊字符，因此会被URI转码，需要进行URI解码后使用，确保无误。*
