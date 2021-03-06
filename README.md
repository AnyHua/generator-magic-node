<h1 align="center">generator-magic-node</h1>
<p align="center">
<a href="https://www.npmjs.com/package/vue" rel="nofollow">
    <img src="https://camo.githubusercontent.com/9a140a4c68e7c178bc660bee7675f4f25ff7ade3/68747470733a2f2f696d672e736869656c64732e696f2f6e706d2f6c2f7675652e737667" alt="License" data-canonical-src="https://img.shields.io/npm/l/vue.svg" style="max-width:100%;">
</a>
</p>
<div align="center">
    English | <a href="./README-zh_CN.md">简体中文</a>
</div>

# Describe    
This is a node back-end service generator that generates scaffolding and entity classes.   

In order to ensure that the background required for a set of front-end projects can switch seamlessly between jhipster and nodejs, the interface specification and jhipster are as consistent as possible, including authorization and authentication methods.

# Conditions  
1. yeoman needs to be installed.
```
npm install -g yo
```
2. The entity class generation section requires the JSON file generated using [JDL], Students who are not familiar with [JDL](https://www.jhipster.tech/jdl) can find relevant documents and help on [jhipster](https://www.jhipster.tech) official website.  
```
npm install -g generator-jhipster
```

# Install      
```
npm install -g generator-magic-node
```

# Use    
* Generating scaffold.
```
mkdir node-project
cd node-project
yo magic-node
```

* Generating entity classes.
1. Please go to [jdl-studio](https://start.jhipster.tech/jdl-studio) to write the data table and export it to the project root directory. For example, the file I exported is jhipster-jdl.jh
    ```
        cp jhipster-jdl.jh node-project/jhipster-jdl.jh
        cd node-project
        jhipster import-jdl jhipster-jdl.jh
    ```
    After successful execution, .jhipster folder will appear in the current project file directory.

2. Execute the command to generate the entity class.
    ```
    yo magic-node --lang=zh --entity=Region
    ```
    Or
    ```
    yo magic-node
    language[en/zh/ja] en
    What generates? Entity class
    Entity name: Region
    ```

# Directory structure
```
.directory
.gitignore
.jhipster   # Data model config.
│-- Region.json 
.yo-rc.json
README.md
config.json
ormconfig.json
package.json
src
│-- controller  # Control layer, interface entry.
│-- entity  # Entity layer, data model class location.
│-- enum  # Enum class.
│-- index.ts
│-- middleware
│-- migration
│-- sample  # Sample data for automatically creating template data at project startup.
│-- subscriber
│-- util
tsconfig.json
```
# Generated control layer example
```
import { Context } from 'koa';
import { getManager } from 'typeorm';
import Region from '../entity/Region';
import { validate } from 'class-validator';
import Elastic from '../util/Elastic';
import { setRoute, RequestMethod } from '../middleware/Routes';
import { getElasticSearchParams, setElasticSearchPagingHeader, deleteSuccessfulResponse, getRequestParamId } from '../util/Tools';
import { BadRequestAlertException, NotFoundAlertException } from '../middleware/RequestError';

export default class RegionAction {
  /**
   * get
   * @param {Application.Context} context
   * @param decoded
   * @returns {Promise<void>}
   */
  @setRoute('/api/regions/:id')
  async getRegion(context: Context, decoded?: any) {
    const id = getRequestParamId(context);
    const regionRepository = getManager().getRepository(Region);
    const region: Region = await regionRepository.findOne(id);
    if (region) {
      context.body = region;
    } else {
      throw new NotFoundAlertException();
    }
  }
  /**
   * post
   * @param {Application.Context} context
   * @param decoded
   * @returns {Promise<void>}
   */
  @setRoute('/api/regions', RequestMethod.POST)
  async createRegion(context: Context, decoded?: any) {
    const regionRepository = getManager().getRepository(Region);
    const region: Region = regionRepository.create(<Region>{
      ...context.request.body
    });
    const errors = await validate(region);
    if (errors.length > 0) {
      throw new BadRequestAlertException(null, errors);
    } else {
      await getManager().save(region);
      await Elastic.syncCreate(region.id, regionRepository.metadata.tableName);
      context.body = region;
    }
  }
  /**
   * put
   * @param {Application.Context} context
   * @param decoded
   * @returns {Promise<void>}
   */
  @setRoute('/api/regions', RequestMethod.PUT)
  async updateRegion(context: Context, decoded?: any) {
    const regionRepository = getManager().getRepository(Region);
    const region: Region = regionRepository.create(<Region>{
      ...context.request.body
    });
    const errors = await validate(region);
    if (errors.length > 0) {
      throw new BadRequestAlertException(null, errors);
    } else {
      await regionRepository.update(region.id, region);
      await Elastic.syncUpdate(region.id, regionRepository.metadata.tableName);
      context.body = region;
    }
  }
  /**
   * delete
   * @param {Application.Context} context
   * @param decoded
   * @returns {Promise<void>}
   */
  @setRoute('/api/regions/:id', RequestMethod.DELETE)
  async deleteRegion(context: Context, decoded?: any) {
    const id = getRequestParamId(context);
    const regionRepository = getManager().getRepository(Region);
    await regionRepository.delete(id);
    await Elastic.syncDelete(id, regionRepository.metadata.tableName);
    deleteSuccessfulResponse(context, id)
  }
  /**
   * search
   * @param {Application.Context} context
   * @param decoded
   * @returns {Promise<void>}
   */
  @setRoute('/api/_search/regions')
  async searchRegion(context: Context, decoded?: any) {
    const res: any = await Elastic.search(getElasticSearchParams(context), 'region');
    setElasticSearchPagingHeader(context, res);
    context.body = res.data;
  }
}

``` 

# Interface example
* Create  
    Request:

    ```
    curl --request POST \
    --url http://localhost:3000/api/regions \
    --header 'cache-control: no-cache' \
    --header 'content-type: application/json' \
    --header 'postman-token: 137cfaa2-bba3-94c6-a73d-858bf899f3c7' \
    --data '{\n	"regionName": "CN"\n}'
    ```
    ```
    POST /api/regions HTTP/1.1
    Host: localhost:3000
    Content-Type: application/json
    Cache-Control: no-cache
    Postman-Token: ffe8ff8c-f9e2-60be-dc1a-0f7e5e2b2383

    {
        "regionName": "CN"
    }
    ```
    Response:
    ```
    {
        "regionName": "CN",
        "id": 1
    }
    ```
* Change  
    Request:
    ```
    curl --request PUT \
    --url http://localhost:3000/api/regions \
    --header 'cache-control: no-cache' \
    --header 'content-type: application/json' \
    --header 'postman-token: e4cae952-07bc-0857-73be-9d2099d3954f' \
    --data '{\n	"id": 1,\n	"regionName": "JP"\n}'
    ```
    ```
    PUT /api/regions HTTP/1.1
    Host: localhost:3000
    Content-Type: application/json
    Cache-Control: no-cache
    Postman-Token: 604881ef-c221-f756-e3c5-6e1e58163b6c

    {
	    "id": 1,
	    "regionName": "JP"
    }
    ```
    Response:
    ```
    {
        "id": 1,
        "regionName": "JP"
    }
    ```
* Delete  
    Request:
    ```
    curl --request DELETE \
    --url http://localhost:3000/api/regions/1 \
    --header 'cache-control: no-cache' \
    --header 'content-type: application/json' \
    --header 'postman-token: d0bd6631-b56d-3c14-565a-f1a6d14aba47'
    ```
    ```
    DELETE /api/regions/1 HTTP/1.1
    Host: localhost:3000
    Content-Type: application/json
    Cache-Control: no-cache
    Postman-Token: 70b023ea-b101-c46e-9240-90f1909c2107
    ```
    Response:
    ```
    {
        "id": "2"
    }
    ```
* get  
    Request:
    ```
    curl --request GET \
    --url http://localhost:3000/api/regions/1 \
    --header 'cache-control: no-cache' \
    --header 'content-type: application/json' \
    --header 'postman-token: 32045bad-4ed5-fbfa-4a7f-9b2686eb3891'
    ```
    ```
    GET /api/regions/1 HTTP/1.1
    Host: localhost:3000
    Content-Type: application/json
    Cache-Control: no-cache
    Postman-Token: ca50cf1d-7e72-70a2-37c8-037bfcc5893f

    ```
    Response:
    ```
    {
        "id": 1,
        "regionName": "CN"
    }
    ```
* Search (Enable elasticsearch)  
    Request:
    ```
    curl --request GET \
    --url 'http://localhost:3000/api/_search/regions?query=regionName%3AJP&page=0&size=10' \
    --header 'cache-control: no-cache' \
    --header 'content-type: application/json' \
    --header 'postman-token: 12afa220-9d89-6f7c-97b9-1dde9f4c8c85'
    ```
    ```
    GET /api/_search/regions?query=regionName:JP&amp;page=0&amp;size=10 HTTP/1.1
    Host: localhost:3000
    Content-Type: application/json
    Cache-Control: no-cache
    Postman-Token: 260f20fc-4697-51a1-38cb-ceeb4259ea23
    ```
    Response:
    ```
    [
        {
            "id": 1,
            "regionName": "JP"
        }
    ]
    ```
# Login 
Authorize the use of the JWT protocol, The interface is the same as jhipster's JWT authorized login interface.
* Get Token  
    Request:
    ```
    curl --request POST \
    --url http://localhost:3000/api/authenticate \
    --header 'cache-control: no-cache' \
    --header 'content-type: application/json' \
    --header 'postman-token: dd8547b8-5c18-e5b1-e0e9-621de99914f1' \
    --data '{\n	"password": "admin",\n	"username": "admin"\n}'
    ```
    ```
    POST /api/authenticate HTTP/1.1
    Host: localhost:3000
    Content-Type: application/json
    Cache-Control: no-cache
    Postman-Token: 53e4cacc-d444-ed3c-be50-c107693b8d1e

    {
	    "password": "admin",
	    "username": "admin"
    }
    ```
    Response:
    ```
    {
        "id_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjp7ImlkIjozLCJsb2dpbiI6ImFkbWluIiwiZmlyc3ROYW1lIjoiQWRtaW5pc3RyYXRvciIsImxhc3ROYW1lIjoiQWRtaW5pc3RyYXRvciIsImVtYWlsIjoiYWRtaW5AbG9jYWxob3N0IiwiaW1hZ2VVcmwiOiJodHRwczovL3dwaW1nLndhbGxzdGNuLmNvbS9mNzc4NzM4Yy1lNGY4LTQ4NzAtYjYzNC01NjcwM2I0YWNhZmUuZ2lmP2ltYWdlVmlldzIvMS93LzgwL2gvODAiLCJhY3RpdmF0ZWQiOjEsImxhbmdLZXkiOiJlbiIsImFjdGl2YXRpb25LZXkiOm51bGwsInJlc2V0S2V5IjpudWxsLCJjcmVhdGVkQnkiOiJzeXN0ZW0iLCJjcmVhdGVkRGF0ZSI6bnVsbCwicmVzZXREYXRlIjpudWxsLCJsYXN0TW9kaWZpZWRCeSI6InN5c3RlbSIsImxhc3RNb2RpZmllZERhdGUiOm51bGx9LCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIiwiUk9MRV9VU0VSIl0sImlhdCI6MTU0ODM5OTQzNCwiZXhwIjoxNTQ5Njk1NDM0fQ.9y9YOKtJaT33T56A2mKKrSeI0ojCmUFPU5JxvLNR3ds"
    }
    ```
* Get Account info  
    Request: 
    ```
    curl --request GET \
    --url http://localhost:3000/api/account \
    --header 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjp7ImlkIjozLCJsb2dpbiI6ImFkbWluIiwiZmlyc3ROYW1lIjoiQWRtaW5pc3RyYXRvciIsImxhc3ROYW1lIjoiQWRtaW5pc3RyYXRvciIsImVtYWlsIjoiYWRtaW5AbG9jYWxob3N0IiwiaW1hZ2VVcmwiOiJodHRwczovL3dwaW1nLndhbGxzdGNuLmNvbS9mNzc4NzM4Yy1lNGY4LTQ4NzAtYjYzNC01NjcwM2I0YWNhZmUuZ2lmP2ltYWdlVmlldzIvMS93LzgwL2gvODAiLCJhY3RpdmF0ZWQiOjEsImxhbmdLZXkiOiJlbiIsImFjdGl2YXRpb25LZXkiOm51bGwsInJlc2V0S2V5IjpudWxsLCJjcmVhdGVkQnkiOiJzeXN0ZW0iLCJjcmVhdGVkRGF0ZSI6bnVsbCwicmVzZXREYXRlIjpudWxsLCJsYXN0TW9kaWZpZWRCeSI6InN5c3RlbSIsImxhc3RNb2RpZmllZERhdGUiOm51bGx9LCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIiwiUk9MRV9VU0VSIl0sImlhdCI6MTU0ODM5OTQzNCwiZXhwIjoxNTQ5Njk1NDM0fQ.9y9YOKtJaT33T56A2mKKrSeI0ojCmUFPU5JxvLNR3ds' \
    --header 'cache-control: no-cache' \
    --header 'postman-token: bc7974d7-c9ff-8ccb-1037-0efdb099db88'
    ```
    ```
    GET /api/account HTTP/1.1
    Host: localhost:3000
    Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjp7ImlkIjozLCJsb2dpbiI6ImFkbWluIiwiZmlyc3ROYW1lIjoiQWRtaW5pc3RyYXRvciIsImxhc3ROYW1lIjoiQWRtaW5pc3RyYXRvciIsImVtYWlsIjoiYWRtaW5AbG9jYWxob3N0IiwiaW1hZ2VVcmwiOiJodHRwczovL3dwaW1nLndhbGxzdGNuLmNvbS9mNzc4NzM4Yy1lNGY4LTQ4NzAtYjYzNC01NjcwM2I0YWNhZmUuZ2lmP2ltYWdlVmlldzIvMS93LzgwL2gvODAiLCJhY3RpdmF0ZWQiOjEsImxhbmdLZXkiOiJlbiIsImFjdGl2YXRpb25LZXkiOm51bGwsInJlc2V0S2V5IjpudWxsLCJjcmVhdGVkQnkiOiJzeXN0ZW0iLCJjcmVhdGVkRGF0ZSI6bnVsbCwicmVzZXREYXRlIjpudWxsLCJsYXN0TW9kaWZpZWRCeSI6InN5c3RlbSIsImxhc3RNb2RpZmllZERhdGUiOm51bGx9LCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIiwiUk9MRV9VU0VSIl0sImlhdCI6MTU0ODM5OTQzNCwiZXhwIjoxNTQ5Njk1NDM0fQ.9y9YOKtJaT33T56A2mKKrSeI0ojCmUFPU5JxvLNR3ds
    Cache-Control: no-cache
    Postman-Token: 0697e795-506d-fb51-f35a-cf6009437a8d

    ```
    Response:
    ```
    {
        "id": 3,
        "login": "admin",
        "firstName": "Administrator",
        "lastName": "Administrator",
        "email": "admin@localhost",
        "imageUrl": "https://wpimg.wallstcn.com/f778738c-e4f8-4870-b634-56703b4acafe.gif?imageView2/1/w/80/h/80",
        "activated": 1,
        "langKey": "en",
        "activationKey": null,
        "resetKey": null,
        "createdBy": "system",
        "createdDate": null,
        "resetDate": null,
        "lastModifiedBy": "system",
        "lastModifiedDate": null,
        "authorities": [
            "ROLE_ADMIN",
            "ROLE_USER"
        ]
    }
    ```

# Technology stack   

<a href="https://nodejs.org">
    <img src="https://raw.githubusercontent.com/little-snow-fox/images/master/logos/node.png" width="20%">
</a>
<a href="https://github.com/koajs/koa">
    <img src="https://raw.githubusercontent.com/koajs/koa/HEAD/docs/logo.png" width="20%">
</a>
<a href="https://www.tslang.cn">
    <img src="https://raw.githubusercontent.com/little-snow-fox/images/master/logos/typescript.png" width="20%">
</a>
<a href="http://typeorm.io">
    <img src="https://raw.githubusercontent.com/little-snow-fox/images/master/logos/typeorm.png" width="20%">
</a>
<a href="https://www.mysql.com">
    <img src="https://raw.githubusercontent.com/little-snow-fox/images/master/logos/mysql.png" width="20%">
</a>
<a href="https://www.elastic.co">
    <img src="https://raw.githubusercontent.com/little-snow-fox/images/master/logos/elastic.png" width="20%">
</a>
<a href="https://yeoman.io">
    <img src="https://raw.githubusercontent.com/little-snow-fox/images/master/logos/yeoman.png" width="20%">
</a>

# License  
MIT © [snowfox](https://github.com/little-snow-fox)
