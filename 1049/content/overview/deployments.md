# Независимые облачные развертывания


В этой статье приведены сведения о различных независимых облачных экземплярах Microsoft Graph и возможностях, доступных разработчикам. 


## Служба Microsoft Graph, предоставляемая 21Vianet в Китае

В этом разделе приведены сведения о службе Microsoft Graph, которую предоставляет оператор 21Vianet, и возможностях, доступных разработчикам.

### Корневые конечные точки службы
| Служба Microsoft Graph, предоставляемая 21Vianet | Microsoft Graph|
|---------------------------|----------------|
| https://microsoftgraph.chinacloudapi.cn | https://graph.microsoft.com|

### Microsoft Graph Explorer
| Служба Microsoft Graph, предоставляемая 21Vianet | Microsoft Graph|
|---------------------------|----------------|
|https://graphexplorerchina.azurewebsites.net| https://graphexplorer2.azurewebsites.net|

### Azure OpenID Connect и OAuth2.0
Конечные точки, которые используются для получения маркеров для входа или для вызова службы Microsoft Graph, предоставляемой 21vianet, отличаются от конечных точек других предложений. 

| Служба Microsoft Graph, предоставляемая 21Vianet | Microsoft Graph|
|---------------------------|----------------|
| https://login.chinacloudapi.cn | https://login.microsoftonline.com|
 
Используйте https://login.chinacloudapi.cn/common/oauth2/authorize, чтобы проверить подлинность пользователя, и https://login.chinacloudapi.cn/common/oauth2/token, чтобы получить маркер для вызова службы Microsoft Graph, которую предоставляет оператор 21Vianet.

> **ПРИМЕЧАНИЕ**. Последние [конечные точки авторизации и маркера версии 2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-appmodel-v2-overview/) НЕДОСТУПНЫ для использования со службой Microsoft Graph, которую предоставляет оператор 21Vianet.  Приложениям доступны только данные организации, но не пользователей. 

### Возможности службы Microsoft Graph, предоставляемой 21Vianet
Следующие функции Microsoft Graph общедоступны (в конечной точке `/v1.0`):

* Пользователи
* Группы
* Files
* Почтовое
* Календарь
* Личные контакты 
* Операции CRUD (создание, чтение, обновление, удаление)
* Поддержка общего доступа к ресурсам независимо от источника (CORS)

Следующие функции Microsoft Graph также доступны в предварительной версии (в конечной точке `/beta`):

* Контакты организации
* Приложения
* Субъекты-службы
* Расширения схемы каталога
