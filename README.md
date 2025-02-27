# oatpp-swagger [![oatpp build status](https://dev.azure.com/lganzzzo/lganzzzo/_apis/build/status/oatpp.oatpp-swagger)](https://dev.azure.com/lganzzzo/lganzzzo/_build?definitionId=2)
Swagger UI for oatpp services

Read more:
- [About oatpp](https://oatpp.io/)  
- [What is Swagger UI](https://swagger.io/tools/swagger-ui/)
- [Endpoint annotation and API documentation in oatpp](https://oatpp.io/docs/components/api-controller/#endpoint-annotation-and-api-documentation).

## Example

For full example project see: [Example CRUD-API project with Swagger UI](https://github.com/oatpp/example-crud)

## Brief

- Use ```oatpp::swagger::Controller``` with ```oatpp::web::server::HttpConnectionHandler```
- Use ```oatpp::swagger::AsyncController``` with ```oatpp::web::server::AsyncHttpConnectionHandler```

- Swagger UI location - ```http://localhost:<PORT>/swagger/ui```
- OpenApi 3.0.0 specification location - ```http://localhost:<PORT>/api-docs/oas-3.0.0.json```

If you are using ```oatpp::web::server::api::ApiController``` most parts of your endpoints are documented automatically like:

- Endpoint name
- Parameters
- Request Body

You may add more information to your endpoint like follows:

```c++
ENDPOINT_INFO(createUser) {
  info->summary = "Create new User";
  info->addConsumes<UserDto>("application/json");
  info->addResponse<UserDto>(Status::CODE_200, "application/json");
  info->addResponse(Status::CODE_500);
}
ENDPOINT("POST", "demo/api/users", createUser,
         BODY_DTO(UserDto, userDto)) {
  return createDtoResponse(Status::CODE_200, m_database->createUser(userDto));
}
```

### How to add Swagger UI to your project

1) Add ```oatpp::swagger::DocumentInfo``` and ```oatpp::swagger::Resources``` components to your AppComponents:

```c++
/**
 *  General API docs info
 */
OATPP_CREATE_COMPONENT(std::shared_ptr<oatpp::swagger::DocumentInfo>, swaggerDocumentInfo)([] {

  oatpp::swagger::DocumentInfo::Builder builder;

  builder
  .setTitle("User entity service")
  .setDescription("CRUD API Example project with swagger docs")
  .setVersion("1.0")
  .setContactName("Ivan Ovsyanochka")
  .setContactUrl("https://oatpp.io/")

  .setLicenseName("Apache License, Version 2.0")
  .setLicenseUrl("http://www.apache.org/licenses/LICENSE-2.0")

  .addServer("http://localhost:8000", "server on localhost");

  // When you are using the AUTHENTICATION() Endpoint-Macro you must add an SecurityScheme object (https://swagger.io/specification/#securitySchemeObject)
  // For basic-authentication you can use the default Basic-Authorization-Security-Scheme like this
  // For more complex authentication schemes you can use the oatpp::swagger::DocumentInfo::SecuritySchemeBuilder builder
  // Don't forget to add info->addSecurityRequirement("basic_auth") to your ENDPOINT_INFO() Macro!
  .addSecurityScheme("basic_auth", oatpp::swagger::DocumentInfo::SecuritySchemeBuilder::DefaultBasicAuthorizationSecurityScheme());

  return builder.build();

}());


/**
 *  Swagger-Ui Resources (<oatpp-examples>/lib/oatpp-swagger/res)
 */
OATPP_CREATE_COMPONENT(std::shared_ptr<oatpp::swagger::Resources>, swaggerResources)([] {
  // Make sure to specify correct full path to oatpp-swagger/res folder !!!
  return oatpp::swagger::Resources::loadResources("<YOUR-PATH-TO-REPO>/lib/oatpp-swagger/res");
}());

```

2) Create ```oatpp::swagger::Controller``` with list of endpoints you whant to document and add it to router:

```c++
auto swaggerController = oatpp::swagger::Controller::createShared(<list-of-endpoints-to-document>);
swaggerController->addEndpointsToRouter(router);
```

**Done!**
