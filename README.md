# Strapi On Azure Windows Web App

## Local test
1. Refer https://docs.strapi.io/dev-docs/quick-start
2. Use yarn
  - `yarn build`
  - `yarn develop`
  - `yarn start`

## Azure Deployment
https://docs.strapi.io/dev-docs/deployment/azure

Azure Comprehensive Guide: https://strapi.io/blog/setting-up-strapi-with-azure-service-apps-a-comprehensive-guide

This repo is a customoziation of the code generated by the Strapi CLI to make it works on Azure Windows WebApp. To reproduce this repo by your own:
1. `npm create-strapi-app my-project --quickstart`
2. In the folder `my-project` add the 2 files:
      - `server.js` this file is needed because the Windows Web App needs a js file as an entry point. Strapi v4 introduces a more modular architecture, and the entry point file is now located in the ./config/server.js file by default.
      - `web.config` this file is use by iisnode which is the bridge between Windows IIS and Node js.
3. Edit the `package.json` file replace the start script `strapi start` by `node server.js`

### server.js file 

This file replace the CLI `strapi start`:
```js 
// ./config/server.js
```

### Web.config file

This file is almost the default one generated automaticaly by Azure on a Git deploy. You notice that it refers to the `server.js`file. The difference with the default one is the security section:
```xml    
<security>
  <requestFiltering>
    <hiddenSegments>
      <remove segment="build"/>
    </hiddenSegments>
  </requestFiltering>
</security>
```
This section allows to access to the `build` folder which contains the Admin panel

## Deploy on Azure

1. Create a Windows Web App in Azure
2. In `Application Settings` add the variables (or import the `.env` file): 
    - `NODE_ENV=PRODUCTION` for starting Strapi with the `Production` configuration
    - `WEBSITE_NODE_DEFAULT_VERSION=10.14.1`
    - `WEBSITE_RUN_FROM_PACKAGE=0`
3. Update the settings in `./config/environments/production/server.json` accordingly. Set `"port": "${process.env.PORT || 1337}"` and change the value of the `"host"` property under `"proxy"` to match the public url of the Web App (see: [Server Configurations](https://strapi.io/documentation/3.0.0-beta.x/concepts/configurations.html#server) for more info)
4. (Only in the context of this repo) Create a CosmosDB instance with a MongoDB Api
    - In the `Preview features`, enable `Aggregation Pipeline`
5. Update the settings in `./config/environments/production/database.json` according to your database. You can also use environment variables and set them through `Application Settings` entries on the Web App, e.g. `DATABASE_HOST`, `DATABASE_USERNAME`, etc.
6. In `./config/environments/production/response.json`, disable `gzip`; Azure Web App will already `gzip` the responses, so we don't need Strapi to do this (otherwise the browser will have issues decoding the content it downloads due to 2x `gzip`)
7. Set local variable `set NODE_ENV=PRODUCTION`
8. Build the Admin Panel: `yarn build`
9. Deploy the app with the Zip Deploy (here from VSCode with the [Azure App Service Extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice)) or through an Azure DevOps Release Pipeline using the [Azure Web App Deployment](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureWebAppV1/README.md) task. Make sure to set the `Deployment Method` to `Zip Deploy`.
10. Browse to your App. It could take a while for the first start. Could even end up with an error. Reload the page.

## Last point....
Don't run  `npm i`, use `yarn` instead to be sure to get the right version package

# Environment
Refer this: https://docs.strapi.io/dev-docs/configurations/environment#strapi-s-environment-variables

## .env Example
NODE_ENV="DEVELOPMENT"
WEBSITE_NODE_DEFAULT_VERSION="20.12.2"
WEBSITE_RUN_FROM_PACKAGE="0"
DATABASE_CLIENT="sqlite"
DATABASE_HOST="127.0.0.1"
DATABASE_PORT=27017
DATABASE_USERNAME=""
DATABASE_PASSWORD=""
DATABASE_SSL=false
DEV_MODE=true

HOST=127.0.0.1
PORT=1337
APP_KEYS={random-string-doesn’t-matter}
API_TOKEN_SALT={random-string-doesn’t-matter}
ADMIN_JWT_SECRET={random-string-doesn’t-matter}
TRANSFER_TOKEN_SALT={random-string-doesn’t-matter}
JWT_SECRET={random-string-doesn’t-matter}

## plugins to install
Documentation plugin
We’ll install the documentation plugin from the Marketplace section for easy access to API details. This plugin will create the swagger specification for the API.

## Setting up Strapi’s JWT-based authentication
Authentication is an important element of any application. Strapi has JWT-based authentication out of the box.

A default key is used for signing JWT. A signing key can be changed in the configuration file /extensions/users-permissions/config/jwt.json. The API for user signup and login is already baked into the platform.
``
{
  "jwtSecret": "f1b4e23e-480b-4e58-923e-b759a593c2e0"
}
``

We’ll use the local provider for authentication. This password and email/username are used to authenticate the user. If we click on “Documentation” from the sidebar, it will provide an option to see the swagger API documentation.


## Authentication roles and permissions
Strapi has two roles that are used to control access to content by default: public and authenticated. The public role is for an unauthenticated user, while the authenticated role is for — you guessed it — an authenticated user.

These roles are automatically assigned to a user based on their authentication status. “Public” users are allowed to read blogs and comments and “Authenticated” users can comment on the blog and edit comments. Roles can be edited in the Roles and Permissions section.

## Switching from SQLite to PostgreSQL
Strapi supports both NoSQL and SQL databases. Changing the database is as simple as changing the env variable in the configuration folder.

By default, Strapi uses SQLite, which is good for local testing, but in production you should use a production-ready database such as PostgreSQL or MySQL. We’ll use PostgreSQL here.

To change the database, edit the config/environments/production/database.json file.

{
  "defaultConnection": "default",
  "connections": {
    "default": {
      "connector": "bookshelf",
      "settings": {
        "client": "postgres",
        "host": "${process.env.DATABASE_HOST }",
        "port": "${process.env.DATABASE_PORT }",
        "database": "${process.env.DATABASE_NAME }",
        "username": "${process.env.DATABASE_USERNAME }",
        "password": "${process.env.DATABASE_PASSWORD }"
      },
      "options": {}
    }
  }
}
Now it will pick database credentials from the environment variable in production.

## Security
- Admin Panel MFA
- SSO with Azure AD https://docs.strapi.io/dev-docs/configurations/sso#microsoft
- Custom SSO - need upgrade: https://github.com/andychoi/strapi-plugin-sso4
- https://docs.strapi.io/dev-docs/providers
- user permissions plugin extension: https://forum.strapi.io/t/looking-to-extend-modify-the-users-permissions-plugins-handling-of-provider-authentication/28524

## Custom development
- Custom uploader: https://oramind.com/develop-strapi-upload-provider/

## Misc
- Strapi starters & templates https://github.com/strapi/starters-and-templates
- Frontend, WYSIWYG editor - https://forum.strapi.io/t/blocks-rich-text-editor/32588/9
- random secret: openssl rand -hex 64