# Open ID authentication with Jakarta EE 10, Security 3.0, and Auth0

Please read [Use Jakarta EE 10 with OpenID Connect Authentication](https://auth0.com/blog/jakarta-ee-oidc/) to see how this example was created. 

## Requirements

Before you start, please make sure you have the following prerequisites installed (or install them now).

- [Java 17](https://adoptium.net/): or use [SDKMAN!](https://sdkman.io/) to manage and install multiple versions (the Jakarta EE spec says 11 and up is supported, but I wrote this tutorial assuming version 17)
- [Auth0 CLI](https://github.com/auth0/auth0-cli#installation): the Auth0 command-line interface
- [HTTPie](https://httpie.org/doc#installation): a simple tool for making HTTP requests from a Bash shell

**You will need a free Auth0 developer account** if you don't already have one. Go ahead and sign up for an Auth0 account using [their sign-up page](https://auth0.com/signup).

 ## Create an OIDC Application
 
You will need to have a free Auth0 developer account and log in to the account using the CLI.

```shell
auth0 login
```
 
Use the Auth0 CLI to create an OpenID Connect (OIDC) application.  From the project base directory, run the following.

```bash
auth0 apps create
```

Use the following values:

- **Name**: `javartaee-demo`
- **Description**: whatever you like, or leave blank
- **Type**: `Regular Web Application`
- **Callback URLs**: `http://localhost:8080/callback`
- **Allowed Logout URLs**: `http://localhost:8080`

The console output shows you the Auth0 domain and the OIDC client ID. However, you also need the client secret, which you have to get by logging into Auth0. Type the following:

```bash
auth0 apps open
```

Select the OIDC app (or client) you just created from the list. This will open the OIDC application on the Auth0 dashboard.

Fill in the three values in `src/main/resources/openid.properties`. Replace the bracketed values with the values from the OIDC application page on the Auth0 dashboard.

```bash
issuerUri=https://<your-auth0-domain>
clientId=<your-client-id>
clientSecret=<your-client-secret>
```

In `src/main/java/com/demo/ProtectedServlet.java`, replace `<your-auth0-domain` in the `@OpenIdAuthenticationMechanismDefinition` annotation with your actual Auth0 domain.

## Configure Roles on Auth0

Open your [Auth0 developer dashboard](https://manage.auth0.com). You need to create a role, assign your user to that role, and create an action that will inject the roles into a custom claim in the JWT.

Under **User Management** click on **Roles**. Click the **Create Role** button. 

**Name** the role `Everyone`. Give it a **Description**, whatever you like. Click **Create**.

The Everyone role panel should be shown. Select the **Users** tab. Click **Add Users**. Assign yourself to the role.

You've now created a role and assigned yourself to it. But this information will not be passed along in the JWT without a little customization. The current best practice is to do this using actions.

Select **Actions** from the left menu in the developer dashboard. Click on **Flows**. Select **Login**.

Add a new action by clicking on the **+** symbol to the right of **Add Action**. Select **Build Custom**.

Give the action a **Name**, such as `Add Roles`. Leave the other two values the same. Click **Create**.

Change the code for the action to the following.

```js
exports.onExecutePostLogin = async (event, api) => {
  const namespace = 'http://www.jakartaee.demo';
  if (event.authorization) {
    api.idToken.setCustomClaim('preferred_username', event.user.email);
    api.idToken.setCustomClaim(`${namespace}/roles`, event.authorization.roles);
    api.accessToken.setCustomClaim(`${namespace}/roles`, event.authorization.roles);
  }
}
```

Click on **Deploy**. 

Click on the **Add to flow** link in the popup window that slides in (if you miss this, you can find the new action under the custom action tab back in the flow panel).

Drag the **Add Roles** action over under the **Rules (legacy)** action. 

Click **Apply** (top right of the panel).

## Start the project

Use this command to start the project.

```bash
./mvnw wildfly:run
```

Using a browser, open `http://localhost:8080/protected`. You should be prompted to log in.

## Links

This example uses the following open source libraries:

* [WildFly](https://www.wildfly.org/)
* [jwks-rsa-java](https://github.com/auth0/jwks-rsa-java)
* [java-jwt](https://github.com/auth0/java-jwt)

## Help

Please post any questions as comments on the [blog post](https://auth0.com/blog/jakarta-ee-oidc/), or visit our [Auth0 Community](https://community.auth0.com/).

## License

Apache 2.0, see [LICENSE](LICENSE).
