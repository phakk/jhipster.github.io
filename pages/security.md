---
layout: default
title: Security
permalink: /security/
redirect_from:
  - /security.html
sitemap:
    priority: 0.7
    lastmod: 2015-01-27T00:00:00-00:00
---

# <i class="fa fa-lock"></i> Securing your application

To use Spring Security with a Single Web Page Application, like the ones generated by JHipster, you need Ajax login/logout/error views. We have configured Spring Security in order to use those views correctly, and of course we generate all the JavaScript and HTML code for you.

By default, JHipster comes with 4 different users:

*   "system", who is mainly used by our audit logs, when something is done automatically
*   "anonymousUser", who is given to anonymous users when they do an action
*   "user", who is a normal user with "ROLE_USER" authorization. His default password is "user"
*   "admin", who is an admin user with "ROLE_USER" and "ROLE_ADMIN" authorizations. His default password is "admin"

For security reasons, you should change those default passwords.

## HTTP Session Authentication

This is the "classical" Spring Security authentication mechanism, but we have improved it quite significantly. It uses the HTTP Session, so it is a stateful mechanism: if you plan to scale your application on multiple servers, you need to have a load balancer with sticky sessions so that each user stays on the same server.

### Improved remember-me mechanism

We have modified the Spring Security remember-me mechanism so that you have a unique token, that is stored in your database (SQL or NoSQL database, depending on your choice during generation!). We also store more information than the standard implementation, so you have a better understanding of where those tokens come from: IP address, browser, date... And we generate a complete administration screen, so that you can invalidate sessions, for example if you forgot to log out on another computer.

### Cookie theft protection

We have added a very complete cookie theft protection mechanism: we store your security information in a cookie, as well as in the database, and each time a user logs in we modify those values and check if they have been altered. That way, if a user ever steals your cookie, he will be able to use only once, at most.

### CSRF protection

Spring Security and AngularJS both have CSRF protection out-of-the-box, but unfortunately they don't use the same cookies or HTTP headers! In practice, you have in fact no protection at all for CSRF attacks. Of course, we re-configure both tools so that they correctly work together.

## Social Login

JHipster provide "social login", using Spring Social, so users can connect to your application using their Google, Facebook or Twitter authentication. This is configured using Sping Boot's starter modules.

## JWT authentication

[JSON Web Token (JWT)](https://jwt.io/) authentication is a stateless security mechanism, so it's a good option if you want to scale your application on several different servers.

Please note that this is the default option when using a [microservices architecture]({{ site.url }}/microservices-architecture/).

This authentication mechanism doesn't exist by default with Spring Security, it's a JHipster-specific integration of [the Java JWT project](https://github.com/jwtk/jjwt). 

This solution uses a secure token that holds the user's login name and authorities. As the token is signed, it cannot be altered by a user.

The secret key should be configured in the `application.yml` file, as the `jhipster.security.authentication.jwt.secret` property.

## OAuth 2.0 Authentication

OAuth is a stateful security mechanism, like HTTP Session. Spring Security provides OAuth 2.0 support, and this is leveraged by JHipster with its `@EnableOAuthSso` annotation.  If you're not sure what OAuth and OpenID Connect (OIDC) are, please see [What the Heck is OAuth?](https://developer.okta.com/blog/2017/06/21/what-the-heck-is-oauth)

To log in to your app, you'll need to have [Keycloak](https://keycloak.org) up and running. The JHipster Team has created a Docker container for you that has the default users and roles. Start Keycloak using the following command.

```
docker-compose -f src/main/docker/keycloak.yml up
```

The security settings in `src/main/resources/application.yml` and `src/test/resources/application.yml` are configured for this image.

```yaml
security:
    basic:
        enabled: false
    oauth2:
        client:
            accessTokenUri: http://localhost:9080/auth/realms/jhipster/protocol/openid-connect/token
            userAuthorizationUri: http://localhost:9080/auth/realms/jhipster/protocol/openid-connect/auth
            clientId: web_app
            clientSecret: web_app
            clientAuthenticationScheme: form
            scope: openid profile email
        resource:
            userInfoUri: http://localhost:9080/auth/realms/jhipster/protocol/openid-connect/userinfo
            tokenInfoUri: http://localhost:9080/auth/realms/jhipster/protocol/openid-connect/token/introspect
            preferTokenInfo: false
```

### Okta

If you'd like to use Okta instead of Keycloak, you'll need to change a few things. First, you'll need to create a free developer account at <https://developer.okta.com/signup/>. After doing so, you'll get your own Okta domain, that has a name like `https://dev-123456.oktapreview.com`.

Modify `src/main/resources/application.yml` to use your Okta settings.

```yaml
security:
    basic:
        enabled: false
    oauth2:
        client:
            accessTokenUri: https://{yourOktaDomain}.com/oauth2/default/v1/token
            userAuthorizationUri: https://{yourOktaDomain}.com/oauth2/default/v1/authorize
            clientId: {clientId}
            clientSecret: {clientSecret}
            clientAuthenticationScheme: form
            scope: openid profile email
        resource:
            userInfoUri: https://{yourOktaDomain}.com/oauth2/default/v1/userinfo
            tokenInfoUri: https://{yourOktaDomain}.com/oauth2/default/v1/introspect
            preferTokenInfo: false
```

Create an OIDC App in Okta to get a `{clientId}` and `{clientSecret}`. To do this, log in to your Okta Developer account and navigate to **Applications** > **Add Application**. Click **Web** and click the **Next** button. Give the app a name you’ll remember, and specify `http://localhost:8080` as a Base URI and Login Redirect URI. Click **Done** and copy the client ID and secret into your `application.yml` file.

Create a `ROLE_ADMIN` and `ROLE_USER` group (**Users** > **Groups** > **Add Group**) and add users to them. You can use the account you signed up with, or create a new user (**Users** > **Add Person**). Navigate to **API** > **Authorization Servers**, click the **Authorization Servers** tab and edit the default one. Click the **Claims** tab and **Add Claim**. Name it "groups" or "roles", and include it in the ID Token. Set the value type to "Groups" and set the filter to be a Regex of `.*`.

**NOTE:** If you want to use Okta all the time (instead of Keycloak), modify JHipster’s Protractor tests to use this account when running. Do this by changing the credentials in `src/test/javascript/e2e/account/account.spec.ts` and `src/test/javascript/e2e/admin/administration.spec.ts`.

After making these changes, you should be good to go! If you have any issues, please post them to [Stack Overflow](https://stackoverflow.com/questions/tagged/jhipster). Make sure to tag your question with "jhipster" and "okta".

You can also use environment variables to override the defaults. For example:

```bash
export SECURITY_OAUTH2_CLIENT_ACCESS_TOKEN_URI="https://{yourOktaDomain}.com/oauth2/default/v1/token"
export SECURITY_OAUTH2_CLIENT_USER_AUTHORIZATION_URI="https://{yourOktaDomain}.com/oauth2/default/v1/authorize"
export SECURITY_OAUTH2_RESOURCE_USER_INFO_URI="https://{yourOktaDomain}.com/oauth2/default/v1/userinfo"
export SECURITY_OAUTH2_RESOURCE_TOKEN_INFO_URI="https://{yourOktaDomain}.com/oauth2/default/v1/introspect"
export SECURITY_OAUTH2_CLIENT_CLIENT_ID="{clientId}"
export SECURITY_OAUTH2_CLIENT_CLIENT_SECRET="{clientSecret}"
```

You can put this in an `~/.okta.env` file and run `source ~/.okta.env` to override Keycloak with Okta.

You can use then set these properties when you deploy to Heroku:

```bash
heroku config:set \
  FORCE_HTTPS="true" \
  SECURITY_OAUTH2_CLIENT_ACCESS_TOKEN_URI="$SECURITY_OAUTH2_CLIENT_ACCESS_TOKEN_URI" \
  SECURITY_OAUTH2_CLIENT_USER_AUTHORIZATION_URI="$SECURITY_OAUTH2_CLIENT_USER_AUTHORIZATION_URI" \
  SECURITY_OAUTH2_RESOURCE_USER_INFO_URI="$SECURITY_OAUTH2_RESOURCE_USER_INFO_URI" \
  SECURITY_OAUTH2_RESOURCE_TOKEN_INFO_URI="$SECURITY_OAUTH2_RESOURCE_TOKEN_INFO_URI" \
  SECURITY_OAUTH2_CLIENT_CLIENT_ID="$SECURITY_OAUTH2_CLIENT_CLIENT_ID" \
  SECURITY_OAUTH2_CLIENT_CLIENT_SECRET="$SECURITY_OAUTH2_CLIENT_CLIENT_SECRET"
```

For Cloud Foundry, you can use something like the following, where `$appName` is the name of your app.

```bash
cf set-env $appName FORCE_HTTPS true
cf set-env $appName SECURITY_OAUTH2_CLIENT_ACCESS_TOKEN_URI "$SECURITY_OAUTH2_CLIENT_ACCESS_TOKEN_URI"
cf set-env $appName SECURITY_OAUTH2_CLIENT_USER_AUTHORIZATION_URI "$SECURITY_OAUTH2_CLIENT_USER_AUTHORIZATION_URI"
cf set-env $appName SECURITY_OAUTH2_RESOURCE_USER_INFO_URI "$SECURITY_OAUTH2_RESOURCE_USER_INFO_URI"
cf set-env $appName SECURITY_OAUTH2_RESOURCE_TOKEN_INFO_URI "$SECURITY_OAUTH2_RESOURCE_TOKEN_INFO_URI"
cf set-env $appName SECURITY_OAUTH2_CLIENT_CLIENT_ID "$SECURITY_OAUTH2_CLIENT_CLIENT_ID"
cf set-env $appName SECURITY_OAUTH2_CLIENT_CLIENT_SECRET "$SECURITY_OAUTH2_CLIENT_CLIENT_SECRET"
```